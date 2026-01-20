# Information Disclosure

# LAB 1 : **Information disclosure in error messages**

### **The Core Concept**

This lab demonstrates **Improper Error Handling**.
When a web application crashes (encounters an error), it should ideally show a generic "Sorry, something went wrong" page.
However, if the developer forgets to turn off "Debug Mode" or catch errors properly, the server might vomit a **Stack Trace**.

A Stack Trace is a log of exactly what the code was doing when it crashed. Hackers love this because it often reveals:

- The framework being used (e.g., Apache Struts, Django, Spring).
- **The exact version number.**
- File paths on the server.

---

### **Step-by-Step Walkthrough**

You can use **Burp Suite** or just your **Browser** for this.

### **Step 1: Identify the Input**

1. Open the lab.
2. Click on any product to view its details.
3. Look at the URL in your browser:
`.../product?productId=1`*(The application expects a **Number** here).*

### **Step 2: Trigger the Error**

We want to break the application logic. The easiest way to crash a function expecting a number is to give it text.

1. Click in the URL bar.
2. Change `productId=1` to `productId=abc` (or just remove the number).
3. Press **Enter**.

### **Step 3: Analyze the Leak**

The page will likely display a "Internal Server Error" (Status 500), but instead of a blank page, you will see a massive wall of text (the Stack Trace).

1. Scroll through the error text.
2. Look for names of technologies. You will likely see references to **"Apache Struts"**.
3. Look closely for the version number next to it.
    - You should see something like `Apache Struts 2 2.3.31`.

### **Step 4: Submit the Solution**

1. Copy the version : `Apache Struts 2 2.3.31`
2. Click the **"Submit solution"** button in the lab header.
3. Paste the version number and submit.

## LAB 2 : **Information disclosure on debug page**

### **The Core Concept**

This lab demonstrates **Leftover Debug Code**.
During development, programmers often create special pages to verify server settings (e.g., checking PHP versions or environment variables). A classic example is a file named `phpinfo.php`.

If developers forget to delete these files before launching the website, anyone who guesses the URL can view sensitive server configuration details—including secret keys used for encryption.

---

### **Step-by-Step Walkthrough**

You can use **Burp Suite** or just your browser's **View Source** feature.

### **Step 1: Reconnaissance (Find the Link)**

Hackers always check the HTML source code for hidden clues left by developers.

1. Open the lab home page.
2. Right-click anywhere on the page and select **View Page Source**.
3. Scroll down or use `Ctrl+F` (Cmd+F) to search for "Debug".
4. You will find an HTML comment that looks like this:
    - *Note:* HTML comments (``) are invisible on the actual webpage but clearly visible in the code.

### **Step 2: Access the Debug Page**

1. Copy the path found in the comment: `/cgi-bin/phpinfo.php`.
2. Paste it at the end of your lab URL in the address bar.
    - Example: `https://YOUR-LAB-ID.web-security-academy.net/cgi-bin/phpinfo.php`
3. Press **Enter**.

### **Step 3: Extract the Secret**

You will see a long, messy table of configuration details (this is the standard PHP Info output).

1. Press `Ctrl+F` (Cmd+F) to open the browser search bar.
2. Type: `SECRET_KEY`.
3. You should find a row labeled **SECRET_KEY** with a random value next to it (e.g., `s3cr3t...`).
4. Copy that value.

### **Step 4: Submit the Solution**

1. Go back to the main lab page.
2. Click **"Submit solution"**.
3. Paste the key you found.

# LAB 3 : **Source code disclosure via backup files**

### **The Core Concept**

This lab combines two common mistakes:

1. **Backup Files:** Developers often save a copy of a file before editing it (e.g., `file.java.bak`). If these are stored in a public folder, anyone can download the original source code.
2. **Hardcoded Secrets:** Developers sometimes write passwords directly into the code to make testing easier, forgetting to remove them later.
3. **The "Robots" Map:** The `robots.txt` file tells Google and other browsers "Don't look in these folders." Paradoxically, this tells hackers exactly where the sensitive folders are!

---

### **Step-by-Step Walkthrough**

You can solve this entirely in your **browser**.

### **Step 1: Check `robots.txt`**

The first thing to check it robots.txt

1. Go to the lab home page.
2. Add `/robots.txt` to the end of the URL.
    - Example: `https://YOUR-LAB-ID.web-security-academy.net/robots.txt`
3. **Read the file.** You will see a line saying:
`Disallow: /backup`
    - *Translation:* "don't look inside the `/backup` folder."

### **Step 2: Browse the Hidden Directory**

1. Change the URL in your browser to visit that folder:
`.../backup`
2. You will see a file listing (Directory Indexing).
3. Click on the file named `ProductTemplate.java.bak`.
    - The `.bak` extension means it's a backup file. Since the browser doesn't know how to run a `.bak` file, it just shows you the text inside (the raw Java source code).

### **Step 3: Hunt for Secrets**

1. Read the code displayed in your browser.
2. Look for the section connecting to the database (usually looks like `ConnectionBuilder`).
3. You will see a line setting the password the hardcoded one .
4. Copy this password.

### **Step 4: Submit the Solution**

1. Go back to the main lab page.
2. Click **"**Submit solution**"**.
3. Paste the database password you found.

# LAB 4 : **Authentication bypass via information disclosure**

### **The Core Concept**

This lab involves a **misconfiguration** in the web server and a logic flaw in how the application trusts inputs.

1. **The Logic Flaw:** The application uses a specific HTTP header to determine your IP address. If that header says `127.0.0.1` (Localhost), the application treats you as an Admin.
2. **The Information Disclosure:** The name of this secret header is hidden. However, the server has the **`TRACE`** method enabled. `TRACE` is a diagnostic tool that tells the server: *"Please send back exactly what you received."*
    - When the front-end proxy adds the secret header to your request, the backend server receives it.
    - If you use `TRACE`, the backend echoes the request back to you—**including the secret header added by the proxy.**

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this lab.

### **Step 1: Discovery (Find the Header)**

1. **Try to access Admin:**
    - Open the lab browser.
    - Navigate to `/admin`.
    - You get a **401 Unauthorized** error: *"Admin interface only available if logged in as an administrator, or if requested from a local IP."*
2. **Send to Repeater:**
    - In Burp **Proxy > HTTP History**, find the `GET /admin` request.
    - Right-click and select **Send to Repeater**.
3. **Use TRACE:**
    - In Repeater, change the method from `GET` to `TRACE`.
    - Click **Send**.
4. **Analyze Response:**
    - Look at the response body. You will see your own request reflected back.
    - Look for a header you didn't write, likely starting with `X-`.
    - You should see: `X-Custom-IP-Authorization: [Your-IP]`.
    - **Copy the header name:** `X-Custom-IP-Authorization`.

### **Step 2: Automate the Bypass (Match and Replace)**

We now know the secret handshake. We need to attach this header to *every* request we make to browse the admin panel comfortably. We will use Burp's "Match and Replace" feature.

1. **Open Proxy Settings:**
    - Go to **Proxy** tab > **Proxy Settings** (or **Options** in older versions).
    - Scroll down to the **"Match and Replace"** section.
2. **Add a Rule:**
    - Click **Add**.
    - **Type:** Select **Request header**.
    - **Match:** Leave this **Empty** (this ensures the rule applies to *every* request).
    - **Replace:** Enter the spoofed header:
    `X-Custom-IP-Authorization: 127.0.0.1`
    - Click **OK**.

### **Step 3: Exploit and Solve**

1. Go back to the lab browser.
2. Refresh the `/admin` page.
    - *Behind the scenes:* Burp silently added `X-Custom-IP-Authorization: 127.0.0.1` to your request.
    - The server now thinks you are logging in from the server itself (Localhost).
3. You should now see the **Admin Panel**.
4. Find the user **carlos** and click **Delete**.

## Without  Automation :

- go to the /admin get request .
- then add the secret header and change your ip with [localhost](http://localhost) ip (127.0.0.1)
- send the request .
- then in the response search for the name (carlos) under the name carlos you will also see and link to delete the user carlos like this **“admin/delete?username=carlos”**
- change the request from /admin to “**admin/delete?username=carlos” and hit send .**

# LAB 5 :  **Information disclosure in version control history**

### **The Core Concept**

This lab demonstrates the danger of leaving the **`.git`** folder exposed on a public web server.
Git is a "Version Control System." It doesn't just store the *current* files; it stores **every change ever made**.

- If a developer accidentally writes a password into a file.
- Then realizes their mistake and "deletes" the password in a new update.
- **The password is still there** in the Git history!

If we can download the `.git` folder, we can travel back in time and read the files exactly as they were before the password was deleted.

---

### **Step-by-Step Walkthrough**

You will need a terminal with **`git`** and **`wget`** installed. (If you are on Windows, use Git Bash or WSL).

### **Step 1: Confirm the Vulnerability**

1. Open the lab home page.
2. Add `/.git` to the end of the URL.
    - Example: `https://YOUR-LAB-ID.web-security-academy.net/.git`
3. If you see a directory listing (folders like `HEAD`, `config`, `objects`), the vulnerability exists.

### **Step 2: Download the Repository**

We need to download this entire folder structure to our computer so we can treat it like a normal code project.

1. Open your terminal.
2. Run this command (replace with your specific lab URL):Bash
    
    `wget -r -np -nc https://YOUR-LAB-ID.web-security-academy.net/.git/`
    
    - `r`: Recursive (download all files and subfolders).
    - `np`: No parent (don't go up directories).
    - `nc`: No clobber (don't overwrite files).
3. Once finished, you will have a folder named `YOUR-LAB-ID.web-security-academy.net`. Go into it:Bash
    
    `cd YOUR-LAB-ID.web-security-academy.net`
    

### **Step 3: Analyze the History**

Now we use standard Git commands to look at what happened in the past.

1. View the commit logs:Bash
    
    `git log`
    
2. Read the commit messages. You are looking for something suspicious.
    - You will likely see a commit message like: **"Remove admin password from config"**.
    - Copy the **Commit Hash** (the long string of numbers/letters) for that specific commit.

### **Step 4: Extract the Secret**

We want to see what was changed in that commit.

1. Run the diff command with the hash you copied:Bash
    
    `git show <COMMIT-HASH>`
    
    *(Alternatively, you can just checkout that commit: `git checkout <COMMIT-HASH>`).*
    
2. Look at the output.
    - Lines starting with  (red) show what was **removed**.
    - Lines starting with `+` (green) show what was **added**.
    - You should see a line removing the hardcoded password (e.g., `ADMIN_PASSWORD = 'super_secret_123'`) and replacing it with an environment variable.

### **Step 5: Login and Solve**

1. Copy the password you found in the "removed" line.
2. Go back to the lab browser.
3. Click **"My account"**.
4. Log in as `administrator` with the password you found.
5. Go to the **Admin Panel** and delete the user `carlos`.