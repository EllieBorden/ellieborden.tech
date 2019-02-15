---
layout: "post"
title: "Domain Testing"
date: "2019-02-12 16:14"
---

Domain testing is a vague, umbrella term which essentially describes the process of testing collections of values as a whole rather than individually. This method is primarily used in functional testing and is classified as a black box technique.

The term **domain** does not regard domain knowledge (knowledge of the product being developed or tested.) 
In domain testing, a domain is synonymous with a set. 

## Purpose

The purpose of domain testing is to reduce the number of tests by excluding test cases that are assumed to have the same result as another test case. 
Testers have a finite amount of time and resources, so it's important to design effective tests and reduce redundancy.  

## Techniques

There are two domain testing techniques.

### Equivalence Partitioning
  
Partitioning, as it relates to domain testing, is the act of splitting a set into subsets.

Equivalence describes a state of equality.

In equivalence partitioning, all potential test values are grouped into classes known as **Equivalence Classes**. All members of an equivalence class are assumed to yield the same result in test. 
	
There are two types of equivalence class:

- **Valid Classes** are expected to be accepted by the application.
	
- **Invalid Classes** are expected to be rejected by the application.

After all equivalence classes have been defined, one or more members of each class are then selected to represent their entire class. 

**Example**

Suppose you are testing an input that requires any **whole number** between **1** and **10000**. Firstly, that requirement is ambiguous because the word "between" could imply either a range of **1** - **10000** or **2** - **9999**, but lets assume we mean the former. It would be a misuse of time to test every number in that range -- much less every input that should be excluded -- so let's consider what variations might exist between them and which value(s) to represent each variation.

Our valid equivalence classes include:

Class Description                   | Class Representative
------------------------------------|:-------------------:
Odd numbers and one-digit numbers.  |          1
Even numbers and two-digit numbers. |          10
Three-digit numbers.                |         100
Four-digit numbers.                 |         1000
Five-digit numbers.                 |        10000

Our invalid equivalence classes include:

Class Description                                                | Class Representative
-----------------------------------------------------------------|:-------------------:
No input.                                                        |         null
Non-numeric inputs.                                              |        'One'
Numbers less than the minimum accepted number.                   |          0
Numbers that have the same absolute value as acceptable numbers. |          -1
Numbers that are not whole.                                      |         1.5
Numbers greater than the maximum accepted number.                |        10001
Numbers formatted with commas.                                   |        10,000

You may not need to test all classes. In the context of our example, if a one-digit input is accepted and a five-digit input is accepted, we can assume values ranging from 2-4 digits will be accepted as well.

Additionally, some of our invalid classes, such as **1,000**, might be valid classes depending on how the system handles those inputs.

### Border Value Analysis

Border value analysis is an extension of equivalence partitioning in which border-values are chosen to represent classes when applicable. Its purpose is to test common "off-by-one" errors which result from loops that iterate once more or less than intended.

Referring back to the previous example, its valid border-values are **1** and **10000**, whereas its invalid border-values are **0** and **10001**.  

![border values graph](assets/images/2019/bva.png){:.img-center .vert-pad}

Since those values were already select to represent classes, we've already performed border value analysis on that input.

## Complications

- Data type
- Value (Date, time, real numbers, etc.)
- Business context
- Not limited to value, can also cover the characteristics of values, such as character length
- Equivalence is subjective and requires experience.

Class Description                                                | Class Representative
-----------------------------------------------------------------|:-------------------:
Numbers that have the same absolute value as acceptable numbers. |          -1
Numbers formatted with commas.                                   |        10,000

## Limitations

- Doesn't address combination-defects
- Efficiency depends on the tester's ability to recognize unique classes, based on the application and business context of the application

## Resources

## References
- Black, Rex. _Pragmatic Software Testing: Becoming an Effective and Efficient Test Professional_ Indianapolis: Wiley Publishing, Inc. 2007. 
- Homès, Bernard. _Fundamentals of Software Testing._ Hoboken: John Wiley & Sons, Inc. 2012.
- Kaner, Cem, Sowmya Padmanabhan, and Hoffman Douglas. _The Domain Testing Workbook._ Context Driven Press. 2013.
- Myers, Glenford J., Tom Badgett, and Corey Sandler. _The Art of Software Testing._ Hoboken: John Wiley & Sons, Inc. 2012.
