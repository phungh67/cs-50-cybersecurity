>[!Definition]
>SQL Injection is a code injection and of course, an attack technique that aim to (but not limited) to exploit, mess up the database and even authenticate without a valid credential. This is one of the most hacking method. 

## First example
Every web page needs to parse user input to authenticate them into the system. Mostly these data will be parsed as a string. Take the code snippet below as an example:
```Javascript
txtUserId = getRequestString("UserId");
txtSQL = "SELECT * FROM Users WHERE UserId = " + txtUserId;
```
The function will get input from user, then passes it to the SQL command. There are several ways to implement an attack:
- A TRUE condition
- Batched statement

### 1. Always TRUE condition
It is simple, relies on a statement that returns TRUE to inject malicious command or to authenticate to the system and bypassed the password check.
```SQL
SELECT * FROM Users WHERE UserId = 105 OR 1=1;
```
In the above statement, the condition `1=1` is always TRUE no matter what. So the command will return the user with Id of 105, without checking any credentials.
The input will be `UserId = 105 OR 1=1`

Another way is to use the empty string comparison.
```SQL
SELECT UserId, Name, Password FROM Users WHERE UserId = 105 AND Pass = "" or ""="";
```
Similarity, the "" = "" is an always TRUE condition because it compares two empty strings. Moreover, this SQL statement can be implemented like this:
```Javascript
uName = getRequestString("username");  
uPass = getRequestString("userpassword");  
  
sql = 'SELECT * FROM Users WHERE Name ="' + uName + '" AND Pass ="' + uPass + '"'
```


```SQL
--Injected statement
SELECT * FROM Users WHERE Name ="" or ""="" AND Pass ="" or ""=""
```
With the inputs are `User Name = " or ""="` and `Password " or""="`. With this attack method, the adversary can get all the password and user information from the database.

### 2. Batched SQL Statement
This is the scenario that they don't want to sneak in our system but want to corrupt it, so that all the sensitive information are lost.
```SQL
SELECT * FROM Users; DROP TABLE Suppliers
```
In the above line, we see that there are 2 instructions, one to query all possible user and to delete a table named `Suppliers`. It does not matter which statement was successfully executed, if not all users were exposed, it would be a table dropped.
The root cause came from this code snippet:
```Javascript
txtUserId = getRequestString("UserId");  
txtSQL = "SELECT * FROM Users WHERE UserId = " + txtUserId;
```
With the following input `105; DROP TABLE Suppliers`. But these attack only successes (maybe) on the old system or old programming language where stacked queries are still allowed. In modern framework (like the `vite` in the assignment, batched is not allowed).

## Second intuition: conducting some recon actions before attacking

In most scenarios, it is not easy to know the exact SQL statement that takes user's input and delivers it to the database. Or maybe, even the exact command was retrieved somehow, to be able to modify illegally, the table's name should be known. This case, we call it **blind-injection**, we don't know how many characters exist in the table name, how many arguments are needed for sending data, or what is the name of the table?

### 1. To find out "how does the SQL statement looks like"

The simplest method is to use `ORDER BY` keyword. It will sort the result based on returned columns. For example, if `ORDER BY 1`, it sorts all results accordingly to column 1 in `SELECT` statement. Also, if with value of 1, website successfully returns a success code (different than 400 or 500), we can safely assume that the query only returns 1 column, and will be look like: `SELECT <res> FROM ??? WHERE ???='' AND ???=''` 

This can be combined with a `Bash` script to find out how many arguments are there in the return result of login statement.

Secondly, `UNION` can be used to find how many results were returned. Because if the statement after `UNIO` keyword returned different number of columns, website would throw an error.

```SQL
admin' UNION SELECT NULL, NULL --
```

With these 2 methods, we can figure out how many arguments in the `res`, in combination with "input" box (username and password), we can reconstruct the SQL statement with a high accuracy.

### 2. To find the name of table

After somehow get into the system, we will want to insert malicious data, or some fake user inside the actual database. Because the code base can be changed, hardened to avoid security risks, but if attackers can put an "agent" into the system, that credential can be used for a relatively long time (at least to the next audit period of the database).

But to do this, there are 2 main things to concern: table name's length and which characters are in the name?

First - to find the name length:
```SQL
admin' AND (SELECT LENGTH(name) FROM sqlite_master WHERE type='table' LIMIT 1) = $i --
```

Focus on the payload, this attack leverage the `AND` logic, if two sides are both correct, we can successfully log in. This intuition can be modified based on the database engine (`MySQL`, `PostgreSQL`, `Microsoft SQL Server`), in here we looked into `master_table` of the `sqlite` to peek into the list of all tables. And of course, it only probes the first entry in this list. The parameter `$i` can be dynamically scanned through, and if `i` equals to length of first table, login would be granted,

In this case, we of course need some knowledge, or so called "social engineering" - you have to guess or propose some theory about "How long a typical table in the database is?". At lest the shortest table name are 4 characters (User) and longest should not exceed 64 characters, included the underline character.

Secondly - to find the actual name:

After successfully guessing the length of table name, we will try to brute force its name. To make everything more easily, the table name is advised to not having special character like @ or % because it will mess the SQL statement up. So we just have uppercase, lowercase, number and dash, hyphen

```SQL
PAYLOAD="admin' AND (SELECT SUBSTR(name, $pos, 1) FROM sqlite_master WHERE type='table' LIMIT 1) = '$CHAR' --
```

We can try the same method as above, and print the "full string".

## Illegally modify the database

This is a tricky part, because as stated before, the website will likely block the stacked query, so that the semicolon could not be used to execute desired statements. That is where pipeline character `||` comes in handy. But as usual, it heavily depends on how the programmers actually coded the website.

```SQL
' || (SELECT sql FROM sqlite_master WHERE name='Users_table') || '
```

In the above statement, if the payload was successfully uploaded to website, the SQL statement would be come `Do something then SELECT ... then continue the usual thing` or the `--` can be combined to completely ignore the rest of statement.

But there is one thing to keep in mind: a service's internal door and external can be protected differently. That is, while the login denies stacked query, the other functions do not. So it is still worthy to try stacked query to find if it is possible to do so:
```SQL
'); INSERT INTO Table_Users (Username, Password, SID) VALUES ('hacker_admin', 'e10adc3949ba59abbe56e057f20f883e', 'hacker_sid'); --
```
