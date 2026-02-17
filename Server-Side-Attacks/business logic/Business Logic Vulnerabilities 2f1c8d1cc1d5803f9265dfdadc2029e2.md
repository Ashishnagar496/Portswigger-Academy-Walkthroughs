# Business Logic Vulnerabilities

# LAB 1 : Excessive trust in client-side controls

### **The Core Concept**

This lab demonstrates a classic **Business Logic Flaw**.
In a secure application, when you add an item to your cart, the browser sends the **Product ID** (e.g., `id=1`), and the server looks up the price in its own database.

- **Secure:** Client says "I want Item #1". Server says "Okay, Item #1 costs $1000."
- **Insecure (This Lab):** Client says "I want Item #1, and it costs $1000". Server says "Okay, I trust you."

Since the user controls the client (the browser), we can simply "lie" to the server about the price.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this lab.

### **Step 1: Log In**

1. Open the lab.
2. Click **"My account"**.
3. Log in using the credentials: `wiener` / `peter`.
    - Notice you have **$100.00** store credit.
4. Go back to the **Home** page.

### **Step 2: Intercept the "Add to Cart" Request**

1. Find the **"Lightweight l33t leather jacket"**. It costs **$1337.00** (too expensive for you).
2. In Burp Suite, go to the **Proxy** tab and turn **Intercept ON**.
3. In the browser, click the **"Add to cart"** button for the jacket.
4. The request will freeze in Burp. Look at the body of the request (at the bottom). You will see parameters like:HTTP
    
    `productId=1&quantity=1&price=133700`
    

### **Step 3: Exploit the Trust**

1. Change the `price` value from `133700` (which represents pennies, i.e., $1337.00) to **`1`** (which is $0.01) or **`5000`** ($50.00).
    - Just make sure it is lower than your $100 credit.
2. Click **Forward** to send the modified request to the server.
3. Turn **Intercept OFF**.

### **Step 4: Checkout**

1. Go to the **Cart** icon in the browser (top right).
2. You should see the Jacket in your cart, but the price is now **$0.01**.
3. Click **"Place order"**.
4. The lab is solved because you successfully purchased the item.

# LAB 2 : **High-level logic vulnerability**

### **The Core Concept**

This lab demonstrates a flaw in integer validation. The server calculates your Total Price using the formula: `Price * Quantity`.

- Normal Logic: `1 Jacket * $1337 = $1337`.
- Flawed Logic: The server allows **Negative Numbers** for the quantity.
- Exploit Logic: If you buy **100** cheap items, the total cost becomes negative (e.g., `$5000`).

If you combine a positive expensive item (the Jacket) with a huge negative quantity of cheap items, the **Grand Total** of your cart can become affordable (or even zero).

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this lab.

### **Step 1: Log In**

1. Open the lab.
2. Click **"My account"**.
3. Log in using: `wiener` / `peter`.
    - You have $100 credit.

### **Step 2: Add the Target Item**

1. Go to the Home page.
2. Add **1** "Lightweight l33t leather jacket" to your cart.
3. Your cart total is now **$1337.00**.

### **Step 3: The Negative Quantity Trick**

We need to lower the cart price. We will use a cheap item to do this.

1. Go back to the Home page.
2. Click on a cheap item (e.g., "Buttons" or "Umbrella").
3. In Burp Suite, turn **Intercept ON**.
4. Click **"Add to cart"**.
5. In the intercepted request, look for the `quantity` parameter.
    - Change `quantity=1` to a negative number, like `quantity=-100`.
    - Since we need to subtract about $1237, if the item costs $10, we need roughly -130 items.
6. Click **Forward** until the request is sent.
7. Turn **Intercept OFF**.

### **Step 4: Check the Cart**

1. Go to your **Cart**.
2. You should see:
    - 1x Leather Jacket ($1337)
    - 100x Cheap Item (Negative Cost)
3. Check the **Total**.
    - If the total is between **$0 and $100**, proceed to Step 5.
    - If the total is **still too high**, repeat Step 3 with more negative items.
    - If the total is **negative** (e.g., -$50), the application might block the checkout (you can't buy something for less than zero). You need to add a few positive cheap items to balance it back into the positive range ($0 - $100).

### **Step 5: Checkout**

1. Once your total is affordable (e.g., $20.00), click **"Place order"**.
2. The lab is solved.

# LAB 3 : Inconsistent security controls

### **The Core Concept**

This lab demonstrates a flaw where security rules are applied strictly in one place but loosely in another.

1. **The Front Door (Registration):** To join, you must prove you own your email address. The system sends a verification link. This stops you from signing up directly as `@dontwannacry.com`.
2. **The Back Door (Profile Update):** Once you are logged in, the application lets you change your email address.
3. **The Flaw:** The "Update Email" feature fails to verify if you own the new domain. It blindly trusts the change. Since the system grants Admin rights to anyone with an `@dontwannacry.com` email, simply changing your text field promotes you to Admin.

---

### **Step-by-Step Walkthrough**

You can solve this entirely in your **browser** (Burp Suite is optional but helpful to see the `/admin` path).

### **Step 1: Create a Normal Account**

First, we need to get into the system legitimately.

1. Open the lab.
2. Click **"Register"**.
3. **Get your email:**
    - Right-click the **"Email client"** button and open it in a new tab.
    - Copy your email address (e.g., `attacker@YOUR-LAB-ID.web-security-academy.net`).
4. **Register:**
    - Paste that email into the registration form.
    - Choose a password (e.g., `123456`).
    - Click **Register**.
5. **Verify:**
    - Go back to the **Email Client** tab.
    - Open the email from the website and click the **verification link**.

### **Step 2: Exploit the "Update Email" Flaw**

Now that you are a verified user, we will swap our identity.

1. Log in with your new account.
2. Go to the **"My account"** page.
3. Look for the **Email** field .
4. Change your email to: `admin@dontwannacry.com`.
5. Click **"Update email"**.
    - The application accepts the change immediately without asking you to verify the new address.

### **Step 3: Access Admin Panel**

Because your profile now says `@dontwannacry.com`, the application treats you as an employee.

1. In the browser URL bar, browse to the admin panel:
`/admin`*(Example: `https://YOUR-LAB-ID.web-security-academy.net/admin`)*
2. You should now see the Admin Interface.
3. Find the user **carlos** and click **Delete**.

# LAB 4 : **Flawed enforcement of business rules**

### **The Core Concept**

This lab suffers from a logic bug in how it remembers "used" coupons.
Normally, a shop should keep a permanent list: `[Used: CouponA, CouponB]`. If you try Coupon A again, it checks the list and says "No."

In this flawed system, the application likely only checks the **last** action or fails to maintain a permanent history across different coupon types.

- You apply **Coupon A**. The system says "Okay."
- You apply **Coupon B**. The system says "Okay."
- You apply **Coupon A** again. The system essentially "forgets" you used it because the last thing it processed was Coupon B.

By **alternating** between two valid codes, you can apply them infinitely until the price is $0.

---

### **Step-by-Step Walkthrough**

You can solve this entirely in your **browser**.

### **Step 1: Gather Your Coupons**

1. Log in to the lab with `wiener` / `peter`.
2. **Code 1:** Look at the homepage banner. You will see the code `NEWCUST5`.
3. **Code 2:** Scroll to the very bottom of the page. Enter any email (e.g., `a@a.com`) into the **"Sign up to our newsletter"** box and click "Sign up".
    - You will receive the code `SIGNUP30`.

### **Step 2: Setup the Cart**

1. Go to the Home page.
2. Add the **"Lightweight l33t leather jacket"** to your cart.
3. Go to your **Cart**. The price is $1337.00.

### **Step 3: The "Alternating" Attack**

1. Enter `NEWCUST5` and click **Apply**. (Price drops).
2. Enter `SIGNUP30` and click **Apply**. (Price drops more).
3. **Now, try `NEWCUST5` again.**
    - It works again! The price drops further.
4. **Try `SIGNUP30` again.**
    - It works again!
5. **Repeat this loop** (Coupon A -> Coupon B -> Coupon A -> Coupon B) about 10-15 times.
    - Watch the Total Price drop until it is below your $100 store credit (or even $0.00).

### **Step 4: Checkout**

1. Once the price is affordable, click **"Place order"**.
2. The lab is solved.

# LAB 5 : Low-level logic flaw ( - )

### **The Core Concept**

This lab involves an **Integer Overflow**.
Computers store numbers in fixed-size containers (usually 32-bit). Imagine a car odometer that only goes up to `999,999`. If you drive one more mile, it rolls over to `000,000`.

In programming with "Signed Integers":

- The maximum value is **2,147,483,647**.
- If you go **one cent** higher, the number doesn't just go to zero; it flips to the **minimum negative value** (-2,147,483,648).

**The Strategy:**
The Leather Jacket costs ~$1337. If we put about 32,000 jackets in the cart, the total price (in cents) exceeds the maximum limit. The total price will flip to a huge negative number. We then add a few more items to bring that negative number up until it is just barely positive (between $0 and $100), allowing us to buy 32,000 jackets for cheap.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite Intruder** for this.

### **Step 1: Prepare the Request**

1. Log in with `wiener` / `peter`.
2. Add **1** "Lightweight l33t leather jacket" to your cart.
3. In **Burp Proxy**, find the `POST /cart` request.
4. Send it to **Intruder**.

### **Step 2: Configure Intruder (The "Spam" Attack)**

We need to add thousands of items. The UI limits us to 2 digits (max 99 per request), so we will use automation to send the "Add 99 items" request hundreds of times.

1. **Positions Tab:**
    - Change the `quantity` parameter value to `99`.
    - Clear all payload markers (`§`).
    - We don't actually need to change any data, we just want to repeat the request. (So no markers are strictly needed
2. **Payloads Tab:**
    - **Payload type:** Select **Null payloads**.
    - **Payload settings:** Select continue indefinitely .
    - why indefinitely ? to reach maximum integer .

### **Step 3: Execute and Adjust**

1. **Start Attack.** Wait for it to finish (it might take a minute).
2. **Check your Cart:** Refresh the cart page in the browser.
    - You should have ~31,978 jackets.
    - The total price should be a negative number! (e.g., something like `$100,000,000`).
    - *Note:* If it is not negative yet, or if it is "too" negative, we need to fine-tune.
    - we need exactly **47** more jackets to hit the "sweet spot".
3. **The Fine Tuning:**
    - Go back to **Repeater**.
    - Set `quantity=47`.
    - Send the request **once**.
4. **Verify:**
    - Refresh the cart.
    - The total price should now be approximately **$1221.96**.

### **Step 4: Fix the Balance**

You cannot check out with a negative total. You need to owe the shop a small amount of money (between $0 and $100).
Currently, the shop owes *you* $1221.96. You need to add items worth roughly that amount to balance the books.

1. Browse the shop for other items.
2. The **"Lightweight l33t leather jacket"** costs $1337.00.
    - If you add **1** more jacket: `$1221.96 + $1337.00 = $115.04`.
    - This is slightly too high (you only have $100 credit).
3. Instead, look for a cheaper item (e.g., an item costing roughly $50-$90).
4. Use **Repeater** (or the browser) to add that item to your cart until your **Total** is positive but less than $100 (e.g., $20.00).

### **Step 5: Checkout**

1. Once the Total is valid (e.g., $15.00), click **"Place order"**.
2. The lab is solved.

# **Lab 6 : Inconsistent handling of exceptional input**

### **The Core Concept**

This lab exploits a difference in how the **Application** (Code) handles data vs. how the **Database** handles data.

1. **The Application:** Checks if your email is "valid". It sees `...long-string...@dontwannacry.com.attacker.net` and says: *"Yes, this is a valid email belonging to `attacker.net`."* It sends the confirmation email there.
2. **The Database:** Has a fixed limit for the email column (e.g., **255 characters**). If you give it 260 characters, it **chops off (truncates)** the extra characters to make it fit.
3. **The Exploit:** We craft an email that is so long that the database chops it off *exactly* after `@dontwannacry.com`.
    - **Input:** `[Lots of A's]@dontwannacry.com.YOUR-LAB-ID.web-security-academy.net`
    - **Truncated (Saved):** `[Lots of A's]@dontwannacry.com`
    - **Result:** The database thinks you are a `dontwannacry.com` employee (Admin), but the confirmation email was still sent to your real inbox because the application used the full address to send it.

---

### **Step-by-Step Walkthrough**

You can use **Burp Suite** Repeater to make counting characters easier, but you can technically do this in the browser or a text editor.

### **Step 1: Get Your Email Domain**

1. Open the lab.
2. Open the **"Email client"** in a new tab.
3. Copy your email domain (everything after the `@`).
    - *Example:* `exploit-0a1b...exploit-server.net` (Let's call this `YOUR-DOMAIN`).

### **Step 2: Determine the Truncation Point**

We need to figure out exactly how many characters the "local part" (the part before the `@`) needs to be.

The target format we want the database to save is:
`[PADDING]@dontwannacry.com`

This entire string needs to be exactly **255 characters**.

- The suffix `@dontwannacry.com` is **17 characters**.
- So, the PADDING must be: `255 - 17 = 238` characters.

### **Step 3: Construct the Payload**

We need an email address that looks like this:
`[238 characters]@dontwannacry.com.[YOUR-DOMAIN]`

1. **Generate the Padding:**
    - You can use a text editor or Python (`print("a"*238)`).
    - Let's assume we use the letter `a` repeated 238 times.
2. **Assemble the Full Email:**
    - `aaaaaaaa...(238 times)...aaaa@dontwannacry.com.YOUR-DOMAIN`
    - *Note:* Replace `YOUR-DOMAIN` with your actual email client domain from Step 1.

### **Step 4: Register the Account**

1. Go to the Lab's **"Register"** page.
2. Paste your massive email address into the Email field.
3. Enter a password (e.g., `123`).
4. Click **Register**.
    - *If you calculated correctly:* The application will send an email to the full address (which ends in your domain), so you will receive it.
    - *Simultaneously:* The database will save only the first 255 characters (ending in `.com`).

### **Step 5: Verify and Login**

1. Go to your **Email Client**.
2. Open the email. (Even though the "To" address looks weird, it arrived in your inbox).
3. Click the confirmation link.
4. **Log in** with the massive email (or try the truncated one if that fails, but usually, you login with what you registered).
5. Go to **"My account"**.
6. Look at the **Email** field displayed on the screen.
    - It should show: `...aaaa@dontwannacry.com`.
    - It should **NOT** show your exploit domain at the end.
    - This confirms the exploit worked!

### **Step 6: Access Admin Panel**

1. Navigate to `/admin`.
2. You should now have access.
3. Delete user **carlos**.

# LAB 7 :**Weak isolation on dual-use endpoint**

### **The Core Concept**

This lab exploits a design pattern called a **Dual-Use Endpoint**.
Developers often write one function to handle two similar tasks to save time.

- **Task A (User):** "I want to change my own password." -> **Requirement:** Must provide Current Password.
- **Task B (Admin):** "I need to reset a user's password." -> **Requirement:** No Current Password needed (Admins don't know users' passwords).

**The Flaw:** The code decides which mode to use based on the **input** (e.g., "Did they send a current password?") instead of the **user's privilege** (e.g., "Is this user actually an Admin?").
If we simply *delete* the `current-password` parameter from our request, the server switches to "Admin Mode" and lets us reset anyone's password without checking if we are actually administrators.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this lab.

### **Step 1: Capture a Valid Request**

1. Log in with your credentials: `wiener` / `peter`.
2. Go to the **"My account"** page.
3. Fill out the "Change Password" form normally:
    - Current: `peter`
    - New: `123`
    - Confirm: `123`
4. In **Burp Proxy**, find the `POST /my-account/change-password` request.
5. Right-click and select **Send to Repeater**.

### **Step 2: Test the Exploit Logic**

We want to see if removing the current password bypasses the check.

1. In Repeater, look at the body parameters:
`username=wiener&current-password=peter&new-password-1=123&new-password-2=123`
2. **Delete** the `current-password=peter&` part entirely.
    - *New Body:* `username=wiener&new-password-1=123&new-password-2=123`
3. Click **Send**.
4. **Result:** You should receive a **200 OK** (or 302 Redirect). This confirms that the server accepted the password change *without* verifying the old password. You have successfully tricked the server into "Admin Mode".

### **Step 3: Attack the Administrator**

Now that we know we can reset passwords without authentication, we target the admin.

1. In the same Repeater tab:
    - Change `username=wiener` to `username=administrator`.
    - Keep the new passwords as `123`.
2. Click **Send**.

### **Step 4: Login as Admin**

1. Go back to the browser and click **"Log out"**.
2. Log in using:
    - Username: `administrator`
    - Password: `123` (The one you just set).
3. Go to the **Admin Panel**.
4. Delete the user **carlos**.

# LAB 8 : insufficient workflow validation

### **The Core Concept**

This lab demonstrates a flaw in the "Order of Operations" (Workflow).
A secure purchase flow typically looks like this:

1. **Add to Cart**
2. **Checkout** (Server validates money & deducts cash)
3. **Confirmation Page** (Server says "Success!")

**The Flaw:** In this lab, the "Confirmation Page" isn't just a static receipt; accessing it **triggers the order completion logic.** If the developer forgot to verify that the checkout step actually happened before loading the confirmation page, you can simply skip the payment and jump straight to the finish line.

It's like walking into a restaurant, skipping the cashier, and sitting directly at a table reserved for "Paid Customers," causing the waiter to assume you've already paid and serve you.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this lab.

### **Step 1: Understand the Normal Flow**

First, we need to buy something legitimately to see what the "Success" URL looks like.

1. Log in with `wiener` / `peter`.
2. Buy a cheap item (e.g., "Umbrella") using your valid store credit.
3. Click **"Place order"**.
4. You will see the message: *"Thank you for your order!"*.
5. Check **Burp Proxy > HTTP History**.
    - Find the `POST /cart/checkout` request (This is the payment step).
    - Find the request *immediately after* it: `GET /cart/order-confirmation?order-confirmed=true`.
    - **Crucial:** This `GET` request is what we want to abuse.

### **Step 2: Prepare the Trap**

We want to skip the `POST` (Payment) and go straight to the `GET` (Confirmation).

1. In Burp history, right-click that `GET /cart/order-confirmation...` request.
2. Select **Send to Repeater**.
3. Go back to your browser and **Add the "Lightweight l33t leather jacket"** to your cart.
    - *Do NOT click "Place order".* Just leave it sitting in the cart.
    - *Note:* The "cart" is stored in your session on the server. So even though you are in Burp Repeater, the server knows your session currently has a jacket in it.

### **Step 3: Execute the Skip**

1. Go to **Burp Repeater**.
2. Click **Send** on the confirmation request you saved in Step 1.
    - *What just happened?* You effectively told the server: "I am at the confirmation screen for the items currently in my cart."
    - The server's flawed logic assumes: "If they are asking for the confirmation page, they must have just successfully passed the checkout."
3. **Result:** The server processes the order for the jacket without checking your balance or deducting the money.

### **Step 4: Verify**

1. Go back to the Lab browser.
2. Click **"My account"**.
3. You should see the Order History updated with the Jacket.
4. The lab is solved.

---

# LAB 9 : **Authentication bypass via flawed state machine**

**The Core Concept**
This lab demonstrates a flaw in the "State Machine" of the login process.
A secure login usually flows strictly: `Step 1 (Creds)` --> `Step 2 (MFA/Role)` --> `Step 3 (Dashboard)`.

**The Flaw:** The application likely trusts the user session *immediately* after Step 1, but relies on the browser to force the user through Step 2.
If we simply **refuse** to load Step 2 (the Role Selector page) and manually force our browser to go straight to the Dashboard, the server might get confused. In this specific lab, the fallback "default" state happens to be the Administrator role.

**Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this lab.

**Step 1: Observe the Normal Behavior**

1. Log in with `wiener` / `peter`.
2. Notice that immediately after entering the password, you are taken to a page that asks you to **"Select your role"** .
3. Since you are a normal user, you can only select "User".
4. Once you select it, you are taken to the Home page.

**Step 2: The Intercept & Drop**
We want to stop the process *before* the application forces us to select the "User" role.
1. **Log out** of the application.
2. In **Burp Suite**, go to the **Proxy** tab and turn **Intercept ON**.
3. In the browser, log in again with `wiener` / `peter`.
4. **Forward** the initial `POST /login` request (this sends your password).
5. **Watch closely:** The next request will be `GET /role-selector`.

    ◦ **DO NOT FORWARD THIS.**
    ◦ Click the **Drop** button in Burp. (This tells your browser *not* to load that page).
    ◦ *Note:* You might need to drop it a couple of times if the browser retries.

**Step 3: Force Browse to Home**
Your browser is now sitting on a blank white page (because we dropped the request and showing that request is dropped by the user ).
1. Turn **Intercept OFF** in Burp.
2. In your browser's URL bar, manually navigate to the home page:( or remove the role-selector from the URL )
`https://YOUR-LAB-ID.web-security-academy.net/`
3. **Result:** You should load the page, but since you skipped the "Downgrade to User" step, the application's default state has left you as an **Administrator**.

**Step 4: Solve the Lab**
1. You should see the **Admin Panel** link in the header.
2. Click it.
3. Delete the user **carlos**.

# LAB 10 : **Infinite money logic flaw**

### **The Core Concept**

This lab works like an "Infinite Money Glitch" in a video game. It relies on a flaw where the cost of buying money is lower than the value of the money itself.

1. **The Math:**
    - A Gift Card costs **$10.00**.
    - You have a coupon (`SIGNUP30`) for **30% off**.
    - New Cost: **$7.00**.
    - Redeem Value: **$10.00**.
2. **The Loop:**
    - Buy a card for $7.
    - Redeem it for $10.
    - **Profit: $3 per cycle.**
3. **The Challenge:** You need $1337. Doing this manually ~450 times is boring. We need to automate it using **Burp Macros**.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** for this.

### **Step 1: Setup and Manual Test**

First, we do the loop once manually to generate the traffic log.

1. **Get the Coupon:**
    - Log in (`wiener` / `peter`).
    - Scroll to the bottom, sign up for the newsletter, and get the code: `SIGNUP30`.
2. **Buy a Gift Card:**
    - Add a **$10 Gift Card** to your cart.
    - Go to Cart, enter `SIGNUP30`, and Apply. (Total is now $7.00).
    - Place Order.
3. **Get the Code:**
    - On the confirmation page, find the code (e.g., `W8S2-....`). Copy it.
4. **Redeem:**
    - Go to **"My account"**.
    - Paste the code and Click **Redeem**.
    - Check your balance: It should have gone up by $3.

### **Step 2: Create the Macro (The Automation)**

We need to teach Burp to do those exact steps automatically.

1. **Open Session Settings:**
    - In Burp, go to **Settings** (or Project Options in older versions).
    - Go to **Sessions**.
    - In the **Macros** section (bottom panel), click **Add**.
2. **Record the Macro:**
    - A generic "Macro Recorder" window opens. It shows your HTTP history.
    - Hold `Ctrl` (or Cmd) and select these 5 requests **in order**:
        1. `POST /cart` (Add to cart)
        2. `POST /cart/coupon` (Apply coupon)
        3. `POST /cart/checkout` (Checkout)
        4. `GET /cart/order-confirmation...` (The page with the code)
        5. `POST /gift-card` (Redeem the code)
    - Click **OK**.
3. **Extract the Gift Card Code:**
    - In the "Macro Editor" window, select the **4th request** (`GET /cart/order-confirmation...`).
    - Click **Configure item**.
    - In the "Custom parameter locations" section, click **Add**.
    - **Name:** `gift-card`
    - **Extraction:** Highlight the gift card code in the response text at the bottom. Burp will auto-generate regex.
    - Click **OK**, then **OK** again.
4. **Inject the Code:**
    - Select the **5th request** (`POST /gift-card`).
    - Click **Configure item**.
    - In the "Parameter handling" section (top right):
        - **Parameter:** `gift-card`
        - **Derive from:** "Prior response" (Response 4).
    - Click **OK**.
5. **Test It:**
    - Click the **Test macro** button.
    - Look at the final response (Request 5). If it says "302 Found" (Redirect) or shows a successful redemption, it works!
    - Click **OK** to save the macro.

### **Step 3: Create the Session Rule**

Now we tell Burp *when* to run this macro.

1. In the **Sessions** settings (top panel "Session handling rules"), click **Add**.
2. **Rule Actions:**
    - Click **Add** -> **Run a macro**.
    - Select the macro you just created.
    - Click **OK**.
3. **Scope Tab:**
    - Select **Include all URLs** (or specifically target the lab).
    - Click **OK**.

### **Step 4: Execute the Attack**

Now, every time we send *any* request, Burp will run that background loop to make us money. We just need to trigger it ~450 times.

1. Go to **Proxy > HTTP History**.
2. Right-click any request (e.g., `GET /my-account`) and send to **Intruder**.
3. **Intruder Config:**
    - **Positions:** Clear all markers.
    - **Payloads:** Select **Null Payloads**.
    - **Generate:** `412` (or 500 to be safe).
4. **Resource Pool (Important):**
    - Go to the **Resource Pool** tab.
    - Select **Create new resource pool**.
    - Set **Maximum concurrent requests** to **1**. (If we go too fast, the cart gets confused).
5. **Start Attack**.
    - Wait for it to finish. You are essentially mining money.

### **Step 5: Cash Out**

1. Refresh your account page in the browser.
2. You should have over **$1337.00**.
3. Add the **"Lightweight l33t leather jacket"** to your cart.
4. Checkout.

# LAB 11 :  **Authentication bypass via encryption oracle  ( - )**

### **The Core Concept**

In this lab :
The application uses the same encryption key for two different things:

1. **The "Stay Logged In" Cookie:** Keeps you logged in (e.g., `username:timestamp`).
2. **The "Invalid Email" Notification:** Encrypts error messages to show them to the user (e.g., `Invalid email address: user-input`).

Because the system allows you to input text that gets encrypted (the "Invalid Email" field), you can use it as an **Encryption Oracle** to create your own fake "Stay Logged In" cookie.

However, there is a catch: the oracle automatically adds the text `"Invalid email address: "` to the beginning of your input. We have to use **Block Alignment** to cut that prefix off.

---

### **Step-by-Step Walkthrough**

You **must** use **Burp Suite** (Repeater and Decoder) for this lab.

### **Step 1: Analyze the "Oracle"**

First, we need to separate the encryption function from the decryption function.

1. **Log in** with `wiener` / `peter` and check **"Stay logged in"**.
2. Post a comment with an **Invalid Email** (e.g., hello hello).
3.       →you will see an error above the image on the comment page like this :    

![image.png](image.png)

1. Look at the HTTP History in Burp:
    - **The Encryption Request:** `POST /post/comment`. The server sees the bad email and sends back a `Set-Cookie: notification=...`.
    
    ```
    POST /post/comment HTTP/2
    Host: portswigger.web-security-academy.net
    Cookie: stay-logged-in=WXM50mKyE0XlDPvCo1%2b%2bvBHUEs2O%2faxc1K6KeK7ZSQM%3d; session=b1pCqmidsy2dyXHPvrq7zS53EBWamvCP
    Content-Length: 113
    Cache-Control: max-age=0
    Sec-Ch-Ua: "Not(A:Brand";v="8", "Chromium";v="144", "Microsoft Edge";v="144"
    Sec-Ch-Ua-Mobile: ?0
    Sec-Ch-Ua-Platform: "Windows"
    Origin: https://0a73008b03c870fa83a7b42500770041.web-security-academy.net
    Content-Type: application/x-www-form-urlencoded
    Upgrade-Insecure-Requests: 1
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36 Edg/144.0.0.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
    Sec-Fetch-Site: same-origin
    Sec-Fetch-Mode: navigate
    Sec-Fetch-User: ?1
    Sec-Fetch-Dest: document
    Referer: https://0a73008b03c870fa83a7b42500770041.web-security-academy.net/post?postId=9
    Accept-Encoding: gzip, deflate, br
    Accept-Language: en-US,en;q=0.9
    Priority: u=0, i
    
    csrf=gpJZywI5kwYfGxdkODi4gMgeRZK9REYJ&postId=9&comment=hello+bro+again+&name=opbande&email=hello+hlello+&website=
    ```
    
    - **The Decryption Request:** `GET /post...`. The browser sends the `notification` cookie, and the server decrypts it and prints the text "Invalid email address: test@ test" on the page.
    - 
    
    ```
    GET /post?postId=9 HTTP/2
    Host: portswigger.web-security-academy.net
    Cookie: notification=1Ro%2bZ0f7rgX6pu8eOpGogsnzl4sb5Wmx2oiT2sNtt0L%2fli827oh8X2E9DdsxUpCj; stay-logged-in=WXM50mKyE0XlDPvCo1%2b%2bvBHUEs2O%2faxc1K6KeK7ZSQM%3d; session=b1pCqmidsy2dyXHPvrq7zS53EBWamvCP
    Cache-Control: max-age=0
    Upgrade-Insecure-Requests: 1
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36 Edg/144.0.0.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
    Sec-Fetch-Site: same-origin
    Sec-Fetch-Mode: navigate
    Sec-Fetch-User: ?1
    Sec-Fetch-Dest: document
    Sec-Ch-Ua: "Not(A:Brand";v="8", "Chromium";v="144", "Microsoft Edge";v="144"
    Sec-Ch-Ua-Mobile: ?0
    Sec-Ch-Ua-Platform: "Windows"
    Referer: https://0a73008b03c870fa83a7b42500770041.web-security-academy.net/post?postId=9
    Accept-Encoding: gzip, deflate, br
    Accept-Language: en-US,en;q=0.9
    Priority: u=0, i
    
    ```
    
2. Send the **POST** request to Repeater (Rename tab: "Encrypt").
3. Send the **GET** request to Repeater (Rename tab: "Decrypt").
4. you can rename a tab just by right clicking on the tab then select rename tab .

### **Step 2: Decrypt your valid cookie**

We need to know the correct format of the login cookie.

1. Copy your `stay-logged-in` cookie from the request headers.
2. Go to your **Decrypt (GET)** tab.
3. Paste the `stay-logged-in` value into the `Cookie: notification=...` header.
4. Click **Send**.
5. Look at the error message in the response. It should contain the decrypted text of your cookie.
    - Format found: `wiener:1598530205184` (Username:Timestamp).
    - 
    
    ![image.png](image%201.png)
    
6. **Copy this timestamp** to your clipboard.

### **Step 3: Construct the Payload (The Math Part)**

We want to forge a cookie for `administrator`.
Target Payload: `administrator:YOUR_TIMESTAMP`

![image.png](image%202.png)

then send the encrypt /post request and the response will look like this (302) : 

![image.png](image%203.png)

now copy the notification cookie in this (302) response and paste this in the notification cookie of the get request and then send the get request and the response of this will contains : 

![image.png](image%204.png)

from this we can conclude that :

The Encryption Oracle forces a prefix: `Invalid email address:`  (23 characters including spaces ).
We need to remove this prefix. Since the encryption is likely block-based (16 bytes per block), we can't cut 23 bytes. We must cut a multiple of 16 (e.g., 32 bytes).

To make the "garbage" prefix exactly 32 bytes long, we need padding.

- Current Prefix: 23 bytes
- Target Length: 32 bytes
- **Padding Needed:** `32 - 23 = 9 bytes`.

**Input to send:** `xxxxxxxxxadministrator:YOUR_TIMESTAMP`*(That is 9 'x's followed by the target).*

### **Step 4: Encrypt and Align**

1. Go to your **Encrypt (POST)** tab.
2. Set the `email` parameter to your padded payload:
`email=xxxxxxxxxadministrator:YOUR_TIMESTAMP`
3. Click **Send**.
4. Copy the new `notification` cookie from the response headers.

### **Step 5: Perform the Encoding and Decoding In (Burp Decoder)**

1. Go to **Burp Decoder**.
2. Paste the encrypted cookie you just copied.
3. **Decode as URL**.
4. **Decode as Base64**.
5. **The Cut:**
    - Look at the Hex editor view.
    - Highlight the **first 32 bytes** (the first 2 rows of 16).
    - Right-click the selection -> **Delete selected bytes**.
    - *Why?* This removes the "Invalid email address: xxxxxxxxx" part, leaving `administrator:YOUR_TIMESTAMP` as the start of the data.
6. **Re-encode:**
    - **Encode as Base64**.
    - **Encode as URL**.
7. Copy this final result.
8. then go to the burp repeater 
9. paste the encoded value in place of notification value in the get request .
10. then send the request .
11. the response will now generate this as an error and the invalid email address text is removed : 
12. 

### **Step 6: Attack**

1. Go to Burp Repeater (or your browser).
2. Send a request to the home page (`GET /`).
3. Replace the `stay-logged-in` cookie with your **Forged Cookie**.
    - *Note:* You might need to remove the `session` cookie to force the server to use the "stay-logged-in" cookie.
4. **Check the Response:** You should see "Welcome, administrator".
5. Navigate to `/admin` and delete **carlos**.

# LAB 12 : **Bypassing access controls using email address parsing discrepancies**

### **The Core Concept**

This lab demonstrates a vulnerability known as **Email Atom Splitting**.
Different software libraries read email addresses differently.

- **The Validator:** The code that checks "Is this allowed?" might look at the *very end* of the string (`@ginandjuice.shop`) and say "Yes, this is an employee."
- **The Mail Server:** The code that actually sends the email might decode special characters (like UTF-7 encoded words) and realize the *real* "At" symbol (`@`) is in the middle of the string.

It's like writing a letter to: `Attacker (ignore the rest) @ CompanyHQ`.
The guard checks "CompanyHQ" and lets it pass. The postman sees the "ignore the rest" note and delivers it to "Attacker".

---

### **Step-by-Step Walkthrough**

You can solve most of this in the **browser**, but **Burp Suite** is recommended if you need to debug the encoding.

### **Step 1: Identify the Restriction**

1. Open the lab.
2. Go to the **"Register"** page.
3. Try to register with your normal exploit server email (e.g., `test@exploit-0a...web-security-academy.net`).
4. **Error:** The application rejects you, saying you must belong to the `ginandjuice.shop` domain.

### **Step 2: Test for Encoding Support**

We want to see if the server supports "Encoded Words" (a way to hide characters using `?utf-?` syntax).unicode transformation format .

1. Try registering with this email:
`=?utf-7?q?&AGEAYgBj-?=foo@ginandjuice.shop`
    - *Explanation:* `&AGEAYgBj-` is UTF-7 for `abc`.
    - So the server sees: `abcfoo@ginandjuice.shop`.
2. **Result:** If the page accepts this (or gives a generic error instead of "Security Blocked"), it means the server **supports UTF-7 decoding**. also  official solution guide confirms this specific lab allows UTF-7 while blocking other formats like UTF-8.

### **Step 3: Craft the Exploit Payload**

We need to craft an email that:

1. **Ends with:** `@ginandjuice.shop` (to pass the Validator).
2. **Actually sends to:** `attacker@YOUR-EXPLOIT-DOMAIN` (by hiding the real `@` symbol inside UTF-7).

**The UTF-7 Formula:**`=?utf-7?q?attacker&AEA-[YOUR-EXPLOIT-DOMAIN]&ACA-?=@ginandjuice.shop`

- `attacker`: The username part of your real email.
- `&AEA-`: This is the UTF-7 encoded version of the `@` symbol.
- `[YOUR-EXPLOIT-DOMAIN]`: Your personal lab domain (e.g., `exploit-0a...net`).
- `&ACA-`: This is the UTF-7 encoded version of a `space` (which helps terminate the address cleanly for some parsers).

### **Step 4: Execute the Attack**

1. Open the **"Email client"** in a new tab and copy your exploit domain (remove the `test@` part).
2. Construct your final payload.
    - Example: `=?utf-7?q?attacker&AEA-exploit-0a1b2c3d.exploit-server.net&ACA-?=@ginandjuice.shop`
3. Go to the **Register** page.
4. Paste the payload into the **Email** field.
5. Enter a password and click **Register**.

### **Step 5: Access the Account**

1. Go to your **Email client**.
2. You should receive the confirmation email!
    - *Why?* The mail server decoded the UTF-7, saw the first `@` (the hidden one), and delivered the mail to your exploit server, ignoring the `@ginandjuice.shop` suffix.
3. Click the link to activate your account.

### **Step 6: Become Admin**

1. Log in with your new account.
2. Since the application thinks you are from `ginandjuice.shop`, you are granted Admin privileges.
3. Click **"Admin panel"**.
4. Delete **carlos .**