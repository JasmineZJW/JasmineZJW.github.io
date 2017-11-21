---
layout:     post
title:      A Macro to Check Title Case When Adding Variables/Values Into SDTM/ADaM
subtitle:   
date:       2017-11-19
author:     Jasmine Zhang
header-img: img/post-bg-2.JPG
catalog: true
tags:
    - SDTM 
    - ADaM 
    - Case
---

> It is usual to add values in supplemental domains or sponsor-defined test names in SDTM domains, as well as adding analysis required variables in ADaM dataset. As stated in CDISC IG, the label of those values/variables should be in title case. This macro is to check whether the manual added labels comply to the CDISC rules.

# Case Rule in SDTM IG
"Adjust the labels of the variables only as appropriate to properly convey the meaning in the context of the data being submitted in the newly created domain. Use title case for all labels (title case means to capitalize the first letter of every word except for articles, prepositions, and conjunctions)." 
                                                     SDTM IG 3.2 (2.6 Creating a New Domain, page 16)
							   
"It is recommended that text data be submitted in upper case text. Exceptions may include long text data (such as comment text); values of --TEST in Findings datasets (which may be more readable in title case if used as labels in transposed views); and certain controlled terminology [see Section 4.1.3.2, Controlled Terminology Text Case ] that are already in mixed case. " 
                                                     SDTM IG 3.2 (4.1.2.4 Case Use of Text in Submitted Data, page 32)

# Logics of the Macro
The macro use the wildcard `/b` in perl regular expression to identify the boundaries among words. Note that the boundaries are not always blank spaces. Sometimes it could be a dash (follow-up) or brackets. Therefore, perl regular expression is the best way to separate each word in the text.

There are four parameters defined in the macro:

-	indt_: specify the input dataset which includes labels to be checked.
-	invar_: specify the variable name that store the label.
-	outdt_: specify the name of output dataset.
-	excl: specify the articles, prepositions, and conjunctions that need to exclude from title case.
	
The rule of the title case is referring to the definition from wikipedia, details can be accessed via below link: ≤http://titlecaseconverter.com≥

There are special cases we have to due with when we convert the text into title cases:

a. Words with characters are in all capitals, those words seem to be an acronym (e.g. HIV, AE, etc.). Such words can not be identified through program since there are so many cases of acronym and we cannot make a list of them. So please be caution when you need to create a label text with such kinds of acronym. You have to upcase the whole term when you create the label. The macro will not process this kind of words. All the capital wordings remains capitals.

b. Units that should follow controlled terminology. This is a special case in clinical trials. You can specify the frequently used units in the macro parameter (excl), those units will be excluded from being changed.

c. This is a rare situation and it is tricky is that many words can be used in different grammatical functions. Take the one example from the title case converter page (≤http://titlecaseconverter.com≥)
	•	by: Stand by Me, but Stand By for Action 
	The first "by" is used as a preposition, so it should be in lower case. However, the second "by" is used as an adverb, therefore the first character should be in uppercase. Such situation is quite rare in clinical trials, but you may pay attention to this once happend.
Since we have special cases that cannot be processed using programs only, in the macro, a new dataset with a new variable naming "LABEL" is generated. If your original text does not match the new variable, records will be listed in the checking output. Then you can check manually to see whether there are special cases.

## 1. To Upcase Each Word in the Text

The CALL PRXNEXT rountine can be used to indentify the location of the first character of each word. It is usefule when we have a regex which matches multiple times (or an unknown number of times) in our string of interest. Then we can use the call routine to get the next occurrence of the matching pattern.

```swift
  START = 1;
  STOP  =length(&invar_.);
  REUC  = prxparse("/\b\w/");
  call prxnext (REUC, START, STOP, LABEL, POS, LENGTH);
  do while (POS > 0);
      call prxnext (REUC, START, STOP, LABEL, POS, LENGTH);
      LABEL = substr(LABEL, 1, START-2)||upcase(substr(LABEL, START-1, 1))||substr(LABEL, START);
  end; 
```

For example, text `"I have a TV and a desk"` will be updated as `"I Have A TV And A Desk"` in the above process.

## 2. To Lowcase the Pre-Defined Values

The below macro cycle is to check whether each pre-defined value separated by a `|` in the macro parameter &EXCL. exist in the text and use PRXCHANGE function to replace it as lowcase text. The `/i` modifier will make the match case insensitive.

```swift
  %let i=1;
  %let val=%scan(&excl.,&i.,"|");
  %do %while(%str(&val.) ne );
      LABEL=cat(substr(LABEL,1,1),prxchange("s/\b&val.\b/&val./i", -1, substr(LABEL,2)));
      %let i=%eval(&i+1);
      %let val=%scan(&excl.,&i.,"|");
  %end;
```
  
The text `"I Have A TV And A Desk"` will be updated as `"I Have a TV and a Desk"` after the above macro cycle. The article `"a"` and the conjunctions `"and"` was changed to lowcase texts.

The full codings can be found via below link: 
≤https://github.com/JasmineZJW/SAS/blob/master/titlecase≥
