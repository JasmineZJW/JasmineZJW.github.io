---
layout:     post
title:      Record-Level Population Flag in ADaM Dataset (1)
subtitle:  
date:       2017-12-08
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
![Aaron Swartz](https://github.com/JasmineZJW/JasmineZJW.github.io/blob/master/img/屏幕快照%202017-12-17%20下午11.49.22.png?raw=true)

Explanation:
The analysis visit windowinga vriables are used to stored the planned schedules defined in the protocol. Note that beginners tend to make mistakes when calculating AWTDIFF: AWTDIFF is the absolute difference between ADY or ARELTM and AWTARGET. It will be necessary to adjust for the fact that there is no day 0 in the event that ADY and AWTARGET are not of the same sign.
In other words:
    if AWU="DAYS", AWDIFF=|ADY-AWTARGET| when ADY and AWTARGET are of the same sign.
 or AWDIFF=|ADY-AWTARGET|+1 when ADY and AWTARGET are of not the same sign. 
    if AWU="h", AWDIFF=|ARELTM-AWTARGET|.

In the above example, ADY for row 1-3 falls within the scheduled windows, these records are included in both the PPS analysis and ITT analysis. Row 4 falls outside the scheduled window, it should be excluded from PPS analysis while included in ITT analysis. In such case, the analyzed endpoint value varies according to the population. The last available values for PPS analysis is row 3. However, for ITT analysis, the last available values for ITT analysis is row 4. Therefore, two derived "ENDPOINT" records are needed for different analysis purposes.
As you can see from the above table, row five is derived from row 4, which is the last record for ITT analysis. ITTRFL is marked as "Y" for row 4. While row 6 is derived from the last record for PPS analysis.

This is the most common proccessing procedures when there are record-level population flags. More details please refer to ADaM IG 1.1, section 4.2.1 Rules for the Creation of Rows and Columns. Sometimes we may meet more complicated situations. Please expect the next post "Record-Level Population Flag in ADaM Dataset (2)".
