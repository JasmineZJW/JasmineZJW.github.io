---
layout:     post
title:      One Step to Combine RTFs into PDF files
subtitle:   
date:       2017-12-17
author:     Jasmine Zhang
header-img: img/post-bg-4.JPG
catalog: true
tags:
    - RFT 
    - Report
---

Combining all the individual TLF outputs into one PDF helps the electronic review easier. Various techniques have been introduced in the PharmaSUG. I read through all these methods and made some improvements based on all the techniques introduced in the papers. By using the improved codes, you just need to define the macro variables in advance and run the macro, it will read all the rtfs in the pre-defined path and combine them into one rtf file. Then a piece of VBScript will convert the combined rtf file into pdf file. The script is wrapped into a SAS macro so it’s convenient to be invoked within SAS program.  

The full codes can be accessed via below link:
<https://github.com/JasmineZJW/SAS/blob/master/rtf_combine.sas>   

More details will be explained in this post.
(To be continued...)

 
