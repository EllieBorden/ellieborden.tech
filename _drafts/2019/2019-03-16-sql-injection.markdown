---
layout: "post"
title: "Testing for SQL Injection Vulnerabilities"
comments: true
date: "2019-03-16 8:00"
---

SQL Injection is the manipulation of input data, through the insertion of SQL syntax, to disrupt or alter a request to a database. 

- common causes
  - lack of validation
- risks
  - database dump

This guide walks through the following operations:

- Introduction to SQL injection fundamentals.
- Introduction to penetration testing standards.
- Installing a test environment.
- Identifying and confirming an SQL injection vulnerability.
- Exploiting the vulnerability.
- Reporting the vulnerability.
- Introduction to penetration testing resources.

## Warnings

Read this section before continuing to the demonstration. I'm not responsible for anything you do with the information in this post.

### DON'T do it Yourself

This guide is intended to serve as an introduction to SQL Injection for testers who are interested in security. It should **NOT** be used by website owners for do-it-yourself security testing. The effectiveness of SQL injection greatly depends on the skill of the tester. Please hire a security expert if you require thorough testing.
 
### DON'T Test Without Permission

- Illegal https://www.law.cornell.edu/uscode/text/18/1030
- Do not begin testing before defining the **Rules of Engagement**.
- Require proof of ownership
- refer to standards
 
### DON'T Test Live Environments

- corrupted database
- server interference

## Test Environment Setup

For this demonstration, I'm using a test-application called bWAPP (Buggy Web Application). 

### Installing bWAPP 

1. Download bWAPP from [itsecgames.com](http://www.itsecgames.com/) as either a compressed folder of source files, which can be installed to any PHP/MySQL development environment such as [WAMP](http://www.wampserver.com/en/), or pre-installed as part of a custom Linux virtual machine called _bee-box_.  
2. Extract the source files, then move the **bWAPP** folder to your root server directory ('C:\wamp64\www' for WAMP).
3. Open **bWAPP/admin/settings.php**, then configure **$db_username** and **$db_password** to match your MySQL login credentials.
4. While your server is running, open **bwapp/install.php** in a browser, then click the installation link. This will create a database for a the application and redirect you to a login page.
5. Login to bWAPP. The default username is **bee** and password is **bug**.
6. Select **SQL Injection (GET/Search)** from the list of list of bugs, then click **Hack**. You should be redirected to **bwapp/sqli_1.php**.

**NOTE:** bWAPP/sqli_1.php connects to a database using **mysql_connect()**. This function was deprecated in PHP 5.5.0 and removed in PHP 7.0.0. If you receive a fatal error: **Call to undefined function mysql_connect()** while trying to use the search feature on this page, consider downgrading from PHP 7 to PHP 5 for this test only.

## Identify Inputs

- GET
- POST
- COOKIES
- Headers
- - Types of sql injection

## Test Inputs

One input at a time

### Bypassing security

- encoding
- Whitespace
- null characters
- commenting

## Confirming SQL Injection

- complete a modified sql query
- stacked queries

## Exploit Vulnerabilities

- fingerprint database management system
- fingerprint user
- get databases
- get tables
- get columns
- get rows
- escalate privileges
- gain access to the server

## Reporting

standards

## Resources

- Learning SQL
- practicing sql
- learning sql Injection
- database-specific cheat sheets
- practice environments
- standards

## References

- Clarke, Justin. _SQL Injection Attacks and Defense, 2nd ed_. Waltham: Syngress. 2012.
- Meucci, Matteo and Andrew Muller. _The OWASP Testing Guide, v4_. 2014.
- PCI Security Standards Council. _PCI Data Security Standard (PCI DSS), v1._. 2015.
