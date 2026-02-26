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
With the following input `105; DROP TABLE Suppliers`

## Practice of Injection

[[Injection assignment]]
