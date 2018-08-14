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

Step 1: 
    The macro will read the TFL definition file from the specific path. This part will need to be changed to accommadate different formats of the TFL definition files. Basic information includes:
    TFLNO: Number of the TFLs.
    SUBTFLNO: Subnumber of the TFLs.
    TITLE: Title of the TFLs, will be printed in the TOC of the combined pdf.

Step 2:

