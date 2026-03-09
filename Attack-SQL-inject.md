# 1. Target
A blog website with upload, comment and login section. The first goal is to *login to this website without any meaningful information about user's credential*. Well, to be straightforward, it is logging in "unauthorized".

>[! First analyze]
>The web page is written with TypeScript, a modern language so, maybe the stacked SQL queries is not the best way to sneak in. So we will rely on two method: trying to commented out any following statement and trying to logging in with an always "TRUE" command.

**Motivation**: refer to [[Injection assignment]] and [[SQL Injection]]

The first trial is about to log in with "admin" user (if exists). The intention is naive, since not every systems have the `admin` user. So this is more or less a lucky guess. But it is clear that, with `--`, the password section can be ignored.
```SQL
Username = admin' --
```
So the SQL statement will receive `admin'` to form a statement, maybe look like this
```SQL
SELECT UUID from Users WHERE Username='admin' -- AND passwordhash=' ...
```
The input above has closing character '  and the comment `--`, so it's not matter what password you input, the statement will be 