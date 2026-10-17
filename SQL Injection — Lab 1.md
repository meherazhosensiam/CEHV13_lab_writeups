# CEH v13 Lab Report: SQL Injection — Lab 1

**Platform:** PortSwigger Web Security Academy  
**Lab Name:** SQL injection vulnerability in WHERE clause allowing retrieval of hidden data  
**Difficulty:** Apprentice   
**Vulnerability Type:** SQL Injection (SQLi)

## 1. Objective

The objective of this lab was to understand how SQL Injection works in a web application and how an attacker can manipulate a database query to retrieve products that are normally hidden from users.

I also wanted to learn how user-controlled input can affect SQL query logic and how this vulnerability can be identified using basic web security testing techniques.

## 2. Tools Used

- PortSwigger Web Security Academy
- Web browser
- Burp Suite Community Edition (optional, for intercepting and modifying HTTP requests)

## 3. Lab Overview

The lab contains a vulnerability in the product category filter. Normally, the application displays products belonging to a selected category and hides unreleased products.

The application uses a query similar to the following:

```sql
SELECT * FROM products
WHERE category = 'Gifts'
AND released = 1;
```

Here, `category = 'Gifts'` filters products by category, while `released = 1` ensures that only released products are displayed.

According to the official lab description, the goal is to manipulate this query so that one or more unreleased products become visible.

## 4. Methodology

### Step 1: Access the lab

I opened the official PortSwigger Web Security Academy lab and launched the target website.

Official lab link: https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data

### Step 2: Understand the normal behavior

I examined the product listing and selected a category from the available filters.

The application returned products belonging to that category. This helped me identify the category filter as a possible input point for testing.

### Step 3: Understand the SQL Injection technique

I studied how an SQL Injection payload can change the logic of a database query.

For this lab, the relevant example payload is:

```sql
' OR 1=1--
```

When supplied in the appropriate context, this payload can change the query's logical condition.

For example, the resulting query may resemble:

```sql
SELECT * FROM products
WHERE category = ''
OR 1=1--'
AND released = 1;
```

The exact behavior depends on the database engine and how the application constructs the query. In this example, `OR 1=1` introduces a condition that is always true, while `--` comments out the remaining SQL on compatible database engines.

### Step 4: Test the category parameter

I would use the browser or Burp Suite to identify the HTTP request responsible for changing the category filter.

I would then modify the category parameter with the payload and resend the request to observe whether the application returns additional products.

### Step 5: Verify the result

The expected result is that the application displays products that were previously hidden, including unreleased products. The lab should report success when its completion condition is satisfied.

**Evidence note:** These are the expected results from the documented lab solution. Actual completion should be confirmed in the running lab.

## 5. Observations

The lab demonstrates several important concepts:

1. User-controlled input can influence the structure of a database query.
2. The `WHERE` clause determines which database records are returned.
3. The `OR` operator can change a filtering condition when an injected expression evaluates to true.
4. SQL comments can prevent the remaining portion of a query from being interpreted as SQL.
5. A simple category filter can expose data that the application intends to keep hidden.

## 6. Vulnerability Analysis

**Root cause:** The vulnerability occurs when untrusted user input is incorporated into an SQL query in a way that allows the input to change the query's intended structure.

**Potential impact:** Depending on the affected query and database permissions, SQL Injection can expose unauthorized records and, in more serious cases, allow other database operations.

**Classification:** CWE-89 — Improper Neutralization of Special Elements Used in an SQL Command ('SQL Injection').

## 7. Recommended Mitigation

The primary defense is to use parameterized queries or prepared statements rather than concatenating user input into SQL commands.

For example, in Python with SQLite:

```python
query = """
    SELECT * FROM products
    WHERE category = ?
    AND released = 1
"""

cursor.execute(query, (category,))
```

The `?` placeholder separates the SQL command from the supplied category value. The database treats the category as data rather than executable SQL syntax.

Additional security measures include:

- Validate user input on the server side.
- Apply least-privilege permissions to database accounts.
- Avoid exposing unnecessary database errors to users.
- Test database-backed features regularly.

## 8. Learning Outcomes

Through this lab, I learned the basic concept of SQL Injection, the purpose of a `WHERE` clause, and how Boolean conditions can alter query behavior. I also learned why parameterized queries are important for secure web application development.

As a beginner studying CEH v13, this exercise helped me connect SQL fundamentals with practical web application security testing.

## 9. Conclusion

This lab demonstrates how an SQL Injection vulnerability in a product category filter can allow users to retrieve hidden database records. The key lesson is that user input must not be allowed to modify the intended structure of a database query.

Understanding this basic technique provides a foundation for studying more advanced SQL Injection vulnerabilities, including authentication bypass, `UNION` attacks, and blind SQL Injection.

**References**

1. PortSwigger Web Security Academy — [SQL Injection: Retrieving hidden data](https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data)
2. PortSwigger — [SQL Injection Learning Path](https://portswigger.net/web-security/learning-paths/sql-injection)
3. MITRE CWE-89 — [SQL Injection](https://cwe.mitre.org/data/definitions/89.html)
