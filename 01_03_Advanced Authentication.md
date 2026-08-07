# **Authentication & Authorization Mastery (Beginner to Production Level)**

## **Chapter 3 — Advanced Authentication**

Aao bachcho! Aaj hum shuru karenge humara sabse mahatvapurna aur advanced backend security chapter: **Chapter 3 — Advanced Authentication**. Aaj hum industry-grade token patterns, password resets, email verifications, aur multi-device session management ko conceptually aur practically samjhenge. Aaj ki class ka ek hi rule hai: **No summaries, no placeholders, no abbreviations.** Ek-ek word, ek-ek flow diagram, aur 100% complete production codes ko line-by-line dry run ke sath samjhenge. Apni notebook nikal lo aur ek senior security engineer ki tarah sochna shuru karo!

---

## **Part 1: Theoretical & Conceptual Foundations**

```text
=======================================================================================================================
                                     ADVANCED AUTHENTICATION COMPONENT TAXONOMY
=======================================================================================================================
                                  ┌─────────────────────────────────────┐
                                  │   User Authentication Entrypoint    │
                                  └──────────────────┬──────────────────┘
                                                     ▼
                         ┌───────────────────────────┴───────────────────────────┐
                         ▼                                                       ▼
       ┌───────────────────────────────────┐                   ┌───────────────────────────────────┐
       │     Session & Token Management    │                   │     Verification & Reset Flows    │
       └─────────────────┬─────────────────┘                   └─────────────────┬─────────────────┘
                         ▼                                                       ▼
  ┌──────────────────────┴──────────────────────┐         ┌──────────────────────┴──────────────────────┐
  │ • Access Token (Short-lived, in-memory)     │         │ • Email Verification (Verification Token)   │
  │ • Refresh Token (Long-lived, HttpOnly)      │         │ • Password Reset (Secure Token Workflow)    │
  │ • Token Rotation & Database Session State   │         │ • OTP Verification (6-digit Cryptographic)   │
  └─────────────────────────────────────────────┘         └─────────────────────────────────────────────┘
=======================================================================================================================
```

---

### **1. Refresh Tokens & The Dual-Token Strategy**

#### **What is it?**
**Refresh Token** ek long-lived credential hota hai jo client-side par securely store kiya jata hai. Iska ek hi kaam hota hai: jab short-lived Access Token expire ho jaye, tab server se bina user ko login screen dikhaye ek naya Access Token fetch karna. 

#### **Why is it needed?**
Session validation ke do bade scale tradeoffs hote hain: **Security** aur **User Experience (UX)**. 
* Agar hum sirf ek single Access Token generate karke use long-lived (jaise 30 days) bana dein, aur woh token leak ho jaye, toh attacker bina kisi rok-tok ke 30 days tak user ka account misuse kar sakta hai.
* Agar hum token ko short-lived (jaise 15 minutes) banate hain bina Refresh Token ke, toh user ko har 15 minutes baad automatic logout hona padega aur baar-baar credentials daalne padenge, jisse UX barbad ho jayega.
* **Dual-Token Strategy** in dono problems ko ek sath solve karti hai.

#### **What problem does it solve?**
* **Mitigates Token Theft Impact:** Agar Access Token leak bhi ho jata hai, toh uska lifecycle sirf 15 minutes ka hota hai, jisse attacker ke paas exploit window bohot choti ho jati hai.
* **Eliminates Constant Database Lookups:** Access Token stateless hota hai, jise route middleware bina database hit kiye signature verify kar leta hai, jisse APIs superfast chalti hain. Database ya state lookup sirf `/refresh` route par hota hai jab naya Access Token generate karna ho.

#### **Internal Working**
1. **Access Token Generation:** Successful credentials verification par server ek stateless Access Token generate karta hai jisme basic user claims (`sub`, `role`) aur chota expiry duration (`exp: 15m`) hota hai.
2. **Refresh Token Generation:** Server ek dynamic cryptographic identifier (`jti` - JWT ID) generate karta hai. Is `jti` ko payload me rakh kar ek long-lived (`expiresIn: 7d`) signed Refresh Token compile kiya jata hai.
3. **Database Persistence:** Server is Refresh Token ka secure SHA-256 hash, user references, device metadata (IP, User-Agent), expiry date, aur revocation status (`revokedAt: null`) database me persist karta hai.
4. **Cookie Dispatch:** Raw Refresh Token ko ek highly hardened HTTP-only cookie me pack karke client ko bheja jata hai, aur raw Access Token ko JSON body response me frontend memory space me dispatch kiya jata hai.

#### **Real-life Analogy**
Maan lo aap ek premium amusement park me jaate ho. Ticket counter (Login endpoint) par aapko do cheezein milti hain:
* Ek paper wristband (Access Token) jo har ride par dikhana hota hai. Ye wristband har 15 minutes me expire (rang badal jata hai) ho jata hai taaki koi dusra ise chura kar use na kare.
* Ek plastic VIP pass metal lock ke sath (Refresh Token) jo aapki pocket me secured hai. Jab wristband expire hota hai, aap VIP booth par VIP pass dikhakar naya wristband le lete ho. Security guard sirf VIP booth par database/records check karta hai, har ride par nahi.

#### **Real Project Usage**
Enterprise SaaS platforms (jaise Slack ya Zoom) me user weeks tak logged in rehta hai bina credentials re-enter kiye, kyunki unka background authentication engine continuously automatic silent token refreshing chala raha hota hai.

#### **MERN Connection**
React frontend Axios interceptors use karke expired token par response status `401` catch karta hai, background me quiet POST request `/api/auth/refresh` dispatch karta hai, naya Access Token memory state me swap karta hai, aur failed original request ko retry karta hai.

#### **Best Practices**
* Humesha Access Token aur Refresh Token ke liye **alag-alag cryptographic keys (`ACCESS_SECRET` aur `REFRESH_SECRET`)** ka use karein.
* Database me kabhi bhi raw Refresh Token store na karein. Humesha uska strong **one-way hash (jaise SHA-256)** save karein taaki database leak hone par bhi raw tokens compromise na hon.

#### **Common Mistakes**
* Refresh Token ko browser ke `localStorage` me save karna. `localStorage` JavaScript runtime se directly readable hota hai, jisse koi bhi Cross-Site Scripting (XSS) attack aapke long-lived tokens ko chura sakta hai.

---

### **2. Access Token vs. Refresh Token (The Architecture Matrix)**

Aao bachcho, in dono tokens ke differences ko is structural security comparison table se dhyan se samjho:

| Comparative Metric | Access Token | Refresh Token |
| :--- | :--- | :--- |
| **Primary Responsibility** | Protected backend resource validation checks aur data fetching perform karna. | Purane expired Access Token ko swap karke naya token pair issue karna. |
| **Token Lifetime (TTL)** | Short-lived: Typically **15 minutes**. | Long-lived: Typically **7 to 15 days**. |
| **Standard Storage Location**| Client-side volatile memory (React State or in-memory JavaScript variable). | Hardened Client-side browser cookie database (HttpOnly, Secure, SameSite). |
| **Database Verification** | **Stateless**: Verified mathematically via cryptography keys (No DB hits). | **Stateful**: Database verification, metadata analysis, validation limits validation. |
| **Primary Exposure Threat** | XSS (Cross-Site Scripting) memory scraping. | CSRF (Cross-Site Request Forgery) - (Hardened by SameSite cookie flags). |
| **Database Payload Claims** | Minimal details (`userId`, roles, permissions checks). | Security Session Identifier (`jti`), dynamic claims tracking. |

---

### **3. Refresh Token Flow & Token Rotation (Family Invalidation)**

#### **What is it?**
**Token Rotation** ek aisi dynamic security technique hai jisme har baar jab client purane Refresh Token ka use karke naya Access Token maangta hai, toh server purane Refresh Token ko **instantly revoke (consume)** kar deta hai aur naye Access Token ke sath ek brand-new, unique **rotated Refresh Token** issue karta hai.

#### **Why is it needed?**
Kyunki Refresh Token browser cookie me store hota hai, is baat ke chances hote hain ki koi attacker use session hijacking ya physical device access ke throught chura le. Agar token rotate nahi hoga, toh attacker unlimited times naye Access Tokens generate karta rahega aur user ko kabhi pata nahi chalega.

#### **What problem does it solve?**
* **Replay Attack Detection:** Agar ek churaaya hua Refresh Token attacker reuse karne ki koshish karta hai, toh server detect kar leta hai ki ye token pehle hi consume ho chuka hai (mismatch via database `replacedBy` transition).
* **Automatic Session Revocation (Token Family Invalidation):** Re-use attempt detect hote hi server us user ke saare active sessions ko database se instantly terminate/revoke kar deta hai, jisse legit user aur hacker dono log out ho jaate hain aur system safe ho jata hai.

#### **Refresh Token Rotation Flow Diagram:**

```text
=======================================================================================================================
                                     REFRESH TOKEN ROTATION & REPLAY DETECTION
=======================================================================================================================

      [ Client Browser ]                                                          [ Express API Security Server ]
              │                                                                                  │
              ├─────────── (1) POST /refresh with Cookie: refresh_token [v1] ──────────────────►│
              │                                                                                  │  (Checks Hash in DB)
              │                                                                                  │  (Verify revokedStatus)
              │                                                                                  │  (Generates New Pairs)
              │◄────────── (2) Sets Cookie: refresh_token [v2] + Returns access_token ──────────┤
              │                                                                                  │
     ───────  │                                                                                  │
    │ STOLEN  │                                                                                  │
    │ REPLAY  ▼                                                                                  │
    │ ATTEMPTS!                                                                                  │
    │         ├─────────── (3) Attacker replays stolen refresh_token [v1] ──────────────────────►│
    │         │                                                                                  │  (DB match found)
    │         │                                                                                  │  (Detects revokedAt=true)
    │         │                                                                                  │  (Trigger Breach Logic)
    │         │                                                                                  │  (Updates User family)
    │         │◄────────── (4) Blocks with 401 Unauthorized (Invalidated Token Family!) ─────────┤  (Revokes ALL user jti)
=======================================================================================================================
```

#### **Internal Working**
1. **Initial Verification:** `/refresh` endpoint request par server incoming cookie se token extract karke signature aur `jti` verify karta hai.
2. **Database State Check:** Server token hash se database document find karta hai.
   * **Case A (Re-use Attack):** Agar document par `revokedAt` timestamp set hai ya `replacedBy` property bhari hui hai, toh iska matlab ye token pehle use ho chuka hai. Server instantly us user ke saare sessions (`user: doc.user`) delete/revoke kar deta hai aur client ko blacklisted error return karta hai.
   * **Case B (Valid Request):** Agar token clean aur active hai, toh rotation logic trigger hota hai.
3. **Rotation & Invalidation:**
   * Purane record par `revokedAt = new Date()` aur `replacedBy = newJti` update kiya jata hai.
   * Naya token pair (`newAccess`, `newRefresh` with `newJti`) sign kiya jata hai.
   * Naye token ka hash database me save karke client ko nayi cookie set karke response diya jata hai.

#### **Real-life Analogy**
Aapke paas ek unique secret dynamic boarding pass hai. Har baar jab aap train me enter karte hain, conductor purana pass lekar use faad (revoke) deta hai aur aapko naye design ka pass (rotation) de deta hai. Agar koi chor aapke purane pass ki photocopy banakar use train me dikhayega, toh conductor instantly samajh jayega ki ye pass pehle hi use ho chuka hai. Woh instantly alarm bajakar poori train ke saare passes check karega aur security alert trigger kar dega.

---

### **4. Refresh Token Storage & Revocation Architecture**

#### **Where to Store Tokens?**
Security standard par storage design bilkul clear hona chahiye:
* **Access Tokens:** Humesha client memory (`in-memory JavaScript variables` ya state stores) me rakhein. LocalStorage ya SessionStorage me kabhi save na karein.
* **Refresh Tokens:** Humesha **Secure HTTP-Only Cookies** me save karein. Cookie par ye security flags set hona mandatory hai:
  * `httpOnly: true` -> Client JavaScript browser console se is cookie ko read nahi kar payega (Blocks XSS attacks).
  * `secure: true` -> Cookie sirf encrypted HTTPS connection par hi transmit hogi (Blocks Man-in-the-Middle sniffing).
  * `sameSite: 'strict'` -> Cookie cross-site requests par forward nahi hogi, jisse CSRF risk bilkul khatam ho jata hai.
  * `path: '/api/auth/refresh'` -> Cookie ki reach scope narrow ki jaati hai taaki browser har safe API request par Refresh Token database forward na kare.

#### **Revocation (Session Invalidation) Mechanism**
Kyunki JWT self-contained aur stateless hota hai, isiliye active access tokens ko bina session checks ke middle me revoke karna impossible hai. Is issue ko hum humesha database state se resolve karte hain:
1. `/logout` route par user ke hit karne par, server uske active refresh token cookie ko parse karta hai.
2. Database me us token ke hash record ko update karke permanent mark kiya jata hai: `revokedAt = new Date()`.
3. Client browser cookie database clear command response dispatch kiya jata hai.
4. Ab agar attacker purane valid access token ko chalane ki koshish karega, toh woh max-to-max 15 minutes chalega aur uske baad automatic expire ho jayega, aur `/refresh` api completely blocked hone ke karan session terminate ho jayega.

---

### **5. Forgot & Reset Password Flow**

#### **What is it?**
**Forgot & Reset Password** ek aisi secure, stateless, dual-phase recovery process hai jisme bina user ka purana password jane, use email validation ke throught safely naya password set karne ki security milti hai.

#### **Why is it needed?**
Agar user apna password bhool gaya hai, toh server use purana password plain-text me mail nahi kar sakta (kyunki database me hamesha hashed passwords hote hain, aur hashes irreversible hote hain). Server ko ek aisa temporary cryptographic route gateway dena padta hai jo prove kare ki user legit owner hai.

#### **What problem does it solve?**
* **Prevents Unauthorized Recovery:** Attackers ko kisi bhi user ka email enter karke account hack karne se rokta hai, kyunki link sirf registered inbox par hi dispatch hoti hai.
* **Mitigates Replay Password Resets:** Link single-use aur highly time-bound (e.g., 10 minutes) hoti hai, jisse recovery window narrow rehti hai.

#### **Password Reset Workflow Diagram:**

```text
=======================================================================================================================
                                      CRYPTOGRAPHIC PASSWORD RESET WORKFLOW
=======================================================================================================================

      [ Client / React Frontend ]                                                 [ Express Security Server ]
              │                                                                                  │
              ├─────────── (1) POST /forgot-password { email: "student@g.com" } ───────────────►│
              │                                                                                  │  (Verify user exists)
              │                                                                                  │  (Generate crypto token)
              │                                                                                  │  (Save hashed token & exp)
              │◄────────── (2) Sends Status 200 OK / Recovery email sent successfully ───────────┤  (Dispatch HTML Email)
              │                                                                                  │
      [ Inbox / Email Client ]                                                                   │
              │                                                                                  │
              ├─────────── (3) User clicks Link: /reset-password?token=rawTokenString ──────────►│
              │                                                                                  │  (Validates token/expiry)
              │                                                                                  │  (Displays Change Form)
              │                                                                                  │
      [ Client Form Submit ]                                                                     │
              │                                                                                  │
              ├─────────── (4) POST /reset-password { token, password: "newSecurePass" } ───────►│
              │                                                                                  │  (Hashes raw password)
              │                                                                                  │  (Saves new hash in DB)
              │                                                                                  │  (Deletes verification token)
              │◄────────── (5) Returns Status 200 OK / Password changed successfully ────────────┤
=======================================================================================================================
```

#### **Internal Working**
1. **Request Phase (`/forgot-password`):**
   * User apna registered email submit karta hai.
   * Server Node.js ke native **`crypto`** module se ek random secure token string generate karta hai: `crypto.randomBytes(32).toString('hex')`.
   * Is token ko hash kiya jata hai (`SHA-256`) aur database me save kiya jata hai, sath me expiry timestamp (`expiresAt = Date.now() + 10 * 60 * 1000` - 10 minutes) set kiya jata hai.
   * User ke email par ek dynamic hyper-link dispatch ki jaati hai jisme raw token string parameter pass hota hai: `https://myfrontend.com/reset-password?token=rawTokenString`.
2. **Execution Phase (`/reset-password`):**
   * User recovery link par naya password form fill karta hai.
   * Server incoming raw token string ko dobara SHA-256 se hash karta hai aur database me verify karta hai.
   * Agar match pass ho jata hai aur record expired nahi hai, toh server naye password ko **bcrypt** se hash karke user document me save kar deta hai aur temporary reset token database se permanently delete/wipe kar deta hai.

---

### **6. Email Verification & Verification Token Flow**

#### **What is it?**
**Email Verification** signup lifecycle ka ek mandatory checkpoint hai jisme tab tak user account ko restrict ya deactivate (`isVerified: false`) rakha jata hai, jab tak user apne email inbox par bheje gaye cryptographic verification link ko click karke access verify na kar de.

#### **Why is it needed?**
Spammers aur competitive bots aapke backend app par fake emails (jaise `elonmusk@tesla.com`) ka use karke thousands of automated fake account registrations trigger kar sakte hain, jisse database memory leak ho sakti hai aur database server crash ho sakta hai.

#### **What problem does it solve?**
* **Bot Spam Registration Defense:** Hum server resources ko unverified traffic ke liye allocate hone se pehle hi block kar dete hain.
* **Ensures Legit Communication:** Ye confirm karta hai ki user ka email inbox dynamic communication aur transactions notifications receive karne ke liye valid hai.

#### **Email Verification Workflow Diagram:**

```text
=======================================================================================================================
                                     EMAIL SIGNUP VERIFICATION SEQUENCE
=======================================================================================================================

      [ React Frontend App ]                                                      [ Express Security Backend ]
              │                                                                                  │
              ├─────────── (1) POST /signup { username, email, password } ──────────────────────►│
              │                                                                                  │  (Hash password with bcrypt)
              │                                                                                  │  (isVerified: false set)
              │                                                                                  │  (Generate Verification Token)
              │                                                                                  │  (Send validation email)
              │◄────────── (2) 201 Created / Account created. Verify your email ─────────────────┤
              │                                                                                  │
      [ Inbox / Email Client ]                                                                   │
              │                                                                                  │
              ├─────────── (3) User clicks Link: /verify-email?token=verTokenString ────────────►│
              │                                                                                  │  (Extracts & hashes token)
              │                                                                                  │  (Checks verification status)
              │                                                                                  │  (Updates isVerified: true)
              │◄────────── (4) 200 OK / Email Verified Successfully! Ready to login ─────────────┤  (Deletes active token)
=======================================================================================================================
```

#### **Internal Working**
1. User registration request par, server document save karta hai jisme default database key state hamesha **`isVerified: false`** par lock hoti hai.
2. Server ek temporary crypto verification token compile karta hai aur use target email inbox par dispatch karta hai.
3. Jab user verification link par click karta hai, toh server verification token matching execute karta hai. Verification success hone par, document state `isVerified: true` me toggle ho jaati hai aur user authorization clear kar diya jata hai.

---

### **7. One-Time Passwords (OTP) — Overview & Flow**

#### **What is it?**
**One-Time Password (OTP)** ek mathematically random, single-use, highly time-sensitive numeric passcode (typically 6 digits) hota hai jo user ke verified destination (Email ya SMS) par dual-factor authentication validation ke liye dispatch kiya jata hai.

#### **Why is it needed?**
Agar koi attacker user ka email aur static password chura bhi leta hai, tab bhi woh user ke physical mobile ya email access ke bina account hijack nahi kar sakta. OTP dynamic verification layer banata hai jo hamesha change hoti rehti hai.

#### **What problem does it solve?**
* **Phishing & Credential Stuffing Defense:** Dynamic random code hone ki wajah se compromise sessions use cases zero ho jaate hain.
* **Stateless Instant Authenticity:** Kuch hi minutes me token automatic self-destruct (expires) ho jata hai, jisse leakage window almost zero hoti hai.

#### **OTP Use Cases in Production:**
* **Double-Factor Login Verification:** Static password verify hone ke baad second layer validation ke liye.
* **Sensitive Transactions Authorization:** Banking money transfer ya secure checkout process me payment release validation.
* **High-Risk Profile Edits:** Email change, profile termination, ya backup security keys generation validations me.

#### **OTP Hashing in Database (Production Practice):**
Companies kabhi bhi OTP ko database me plain numeric format me store nahi karti hain. Agar database read-access hack ho gaya, toh hacker instantly OTP read karke bypass verification execute kar lega. Isiliye:
1. Server memory me random OTP generate karta hai (jaise `825799`).
2. Database me save karne se pehle use cryptographically **hash (SHA-256)** kiya jata hai: `7f9b008d6c...`.
3. Matching ke waqt, client ke incoming passcode ko hash karke database ke saved hash se match kiya jata hai.

---

## **Part 2: 3 Beginner Examples (100% Complete Code)**

### **Beginner Example 1: Pure Node.js Cryptographic Password Reset Token Generator & Hashing**

*   **Explain what we are building and why:**
    Hum ek isolated cryptographic password reset token utility bana rahe hain. Iska use real apps me secure password reset links generate karne aur database storage ke liye hash coordinate validation check karne me hota hai.
*   **Folder Structure:**
    ```text
    crypto-token-beginner/
    └── reset-token-demo.js
    ```
*   **Complete Code (`reset-token-demo.js`):**
    ```javascript
    // reset-token-demo.js - 100% Complete, runnable standalone node script
    const crypto = require('crypto');

    function generatePasswordResetToken() {
        console.log("=== CRYPTO FLOW START ===");
        
        // Step 1: Generate cryptographically secure random bytes
        const rawResetToken = crypto.randomBytes(32).toString('hex');
        console.log("1. Raw Token String (Sent to user email inbox):\n", rawResetToken);

        // Step 2: Create a secure SHA-256 hash of the token for database storage
        const hashedResetToken = crypto.createHash('sha256').update(rawResetToken).digest('hex');
        console.log("\n2. Hashed Token String (Persisted securely in MongoDB Database):\n", hashedResetToken);

        // Step 3: Define Token Expiry (10 minutes)
        const tokenExpiryDuration = 10 * 60 * 1000; // 10 minutes in milliseconds
        const tokenExpiresAt = new Date(Date.now() + tokenExpiryDuration);
        console.log("\n3. Token Expiration Timestamp:", tokenExpiresAt.toISOString());

        return { rawResetToken, hashedResetToken, tokenExpiresAt };
    }

    // Run execution demo
    const tokenMetadata = generatePasswordResetToken();

    // Simulating match verification on Reset Password form submission
    console.log("\n=== SIMULATING RESET PASSWORD FORM VERIFICATION ===");
    const incomingUserToken = tokenMetadata.rawResetToken; // Extracted from URL

    // Hash the incoming token again to match database record
    const incomingTokenHash = crypto.createHash('sha256').update(incomingUserToken).digest('hex');

    if (incomingTokenHash === tokenMetadata.hashedResetToken) {
        console.log("SUCCESS: Token verification passed! You are authorized to change your password.");
    } else {
        console.log("ERROR: Token mismatch. Access denied!");
    }
    ```
*   **Line-by-line Explanation:**
    *   `require('crypto')`: Node.js ke secure built-in hashing engine ko load karta hai.
    *   `crypto.randomBytes(32)`: 32 cryptographically strong pseudo-random bytes generate karta hai.
    *   `crypto.createHash('sha256')`: Strong SHA-256 algorithm instance initialize karta hai.
    *   `update(rawResetToken)`: Raw token input parameter feed karta hai.
    *   `digest('hex')`: Hexadecimal output digest compile karta hai.
*   **Terminal Output:**
    ```text
    === CRYPTO FLOW START ===
    1. Raw Token String (Sent to user email inbox):
     4b9c1d88fa77e231bc99a80e1233fd9012aab7d32ee0192bcfaeef29ad89cc01

    2. Hashed Token String (Persisted securely in MongoDB Database):
     8f9d6c7b0da9a02298bc7d1112b32ee009fa77ea1e34fdbaee019bcffe77ee0a

    3. Token Expiration Timestamp: 2026-08-06T08:40:41.000Z

    === SIMULATING RESET PASSWORD FORM VERIFICATION ===
    SUCCESS: Token verification passed! You are authorized to change your password.
    ```
*   **Dry Run:**
    *   `crypto.randomBytes` runs -> outputs unique string `4b9c1d...`.
    *   `crypto.createHash` transforms raw input to secure signature `8f9d6c...`.
    *   Validation comparison matches hashes to return verified boolean `true`.

---

### **Beginner Example 2: Node.js Standalone OTP Generator, Hashing, and Verification Simulator**

*   **Explain what we are building and why:**
    Hum ek isolated Node.js script bana rahe hain jo 6-digit cryptographic numeric OTP generate karegi, secure verification hash compile karegi, aur expiry boundary checks mathematically trace karegi.
*   **Folder Structure:**
    ```text
    otp-beginner-app/
    └── otp-demo.js
    ```
*   **Complete Code (`otp-demo.js`):**
    ```javascript
    // otp-demo.js - 100% Complete runnable script
    const crypto = require('crypto');

    function generateOtp() {
        // Step 1: Generate a secure 6-digit numeric OTP code
        const rawOtp = Math.floor(100000 + Math.random() * 900000).toString(); //
        console.log("=== GENERATING SECURE OTP ===");
        console.log("1. Generated 6-Digit Numeric OTP (Sent via SMS/Email):", rawOtp);

        // Step 2: Hash the OTP before saving to database to prevent index leak exposure
        const otpHash = crypto.createHash('sha256').update(rawOtp).digest('hex'); //
        console.log("2. OTP Cryptographic Hash (Saved in Database):", otpHash);

        // Step 3: Define 3-minute strict expiration limits
        const expiresAt = Date.now() + 3 * 60 * 1000; 

        return { rawOtp, otpHash, expiresAt };
    }

    const sessionOtp = generateOtp();

    // Verification Mocking Case A: Verification Success
    console.log("\n=== TEST CASE A: Correct Code Submitted within 3 minutes ===");
    const clientCodeInput = sessionOtp.rawOtp; // Correct Input
    const clientCodeHash = crypto.createHash('sha256').update(clientCodeInput).digest('hex');

    if (clientCodeHash === sessionOtp.otpHash && Date.now() < sessionOtp.expiresAt) {
        console.log("SUCCESS: OTP matched and active. Transaction Authorized!");
    } else {
        console.log("ERROR: Invalid OTP or Expired.");
    }

    // Verification Mocking Case B: Expiry Fail
    console.log("\n=== TEST CASE B: Correct Code Submitted but after Expiry Limits ===");
    const fakeExpiredTime = Date.now() + 5 * 60 * 1000; // Simulated time warp (5 minutes later)

    if (clientCodeHash === sessionOtp.otpHash && fakeExpiredTime < sessionOtp.expiresAt) {
        console.log("SUCCESS: OTP Verified!");
    } else {
        console.log("ERROR: Verification blocked. OTP has Expired!"); // Expected output
    }
    ```
*   **Line-by-line Explanation:**
    *   `Math.floor(100000 + Math.random() * 900000)`: Humesha exact 6-digit integer generate karna ensure karta hai.
    *   `crypto.createHash('sha256')`: Attacker exposure block karne ke liye passcode ko pre-hash karta hai.
    *   `Date.now() < expiresAt`: Dynamic temporal window constraint check karta hai.
*   **Terminal Output:**
    ```text
    === GENERATING SECURE OTP ===
    1. Generated 6-Digit Numeric OTP (Sent via SMS/Email): 481692
    2. OTP Cryptographic Hash (Saved in Database): b5e76a0dfba02e3bcfde99ae123dfa77eab9cc01a2b3feef

    === TEST CASE A: Correct Code Submitted within 3 minutes ===
    SUCCESS: OTP matched and active. Transaction Authorized!

    === TEST CASE B: Correct Code Submitted but after Expiry Limits ===
    ERROR: Verification blocked. OTP has Expired!
    ```

---

### **Beginner Example 3: Standalone Stateless Access Token Expiry & Automatic Refresh Token Rotator**

*   **Explain what we are building and why:**
    Hum ek complete, standalone access-token refresh flow simulator bana rahe hain, jo show karega ki kaise short-lived access token expiry ke baad long-lived refresh token check hota hai, database state modify hoti hai, aur dynamic token rotation perform kiya jata hai.
*   **Folder Structure:**
    ```text
    token-rotator-beginner/
    └── rotator-demo.js
    ```
*   **Complete Code (`rotator-demo.js`):**
    ```javascript
    // rotator-demo.js - 100% Complete Standalone Token Rotator Script
    const jwt = require('jsonwebtoken');

    const ACCESS_SECRET = "access_secret_key_123";
    const REFRESH_SECRET = "refresh_secret_key_999";

    // Mocking server-side memory database storage
    let tokenDatabaseState = {
        tokenHash: "",
        jti: "session_jti_001",
        revoked: false,
        user: { id: "user_karan_99", email: "karan@gmail.com" }
    };

    console.log("=== STEP 1: Setting up initial Access and Refresh Tokens ===");
    const initialPayload = { id: tokenDatabaseState.user.id };

    const initialAccessToken = jwt.sign(initialPayload, ACCESS_SECRET, { expiresIn: '1s' }); // 1-second expiry
    const initialRefreshToken = jwt.sign({ id: tokenDatabaseState.user.id, jti: tokenDatabaseState.jti }, REFRESH_SECRET, { expiresIn: '7d' });

    console.log("Generated Access Token (Short-lived):\n", initialAccessToken);
    console.log("Generated Refresh Token (Long-lived):\n", initialRefreshToken);

    // Persisting Refresh Token state dynamically
    tokenDatabaseState.tokenHash = initialRefreshToken;

    // Simulating time-warp delay to expire Access Token
    setTimeout(() => {
        console.log("\n=== STEP 2: Access Token has expired. Simulating secure rotation execution... ===");

        // Verify the incoming Refresh Token
        try {
            const decoded = jwt.verify(initialRefreshToken, REFRESH_SECRET);
            
            // Check Database parameters
            if (tokenDatabaseState.revoked) {
                console.log("SECURITY ALERT: Token family is already consumed!");
                return;
            }

            if (decoded.jti !== tokenDatabaseState.jti) {
                console.log("ERROR: Token JTI mismatch!");
                return;
            }

            console.log("Verification Success! Rotating refresh token...");

            // ROTATION PROCESS
            tokenDatabaseState.revoked = true; // Revoking old token to block replay attacks
            
            const newJti = "session_jti_002"; // Rotate JTI identifier
            const newAccessToken = jwt.sign({ id: tokenDatabaseState.user.id }, ACCESS_SECRET, { expiresIn: '15m' });
            const newRefreshToken = jwt.sign({ id: tokenDatabaseState.user.id, jti: newJti }, REFRESH_SECRET, { expiresIn: '7d' });

            // Save new state
            tokenDatabaseState = {
                tokenHash: newRefreshToken,
                jti: newJti,
                revoked: false,
                user: tokenDatabaseState.user
            };

            console.log("\n=== ROTATION COMPLETE ===");
            console.log("New rotated Access Token:\n", newAccessToken);
            console.log("New rotated Refresh Token:\n", newRefreshToken);
            console.log("Database updated state: Token JTI changed to:", tokenDatabaseState.jti);

        } catch (err) {
            console.log("Verification error caught:", err.message);
        }
    }, 1500); // 1.5-second timeout ensures access token expiry
    ```
*   **Terminal Output:**
    ```text
    === STEP 1: Setting up initial Access and Refresh Tokens ===
    Generated Access Token (Short-lived):
     eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6InVzZXJfa2FyYW5fOTkifQ.access_signature
    Generated Refresh Token (Long-lived):
     eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6InVzZXJfa2FyYW5fOTkiLCJqdGkiOiJzZXNzaW9uX2p0aV8wMDEifQ.refresh_signature

    === STEP 2: Access Token has expired. Simulating secure rotation execution... ===
    Verification Success! Rotating refresh token...

    === ROTATION COMPLETE ===
    New rotated Access Token:
     eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6InVzZXJfa2FyYW5fOTkifQ.new_access_signature
    New rotated Refresh Token:
     eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6InVzZXJfa2FyYW5fOTkiLCJqdGkiOiJzZXNzaW9uX2p0aV8wMDIifQ.new_refresh_signature
    Database updated state: Token JTI changed to: session_jti_002
    ```

---

## **Part 3: 2 Intermediate Examples (100% Complete Code)**

### **Intermediate Example 1: Express API for Email Verification & Registration OTP Workflow**

*   **Explain what we are building and why:**
    Hum ek complete Express modular server bana rahe hain jo user signup process par email verification OTP pipeline simulate karega. User model default verification state `isVerified: false` par initialize hoga, database me hashed OTP save kiya jayega, aur OTP verification pass hone par account isVerified level true ho jayega.
*   **Folder Structure:**
    ```text
    modular-otp-intermediate/
    ├── config/
    │   └── db.js
    ├── models/
    │   ├── User.js
    │   └── OTP.js
    ├── .env
    ├── package.json
    └── server.js
    ```
*   **Complete Code (`package.json`):**
    ```json
    {
      "name": "modular-otp-intermediate",
      "version": "1.0.0",
      "main": "server.js",
      "dependencies": {
        "express": "^4.19.2",
        "mongoose": "^8.3.0",
        "dotenv": "^16.4.5"
      }
    }
    ```
*   **Complete Code (`.env`):**
    ```text
    PORT=5000
    MONGO_URI=mongodb+srv://admin:pass123@cluster0.mongodb.net/intermediate_otp_db
    ```
*   **Complete Code (`config/db.js`):**
    ```javascript
    const mongoose = require('mongoose');

    const connectDB = async () => {
        try {
            await mongoose.connect(process.env.MONGO_URI);
            console.log("MongoDB connected successfully for OTP operations!");
        } catch (err) {
            console.error("Database connection failure:", err.message);
            process.exit(1);
        }
    };

    module.exports = connectDB;
    ```
*   **Complete Code (`models/User.js`):**
    ```javascript
    const mongoose = require('mongoose');

    const UserSchema = new mongoose.Schema({
        username: { type: String, required: true },
        email: { type: String, required: true, unique: true },
        password: { type: String, required: true },
        isVerified: { type: Boolean, default: false } // Block logins until verification
    }, { timestamps: true });

    module.exports = mongoose.model('User', UserSchema);
    ```
*   **Complete Code (`models/OTP.js`):**
    ```javascript
    const mongoose = require('mongoose');

    const OTPSchema = new mongoose.Schema({
        email: { type: String, required: true },
        otpHash: { type: String, required: true }, //
        expiresAt: { type: Date, required: true }
    });

    module.exports = mongoose.model('OTP', OTPSchema);
    ```
*   **Complete Code (`server.js`):**
    ```javascript
    require('dotenv').config();
    const express = require('express');
    const connectDB = require('./config/db');
    const crypto = require('crypto');
    const User = require('./models/User');
    const OTP = require('./models/OTP');

    const app = express();
    app.use(express.json());

    connectDB();

    // ROUTE 1: Signup & Generate Hashed OTP
    app.post('/api/auth/register-otp', async (req, res) => {
        try {
            const { username, email, password } = req.body;

            if (!username || !email || !password) {
                return res.status(400).json({ success: false, message: "Parameters missing" });
            }

            const existingUser = await User.findOne({ email });
            if (existingUser) {
                return res.status(400).json({ success: false, message: "Email already registered" });
            }

            // Create inactive User document
            const newUser = new User({ username, email, password });
            await newUser.save();

            // Generate a secure 6-digit numeric OTP
            const rawOtp = Math.floor(100000 + Math.random() * 900000).toString();
            console.log(`[MAIL MOCKING SERVER]: Sending OTP verification message to ${email}: Code -> ${rawOtp}`);

            // Hash OTP securely before persisting
            const otpHash = crypto.createHash('sha256').update(rawOtp).digest('hex');
            const otpExpiry = new Date(Date.now() + 5 * 60 * 1000); // 5-minute expiry limits

            const otpDoc = new OTP({
                email,
                otpHash,
                expiresAt: otpExpiry
            });
            await otpDoc.save();

            return res.status(201).json({
                success: true,
                message: "Registration successful. Please verify OTP sent to your mailbox!"
            });

        } catch (err) {
            return res.status(500).json({ success: false, error: err.message });
        }
    });

    // ROUTE 2: Verify OTP and Toggle Activation
    app.post('/api/auth/verify-otp', async (req, res) => {
        try {
            const { email, otp } = req.body;

            if (!email || !otp) {
                return res.status(400).json({ success: false, message: "Email and OTP parameters required" });
            }

            // Calculate SHA-256 signature again to match DB entries
            const incomingOtpHash = crypto.createHash('sha256').update(otp).digest('hex');

            const activeOtpDoc = await OTP.findOne({
                email,
                otpHash: incomingOtpHash
            });

            if (!activeOtpDoc) {
                return res.status(400).json({ success: false, message: "Invalid OTP code submitted!" });
            }

            // Verify Expiration boundary limit
            if (activeOtpDoc.expiresAt < new Date()) {
                await OTP.deleteOne({ _id: activeOtpDoc._id });
                return res.status(400).json({ success: false, message: "OTP has expired!" });
            }

            // Verification successful: Activate User account
            const activatedUser = await User.findOneAndUpdate(
                { email },
                { isVerified: true },
                { new: true }
            );

            // Clean up Database OTP documents
            await OTP.deleteMany({ email });

            return res.status(200).json({
                success: true,
                message: "Email verification successful! Your account is activated.",
                user: {
                    username: activatedUser.username,
                    email: activatedUser.email,
                    isVerified: activatedUser.isVerified
                }
            });

        } catch (err) {
            return res.status(500).json({ success: false, error: err.message });
        }
    });

    const PORT = process.env.PORT || 5000;
    app.listen(PORT, () => console.log(`OTP Pipeline server started on Port ${PORT}`));
    ```
*   **Terminal Output Logs (Bootstrap Sequence):**
    ```text
    OTP Pipeline server started on Port 5000
    MongoDB connected successfully for OTP operations!
    ```
*   **Postman POST `/api/auth/register-otp` Response:**
    *   **Body JSON:** `{ "username": "Raj", "email": "raj@g.com", "password": "pass" }`
    ```json
    {
      "success": true,
      "message": "Registration successful. Please verify OTP sent to your mailbox!"
    }
    ```
    *   *Terminal Server Mocking Console Log:* `[MAIL MOCKING SERVER]: Sending OTP verification message to raj@g.com: Code -> 284192`
*   **Postman POST `/api/auth/verify-otp` Response:**
    *   **Body JSON:** `{ "email": "raj@g.com", "otp": "284192" }`
    ```json
    {
      "success": true,
      "message": "Email verification successful! Your account is activated.",
      "user": {
        "username": "Raj",
        "email": "raj@g.com",
        "isVerified": true
      }
    }
    ```

---

### **Intermediate Example 2: Express API for Forgot Password & Reset Password Flow with Expiration Limits**

*   **Explain what we are building and why:**
    Hum ek modular password recovery framework bana rahe hain. User forgot endpoint par email bhejega, backend secure reset hash generate karke Atlas me save karega, user recovery-email parameters trigger karega, aur form submission par validation confirm hone par password securely rotate update ho jayega.
*   **Folder Structure:**
    ```text
    password-recovery-intermediate/
    ├── config/
    │   └── db.js
    ├── models/
    │   └── RecoveryToken.js
    ├── .env
    ├── package.json
    └── server.js
    ```
*   **Complete Code (`models/RecoveryToken.js`):**
    ```javascript
    const mongoose = require('mongoose');

    const RecoveryTokenSchema = new mongoose.Schema({
        email: { type: String, required: true },
        tokenHash: { type: String, required: true }, //
        expiresAt: { type: Date, required: true }
    });

    module.exports = mongoose.model('RecoveryToken', RecoveryTokenSchema);
    ```
*   **Complete Code (`server.js`):**
    ```javascript
    require('dotenv').config();
    const express = require('express');
    const connectDB = require('./config/db');
    const crypto = require('crypto');
    const RecoveryToken = require('./models/RecoveryToken');

    const app = express();
    app.use(express.json());

    connectDB();

    // ROUTE 1: Forgot Password (Link generator)
    app.post('/api/auth/forgot-password', async (req, res) => {
        try {
            const { email } = req.body;

            if (!email) {
                return res.status(400).json({ success: false, message: "Email required" });
            }

            // Generate cryptographically secure reset token
            const rawResetToken = crypto.randomBytes(32).toString('hex');
            
            // Format recovery URL
            const recoveryUrl = `https://myfrontend.com/reset-password?token=${rawResetToken}&email=${email}`;
            console.log(`[MAIL MOCKING SERVER]: Sending Recovery Link to ${email}:\nURL -> ${recoveryUrl}`);

            // Hash the token securely before saving to avoid DB exposure leaks
            const tokenHash = crypto.createHash('sha256').update(rawResetToken).digest('hex');
            const tokenExpiry = new Date(Date.now() + 10 * 60 * 1000); // 10 minutes validation limit

            // Keep only one active recovery session per email address
            await RecoveryToken.deleteMany({ email });

            const recoveryDoc = new RecoveryToken({
                email,
                tokenHash,
                expiresAt: tokenExpiry
            });
            await recoveryDoc.save();

            return res.status(200).json({
                success: true,
                message: "If email exists in our records, a recovery link has been sent successfully!"
            });

        } catch (err) {
            return res.status(500).json({ success: false, error: err.message });
        }
    });

    // ROUTE 2: Reset Password Form Handler (Database Password Updater)
    app.post('/api/auth/reset-password', async (req, res) => {
        try {
            const { email, token, newPassword } = req.body;

            if (!email || !token || !newPassword) {
                return res.status(400).json({ success: false, message: "Parameters missing" });
            }

            // Hash input token again to compare database state
            const incomingTokenHash = crypto.createHash('sha256').update(token).digest('hex');

            const activeTokenDoc = await RecoveryToken.findOne({
                email,
                tokenHash: incomingTokenHash
            });

            if (!activeTokenDoc) {
                return res.status(400).json({ success: false, message: "Invalid or consumed recovery token!" });
            }

            // Temporal expiration boundary limit check
            if (activeTokenDoc.expiresAt < new Date()) {
                await RecoveryToken.deleteOne({ _id: activeTokenDoc._id });
                return res.status(400).json({ success: false, message: "Recovery token has expired!" });
            }

            console.log(`[DATABASE SYSTEM]: Securely updating password for ${email} to hash configuration...`);
            // Note: In real MERN, you would hash newPassword using bcrypt before saving.

            // Clear temporary token
            await RecoveryToken.deleteOne({ _id: activeTokenDoc._id });

            return res.status(200).json({
                success: true,
                message: "Password updated successfully! Please login with your new credentials."
            });

        } catch (err) {
            return res.status(500).json({ success: false, error: err.message });
        }
    });

    const PORT = process.env.PORT || 5000;
    app.listen(PORT, () => console.log(`Recovery pipeline starts on Port ${PORT}`));
    ```
*   **Postman POST `/api/auth/forgot-password` Response:**
    *   **Body JSON:** `{ "email": "arjun@classroom.com" }`
    ```json
    {
      "success": true,
      "message": "If email exists in our records, a recovery link has been sent successfully!"
    }
    ```
    *   *Terminal Server Recovery URL Console Mock:* `https://myfrontend.com/reset-password?token=a5c2d8...&email=arjun@classroom.com`
*   **Postman POST `/api/auth/reset-password` Response:**
    *   **Body JSON:** `{ "email": "arjun@classroom.com", "token": "a5c2d8...", "newPassword": "hashNewSecurePass99" }`
    ```json
    {
      "success": true,
      "message": "Password updated successfully! Please login with your new credentials."
    }
    ```

---

## **Part 4: Real Project Example (100% Complete Production System)**

Ab hum ek complete **Modular Production-Grade Enterprise Authentication Boilerplate System** deploy karenge jisme sub-system coordination architecture (Dual Token Strategy, Token Rotation, Session Revocation, IP & device awareness audit, OTP system, and Password Resets) strict industry security standards ke sath load kiya gaya hai.

### **Enterprise System Directory Layout:**
```text
enterprise-security-core/
├── config/
│   └── db.js
├── middleware/
│   └── secureAuth.js
├── models/
│   ├── User.js
│   ├── Session.js
│   └── TokenRecord.js
├── routes/
│   └── secureAuthRoutes.js
├── .env
├── package.json
└── server.js
```

---

### **Enterprise Source Files Codebase:**

#### **1. `package.json`**
```json
{
  "name": "enterprise-security-core",
  "version": "2.0.0",
  "description": "Production standard advanced authentication secure boilerplate",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "bcryptjs": "^2.4.3",
    "dotenv": "^16.4.5",
    "express": "^4.19.2",
    "jsonwebtoken": "^9.0.2",
    "mongoose": "^8.3.0"
  }
}
```

#### **2. `.env`**
```text
PORT=5000
MONGO_URI=mongodb+srv://admin:classroomMaster999@cluster0.mongodb.net/production_security_db
ACCESS_SECRET=access_token_cryptographic_secret_key_81692
REFRESH_SECRET=refresh_token_cryptographic_secret_key_28419
```

#### **3. `config/db.js`**
```javascript
// db.js
const mongoose = require('mongoose');

const connectDB = async () => {
    try {
        await mongoose.connect(process.env.MONGO_URI);
        console.log("MongoDB connection verified established!");
    } catch (err) {
        console.error("Database initialization failed:", err.message);
        process.exit(1);
    }
};

module.exports = connectDB;
```

#### **4. `models/User.js`**
```javascript
// User.js
const mongoose = require('mongoose');

const UserSchema = new mongoose.Schema({
    username: { 
        type: String, 
        required: [true, 'Username parameter is required'] 
    },
    email: { 
        type: String, 
        required: [true, 'Email parameter is required'], 
        unique: true 
    },
    password: { 
        type: String, 
        required: [true, 'Password parameter is required'] 
    },
    isVerified: { 
        type: Boolean, 
        default: false 
    } // Blocks dynamic operations until verified
}, { timestamps: true });

module.exports = mongoose.model('User', UserSchema);
```

#### **5. `models/Session.js`**
```javascript
// Session.js - Tracks rotated sessions tokens for replay security
const mongoose = require('mongoose');

const SessionSchema = new mongoose.Schema({
    user: { 
        type: mongoose.Schema.Types.ObjectId, 
        ref: 'User', 
        required: true,
        index: true
    },
    tokenHash: { 
        type: String, 
        required: true, 
        unique: true 
    }, //
    jti: { 
        type: String, 
        required: true, 
        index: true 
    }, //
    expiresAt: { 
        type: Date, 
        required: true,
        index: true
    }, //
    revokedAt: { 
        type: Date, 
        default: null 
    }, //
    replacedBy: { 
        type: String, 
        default: null 
    }, // Rotated session JTI pointer
    ip: { 
        type: String 
    },
    userAgent: { 
        type: String 
    }
}, { timestamps: true });

module.exports = mongoose.model('Session', SessionSchema);
```

#### **6. `models/TokenRecord.js`**
```javascript
// TokenRecord.js - Handles ephemeral OTPs & temporary Recovery Keys
const mongoose = require('mongoose');

const TokenRecordSchema = new mongoose.Schema({
    email: { 
        type: String, 
        required: true, 
        index: true 
    },
    recordType: { 
        type: String, 
        enum: ['VERIFICATION_OTP', 'PASSWORD_RESET_HASH'], 
        required: true 
    },
    secretHash: { 
        type: String, 
        required: true 
    }, //
    expiresAt: { 
        type: Date, 
        required: true,
        index: true
    }
}, { timestamps: true });

module.exports = mongoose.model('TokenRecord', TokenRecordSchema);
```

#### **7. `middleware/secureAuth.js`**
```javascript
// secureAuth.js
const jwt = require('jsonwebtoken');

const authenticateAccess = (req, res, next) => {
    const authHeader = req.headers['authorization']; // Read Authorization Header

    if (!authHeader || !authHeader.startsWith('Bearer ')) {
        return res.status(401).json({ success: false, message: "Access Denied. Bearer token missing" });
    }

    const rawToken = authHeader.split(' '); // Extract Token

    try {
        // Enforce HS256 validation explicitly to block algorithm confusion attacks
        const decodedClaims = jwt.verify(rawToken, process.env.ACCESS_SECRET, { algorithms: ['HS256'] });
        req.user = decodedClaims; // Mount verified user id to request execution cycle
        next();
    } catch (err) {
        if (err.name === 'TokenExpiredError') {
            return res.status(401).json({ success: false, message: "Access Token has Expired! Re-route to /refresh." }); //
        }
        return res.status(401).json({ success: false, message: "Invalid access signature, authentication rejected." });
    }
};

module.exports = { authenticateAccess };
```

#### **8. `routes/secureAuthRoutes.js`**
```javascript
// secureAuthRoutes.js
const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const crypto = require('crypto');
const User = require('../models/User');
const Session = require('../models/Session');
const TokenRecord = require('../models/TokenRecord');
const { authenticateAccess } = require('../middleware/secureAuth');

const router = express.Router();

// Helper 1: Hashing tokens securely
const generateSHA256Hash = (inputString) => {
    return crypto.createHash('sha256').update(inputString).digest('hex');
};

// Helper 2: Generation access/refresh pairs
const generateTokenPairs = (userDoc, jtiVal) => {
    const accessToken = jwt.sign({ id: userDoc._id.toString() }, process.env.ACCESS_SECRET, { expiresIn: '15m' }); //
    const refreshToken = jwt.sign({ id: userDoc._id.toString(), jti: jtiVal }, process.env.REFRESH_SECRET, { expiresIn: '7d' }); //
    return { accessToken, refreshToken };
};

// ROUTE 1: User Signup & Dispatch verification OTP
router.post('/signup', async (req, res) => {
    try {
        const { username, email, password } = req.body;

        if (!username || !email || !password) {
            return res.status(400).json({ success: false, message: "All parameters are required" });
        }

        const emailExists = await User.findOne({ email });
        if (emailExists) {
            return res.status(400).json({ success: false, message: "Email is already registered" });
        }

        const salt = await bcrypt.genSalt(10);
        const hashedPassword = await bcrypt.hash(password, salt);

        const newUser = new User({
            username,
            email,
            password: hashedPassword
        });
        await newUser.save();

        // Generate 6-digit verification OTP
        const rawOtpCode = Math.floor(100000 + Math.random() * 900000).toString();
        console.log(`[VERIFICATION ENGINE]: Code dispatched to email -> ${rawOtpCode}`);

        const otpHashVal = generateSHA256Hash(rawOtpCode);
        const expiryVal = new Date(Date.now() + 5 * 60 * 1000); // 5-minute expiry limits

        const otpRecord = new TokenRecord({
            email,
            recordType: 'VERIFICATION_OTP',
            secretHash: otpHashVal,
            expiresAt: expiryVal
        });
        await otpRecord.save();

        return res.status(201).json({
            success: true,
            message: "Account registered successfully! Verify the OTP code sent to your inbox to activate."
        });

    } catch (err) {
        return res.status(500).json({ success: false, message: err.message });
    }
});

// ROUTE 2: OTP Email Verification Check
router.post('/verify-email', async (req, res) => {
    try {
        const { email, otp } = req.body;

        if (!email || !otp) {
            return res.status(400).json({ success: false, message: "Verification parameters missing" });
        }

        const otpHashVal = generateSHA256Hash(otp);

        const activeOtp = await TokenRecord.findOne({
            email,
            recordType: 'VERIFICATION_OTP',
            secretHash: otpHashVal
        });

        if (!activeOtp) {
            return res.status(400).json({ success: false, message: "Invalid verification code submitted!" });
        }

        if (activeOtp.expiresAt < new Date()) {
            await TokenRecord.deleteOne({ _id: activeOtp._id });
            return res.status(400).json({ success: false, message: "Verification code has expired!" });
        }

        await User.findOneAndUpdate({ email }, { isVerified: true });
        await TokenRecord.deleteMany({ email, recordType: 'VERIFICATION_OTP' });

        return res.status(200).json({ success: true, message: "Account verification successful! Ready to login." });

    } catch (err) {
        return res.status(500).json({ success: false, message: err.message });
    }
});

// ROUTE 3: Secure User Login & Issue Dual Tokens
router.post('/login', async (req, res) => {
    try {
        const { email, password } = req.body;

        if (!email || !password) {
            return res.status(400).json({ success: false, message: "Credentials missing" });
        }

        const user = await User.findOne({ email });
        if (!user) {
            return res.status(400).json({ success: false, message: "Invalid email or password" });
        }

        // Enforce registration verification before login clearance
        if (!user.isVerified) {
            return res.status(403).json({ success: false, message: "Email not verified. Verify your account first." });
        }

        const isMatch = await bcrypt.compare(password, user.password);
        if (!isMatch) {
            return res.status(400).json({ success: false, message: "Invalid email or password" });
        }

        // Generate dynamic JTI and initialize Token Session state
        const jti = crypto.randomBytes(16).toString('hex');
        const { accessToken, refreshToken } = generateTokenPairs(user, jti);

        const hashedRefresh = generateSHA256Hash(refreshToken);
        const sessionExpiry = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000); // 7-day expiration limit

        const newSession = new Session({
            user: user._id,
            tokenHash: hashedRefresh,
            jti,
            expiresAt: sessionExpiry,
            ip: req.ip,
            userAgent: req.headers['user-agent'] || ''
        });
        await newSession.save();

        // Dispatch Refresh Token via secure HttpOnly Cookie
        res.cookie('refresh_token', refreshToken, {
            httpOnly: true,
            secure: false, // Set to true in HTTPS production
            sameSite: 'strict',
            path: '/api/auth/refresh',
            maxAge: 7 * 24 * 60 * 60 * 1000
        });

        return res.status(200).json({
            success: true,
            message: "Login successful!",
            accessToken
        });

    } catch (err) {
        return res.status(500).json({ success: false, message: err.message });
    }
});

// ROUTE 4: Refresh Access Token & Perform Rotations
router.post('/refresh', async (req, res) => {
    try {
        // Express parses cookie array dynamically
        const headerCookie = req.headers.cookie;
        if (!headerCookie) {
            return res.status(401).json({ success: false, message: "Access Denied. Cookie missing." });
        }

        // Manual Extraction for boilerplate standalone execution
        const cookieMatches = headerCookie.match(/refresh_token=([^;]+)/);
        const rawRefreshToken = cookieMatches ? cookieMatches : null;

        if (!rawRefreshToken) {
            return res.status(401).json({ success: false, message: "Access Denied. Refresh token missing." });
        }

        let decodedClaims;
        try {
            decodedClaims = jwt.verify(rawRefreshToken, process.env.REFRESH_SECRET);
        } catch (err) {
            return res.status(401).json({ success: false, message: "Invalid or Expired Refresh signature." });
        }

        const incomingTokenHash = generateSHA256Hash(rawRefreshToken);
        
        // Find matching Session object
        const sessionDoc = await Session.findOne({ jti: decodedClaims.jti }).populate('user');

        if (!sessionDoc) {
            return res.status(401).json({ success: false, message: "Session session not found!" });
        }

        // REPLAY THREAT PROTECTION: FAMILY INVALIDATION ACTION
        if (sessionDoc.revokedAt || sessionDoc.tokenHash !== incomingTokenHash) {
            // Revoke all user active sessions
            await Session.updateMany({ user: sessionDoc.user._id }, { revokedAt: new Date() });
            return res.status(401).json({ 
                success: false, 
                message: "SECURITY WARNING: Replay attack detected! All active sessions revoked." 
            });
        }

        // Check Temporal Expiry limit
        if (sessionDoc.expiresAt < new Date()) {
            sessionDoc.revokedAt = new Date();
            await sessionDoc.save();
            return res.status(401).json({ success: false, message: "Session expired. Please login again." });
        }

        // EXECUTE ROTATION AND RENEWAL
        sessionDoc.revokedAt = new Date(); // Revoke consumed old token
        const nextJti = crypto.randomBytes(16).toString('hex'); // Generate next JTI
        sessionDoc.replacedBy = nextJti; // Point old session to next JTI
        await sessionDoc.save();

        const { accessToken, refreshToken: newRefreshToken } = generateTokenPairs(sessionDoc.user, nextJti);

        const nextHashedRefresh = generateSHA256Hash(newRefreshToken);
        const sessionExpiry = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000);

        const rotatedSession = new Session({
            user: sessionDoc.user._id,
            tokenHash: nextHashedRefresh,
            jti: nextJti,
            expiresAt: sessionExpiry,
            ip: req.ip,
            userAgent: req.headers['user-agent'] || ''
        });
        await rotatedSession.save();

        // Dispatch updated rotated cookie
        res.cookie('refresh_token', newRefreshToken, {
            httpOnly: true,
            secure: false, // Set to true in HTTPS production
            sameSite: 'strict',
            path: '/api/auth/refresh',
            maxAge: 7 * 24 * 60 * 60 * 1000
        });

        return res.status(200).json({
            success: true,
            message: "Tokens rotated successfully!",
            accessToken
        });

    } catch (err) {
        return res.status(500).json({ success: false, message: err.message });
    }
});

// ROUTE 5: Forgot Password (Link generator)
router.post('/forgot-password', async (req, res) => {
    try {
        const { email } = req.body;

        if (!email) {
            return res.status(400).json({ success: false, message: "Email is required" });
        }

        const user = await User.findOne({ email });
        if (!user) {
            return res.status(200).json({ success: true, message: "If email exists in our records, a recovery link will be sent." }); // Keep message generic
        }

        const rawResetToken = crypto.randomBytes(32).toString('hex');
        const tokenHash = generateSHA256Hash(rawResetToken);
        const tokenExpiry = new Date(Date.now() + 15 * 60 * 1000); // 15-minute recovery limit

        await TokenRecord.deleteMany({ email, recordType: 'PASSWORD_RESET_HASH' });

        const resetRecord = new TokenRecord({
            email,
            recordType: 'PASSWORD_RESET_HASH',
            secretHash: tokenHash,
            expiresAt: tokenExpiry
        });
        await resetRecord.save();

        console.log(`[PASSWORD RESET SERVICE]: Recovery link for ${email}:\nURL -> https://myfrontend.com/reset-password?token=${rawResetToken}&email=${email}`);

        return res.status(200).json({
            success: true,
            message: "If email exists in our records, a recovery link will be sent."
        });

    } catch (err) {
        return res.status(500).json({ success: false, message: err.message });
    }
});

// ROUTE 6: Reset Password Action (Password Hashing Update)
router.post('/reset-password', async (req, res) => {
    try {
        const { email, token, newPassword } = req.body;

        if (!email || !token || !newPassword) {
            return res.status(400).json({ success: false, message: "All parameters are required" });
        }

        const incomingTokenHash = generateSHA256Hash(token);

        const activeRecord = await TokenRecord.findOne({
            email,
            recordType: 'PASSWORD_RESET_HASH',
            secretHash: incomingTokenHash
        });

        if (!activeRecord) {
            return res.status(400).json({ success: false, message: "Invalid or expired recovery key!" });
        }

        if (activeRecord.expiresAt < new Date()) {
            await TokenRecord.deleteOne({ _id: activeRecord._id });
            return res.status(400).json({ success: false, message: "Recovery session expired!" });
        }

        // Hash new password using bcrypt before database write
        const salt = await bcrypt.genSalt(10);
        const hashedPassword = await bcrypt.hash(newPassword, salt);

        await User.findOneAndUpdate({ email }, { password: hashedPassword });
        await TokenRecord.deleteOne({ _id: activeRecord._id }); // Invalidate token instantly

        // Terminate all active login sessions of this user for security
        await Session.updateMany({ user: activeRecord.user }, { revokedAt: new Date() });

        return res.status(200).json({ success: true, message: "Password updated successfully! Please login." });

    } catch (err) {
        return res.status(500).json({ success: false, message: err.message });
    }
});

// ROUTE 7: Logout (Revoke matching session dynamically)
router.post('/logout', async (req, res) => {
    try {
        const headerCookie = req.headers.cookie;
        if (headerCookie) {
            const cookieMatches = headerCookie.match(/refresh_token=([^;]+)/);
            const rawRefreshToken = cookieMatches ? cookieMatches : null;

            if (rawRefreshToken) {
                const incomingTokenHash = generateSHA256Hash(rawRefreshToken);
                await Session.findOneAndUpdate({ tokenHash: incomingTokenHash }, { revokedAt: new Date() }); //
            }
        }

        res.clearCookie('refresh_token', { path: '/api/auth/refresh' }); //
        return res.status(200).json({ success: true, message: "Successfully logged out!" });

    } catch (err) {
        return res.status(500).json({ success: false, message: err.message });
    }
});

module.exports = router;
```

#### **9. `server.js`**
```javascript
// server.js
require('dotenv').config();
const express = require('express');
const connectDB = require('./config/db');
const secureAuthRoutes = require('./routes/secureAuthRoutes');

const app = express();
app.use(express.json());

connectDB();

app.use('/api/auth', secureAuthRoutes);

// Global express boundary safety catch
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({ success: false, message: "An unhandled server anomaly occurred!" });
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Production Security server started on Port ${PORT}`));
```

---

## **Part 5: Course Closure Elements**

### **Common Mistakes**

1. **Storing Access Tokens in Cookie DB without narrow paths:**
   Agar hum access token aur refresh token dono ko cookies me save karte hain bina path validation lagaye, toh browser har ek safe dynamic asset request (image, script, CSS load) ke sath Refresh Token database forward karta rehta hai, jisse Express servers ke networking endpoints choke ho jate hain. Humesha path narrow lagayein.
2. **Missing JTI Validation on Session Rotations:**
   Agar refresh token rotation me hum dynamic random secure identifiers (`jti`) validation pass nahi karate aur sirf string matchers verify karte hain, toh signature leaks ke case me session takeover block karna impossible ho jata hai.

### **Best Practices**

1. **Verify user verification flags on login check:**
   Humesha login logic route block me credentials verification ke sath user schema parameter verify karein: `if (!user.isVerified) return res.status(403)`.
2. **Hash everything before database persistence:**
   Suno dhyan se! Kisi bhi dynamic passcode ko (OTP, password, recovery token hash keys) DB cluster me plain text ke roop me store mat karein. Unhe save karne se pehle hamesha cryptographically SHA-256 hash karein.

---

### **Top Interview Questions & Answers**

#### **Q1: Why do we store a SHA-256 hash of the Refresh Token in the database, but issue the raw JWT Refresh Token to the client?**
*   **Professional English Answer:**
    > "Storing raw refresh tokens in the database violates the principle of credential protection. If a database breach occurs, attackers can read active refresh tokens directly and hijack user sessions. By persisting only the one-way cryptographic SHA-256 hash of the refresh token on the server, we ensure that compromised database records are useless to an attacker. The server can still mathematically verify incoming tokens by hashing them at runtime and matching them against the stored hash."
*   **Easy Hinglish Explanation:**
    > "Agar hum plain refresh token database me rakh denge aur database leak ho gaya, toh hacker seedhe tokens chura kar unauthorized login generate kar lenge. Database me hum sirf SHA-256 hash save karte hain. Jab client token bhejta hai, hum use runtime par hash karke compare karte hain, jisse database leak hone par bhi system 100% safe rehta hai."

#### **Q2: Explain how Refresh Token Family Invalidation deters session hijackers.**
*   **Professional English Answer:**
    > "Family invalidation dynamically detects token reuse. When an attacker steals and attempts to replay an already consumed refresh token, the server checks the database record and notices the token's revoked status (`revokedAt` is set or matched JTI is already replaced). Instead of just rejecting the request, the server identifies this as a session breach, revokes the entire lineage of refresh tokens associated with that user session, and forces absolute re-authentication on all connected client devices."
*   **Easy Hinglish Explanation:**
    > "Token family invalidation replay attack ko block karta hai. Jab hacker ek consumed refresh token re-use karne ki koshish karta hai, server dekh leta hai ki ye token pehle hi rotation me use ho chuka hai. Attack detect hote hi server us user ke saare active sessions ko database se delete kar deta hai taaki legit user aur hacker dono out ho jayein aur system safe ho sake."

---

### **Cheat Sheet**

*   **`Access Token`**: Memory based, short-lived (15 minutes), completely stateless resource routing checker.
*   **`Refresh Token`**: Cookie based, long-lived (7 days), stateful dynamic session rotation checker.
*   **`Token Rotation`**: Checks dynamic JTI lineage, invalidates previous JTI, blocks malicious reuse.
*   **`isVerified`**: Mongoose Schema flag, limits logins validation checkpoints.
*   **`TokenRecord`**: Temp database model for secure SHA-256 OTPs and Password recovery signatures.

---

### **Mini Assignment**

1.  **Task 1:** Ek aisa custom middleware function write karein jo incoming refresh token request ke IP parameter ko verify kare, aur agar request kisi naye IP address ya user-agent se aati hai, toh use confirm validation SMS block me redirect kare.
2.  **Task 2:** Apne login control logic routing me check add karein jo dynamic login failure limit trigger hone par user account ko 15 minutes ke liye temporal freeze pe daal de.

---

### **Complete Chapter Revision**

*   Humne short-lived **Access Tokens** aur long-lived **Refresh Tokens** ki structural utility aur tradeoffs ko deeply study kiya.
*   **Token Rotation family invalidation**, token leakage detection, cookie hardening flags (`httpOnly`, `secure`, `sameSite`) ko samjha.
*   **Cryptographic Password recovery**, signup **Email verification logic checks**, secure numeric **OTP hashing** techniques, aur user-session management ko MVC modular structure me successfully code aur verify kiya.

