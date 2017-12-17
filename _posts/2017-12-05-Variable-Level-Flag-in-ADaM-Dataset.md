---
layout:     post
title:      Record-Level Population Flag in ADaM Dataset
subtitle:   
date:       2017-12-08
author:     Jasmine Zhang
header-img: img/post-bg-3.JPG
catalog: true
tags:
    - ADaM 
    - Record-level flag
---

Record-level flags can be used when the subject is included in the subject-level population, but there are records for the subject that do not satisfy requirements for the population. Below I will share experiences of processing with these record-level flags.



# Introduction of the Record-Level Population Flags

There are 5 records-level population flags defined in the ADaM IG 1.1: 

Variable Name | Variable Label
--------------|---------------
ITTRFL        |Intent-To-Treat Record-Level Flag               
SAFRFL        |Safety Analysis Record-Level Flag 
SAFRFL        |Full Analysis Set Record-Level Flag
PPROTRFL      |Per-Protocol Record-Level Flag
COMPLRFL      |Completers Record- Level Flag

As stated in ADaM IG 1.1, additional flags can be added as long as the variable comply the following conventions:

1. Record-level population flag names end with "RFL".
2. Values of record-level population flags follows the "Y" controlled terminology. Only "Y" or null value is allowed. This is different from the subject-level population flags, which do not allow null values.
3. Corresponding numeric flags are also permissible if the character flags exist. For the numeric flags, variable names should be ended with "RFN". Only 1 or null values are allowed.

# Handling Derived Analysis Timepoint when Record-Level Flags Exist

Assume that it was stated in the SAP that only those records collected within the permissible time window should be included in the Per-Protocol analysis. In that case, records which was conducted out of the scheduled window will be excluded from the Per-Protocol analysis. However, those records will be included in the Itention-To-Treat analysis. ENDPOINT might be different for Per-Protocol analysis or Itention-To-Treat analysis. 

Example of different timepoints for different analysis population:

Explanation


