---
layout: "post"
title: "Testing for SQL Injection Vulnerabilities"
comments: true
date: "2019-03-16 8:00"
---

SQL injection is the submission of SQL syntax to a vulnerable input field, which is then used by an application in the dynamic construction of a database query. This attack is possible when user input is insufficiently validated and can be interpreted by a database management system (DBMS) as code. A malicious user can inject SQL to disrupt or alter queries to the database, potentially leading to the exposure of sensitive data such as usernames, passwords, and credit card numbers.

This guide aims to serve as an introduction to testing for SQL injection and walks through the following operations:

- Installation of a test environment.
- Identification of an SQL injection vulnerability.
- Confirmation and exploitation the vulnerability.
- Best practices for reporting security vulnerabilities.

## Test Responsibly

Read this section before continuing to the demonstration. I'm not accountable for anything you break using the information in this post.

### DON'T do it Yourself

This guide is intended for people who are interested in learning about security. It should **NOT** be used by website owners for do-it-yourself security testing. The effectiveness of SQL injection greatly depends on the skill of the tester. Please hire a security expert or team of experts if you require thorough testing.
 
### DON'T Test Without Permission

- Illegal https://www.law.cornell.edu/uscode/text/18/1030
- Do not begin testing before defining the **Rules of Engagement**.
- Require proof of ownership

### Avoid Testing Live Environments

- corrupted database
- server interference

## Test Environment Setup

For this demonstration, I'm using a test-application called bWAPP (Buggy Web Application). 

### Installing bWAPP 

1. Download bWAPP from [itsecgames.com](http://www.itsecgames.com/) as either a compressed folder of source files, which can be installed to any PHP/MySQL development environment such as [WAMP](http://www.wampserver.com/en/) or pre-installed as part of a custom Linux virtual machine called _bee-box_.  
2. Extract the source files, then move the **bWAPP** folder to your root server directory ('C:\wamp64\www' for WAMP).
3. Open **bWAPP/admin/settings.php**, then configure **$db_username** and **$db_password** to match your MySQL login credentials.
4. While your server is running, open **bwapp/install.php** in a browser, then click the installation link. This will create a database for the application and redirect you to a login page.
5. Login to bWAPP. The default username is **bee** and password is **bug**.
6. Select **SQL injection (GET/Search)** from the list of bugs, then click **Hack**. You should be redirected to **bwapp/sqli_1.php**.

> **NOTE:** I'm using PHP 5.6.40 and MySQL 5.7.24. You may experience different results using other versions in your test environment.

Installation steps and detailed requirements are listed in **install.txt**, which is located in the root directory of the folder you downloaded in Step 1.

### Troubleshooting

bWAPP/sqli_1.php connects to a database using **mysql_connect()**. This function was deprecated in PHP 5.5.0 and removed in PHP 7.0.0. If you receive a fatal error: **Call to undefined function mysql_connect()** while trying to use the search feature on this page, consider downgrading from PHP 7 to PHP 5 for this test only.

If you encounter other errors, post them in a comments section below.

## Testing

- beginning to test
- presumably have permission

### Identify Inputs

- GET
- POST
- COOKIES
- Headers
- - Types of sql injection

### Test Inputs

One input at a time

- bypassing intrusion detection
  - encoding
  - Whitespace
  - null characters
  - commenting

### Confirm SQL Injection

- complete a modified sql query
- stacked queries

### Identify A Vulnerability

- fingerprint database management system
- fingerprint user
- get databases
- get tables
- get columns
- get rows
- escalate privileges
- gain access to the server

## Reporting

- standards
- best practices

## Resources

- we only covered the basics
- continue learning
- focus on the behaviors of popular DBMS or your organization's DBMS

### Learning SQL

- [SQLBolt](https://sqlbolt.com/)
- [Code Academy](https://www.codecademy.com/learn/learn-sql)
- [Hacker Rank](https://www.hackerrank.com/domains/sql?filters%5Bstatus%5D%5B%5D=unsolved&badge_type=sql) - This site doesn't teach SQL but hosts a collection of SQL practice-questions ranging in complexity.

### Learning SQL Injection

- [SQL Injection Attacks and Defense, 2n ed](https://www.amazon.com/Injection-Attacks-Defense-Justin-Clarke/dp/1597499633/ref=sr_1_2?keywords=sql+injection+book&qid=1552931384&s=gateway&sr=8-2) - The most thorough introduction to SQL injection available. (Published in 2012.)
  
- [OWASP - Vulnerable web applications](https://www.owasp.org/index.php/OWASP_Vulnerable_Web_Applications_Directory_Project#tab=Main) - A list of vulnerable web applications for practicing security testing.

- [OWASP - Penetration testing methodologies](https://www.owasp.org/index.php/Penetration_testing_methodologies)

### Testing References

Known Vulnerabilities:

- [Exploit Database](https://www.exploit-db.com/)
- [National Vulnerability Database](https://nvd.nist.gov/vuln/search)

Payloads:

- [NetSPI - SQL attack queries](https://sqlwiki.netspi.com/attackQueries/)
- [Bug bounty cheat sheet - SQLi](https://github.com/EdOverflow/bugbounty-cheatsheet/blob/master/cheatsheets/sqli.md) - An unordered list of relevant cheat sheets. (Last updated in 2017.)

Miscellaneous:

- [OWASP - Bypassing firewalls](https://www.owasp.org/index.php/SQL_Injection_Bypassing_WAF)

## References

- Clarke, Justin. _SQL Injection Attacks and Defense, 2nd ed_. Waltham: Syngress. 2012.
- Meucci, Matteo and Andrew Muller. _The OWASP Testing Guide, v4_. 2014.
- PCI Security Standards Council. _PCI Data Security Standard (PCI DSS), v1._. 2015.
