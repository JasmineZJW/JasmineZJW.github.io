---
layout:     post
title:      A Macro to Check Title Case When Adding Variables/Values Into SDTM/ADaM
subtitle:   
date:       2017-11-19
author:     Jasmine Zhang
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - SDTM
    - ADaM 
    - Case
---

> It is usual to add values in supplemental domains or sponsor-defined test names in SDTM domains, as well as adding analysis required variables in ADaM dataset. As stated in CDISC IG, the label of those values/variables should be in title case. This macro is to check whether the manual added labels comply to the CDISC rules.

# Case Rule in SDTM IG

"Adjust the labels of the variables only as appropriate to properly convey the meaning in the context of the data being submitted in the newly created domain. Use title case for all labels (title case means to capitalize the first letter of every word except for articles, prepositions, and conjunctions)."
                                                         ---SDTM IG 3.2 (2.6 Creating a New Domain, page 16)
                                                              
"It is recommended that text data be submitted in upper case text. Exceptions may include long text data (such as comment text); values of --TEST in Findings datasets (which may be more readable in title case if used as labels in transposed views); and certain controlled terminology [see Section 4.1.3.2, Controlled Terminology Text Case ] that are already in mixed case. "
                                                         ---SDTM IG 3.2 (4.1.2.4 Case Use of Text in Submitted Data, page 32)

# Logics of the Macro
The macro use the wildcard "/b" in perl regular expression to identify the boundaries among words. Note that the boundaries are not always blank spaces. Sometimes it could be a dash (follow-up) or brackets. Therefore, perl regular expression is the best way to separate each word in the text.

There are four parameters defined in the macro:
1. indt_: specify the input dataset which includes labels to be checked.
2. invar_: specify the variable name that store the label.
3. outdt_: specify the name of output dataset. 
4. excl: specify the articles, prepositions, and conjunctions that need to exclude from title case.

The rule of the title case is referring to the definition in wiki, details can be accessed via below link:
http://titlecaseconverter.com


## To Upcase Each Word in the Text

      START = 1;
      STOP  =length(&invar_.);
      REUC  = prxparse("/\b\w/");
      call prxnext (REUC, START, STOP, LABEL, POS, LENGTH);
      do while (POS > 0);
          call prxnext (REUC, START, STOP, LABEL, POS, LENGTH);
          LABEL = substr(LABEL, 1, START-2)||upcase(substr(LABEL, START-1, 1))||substr(LABEL, START);
      end; 
