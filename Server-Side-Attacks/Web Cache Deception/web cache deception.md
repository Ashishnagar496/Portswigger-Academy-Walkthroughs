# web cache deception :

# **Lab 1:  Exploiting path mapping for web cache deception**

## The Core Concept

This lab demonstrates **Web Cache Deception (WCD)**.
Web Cache Deception occurs when an attacker tricks a caching proxy into storing a sensitive, private response (like a user's account page) and making it available via a public URL. This is possible due to a **path mapping discrepancy**:

1. **The Origin Server:** Is configured to ignore extra path segments. It sees `/my-account/abc.js` and simply serves the private `/my-account` page.
2. **The Cache:** Sees the `.js` extension and assumes it is a public, static file. It then caches the private response and serves it to anyone who visits that exact URL.

---

## Step-by-Step Walkthrough

You will need to use your own account to find the vulnerability, then use the **Exploit Server** to target the victim.

### Step 1: Detect Path Mapping Abstraction

- Log in as `wiener:peter`.
- In **Burp Repeater**, send a request to `/my-account`. You see your API key.
- Now, change the path to `/my-account/abc`.
- **Observation:** The server still returns your account page. This means the server "abstracts" the path—it ignores the `/abc` and treats it as a request for `/my-account`.

### Step 2: Test Cache Rules

- Change the path to `/my-account/test.js`.
- Check the response headers:
    - Look for `X-Cache: miss`.
    - Look for `Cache-Control: max-age=30`.
- **The Test:** Send the request again immediately. If `X-Cache` changes to **hit**, the cache has been tricked into storing your private account page because it thinks it’s a public JavaScript file.

### Step 3: Craft and Deliver the Exploit

- Go to the **Exploit Server**.
- In the **Body**, add a script that forces the victim (**carlos**) to visit a unique "static" URL on the lab site:
`<script>document.location="https://YOUR-LAB-ID.web-security-academy.net/my-account/exploit.js"</script>`
- Click **Deliver exploit to victim**.
- The victim's browser will now request their account page through the cache at that specific URL. The cache will save Carlos's private data under the name `exploit.js`.

### Step 4: Exfiltrate the Data

- Quickly visit the URL you sent to Carlos in your own browser:
`https://YOUR-LAB-ID.web-security-academy.net/my-account/exploit.js`
- Because the response is cached, you will see **Carlos's account details** instead of your own.
- Copy his **API key** and submit it to solve the lab.

---

# **Lab 2 : Exploiting path delimiters for web cache deception :**

### **The Core Concept**

This lab demonstrates **Web Cache Deception (WCD)**.
This vulnerability occurs when a web cache and an origin server (the backend application) disagree on how to interpret a URL.

- **The Cache's View:** The cache sees a URL ending in `.js` (e.g., `/my-account;wcd.js`) and thinks, "Oh, this is a static JavaScript file. I should save this so I can serve it faster next time."
- **The Origin's View:** The origin server sees the semicolon `;` as a delimiter (a separator). It ignores everything after it. It sees `/my-account;wcd.js` and thinks, "The user wants `/my-account`. I will serve their private account page."
- **The Exploit:** We trick the victim into visiting `/my-account;wcd.js`. The server generates their private page (with API key). The cache saves it because it looks like a JS file. Then, *we* visit that same URL, and the cache serves us the victim's private page.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this lab.

### **Step 1: Identify the Target and Behavior**

1. **Log in:** Access the lab and log in with `wiener` / `peter`.
2. **Observe:** Your API key is displayed on the `/my-account` page. This is the secret we want to steal from Carlos.
3. **Capture Traffic:** In **Burp Proxy > HTTP History**, find the `GET /my-account` request.
4. **Send to Repeater:** Right-click and select **Send to Repeater**.

### **Step 2: Hunt for Delimiters**

We need to find a character that the server treats as a "stop" sign, but the cache treats as part of the filename.

1. **Test 1 (Standard Path):** Change the path to `/my-account/abc`.
    - *Result:* **404 Not Found**. The server thinks you are looking for a sub-folder that doesn't exist.
2. **Test 2 (Arbitrary String):** Change path to `/my-accountabc`.
    - *Result:* **404 Not Found**.
3. **Fuzzing for Delimiters:**
    - Right-click the request -> **Send to Intruder**.
    - **Position:** Set the payload position between `account` and `abc`: `/my-account§§abc`.
    - **Payloads:** Use a list of special characters (e.g., `;`, `?`, `#`, `:`, `,`). *Note: The lab provides a specific list if you need it.*
    - **Encoding:** Uncheck "URL-encode these characters" at the bottom.
    - **Launch Attack.**
4. **Analyze Results:**
    - Sort by Status Code.
    - You should see that **`;` (semicolon)** and **`?` (question mark)** return a **200 OK**.
    - *Meaning:* The server sees `/my-account;abc` and ignores the `;abc` part, serving the normal `/my-account` page.

### **Step 3: Check for Caching Discrepancies**

Now we check if the cache is fooled by these characters.

1. **Test `?`:**
    - In Repeater, try `/my-account?abc.js`.
    - Send it twice. Check the headers.
    - *Result:* Likely `X-Cache: miss` or no cache headers. Caches usually know that `?` starts a query string and rarely cache dynamic queries by default.
2. **Test `;`:**
    - In Repeater, try `/my-account;abc.js`.
    - Send it. Response: `X-Cache: miss`.
    - Send it **again**. Response: `X-Cache: hit`.
    - *Bingo!* The cache saw `.js` at the end and cached the response. The server saw `;` and served the sensitive account page. We have found our exploit vector.

### **Step 4: Craft the Exploit**

We need to make Carlos visit this URL so his data gets cached.

1. Go to the **Exploit Server** (button in top banner).
2. In the **Body** section, write a script to force Carlos to visit the URL:HTML
    
    `<script>document.location="https://YOUR-LAB-ID.web-security-academy.net/my-account;wcd.js"</script>`
    
    - *Important:* Use a unique string (like `wcd.js` or `stealing.js`) to ensure you are creating a fresh cache entry.
3. Click **Store**.
4. Click **Deliver exploit to victim**.

### **Step 5: Steal the Data**

1. Wait a few seconds for the victim to "visit" the link.
2. In your own browser (or Incognito window), navigate to the exact URL you sent the victim:
`https://YOUR-LAB-ID.web-security-academy.net/my-account;wcd.js`
3. **Success:** You should see the `/my-account` page. But wait... whose account is it?
    - Check the API Key on the screen. It is **Carlos's key**, not yours! You are viewing the cached copy of his private page.
4. Copy the API Key.

### **Step 6: Solve the Lab**

1. Click **Submit solution** in the lab banner.
2. Paste Carlos's API Key and submit.

# **Lab 3 : Exploiting origin server normalization for web cache deception**

### **The Core Concept**

This lab demonstrates **Web Cache Deception via URL Normalization**.

- **Normalization:** Web servers are "smart." If you request `/folder/../page`, the server simplifies (normalizes) this to just `/page`.
- **The Discrepancy:** The cache might be "dumb." It looks at the URL `/resources/..%2fmy-account` and thinks, *"This starts with `/resources`, which is a public static folder. I should cache this!"*
- **The Exploit:**
    1. We send the victim to `/resources/..%2fmy-account`.
    2. The **Origin Server** normalizes the path, realizes it means `/my-account`, and generates the victim's private profile.
    3. The **Cache** sees the `/resources` prefix, assumes it's safe static content, and **saves the private profile**.
    4. We visit that same URL and download the victim's cached profile.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this lab.

### **Step 1: Verify Normalization**

First, we need to prove that the server "fixes" weird paths (decodes and traverses directories).

1. **Log in:** Access the lab and log in with `wiener` / `peter`.
2. **Capture Traffic:** In **Burp Proxy > HTTP History**, find the `GET /my-account` request. Send it to **Repeater**.
3. **Test Traversal:**
    - Change the path to: `/aaa/..%2fmy-account`
    - *Note:* `..%2f` is the URL-encoded version of `../`.
    - Click **Send**.
4. **Analyze:**
    - If you get a **200 OK** and the response contains your account details, the server successfully normalized the path (it went into `/aaa`, then back out `..`, then into `/my-account`).

### **Step 2: Identify the Cache Rule**

Now we need to find a "sticky" directory—a folder where the cache aggressively saves everything.

1. Look at your HTTP history. Notice that static files (images, CSS) are loaded from `/resources/`.
2. **Test the Rule:**
    - Send a request to `/resources/test`.
    - Check the response headers. You might see `X-Cache: miss`.
    - Send it again.
    - If you see **`X-Cache: hit`**, it confirms that **anything** starting with `/resources/` gets cached.

### **Step 3: Combine the Exploit**

We need to combine the "sticky" cache directory (`/resources`) with the normalization trick (`..%2fmy-account`).

1. In Repeater, construct this specific path:
`/resources/..%2fmy-account`
2. **Send Request 1:** You should see your account page (200 OK) and `X-Cache: miss`.
3. **Send Request 2:** Send it again.
4. **Verify:** You should see **`X-Cache: hit`**.
    - *Why this works:* The Cache saw `/resources...` and saved it. The Server saw `../my-account` and served private data.

### **Step 4: Craft the Attack Script**

We need to force Carlos to visit this URL.

1. Go to the **Exploit Server** (button in the top banner).
2. In the **Body** section, paste this script:HTML
    
    `<script>document.location="https://YOUR-LAB-ID.web-security-academy.net/resources/..%2fmy-account?carlos"</script>`
    
    - *Important:* Replace `YOUR-LAB-ID` with your actual lab domain.
    - *Note:* I added `?carlos` as a "cache buster" to ensure we create a brand new cache entry for the victim.
3. Click **Store**.
4. Click **Deliver exploit to victim**.

### **Step 5: Steal the API Key**

1. Wait a few seconds.
2. In your own browser (or Burp Repeater), navigate to the exact URL you sent Carlos:
`https://YOUR-LAB-ID.web-security-academy.net/resources/..%2fmy-account?carlos`
3. **Success:** You will see a cached copy of the account page.
4. Look for the **API Key**. It belongs to Carlos.

### **Step 6: Solve the Lab**

1. Copy the API Key.
2. Click **Submit solution**.
3. Paste the key and click OK.

# LAB 4 : **Exploiting cache server normalization for web cache deception :**

### **The Core Concept**

This lab demonstrates **Web Cache Deception via Cache Normalization**.
This is the opposite of the previous lab.

- **The Origin Server:** Is "dumb." It does *not* normalize paths. If you send `/folder/..%2fpage`, it gets confused (404). However, it *does* recognize delimiters like `#` (encoded as `%23`) and stops reading the path there.
- **The Cache:** Is "smart." It aggressively decodes the URL and normalizes it *before* checking its rules.
- **The Exploit:**
    1. We construct a URL like `/my-account%23%2f%2e%2e%2fresources`.
    2. **Origin View:** Sees `/my-account` followed by a `#` delimiter. It ignores the rest and serves the private profile.
    3. **Cache View:** Decodes `%23` to `#`, but importantly, it also decodes `%2f%2e%2e%2f` to `/../`. It normalizes the path: `/my-account` -> (stop at hash?) -> actually, the lab implies the cache sees the directory traversal.
    - *Correction for this specific lab logic:* The cache decodes the encoded characters. It sees `/my-account` + `#` + `/../resources`. Because of how it processes the traversal (or how it matches rules), it thinks this request maps to a static resource (or it simply caches the response because it thinks it matches a rule for `/resources`).
    - *Simpler explanation:* We trick the cache into thinking the request is for a static resource (by traversing to a static folder *after* a delimiter), while the origin server stops processing *at* the delimiter and serves the sensitive page.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this lab.

### **Step 1: Identify Delimiters**

First, find a character that makes the origin server "stop reading" the URL but still serves the page.

1. **Log in:** Access the lab and log in with `wiener` / `peter`.
2. **Capture Traffic:** In **Burp Proxy > HTTP History**, find the `GET /my-account` request. Send it to **Repeater**.
3. **Fuzzing:**
    - Send to **Intruder**.
    - Payload position: `/my-account§§abc`.
    - Payloads: Common delimiters (`?`, `#`, `;`, etc.). **Important:** Ensure you test URL-encoded versions like `%23` (for `#`) and `%3f` (for `?`).
    - *Result:* You will find that **`%23`** (encoded `#`) returns a **200 OK**.
    - *Why `%23`?* The browser doesn't send `#` to the server (it stays client-side), but it *does* send `%23`. The origin server decodes it, sees `#`, and treats it as the end of the URL.

### **Step 2: Check Normalization Behavior**

We need to see who normalizes paths (decodes `..%2f` to `../` and moves up a directory).

1. **Test Origin:**
    - Send `/aaa/..%2fmy-account`.
    - *Result:* **404 Not Found**. The origin server does *not* normalize. It thinks you want a literal file named `..%2fmy-account`.
2. **Test Cache:**
    - Find a static resource in your history (e.g., `/resources/image.png`).
    - Send `/aaa/..%2fresources/image.png` to Repeater.
    - Send it twice.
    - *Result:* **`X-Cache: hit`**.
    - *Meaning:* The cache **does** normalize! It resolved `/aaa/..%2f` back to root, saw `/resources/...`, and applied its caching rule.

### **Step 3: Construct the Exploit Path**

We need a path that:

1. Starts with the sensitive page: `/my-account`.
2. Includes the delimiter (`%23`) so the origin stops reading there.
3. Includes a traversal to a static folder (`/../resources`) so the cache thinks it's a static file.

**The Payload Construction:**

- Start: `/my-account`
- Delimiter: `%23`
- Traversal (Encoded): `%2f%2e%2e%2f` (This is `/../`)
- Static Folder: `resources`

**Final Path:** `/my-account%23%2f%2e%2e%2fresources`

### **Step 4: Verify the Exploit**

1. In Repeater, paste that path: `/my-account%23%2f%2e%2e%2fresources`.
2. **Send Request 1:** Response contains your API key. Header: `X-Cache: miss`.
3. **Send Request 2:** Response contains your API key. Header: **`X-Cache: hit`**.
4. It works! The cache normalized the path (ignoring the `#` effectively for rule matching or processing the traversal) and saved the response.

### **Step 5: Deliver to Victim**

1. Go to the **Exploit Server**.
2. **Body:**HTML
    
    `<script>document.location="https://YOUR-LAB-ID.web-security-academy.net/my-account%23%2f%2e%2e%2fresources?wcd"</script>`
    
    - *Note:* The `?wcd` is just a cache buster.
3. Click **Deliver exploit to victim**.

### **Step 6: Steal the Data**

1. Wait a moment.
2. Visit the exact URL you sent the victim:
`https://YOUR-LAB-ID.web-security-academy.net/my-account%23%2f%2e%2e%2fresources?wcd`
3. You will get the cached response containing **Carlos's API Key**.
4. Copy the key and submit the solution.

# **Lab 5: Exploiting exact-match cache rules for web cache deception**

### **The Core Concept**

This lab combines **Web Cache Deception** with **CSRF (Cross-Site Request Forgery)**.

- **The Cache Rule:** The cache is configured to always cache specific "exact match" files, like `/robots.txt`. If a URL ends in `/robots.txt` (after normalization), the cache saves it.
- **The Deception:** We construct a URL like `/my-account;%2f%2e%2e%2frobots.txt`.
    - **Origin:** Sees `/my-account` (stops at `;`) and generates the user's private page (which includes their **CSRF Token**).
    - **Cache:** Normalizes the path (`/../robots.txt`), matches the rule for `/robots.txt`, and caches the private response.
- **The Attack:** We steal the administrator's CSRF token from the cache, then use it to force them to change their email address.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this lab.

### **Step 1: Identify the Target Data**

1. **Log in:** Access the lab and log in with `wiener` / `peter`.
2. **Inspect:** Go to the "Change Email" section. View the page source or inspect the form.
3. **Observation:** The form requires a **CSRF token** (a hidden field) to submit changes. We need to steal the administrator's token to change their email.

### **Step 2: Find the Delimiter**

We need a character that makes the server stop reading the path but keeps serving the page.

1. **Capture Traffic:** In **Burp Proxy > HTTP History**, find `GET /my-account`. Send to **Repeater**.
2. **Fuzzing:**
    - Try `/my-account;abc`. Result: **200 OK**.
    - Try `/my-account?abc`. Result: **200 OK**.
    - *Result:* The server uses both `;` and `?` as delimiters.

### **Step 3: Find the Cache Rule**

We need to find a file that the cache *always* saves, regardless of where it is found.

1. **Test Static Directories:** Check if `/resources/` is cached. (In this lab, it might not be).
2. **Test Exact Matches:** Try standard files like `/robots.txt`, `/favicon.ico`, or `/sitemap.xml`.
    - Send `GET /robots.txt` to Repeater.
    - Send it twice.
    - If `X-Cache: hit`, then `/robots.txt` is a cacheable target.
3. **Check Normalization:**
    - Send `/aaa/..%2frobots.txt`.
    - If `X-Cache: hit`, the cache normalizes paths!

### **Step 4: Construct the Deception URL**

We need a URL that:

1. Starts with `/my-account` (for the server).
2. Uses a delimiter (`;`).
3. Traverses to the cacheable file (`/../robots.txt`).

**Payload:** `/my-account;%2f%2e%2e%2frobots.txt`

1. **Verify:**
    - Send this payload in Repeater.
    - **Origin:** Sees `/my-account`, generates the private page.
    - **Cache:** Sees `/robots.txt` (after normalization), saves the page.
    - **Check:** Look for `X-Cache: hit` on the second attempt.

### **Step 5: Steal the Administrator's CSRF Token**

1. Go to the **Exploit Server**.
2. **Body (Script 1):** Force the admin to cache their page.HTML
    
    `<script>document.location="https://YOUR-LAB-ID.web-security-academy.net/my-account;%2f%2e%2e%2frobots.txt?wcd"</script>`
    
3. Click **Deliver exploit to victim**.
4. **Retrieve the Token:**
    - Immediately go to Burp Repeater.
    - Send a request to the *exact URL* you sent the victim:
    `GET /my-account;%2f%2e%2e%2frobots.txt?wcd`
    - **Search the Response:** Look for `name="csrf" value="..."`.
    - **Copy the Token:** This is the administrator's token!

### **Step 6: Execute the CSRF Attack**

Now we use the stolen token to change the admin's email.

1. In **Burp Proxy**, find your previous `POST /my-account/change-email` request.
2. Send it to **Repeater**.
3. **Modify the Request:**
    - **CSRF Token:** Paste the administrator's token you just stole.
    - **Email:** Change to an attacker email (e.g., `hacker@evil.com`).
4. **Generate CSRF HTML:**
    - Right-click the request -> **Engagement tools** -> **Generate CSRF PoC**.
    - (Alternatively, you can write the HTML form manually).
    - *Tip:* Ensure the form auto-submits.
5. **Final Exploit:**
    - Copy the HTML.
    - Go back to the **Exploit Server**.
    - Paste the HTML into the **Body**.
    - Click **Deliver exploit to victim**.

### **Step 7: Solve the Lab**

- The victim (administrator) will visit your exploit page.
- The script submits the form using their valid session and the valid CSRF token you stole.
- Their email changes, and the lab is solved.

  ****