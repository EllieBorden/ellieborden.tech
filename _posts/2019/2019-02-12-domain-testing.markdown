---
layout: "post"
title: "Domain Testing"
date: "2019-02-12 16:14"
---

Domain testing is a vague, umbrella term which essentially describes the process of testing a set of values as a whole rather than individually. This method is primarily used in functional testing and is classified as a black box technique.

The term **domain** does not regard domain knowledge (knowledge of the product being developed or tested.) 
In domain testing, a domain is synonymous with a set. 

## Purpose

The purpose of domain testing is to reduce the number of tests by excluding test cases that are assumed to have the same result as another test case. 
Testers have a finite amount of time and resources, so it's important to design effective tests and reduce redundancy.  

## Techniques

There are two domain testing technique, Equivalence Partitioning and Boundary Value Analysis.

### Equivalence Partitioning
  
Partitioning, as it relates to domain testing, is the act of splitting a set into subsets.

Equivalence describes a state of equality.

In equivalence partitioning, all potential test values are grouped into classes known as **Equivalence Classes (ECs)**. All members of an equivalence class are assumed to yield the same result in test. 
	
There are two types of equivalence classes:

- **Valid Classes** are expected to be accepted by the application.
	
- **Invalid Classes** are expected to be rejected by the application.

After an equivalence class has been defined, one or more members of that class are selected to represent the entire class. 

**Example**

Suppose you are testing an input that requires any **whole number** between **1** and **10000**. Firstly, that requirement is ambiguous because the word "between" could imply either a range of **1** - **10000** or **2** - **9999**, but let's assume we mean the former. It would be a misuse of time to test every number in that range -- much less every input that should be excluded -- so let's consider what variations might exist between them and which value(s) to represent each variation.

Our valid ECs include:

Class Description                   | Class Representative
------------------------------------|:-------------------:
Odd numbers and one-digit numbers.  |          1
Even numbers and two-digit numbers. |          10
Three-digit numbers.                |         100
Four-digit numbers.                 |         1000
Five-digit numbers.                 |        10000

Our invalid ECs include:

Class Description                                                | Class Representative
-----------------------------------------------------------------|:-------------------:
No input.                                                        |         null
Non-numeric inputs.                                              |        'One'
Numbers less than the minimum acceptable number.                   |          0
Numbers that are not whole.                                      |         1.5
Numbers greater than the maximum acceptable number.                |        10001
Numbers that have the same absolute values as valid numbers. |          -1
Numbers formatted with commas.                                   |        10,000
Numbers that are preceded by spaces  |&nbsp;&nbsp;&nbsp;&nbsp; 1
Numbers that are preceded by zeros  |  00001 
Expressions that equal a number within the valid range.  |  1 + 1

You may not need to test all those classes. In the context of our example, if a one-digit input is accepted and a five-digit input is accepted, we can assume values ranging from 2-4 digits will be accepted as well.

Additionally, some of our invalid classes might actually be valid classes depending on how the system handles those inputs. 

### Boundary Value Analysis

Boundary value analysis is an extension of equivalence partitioning in which boundary values are chosen to represent their classes when applicable. Its purpose is to test common "off-by-one" errors which result from loops that iterate once more or less than intended.

Referring to the previous example, its valid boundary values are **1** and **10000**, whereas its invalid boundary values are **0** and **10001**.  

![border values graph](assets/images/2019/bva.png){:.img-center .vert-pad}

Since those values were already selected to represent their classes, we've already performed boundary value analysis on that input. 

## Limitations.

Equivalence is subjective. The effectiveness of this method depends on the testers ability to determine equivalence classes and to select representatives of those classes. Values that have the highest potential to cause errors (such as boundary values) should be chosen to represent their equivalence class when possible.  

Additionally, domain testing is one of many techniques and should not be the only one considered when testing.

## Resources

The example we discussed is very simple. Domain testing can be executed on a variety of complex problems. [_The Domain Testing Workbook_](https://www.amazon.com/Domain-Testing-Workbook-Cem-Kaner/dp/0989811905) contains examples of some of those use cases and elaborates on these techniques in greater detail. It's the best resource I've found on this subject. Other decent books are listed in the references section below.

[_Big List of Naughty Strings_](https://github.com/minimaxir/big-list-of-naughty-strings/blob/master/blns.txt) is an evolving list of potential inputs that have a high probability of causing errors. 

## References
- Black, Rex. _Pragmatic Software Testing: Becoming an Effective and Efficient Test Professional_ Indianapolis: Wiley Publishing, Inc. 2007. 
- Homès, Bernard. _Fundamentals of Software Testing._ Hoboken: John Wiley & Sons, Inc. 2012.
- Kaner, Cem, Sowmya Padmanabhan, and Hoffman Douglas. _The Domain Testing Workbook._ Context Driven Press. 2013.
- Myers, Glenford J., Tom Badgett, and Corey Sandler. _The Art of Software Testing._ Hoboken: John Wiley & Sons, Inc. 2012.
