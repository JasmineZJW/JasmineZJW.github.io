---
layout:     post
title:      One Step to Combine RTFs into PDF files
subtitle:   
date:       2017-12-17
author:     Jasmine Zhang
header-img: img/post-bg-4.JPG
catalog: true
tags:
    - RTF 
    - Report
    - TFL
---

Combining all the individual TLF outputs into one PDF helps the electronic review easier. Various techniques have been introduced in the PharmaSUG. I read through all these methods and made some improvements based on all the techniques introduced in the papers. By using the improved codes, you just need to define the macro variables in advance and run the macro, it will read all the rtfs in the pre-defined path and combine them into one rtf file. Then a piece of VBScript will convert the combined rtf file into pdf file. The script is wrapped into a SAS macro so it’s convenient to be invoked within SAS program.  

The full codes can be accessed via below link:
<https://github.com/JasmineZJW/SAS/blob/master/rtf_combine.sas>   

Macro parameters:
path   : Path of rtf files or path of the subfolders. If file size is too large, please 
            separate them into several sub-folders. 
            In such case, you only need to specify the path of the subfolders.
tocpath: Path of TFL definition file. The file should be stored in xlsx format.  
toc    ： The file name of the TFL definition file, no need to indicate the suffix of the file.        
outpath: The path you want to store the combine files.
outfile: The output file name.                          
split  : Indicate whether to split or not. Values are indicated as Y/N.
            * Y=split the output files as Table/Figure or Listing 
            * N=combine as one big file
keepdocx： Indicate whether to keep the combined docx file(s)  
            * Y=keep the combined docx file(s)
            * N=remove the docx file(s)                        

# Step 1 
The macro will read the TFL definition file from the specific path. This part will need to be changed to accommadate different formats of the TFL definition files. Basic information includes:
    TFLNO: Number of the TFLs.
    SUBTFLNO: Subnumber of the TFLs.
    TITLE: Title of the TFLs, will be printed in the TOC of the combined pdf.

# Step 2
To read all the rtf files from the speicified path or subfolders into sas datasets. 
Scanning a folder using SAS File I/O functions to identify the rtf files to be combined and create the rtf_listdataset:
```swift
filename rtf_list pipe "dir ""&path"" /b ";

data rtf_list ;
  infile rtf_list length=reclen ;
  input FILENAME $varying3500. reclen ;
  if scan(FILENAME,2,'.') in ('doc' 'rtf');
  length TFLNO SUBTFLNO $20.;
  TFLNO = tranwrd(scan(scan(FILENAME,1,"."),1,"-"), '_','.');
  SUBTFLNO = scan(scan(scan(FILENAME,-1,'\'),1,"."),-1,"-");
   FILENAME="%"||"nrstr("||strip(tranwrd("&path.", "&", cats("%","nrstr", "(&)")))||'\'||strip(FILENAME)||")";
run;
```
Now let's take a look at the structure of the rtf files. An RTF file consists of an opening section, a content section and a closingsection.
    1) Opening section is the RTF code until the first \sectd of the file, which includes information of the layout of
the file is when viewing as MS Word, etc., e.g. font styles and colors;
    2) Content section includes the content to be displayed. The RTF code for each page starts with \sect\sectd or \sectd; 
    3) Closing section is the RTF code after the last \pard of the file, typically only a closing curly brace.

More details of the rtf codes please refer to the below link:
<http://latex2rtf.sourceforge.net/rtfspec_7.html#rtfspec_tabledef>

Below is the piece of SAS code to read all the rtf files into SAS dataset:
```swift
%macro read_rtf (i=, in_file=, bookmark=);
    data rtff&i.;
      length FNAME&i. $80. BOOKMARK $40.;
      infile "&in_file." length=LINELEN
      FILENAME=FNAME&i. lrecl=32767 recfm=v end=eof;
      input LINEIN $varying3500. LINELEN; 
      FILENAME=FNAME&i.;
      SEQ=&i.;
      BOOKMARK="&bookmark";
      SEQ2=_n_;
    run;
%mend;

data _null_;
    set rtf_list;
    call execute('%nrstr(%read_rtf(i='||cats(SEQ)||', in_file='||cats(FILENAME)||', bookmark='||cats(BOOKMARK)||'))');
run;
```

RTF files can be concatenated by the following steps:
1. Read in RTF files and output the RTF code to a SAS dataset.
2. Remove the closing section of the first file;
3. Remove the opening section of the last file;
4. Remove both the opening section and closing section of all other files;
5. Insert \sect in front of the first page of each RTF file except for the first one. The RTF code \sect starts a new
page and \sectd sets the section properties to the default. Not surprisingly, the first section (first page) of an RTF
file does not have \sect and always begins with \sectd while the rest sections always with \sect\sectd. In the combined file, the first page of each individual file is no longer the first page, except for the first file.\sect
therefore needs to be inserted in front of the first \sectd of those files.
6. Output above RTF code to one single file in such an order: the first file, the rest files and the last file.

when combining the rtf files, we have to calculate the page numbers to be showed in the TOC.

# Step 3
To create the TOC.
Below is the small piece of macro to generate the toc for each rtf and link to the specific page in the combined pdf:

```swift
%macro toc (bookmark=, tflno=, title=, nspace=, firstp=);
    LINEIN = '\pard\sa200\li2000\f0\fs20\b0\ql{\field{\*\fldinst HYPERLINK  \\l "'||strip(&bookmark.)||'"}'; output;
    LINEIN = '{\fldrslt \li18000\fi200 '||strip(&tflno.)||' \tx2000\tab '||strip(&title.)||' \ptabldot\pindtabqr '||strip(&firstp.)||'}}'; output;
    LINEIN = '\ri2000\par';output;```swift 
%mend;
```
The first LINEIN creates the hyperlink to each individual rtf:
```swift 
 {\field {\*\fldinst HYPERLINK \\l "bookmark_name"}{\fldrslt title_of_RTF_output}}
```
The second LINEIN defines the format of the TOC. The rtf code ```swift \ptabldot\pindtabqr ``` right aligns page numbers and inserts dotted lines between titles and page numbers.

The third LINEIN closes the corresponding TOC line.

# Step 4
The final step is to generate the VBA script and execute it using SAS program.
```swift
filename vbs ""&vbs."";
data _null_;
  file vbs;
  length vbscmd $200.;
  put 'Dim objWord';
  put 'Dim objWordDoc';
  put 'Set objWord = CreateObject("Word.Application")';
  put 'Wscript.Sleep 500';
  put 'objWord.Visible = False';
  vbscmd='Set objWordDoc = objWord.Documents.Open("'|| "&docx." ||'")';
  put vbscmd;
  put 'Wscript.Sleep 300';
  vbscmd='objWordDoc.ExportAsFixedFormat "'||"&pdf."||'", 17, False,1,0,1,1,0,False,True,2,False,True,False';
  put vbscmd;
  put 'objWordDoc.Close False';
  put 'objWord.Quit';
run;

filename vbs;

data _null_;
  /* execute vbs */
  call system(""&vbs."");
  /* clean up: delete  vbs */
  call system("del /q "&vbs."");
  /* clean up: delete  docx */
run;
```

# Conclusion
This post explain the rationals of the rtf_combine macro. By using the macro, you just have to simply define the path of the files in the macro parameters and execute it. The macro will help to combine the rtfs into 1 or several pdf files with TOC and bookmark included.

Referece:
A Fully Automated Approach to Concatenate RTF outputs and Create TOC (<https://www.lexjansen.com/pharmasug-cn/2015/DV/PharmaSUG-China-2015-DV28.pdf>)
