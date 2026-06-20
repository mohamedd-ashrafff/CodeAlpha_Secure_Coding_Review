# Secure Coding Review Report

## 1. Target Application
- **Language:** Python (Flask) / SQLite
- **Component:** User Login Authentication Module
- **Review Type:** Manual Code Inspection

## 2. Vulnerable Code Snippet
The following code was found to be directly concatenating user input into the SQL query, which is a classic security flaw.

```python
from flask import Flask, request
import sqlite3

app = Flask(__name__)

@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']
    
    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()
    
    # VULNERABILITY: SQL Injection 
    query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
    cursor.execute(query)
    
    user = cursor.fetchone()
    if user:
        return "Login Successful!"
    else:
        return "Invalid Credentials!"
```

## 3. Vulnerability Identified: SQL Injection (SQLi)
- **Severity:** Critical
- **Description:** The application constructs a SQL query using unsanitized user input (`f-strings`). An attacker can bypass authentication by injecting SQL payloads. 
- **Proof of Concept (PoC) Input:** If an attacker enters `' OR '1'='1` in the username field, the query becomes:
  `SELECT * FROM users WHERE username = '' OR '1'='1' AND password = ''`
  This evaluates to True, bypassing the password check entirely.

## 4. Remediation & Best Practices
To fix this, the application must use **Parameterized Queries (Prepared Statements)**. This ensures that the database treats user input strictly as data, not as executable SQL commands.

## 5. Secure Code (Fixed)
Here is the refactored, secure version of the code:

```python
from flask import Flask, request
import sqlite3

app = Flask(__name__)

@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']
    
    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()
    
    # SECURE: Using Parameterized Queries
    query = "SELECT * FROM users WHERE username = ? AND password = ?"
    cursor.execute(query, (username, password))
    
    user = cursor.fetchone()
    if user:
        return "Login Successful!"
    else:
        return "Invalid Credentials!"
```
