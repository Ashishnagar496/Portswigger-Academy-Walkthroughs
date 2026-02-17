# Path Traversal

# LAB 1 :  **File path traversal, simple case**

**The Core Concept**
This vulnerability (also called Directory Traversal) happens when a website loads a file based on user input without checking *where* that file is located.
Imagine the server keeps images in a folder like `/var/www/images/`.
• **Normal Request:** `filename=cat.jpg` → Server reads `/var/www/images/cat.jpg`.
• **Attack:**  for `../../etc/passwd`.
    ◦ `../` means "Step back one folder".
    ◦ So the server reads: `/var/www/images/../../etc/passwd`.
    ◦ Mathematically: `/var/www/images/` (step back) → `/var/www/` (step back) → `/var/` (step back) → `/` (Root directory) → `/etc/passwd`.
We use `../` multiple times to ensure we go all the way up to the root of the computer's file system.

**Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this.

**Step 1: Identify the Target Request**
1. Open the lab.
2. Right-click on any product image (e.g., the backpack or camera) and select **"Open image in new tab"**.
    ◦ *Alternatively:* Just refresh the main page while Burp is running.
3. Go to **Burp Proxy > HTTP History**.
4. Look for a request that fetches an image. It will look like:
GET /image?filename=70.jpg
**Step 2: Inject the Payload**
1. Right-click that request and select **Send to Repeater**.
2. In Repeater, look at the `filename` parameter.
3. Replace the image name (e.g., 70.jpg) with the traversal payload:
../../../etc/passwd
    ◦ *Why 3 times?* We are guessing the images are stored 3 folders deep (e.g., `/var/www/html/images`). Using "too many" `../` doesn't hurt (you just hit the root and stay there), but using too few won't work.

**Step 3: Launch and Verify**
1. Click **Send**.
2. **Analyze the Response:**
    ◦ You should see a **200 OK** status.
    ◦ The "Body" of the response will not be an image anymore. It will be text starting with:
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
3. The lab is now solved.

# LAB 2 : File path traversal, traversal sequences blocked with absolute path bypass

### **The Core Concept**

Sometimes developers try to fix path traversal by explicitly blocking the `../` characters (the "dot-dot-slash"). 

However, they often forget to block **Absolute Paths**.

- **Relative Path:** `../../etc/passwd` (Navigate relative to where I am now).
- **Absolute Path:** `/etc/passwd` (Navigate directly to this specific address on the hard drive).

If the server code simply concatenates the input (e.g., `File("/var/www/images/" + userInput)`), many programming languages will ignore the first part if `userInput` starts with a root slash `/`. It essentially overrides the base directory.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this.

### **Step 1: Identify the Target Request**

1. Open the lab.
2. Refresh the page to load the product images.
3. In **Burp Proxy > HTTP History**, find the `GET /image?filename=...` request.
4. Right-click it and select **Send to Repeater**.

### **Step 2: Test the Block**

1. First, try the standard attack: `../../../etc/passwd`.
2. Click **Send**.
3. You will likely get a **400 Bad Request** or **"No such file"** because the application detected the `../` and blocked it.

### **Step 3: The Absolute Bypass**

1. Now, remove the `../` entirely.
2. Change the `filename` parameter to the direct absolute path:
`/etc/passwd`
3. Click **Send**.

### **Step 4: Verify Success**

1. Check the response body.
2. You should see the content of the file (`root:x:0:0...`).
3. The lab is marked as Solved.

# LAB 3 : **File path traversal, traversal sequences stripped non-recursively**

### **The Core Concept**

This lab demonstrates a poorly implemented security filter. The developer tried to protect the site by deleting the ../ characters in the filename .

However, the computer only does this **once**.

- **The Trick:** We can hide a `../` inside another `../`.
- **Input:** `....//`
- **Filter Action:** The filter sees the middle `../` and deletes it.
- **Result:** The remaining `..` and `/` collapse together to form a new `../` that the filter misses!

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this.

### **Step 1: Capture the Request**

1. Open the lab.
2. Refresh the page to load images.
3. In **Burp Proxy > HTTP History**, find the `GET /image?filename=...` request.
4. Right-click it and select **Send to Repeater**.

### **Step 2: Construct the Nested Payload**

We want to go up 3 directories (`../../../`). But since the server deletes `../`, we need to double up.

- **Standard:** `../`
- **Nested:** `....//` (Two dots, two dots, slash, slash)
- **Full Payload:** `....//....//....//etc/passwd`

### **Step 3: Execute the Attack**

1. In Repeater, replace the filename with the nested payload:
`....//....//....//etc/passwd`
2. Click **Send**.
3. **What happens on the server:**
    - Server receives: `....//....//....//etc/passwd`
    - Server filter deletes the `../` it finds in the middle.
    - Server is left with: `../../../etc/passwd`.
    - Server executes the path.

### **Step 4: Verify Success**

1. Check the response body.
2. You should see the content of `/etc/passwd`.
3. The lab is marked as Solved.

# LAB 4 : **File path traversal, traversal sequences stripped with superfluous URL-decode**

**The Core Concept**
This lab exploits a timing mismatch in how the server processes data. It involves **Double URL Encoding**.

1. **The Filter:** The security looks for the dangerous sequence `../`.
2. **The Encoding:** In URLs, special characters are encoded (e.g., `/` becomes `%2f`).
3. **The Flaw:** The server decodes the URL **twice**.
    ◦ If we send `../` encoded as `%2f`, the security might decode it once, see the `/`, and block it.
    ◦ **The Trick:** We encode the `%` symbol itself!
    ◦ `%` becomes `%25`.
    ◦ So, `/` → `%2f` →`%252f`.

**The Attack Flow:**
1. **We send:** `..%252f`
2. **Server Decode 1:** `%25` becomes `%`. The string becomes `..%2f`.
3. **Server Decode 2 (The Bug):** `%2f` becomes `/`. The string becomes `../`.
4. **File System:** Sees `../../` and executes the hack.

**Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this.

**Step 1: Capture the Request**
1. Open the lab.
2. Refresh the page to load images.
3. In **Burp Proxy > HTTP History**, find the `GET /image?filename=...` request.
4. Right-click it and select **Send to Repeater**.

**Step 2: Construct the Double-Encoded Payload**

We need to replace every slash `/` in our standard attack with `%252f`.
• **Standard:** `../../../etc/passwd`
• **Single Encoded:** `..%2f..%2f..%2fetc/passwd`
• **Double Encoded:** `..%252f..%252f..%252fetc/passwd`

**Step 3: Execute the Attack**

1. In Repeater, replace the filename value with the double-encoded payload:
..%252f..%252f..%252fetc/passwd
2. Click **Send**.

**Step 4: Verify Success**

1. Check the response body.
2. You should see the content of `/etc/passwd`.
3. The lab is marked as Solved.
*(Note: Sometimes you only need to double-encode the slash. Sometimes you need to double-encode the dots too, e.g., `%252e%252e%252f`. But here, encoding the slash is enough).*

# LAB 5 : File path traversal, validation of start of path

### **The Core Concept**

This lab demonstrates a flawed whitelist defense. 

It is like a security guard who checks if you are wearing the company uniform at the entrance, but then ignores whatever you do once you are inside.

- **The Check:** "Does the path start with `/var/www/images/`?"
- **The Bypass:** We give them exactly what they want at the beginning, but then immediately use `../` to walk back out of that folder and go wherever we want.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this.

### **Step 1: Analyze the Original Request**

1. Open the lab.
2. Refresh the page to load images.
3. In **Burp Proxy > HTTP History**, find the `GET /image?filename=...` request.
4. **Observe the parameter:** You will likely see that the application is requesting the *full path* by default, not just the filename.
    - Example: `filename=/var/www/images/25.jpg`

### **Step 2: Construct the Payload**

We need to keep the start of the string identical to satisfy the security check.

- **Required Prefix:** `/var/www/images/`
- **Traversal:** `../../../` (to step back 3 times to root).
- **Target:** `etc/passwd`

**Combined Payload:**`/var/www/images/../../../etc/passwd`

### **Step 3: Execute the Attack**

1. Right-click the request and select **Send to Repeater**.
2. Replace the `filename` value with your payload:
`/var/www/images/../../../etc/passwd`
3. Click **Send**.

### **Step 4: Verify Success**

1. Check the response body.
2. You should see the content of `/etc/passwd`.
3. The lab is marked as Solved.

# LAB 6 : **File path traversal, validation of file extension with null byte bypass**

### **The Core Concept**

This lab exploits a difference in how high-level web languages (like PHP or Java) and low-level operating systems (like C/Linux) read text strings.

1. **The Constraint:** The application checks: *"Does this filename end with `.png`?"* If not, it blocks you.
2. **The Bypass (Null Byte `%00`):** In low-level programming, a "Null Byte" (Hex `00`) acts as a **Stop Sign**. It tells the system "This is the end of the string."

**The Attack:**
We send: `../../../etc/passwd%00.png`

- **The Web App Check:** It looks at the very end. "Does it end in `.png`? Yes." -> **Passes Security.**
- **The Operating System:** It starts reading the file path. It sees `passwd`, then hits the `%00` (Stop Sign). It stops reading immediately, ignoring the `.png` part.
- **Result:** The system loads `../../../etc/passwd`.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this.

### **Step 1: Capture the Request**

1. Open the lab.
2. Refresh the page to load images.
3. In **Burp Proxy > HTTP History**, find the `GET /image?filename=...` request.
4. Right-click it and select **Send to Repeater**.

### **Step 2: Construct the Null Byte Payload**

We need to satisfy the `.png` requirement but "cut it off" before the OS sees it.

- **Target:** `../../../etc/passwd`
- **The Terminator:** `%00` (URL-encoded null byte)
- **The Fake Extension:** `.png`

**Combined Payload:**`../../../etc/passwd%00.png`

### **Step 3: Execute the Attack**

1. In Repeater, replace the `filename` value with your payload:
`../../../etc/passwd%00.png`
2. Click **Send**.

### **Step 4: Verify Success**

1. Check the response body.
2. You should see the content of `/etc/passwd`.
3. The lab is marked as Solved.

*(Note: This technique is effectively "extinct" in modern PHP (versions 5.3+) and Java (versions 7u40+), but it is crucial for understanding legacy systems and C-based applications).*