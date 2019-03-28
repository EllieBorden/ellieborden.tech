---
layout: "post"
title: "Testing for SQL Injection Vulnerabilities"
comments: true
date: "2019-03-16 8:00"
---

SQL injection (SQLi) is the submission of SQL syntax to a vulnerable input field, which is then used by an application in the dynamic construction of a database query. This attack is possible when user input is insufficiently validated and can be interpreted by a database management system (DBMS) as code. A malicious user can inject SQL to disrupt or alter queries to the database, potentially leading to the exposure of sensitive data such as usernames, passwords, and credit card numbers.

### Agenda

This guide aims to serve as an introduction to testing for SQL injection and walks through the following operations:

- Installation of a test environment.
- Identification of an SQL injection vulnerability.
- Confirmation and exploitation the vulnerability.
- Best practices for reporting security vulnerabilities.

Tools are not used in this exercise, however, some popular tools will be listed in the **Resources** section below.

Code review for SQL injection will be covered in a later post.

### Prerequisites

To get the most out of this guide, you'll need to be familiar with the HyperText Transfer Protocol (HTTP) and the following languages:

- HTML
- PHP
- SQL

If you wish to follow along, you'll also need to configure an Apache/PHP/MySQL environment.

## Test Responsibly

Read this section before continuing to the demonstration. I'm not accountable for anything you break using the information in this post.

### DON'T do it Yourself

This guide is intended for people who are interested in learning about security. It should **NOT** be used by website owners for do-it-yourself security testing. The effectiveness of SQL injection greatly depends on the skill of the tester. Please hire a security expert or team of experts if you require thorough testing.

### DON'T Test Without Permission

Testing a live application without its owner's consent can result in legal ramifications. Do not begin testing without written permission, a defined scope, and defined rules of engagement. You should also be confident that the person permitting security testing owns the property or has the authority to authorize testing.

### Avoid Testing Live Environments

It is possible to delete or overwrite data by injecting SQL into a database query. I highly recommend creating a copy of your application to be used specifically for testing -- especially while you're learning. If you must test an application in production, confirm there are backups of its data first.

## Test Environment Setup

For this demonstration, I'm using a test-application called bWAPP (Buggy Web Application).

### Installing bWAPP

1. Download bWAPP from [itsecgames.com](http://www.itsecgames.com/) as either a compressed folder of source files, which can be installed to any PHP/MySQL development environment such as [WAMP](http://www.wampserver.com/en/) or pre-installed as part of a custom Linux virtual machine called _bee-box_.  
2. Extract the source files, then move the **bWAPP** folder to your root server directory (E.g. 'C:\wamp64\www' for WAMP).
3. Open **bWAPP/admin/settings.php**, then configure **$db_username** and **$db_password** to match your MySQL login credentials.
4. While your server is running, open **bwapp/install.php** in a browser, then click the installation link. This will create a database for the application and redirect you to a login page.
5. Login to bWAPP. The default username is **bee** and password is **bug**.
6. Select **SQL injection (GET/Search)** from the list of bugs, then click **Hack**. You should be redirected to **bwapp/sqli_1.php**.

> **NOTE:** I'm using PHP 5.6.40 and MySQL 5.7.24. You may experience different results using other versions in your test environment.

Installation steps and detailed requirements are listed in **install.txt**, which is located in the root directory of the folder you downloaded in Step 1.

### Troubleshooting

bWAPP/sqli_1.php connects to a database using **mysql_connect()**. This function was deprecated in PHP 5.5.0 and removed in PHP 7.0.0. If you receive a fatal error: **Call to undefined function mysql_connect()** while trying to use the search feature on this page, consider downgrading from PHP 7 to PHP 5 for this test only. Real PHP applications should run on an updated version of PHP and use [PHP Data Objects (PDO)](https://www.php.net/manual/en/book.pdo.php) to perform database queries.

If you encounter other errors, post them in a comments section below.

## Testing

We are testing the search box on **bwapp/sqli_1.php**. Navigate to this page to begin.

### Identify Inputs

First, we need to identify all sources of user input within the scope of our test. In this example, we are exclusively testing the **Search for a movie** form, however, a typical scope could potentially include one or more inputs, features, or pages.

User input is commonly submitted in the form of a GET or POST request, but can also be included in cookies and request headers. Any input on the client-side can be manipulated by a user. This also applies to obscure elements such as hidden form inputs, which are not displayed on a page but can be seen and manipulated in the page's source code. 

Radio buttons and checkboxes are other examples. While those elements are typically only checked or unchecked, their values can be edited in the source code.

> **NOTE**: This is why it is imperative to validate input server-side. If a form is only validated by the user's client, they can easily edit or delete the validation before submission.

[View the page source](https://www.lifewire.com/view-web-source-code-4151702) and scroll down to the **Search for a movie** form.


```html
<form action="/sqli_1.php" method="GET">

  <p>

  <label for="title">Search for a movie:</label>
  <input type="text" id="title" name="title" size="25">

  <button type="submit" name="action" value="search">Search</button>

  </p>

</form>
```

We can see two input variables:

- **title** is defined in the **input** element and is assigned a value of whichever string is in the textbox upon form-submission.
- **action** is defined in the **button** element and is assigned the value **"search"** upon submission.

We can also confirm that there are no hidden inputs and that the data is submitted through a GET request. This is reflected in the URL when we enter the word "test" into the form, then click **Search**:

{:style="text-align: center;"}
```url
bwapp/sqli_1.php?title=test&action=search

```

For other request methods, you may need to use a proxy server, such as the one available in the free and open source application [ZAP](https://github.com/zaproxy/zaproxy), to see and edit the data being transferred.

### Identify the Inputs' Applications

Now that we have identified the inputs in the **Search for a movie** form, we can try to identify how those inputs are being used. It is important to keep all variables constant except the one being tested so that the results can be attributed to a single cause.

Alter the URL parameters to perform the following tests on the **title** variable:

URL Parameters                | Result
------------------------------|---------------------------------------------------------------------------------
?title=1&action=search        | 'No movies were found!' is returned.
?title=&action=search         | Presumably all movies are returned.
?action=search                | Nothing is returned.
?title=**iron**&action=search | Presumably all movies containing the word 'the' (case-insensitive) are returned.

Given these results, we can determine that the **title** variable is being used in an SQL query that probably looks similar to: 

```sql
SELECT title, release, character, genre, imdb 
FROM movie
WHERE title LIKE '%title%'
ORDER BY title ASC;

/* Where in "..LIKE '%title%'", title is equal to $_GET["title"], which may or may not be validated, and % is a wildcard representing one or more characters. */
```

Now test the **action** variable:

URL Parameters      | Result
--------------------|------------------------------------
?title=&action=1    | Presumably all movies are returned.
?title=&action=test | Presumably all movies are returned.
?title=&action=     | Presumably all movies are returned.
?title=             | Presumably all movies are returned.

The **action** variable is not affecting an SQL query within our test scope and is therefore probably not a potential candidate for SQL injection.

> **NOTE**: We are performing black box testing, however, if we were referencing the server-side code in our test, we could confirm that **$\_GET["action"]** is not used anywhere within scope.

### Inject SQL Payloads

Types of SQL injection

- bypassing intrusion detection
  - encoding
  - Whitespace
  - null characters
  - commenting

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

We have only covered a basic example of SQL injection, which does not account for custom error messages, out-of-band injection, partial validation, intrusion prevention systems, or other advanced defenses that are encounter in real-world testing. See the resources provided below to continue learning and become a more efficient tester.

All resources are free, apart from _SQL Injection Attacks and Defense, 2nd ed_. Most items are followed by a year in parenthesis which represents the last year that resource was updated as of the date of this publication.

### Learning SQL

- [SQLBolt](https://sqlbolt.com/) - Interactive website for learning SQL.
- [Hacker Rank](https://www.hackerrank.com/domains/sql?filters%5Bstatus%5D%5B%5D=unsolved&badge_type=sql) - This site doesn't teach SQL but hosts a collection of SQL practice-questions ranging in complexity.

### Learning SQL Injection

- [SQL Injection Attacks and Defense, 2nd ed](https://www.amazon.com/Injection-Attacks-Defense-Justin-Clarke/dp/1597499633/ref=sr_1_2?keywords=sql+injection+book&qid=1552931384&s=gateway&sr=8-2) - The most thorough introduction to SQL injection available. (2012)

- [OWASP - Vulnerable web applications](https://www.owasp.org/index.php/OWASP_Vulnerable_Web_Applications_Directory_Project#tab=Main) - A list of vulnerable web applications, written in various technologies, for practicing security testing. (2018)

- [Devilbox](https://github.com/cytopia/devilbox) - A Docker composition providing local, quickly customizable, LAMP and MEAN development environments. (2019)

- [OWASP - Penetration testing methodologies](https://www.owasp.org/index.php/Penetration_testing_methodologies) (2019)

### Testing References

Known Vulnerabilities:

- [Exploit Database](https://www.exploit-db.com/)
- [National Vulnerability Database](https://nvd.nist.gov/vuln/search)

Payloads:

- [NetSPI - SQL attack queries](https://sqlwiki.netspi.com/attackQueries/) (2019)
- [Bug bounty cheat sheet - SQLi](https://github.com/EdOverflow/bugbounty-cheatsheet/blob/master/cheatsheets/sqli.md) - An unordered list of relevant cheat sheets. (2017)

Miscellaneous:

- [OWASP - Bypassing firewalls](https://www.owasp.org/index.php/SQL_Injection_Bypassing_WAF) (2017)

### Open Source Tools

- [Metasploit](https://github.com/rapid7/metasploit-framework) (2019)
- [OWASP-ZAP](https://github.com/zaproxy/zaproxy) (2019)
- [SQLMap](https://github.com/sqlmapproject/sqlmap) (2019)
- [SQLNinja](https://sourceforge.net/projects/sqlninja/) (2014)

### Reporting

- [OWASP - Vulnerability disclosure cheat sheet](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Vulnerability_Disclosure_Cheat_Sheet.md) (2019)

### Legal

- [Computer Crime Research Center - Legislation](http://www.crime-research.org/legislation/)
- [Fraud and related activity in connection with computers (U.S.)](https://www.law.cornell.edu/uscode/text/18/1030)

## References

- Clarke, Justin. _SQL Injection Attacks and Defense, 2nd ed_. Waltham: Syngress. 2012.
- Meucci, Matteo and Andrew Muller. _The OWASP Testing Guide, v4_. 2014.
- PCI Security Standards Council. _PCI Data Security Standard (PCI DSS), v1._. 2015.
