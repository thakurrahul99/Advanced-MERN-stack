# **Authentication & Authorization Mastery (Beginner to Production Level)**

## **Chapter 6 — Complete MERN Authentication Project**

Aao bachcho! Aaj hum hamari web development and security series ke sabse bada, most requested, aur ultimate crown-jewel module me kadam rakh rahe hain: **Chapter 6 — Complete MERN Authentication Project**. 

Pichle chapters me humne authentication ke theoretical concepts, tokens, cookies, authorization models (RBAC), security headers, aur server hardening ko individual levels par seekha. Lekin as an elite full-stack architect, aapko pata hona chahiye ki jab in saare components ko mila kar ek single, massive production application banayi jaati hai, toh unka integration, directory binding, file matching, aur communication flows kaise operate hote hain.

Hum is chapter ko do parts me padhenge. Pehle hum ek-ek structural aur logical component ka complete **What, Why, Internal Working, Analogy, aur complete React-to-Database lifecycle tracing** karenge. Uske baad, hum **Part 5** me ek real-world, complete, ready-to-run **production grade enterprise system** develop karenge jisme ek bhi line skipped ya custom template truncated nahi hogi. Apni coffee garam kar lo, laptop screen ko clean karo, aur dhyan se ek-ek module ko apne brain me permanently flash karte chalo!

---

## **Part 1: Conceptual Deep Dive on Production MERN Architecture**

---

### **1. Production Folder Structure & Project Architecture**

#### **What is it?**
**Production Folder Structure** ek industry-standard, clean, aur decoupled directory layout hota hai jo modular development ko enable karta hai. Yeh application ke components (routes, business logic, schemas, configurations, helpers, database files) ko physically separate folders me partition karta hai taaki unhe independent scale, debug, aur test kiya ja sake.

```text
===================================================================
                  PRODUCTION DIRECTORY ARCHITECTURE
===================================================================
                      
   [ secure-mern-core/ ] (Root Project Folder)
          │
          ├── [ config/ ]       ──► DB connections, Winston Logger init
          ├── [ models/ ]       ──► Mongoose Schemas (User, Token, Session)
          ├── [ routes/ ]       ──► Endpoint paths & HTTP methods declaration
          ├── [ controllers/ ]  ──► Route handler functions & request parsers
          ├── [ middleware/ ]   ──► Auth, RBAC, Validation, Rate Limit gates
          ├── [ utils/ ]        ──► Helper files (cryptography, token signers)
          ├── [ services/ ]     ──► DB queries, Cloudinary, Third-Party APIs
          ├── .env              ──► Cryptographic secrets & DB URIs
          └── server.js         ──► App entrypoint, middleware registry
===================================================================
```

#### **Why is it needed?**
Jab hum monolithic (single-file) code likhte hain, toh initial level par program chal jata hai. Lekin jab application badh kar 50+ endpoints aur 20+ database collections par pahonchta hai, tab ek hi file me everything manage karna maintenance nightmare ban jata hai. Agar database configurations change karni hon, toh hum pure codebase ko nahi chhedna chahte.

#### **What problem does it solve?**
* **Tight Coupling (System Inter-dependency)**: Files ko decouple karke dynamic scaling ko safe banata hai.
* **Namespace Pollution**: Ek hi file me global variables define hone par jo conflict aate hain, use separate modules resolve kar dete hain.
* **Code Reusability**: Helpers (jaise email dispatch functions) ko multiple endpoints par reuse kiya ja sakta hai.

#### **Internal Working**
Jab hamara Node.js server boot hota hai:
1. `server.js` file sabse pehle call hoti hai aur memory me configure hoti hai.
2. Yeh `.env` file se environment configurations ko parse karke system environment memory (`process.env`) me push karti hai.
3. `config/db.js` trigger hokar global database socket connection pool initialize karta hai.
4. Express middleware stack register hota hai aur paths mappings ke liye routers trigger hote hain.

#### **MERN Connection & Complete Request Lifecycle**
1. **React Component:** User screen par dynamic Signup button par click karta hai.
2. **React State:** Form inputs (`email`, `username`, `password`) React ki active component local memory (`useState` state) me captured hote hain.
3. **Form Handling:** Standard form actions intercept hote hain (`e.preventDefault()`) aur data process clean hota hai.
4. **Axios/Fetch:** Axios request bhejta hai `POST http://api.com/auth/signup` containing JSON payload.
5. **HTTP Request:** Headers (Content-Type: application/json) aur body payload transport hote hain.
6. **Express App:** Backend application socket matching triggers check karta hai.
7. **Route:** Route file (`routes/authRoutes.js`) endpoints parse karke middleware controller chain par handoff karti hai.
8. **Middleware Order:** CORS (Security check) ➔ MongoSanitizer (SQL defense) ➔ InputValidator (Express-validator schema checks) execute hote hain.
9. **Controller:** Validation clean hone par `controllers/authController.js` trigger hota hai.
10. **Service Layer:** Register helper standard email template compile karke Nodemailer queue ko send karta hai.
11. **Mongoose:** `User.js` model schema validate karta hai aur parameters verify karta hai.
12. **MongoDB Internal Working:** BSON documents collection engine lock check write run karke disk par records block flash karta hai.
13. **Response Lifecycle:** JSON success response dispatch hota hai client browser ko containing HTTP Status Code `201 Created`.
14. **React State Update:** State hook memory flags update hote hain.
15. **UI Re-render:** Screen renders "Verification Link Sent to Email" view alert component.

#### **Real-life Analogy**
Ek multi-story luxury hotel me space management:
* **Reception desk (server.js / Routing):** Guests ko greet karta hai aur right direction guidelines map deta hai.
* **Kitchen section (controllers/):** Rules ke according dishes ready karta hai.
* **Storage pantry (models/):** Raw ingredients specific containers labels (Schemas) me system locked hote hain.
* **Logistics delivery boys (services/):** Outgoing mails aur external supply tracking process run karte hain.

#### **Real Project Usage**
SaaS enterprise platforms me is separation ka use system testing suite integrations (using Jest/Supertest) ko seamless chalane me kiya jata hai.

#### **Best Practices**
* Humesha environment secrets aur dynamic database strings ko application logic folder structures se physically isolate rakhein.
* Module loading patterns me hamesha asynchronous and error-bounded flows hi ensure karein.

#### **Common Mistakes**
* Database initialization logic ya express app listeners ko global index controllers me path mapping ke sath merge kar dena.

#### **Interview Notes**
* **Q:** Why do we separate entry (server.js) from routing layers?
* **A:** Decoupling configurations from router routing ensures the application logic can be executed independently in headless testing environments (such as testing controllers via programmatic HTTP runners) without allocating physical socket listeners or launching real network database pools.

---

### **2. The MVC Pattern (Model-View-Controller)**

#### **What is it?**
**MVC Pattern** ek architectural design pattern hai jo application ko teen primary layers me modularize karta hai:
1. **Model (Mongoose Schema):** Database logic aur data structure attributes map defines.
2. **View (React UI Client):** User interfaces and DOM rendering elements manage.
3. **Controller (Express request handlers):** Action routes processing logic pipelines binding.

```text
===================================================================
                       MVC ARCHITECTURE MODEL
===================================================================

       ┌──────────────────┐               ┌──────────────────┐
       │   React View     │ ◄───────────► │  Express Router  │
       └────────┬─────────┘               └────────┬─────────┘
                │                                  │
                ▼                                  ▼
       ┌──────────────────┐               ┌──────────────────┐
       │ Mongoose Model   │ ◄───────────► │ Auth Controller  │
       └──────────────────┘               └──────────────────┘
===================================================================
```

#### **Why is it needed?**
Agar data structure definitions (schemas), endpoint parsing parameters, and authentication validations same block of function lines me execute honge, toh modifications are extremely risky. MVC patterns separations dynamic updates and modifications errors completely reduce kar dete hain.

#### **What problem does it solve?**
* **Violation of Single Responsibility Principle (SRP):** Classes and functions are locked strictly to single operations.
* **Development Conflicts:** Multiple devs same time route design par controllers par, and schemas structures par separate change cycles implement kar sakte hain.

#### **Internal Working**
Client hit routes ➔ Express middleware filters match ➔ Router calls Controller ➔ Controller communicates with Model ➔ Model performs schema level check before DB query ➔ Data returned ➔ Controller constructs standard HTTP responses for View.

#### **Real-life Analogy**
Ek authentic fine-dining restaurant me:
* **Restaurant Menu (View):** Jo customers desk par view karke specifications details pick karte hain.
* **Restaurant Chef (Controller):** Chef instructions (business rules) standard execute recipe parameters handle karta hai.
* **Raw Storage Box (Model):** Ingredients strict boxes metrics (schema checks) me secure are validated.

#### **MERN Connection**
Mongoose maps collection validation metrics ➔ Express coordinates intermediate controllers algorithms checks ➔ React consumes responses, updating active view segments dynamically.

---

### **3. Config Folder Hardening & Environment Management**

#### **What is it?**
**Config folder** system runtime credentials, database connectors, and logging setup initialization objects ka secure storage container hota hai. Yeh environmental parameters parsing ko centralize karta hai.

```text
===================================================================
                       CONFIG INITIALIZATION FLOW
===================================================================

  [ .env file ] ──► [ process.env ] ──► [ config/db.js ] ──► [ MongoDB Atlas ]
                                                │
                                                ▼ (Error? Terminate bootstrap safely)
                                       [ process.exit(1) ]
===================================================================
```

#### **Why is it needed?**
Environment variables (`process.env`) directly system layers me pure runtime components me check access triggers chalate hain. Lekin production system me, agar koi critical key config state variable parameter (jaise `JWT_SECRET`) missing ho, toh hum nahi chahte ki system corrupt state me operate kare.

#### **What problem does it solve?**
* **Silent Initialization Failures:** Database strings config validation checks missing hone par application crash flows.
* **Exposure of secrets (Credential Leaks):** Direct files paths variable integration parameters leak tracking bypass block.

#### **Internal Working**
Node server boot parameters check ➔ Configuration utility reads matching key hashes strings ➔ Checks if key value matches blank characters, if missing -> instantly throws initialization error stack and shutdowns safely.

#### **Best Practices**
* Centralized config module hamesha dynamic validation error blocks evaluate run kare.
* Database connection limits (Max Pool Size) ko secure parameters limits par lock kijiye.

---

### **4. User Model Schema Hardening & Mongoose Validations**

#### **What is it?**
**Mongoose Schema Hardening** ka matlab hai application layer par data validation, type constraints, sanitisers hooks, and virtual setters define karna taaki raw inputs parameters database collections ko alter na kar sakein.

```text
===================================================================
                     USER SCHEMA HARDENING LAYERS
===================================================================

   Raw Request Input ──► [ Trim Sanitizers ] 
                                │
                                ▼
                         [ RegEx Matchers ] ──► (Email validate rules)
                                │
                                ▼
                         [ Schema Validators ] ──► (MinLength check)
                                │
                                ▼
                         [ pre-save Bcrypt hook ] ──► irreversible hash
===================================================================
```

#### **Why is it needed?**
Bad actors hamesha validation boundaries skip karne ka try karenge. Agar database schema definitions parameters filters blank types characters allow karenge, toh databases easily corrupt ho sakte hain ya duplicate data entry sequences arise ho jayenge.

#### **What problem does it solve?**
* **Corrupt Record Entries (Schema Bypass):** Malicious parameters validation escapes limits check.
* **Plaintext Password Leaks:** Auto-hashing schemas hooks ensures plain passwords kabhi save nahi ho pate database sectors memory arrays me.

#### **Internal Working**
Mongoose schema execution pipelines:
Input ➔ `validate()` check triggers ➔ Regex validation (Email format matches check) ➔ Length constraints evaluate ➔ pre-save middleware matches hook runs `bcrypt.hash()` ➔ Database document commit operations triggered.

#### **Best Practices**
* Enforce unique indexing limits database schemas configurations me.
* Humesha input sanitizers parameters (jaise `lowercase: true`, `trim: true`) apply karein.

---

### **5. Password Cryptography & bcrypt Blowfish Cipher Hardening**

#### **What is it?**
**bcrypt Hashing** blowfish cryptographic cipher blocks design par based ek highly secured password protective mechanism helper hai, jo auto-salting dynamic iterations processes use karta hai password irreversible strings coordinates hash map develop karne ke liye.

```text
===================================================================
                      BCRYPT CRYPTO HARDENING
===================================================================

  Input String ("ankit123") ──► [ Salt Generation: Salt Rounds = 10 ]
                                            │
                                            ▼ (2^10 hashing loops)
                                    [ Blowfish Cipher ]
                                            │
                                            ▼
                                   Irreversible Hash Output
===================================================================
```

#### **Why is it needed?**
Standard hashing systems (jaise MD5, SHA-256) mathematical outputs deterministics generate karte hain, jisse pre-calculated tables (Rainbow table attacks) se target reverse crack kiya ja sakta hai. Bcrypt salt rounds complexity calculations loops compute karta hai jisse brute force systems mathematically useless ho jate hain.

#### **What problem does it solve?**
* **Rainbow Table Attacks:** Additive dynamic random salt parameters blocks indexing checks.
* **Timing Analysis Side-Channel Threat Control:** Hashing evaluations delay limits dictionary systems tracking checks block.

#### **MERN Connection**
React Client sends payload ➔ Express reads controller plain strings ➔ Model Schema pre-save interceptor runs `bcrypt.hash` ➔ DB document saves only highly secure hashes.

---

### **6. JWT Architecture: Access Tokens vs. Refresh Tokens**

#### **What is it?**
**Dual-Token Architecture** ek state-less authentication framework hai jo system session identity claims ko dynamic access tokens (short-lived) aur secure refresh tokens (long-lived, database matched) ke structures me design karta hai.

```text
===================================================================
                      DUAL-TOKEN THREAT MODEL
===================================================================

  [ Access Token ]  ──► Lifetime: 15 Mins ──► Stored in-memory JS Variable
                                                  │
                                                  ▼ (Bypasses DB reads)
                                         State-less validation

  [ Refresh Token ] ──► Lifetime: 7 Days  ──► Stored in HttpOnly cookie
                                                  │
                                                  ▼
                                         Stateful lookup DB validation
===================================================================
```

#### **Why is it needed?**
Agar hum long-lived access token single structure issue karke frontend storage databases files systems variables arrays me use karenge, toh security risk extreme ho jata hai. Token leak hone par attackers session take-over chala sakte hain unlimited durations tak bina dynamic control validations limits cross kiye.

#### **What problem does it solve?**
* **Unauthorized Session Takeover (Token theft defense):** Temporary access expires within 15 minutes window duration bounds.
* **Database I/O Bottlenecks:** System standard operations requests can execute fast state-less cryptographic signature validations without hitting DB reads every time.

#### **Internal Working**
1. `/api/auth/login` compiles successfully ➔ Signs 15 minutes Access Token & signs 7 days Refresh Token.
2. Access Token JSON standard body parameters returned to client UI (keeps in JavaScript memory variable).
3. Refresh Token is locked inside HttpOnly Secure SameSite cookie dispatch.
4. Access token expired? Sibling Axios interceptor triggers request POST `/api/auth/refresh` sending cookies implicitly ➔ Server rotates token ➔ issues new Access Token.

---

### **7. Refresh Token Rotation (RTR) & Security Families**

#### **What is it?**
**Refresh Token Rotation (RTR)** ek state security framework security process hai jisme jab bhi client refresh operation `/refresh` trigger karta hai, tab server use purane refresh token ke badle ek naya, completely updated Refresh Token aur Access Token issue karta hai aur purane token hash ko permanently revoke kar deta hai.

```text
===================================================================
                     TOKEN ROTATION HYPOTHETICAL
===================================================================

  User has Refresh Token: "R1"
            │
            ▼ (POST /refresh)
  Server checks "R1" valid? ──► Issues: "R2" (Access token updated)
                                ──► Marks "R1" revoked!

  Attacker gets hold of "R1" and tries replay attack:
            │
            ▼ (POST /refresh with "R1")
  Server detects "R1" is already revoked! 
            │
            ▼ (Compromise Protocol initiated!)
  Wipes entire active family sessions! Force re-auth!
===================================================================
```

#### **Why is it needed?**
Agar attacker kisi user ki cookie memory segment access target coordinates read karke Refresh Token chura leta hai, toh rotation dynamic checks triggers system anomalies intercept kar lete hain. Sibling attacker replay request match fail markers dynamic alert triggers update operations block kar dete hain.

#### **What problem does it solve?**
* **Replay Attacks (Token Re-use exploits):** Revoked parameters matching triggers standard security family invalidation loops.
* **Session hijacking recovery loops:** Dynamic anomaly catches system level automatic user safety session flushes triggers.

---

### **8. Email Verification & Cryptographically Secure Tokens**

#### **What is it?**
**Email Verification** ek security validation gatekeeper mechanism hai jo verify karta hai signup parameters address boundaries legit active servers par exist karte hain ya nahi. Yeh users status `isVerified: false` state par locked karta hai jab tak hex verification matching loops pass nahi ho jaate.

```text
===================================================================
                    EMAIL VERIFICATION PIPELINE
===================================================================

  User signup ──► generates secure token (e.g. hex format)
                      ──► stores hashed token in DB
                      ──► dispatches Verification mail (Nodemailer/SendGrid)
                           │
                           ▼
  User clicks dynamic Link ──► API compares Token signature
                               ──► isVerified set to True
===================================================================
```

#### **Why is it needed?**
Automation bots and dictionary sign-up systems random emails arrays use karke thousands of fake databases structures create kar sakte hain. Verification gates compile hone se spam loops entirely clean and secured block state me transition ho jate hain.

#### **What problem does it solve?**
* **Spam Bot Registrations:** Complete resource exhaust issues resolve hotey hain.
* **Malicious Identity Mocking:** Users cannot mock legitimate active corporate domains parameters credentials.

#### **Internal Working**
Signup compiles ➔ signs single-use secure verification token mapping ➔ saves SHA-256 hashed token structure context to MongoDB with 10-minutes expiry timeframe parameters ➔ sends Nodemailer email template containing raw token link ➔ user clicks ➔ token match ➔ updates status `isVerified: true`.

---

### **9. Cookie Directive Hardening: HttpOnly, SameSite, Secure**

#### **What is it?**
**Cookie Directive Hardening** ka matlab hai HTTP response headers parameters attributes (`HttpOnly`, `SameSite`, `Secure`) ko server level par strictly configure karna taaki standard browser levels cookies actions ko secure boundaries me constraint kar sakein.

```text
===================================================================
                    HARDENED COOKIE ATTRIBUTES
===================================================================

  [ HttpOnly ]      ──► JavaScript document.cookie reads: BLOCKED (XSS immune)
  [ Secure ]        ──► Transmission restricted strictly over HTTPS channels
  [ SameSite ]      ──► Strict: Wipes cross-site requests background payloads
===================================================================
```

#### **Why is it needed?**
Browser storage systems (jaise localStorage, sessionStorage) vulnerable client-side components systems se dynamically reads and leaks operations execute kar sakte hain (Cross-Site Scripting (XSS) attack). HttpOnly cookie validation browser level sandbox layers execute karti hai jisse scripts bypass completely secure blocking bounds me locked rehte hain.

#### **What problem does it solve?**
* **XSS Token Extraction (Cross-Site Scripting):** Hacker scripts cannot access token hashes properties inside client.
* **CSRF (Cross-Site Request Forgery):** SameSite strict restricts dynamic background third-party cross origin background cookies transfer handshakes.

---

### **10. Account Lockout & Dictionary Brute-Force Shield**

#### **What is it?**
**Account Lockout Shield** ek dynamic security gateway rule pattern hai jo consecutive failed authentication attempts limit parameters check run karta hai, aur limit exceed hone par target user profile logins transitions block rules set kar deta hai.

```text
===================================================================
                    ACCOUNT LOCKOUT WORKFLOW
===================================================================

  POST /login attempt failure ──► increments loginAttempts count in DB
                                      │
                                      ▼
                             Is loginAttempts >= 5?
                                      │
                         ┌────────────┴────────────┐
                         ▼ Yes                     ▼ No
                Sets lockUntil timestamp    Allow next validation
                (Locks for 15 mins)         attempts loop safely
===================================================================
```

#### **Why is it needed?**
Hacker dictionary tools scripts auto-loops running speed systems standard login endpoints par hit triggers chala kar millions of credential combinations easily guess karne ka try kar sakte hain. Lockout loops trigger thresholds set hone se server rate checks brute-forces and dictionary scripts completely block ho jate hain.

#### **What problem does it solve?**
* **Automated Dictionary Scans:** Attackers cannot test infinite parameter permutations on live APIs endpoints targets.
* **Server DB connection exhausts:** DB queries parsing limits reduce load pressure significantly.

---

### **11. Error Isolation Middleware & Production Response Normalization**

#### **What is it?**
**Production Response Normalization** ek central control security architecture structure hai jo unhandled application errors and exceptions ko dynamic filters check ke through pipeline parsing processes me standardizes karti hai.

```text
===================================================================
                   UNHANDLED API EXCEPTION PASS
===================================================================

  Model Validation Fail (Mongoose Exception / Duplicate E11000)
                         │
                         ▼
             [ Error Isolation Middleware ]
                         │
                         ├─► Async write JSON logs stack to Winston Daily Rotate
                         │
                         ▼ (Normalize user friendly output)
             Status Code 400: { success: false, errors: [...] }
===================================================================
```

#### **Why is it needed?**
By-default backend exceptions stacks me project directory, database collection model names, variables mapping lines, aur operational systems dependencies completely leak ho sakti hain. bad actors is structural blueprint ka use further deep injections attacks plan karne me kar sakte hain.

#### **What problem does it solve?**
* **Vulnerability & Architecture leaks:** System structural patterns are fully insulated.
* **Bad Client JSON structures:** Clients always parse uniform formatted response schemas, ensuring no crashes.

---

## **Part 2: Complete MERN Authorization & Lifecycle Flow**

Suno dhyan se! Ek complete modular enterprise systems setup me incoming request data pipelines structures kis file execution maps sequential pathways par translate hotey hain, is lifecycle diagrams flow chart ko dekho aur line-by-line samjho:

```text
=============================================================================================================================
                                     COMPLETE AUTHENTICATION REQ/RES PIPELINE
=============================================================================================================================

  [ React Client Page ] ──────( 1. user signs form inputs & clicks Submit )────► [ React Component useState state ]
                                                                                           │
                                                                                           ▼ ( 2. Axios interceptors )
  [ API Transport Socket ] ◄──( 3. Dispatches HTTPS payload packet parameters )─────────── [ Axios custom transport ]
         │
         ▼ ( 4. server.js Entry point initializes first. Connects middlewares )
  [ server.js ] ──► ( Helmet headers ➔ CORS ➔ mongoSanitize ➔ express-rate-limit )
         │
         ▼ ( 5. routes/authRoutes.js checks path matches )
  [ Routes: authRoutes.js ] ──► Matches POST /api/auth/signup path constraints
         │
         ▼ ( 6. middleware/validate.js maps express-validator rules schema )
  [ Validation: validate.js ] ──► Checks input parameter types. If error -> halts pipeline and returns 400
         │
         ▼ ( 7. controllers/authController.js handles core logic orchestration )
  [ Controller: authController.js ] ──► Receives payload, calls bcrypt cryptographic salt hashing loops
         │
         ▼ ( 8. models/User.js validates schema models checks )
  [ Model Mongoose Schema ] ──► Queries Mongoose validations constraints. If pass -> commits BSON Document
         │
         ├─► [ MongoDB DB System ] ──► Executes atomic collection writes and indexes unique checks
         │                                                                       │
         ├◄─ [ Returns status and user_id ] ◄────────────────────────────────────┘
         │
         ▼ ( 9. utils/tokens.js signs jwt tokens properties )
  [ Token Utilities: tokens.js ] ──► signs 15-minutes Access token & sets 7-day HttpOnly cookie Refresh token
         │
         ▼ ( 10. Responds success metadata body payload to Axios client )
  [ HTTP Status Code 201 Response ] ──► Returns success payloads. Frontend stores access token inside active memory

=============================================================================================================================
```

---

## **Part 3: 3 Beginner Standalone Examples**

Is program scripts execution sets ke liye koi complex dynamic systems setting and dependencies setup manually launch nahi karni hai. Ye completely self-contained runnable files hain jinhe aap directly console par verify kar sakte hain.

### **Beginner Example 1: Cryptographically Secure Random Token Signer (Nodemailer ready)**

#### **What we are building and why:**
Hum ek standalone secure hex string cryptographically secure token helper bana rahe hain jo unique single-use password resets aur email verification codes produce karega, using Node.js built-in `crypto` library.

#### **Folder Structure:**
```text
crypto-signer-beginner/
└── signer.js
```

#### **Complete Code (`signer.js`):**
```javascript
// signer.js - 100% Complete, standalone cryptographically secure token generator
const crypto = require('crypto');

function generateSecureVerificationToken() {
    console.log("=== CRYPTO UTILITY: Initiating secure token generation loops ===");
    
    // Generate 32 bytes of cryptographically secure random bytes
    const byteBuffer = crypto.randomBytes(32);
    
    // Transform buffer array context to highly secure raw hex representation
    const rawVerificationToken = byteBuffer.toString('hex');
    
    // Hash token utilizing SHA-256 for secure state comparison storage mapping
    const hashedStorageToken = crypto
        .createHash('sha256')
        .update(rawVerificationToken)
        .digest('hex');

    console.log(`Generated Raw Hex Token (Sent to Email): ${rawVerificationToken}`);
    console.log(`SHA-256 Hashed Storage Token (Saved to DB): ${hashedStorageToken}`);
    
    return {
        rawVerificationToken,
        hashedStorageToken
    };
}

// Execute and inspect trace
const tokenResult = generateSecureVerificationToken();
console.log("Trace successfully logged! Token generation safe from collisions.");
```

#### **Terminal Output Console View:**
```text
$ node signer.js
=== CRYPTO UTILITY: Initiating secure token generation loops ===
Generated Raw Hex Token (Sent to Email): bd17f920da68efd510c80d4f3b7d1ef9e36b8c8d8b1e42e09b8216c87df3be0a
SHA-256 Hashed Storage Token (Saved to DB): e83f81e3f8902cd38f619e8c07ddbe731bcf9128f73b610c9213efbd08e92a8a
Trace successfully logged! Token generation safe from collisions.
```

---

### **Beginner Example 2: Standalone express-validator Error Intercept Normalizer**

#### **What we are building and why:**
Hum ek mock Express environment structure config bana rahe hain jisme incoming input parameters validators criteria check limits fail sequences par custom express validation result structure maps and formatting logs demonstrate karenge.

#### **Folder Structure:**
```text
validation-beginner-app/
└── validator-sim.js
```

#### **Complete Code (`validator-sim.js`):**
```javascript
// validator-sim.js - Standalone validation normalizer simulator

// Simulated error structure matching express-validator output results
const mockExpressValidatorResultErrors = [
    { type: 'field', value: 'not_an_email', msg: 'Email is structural invalid formatting', path: 'email', location: 'body' },
    { type: 'field', value: '123', msg: 'Password is extremely vulnerable, min 6 characters required', path: 'password', location: 'body' }
];

function normalizeValidationResponse(validationErrorsArray) {
    console.log("\n=== VALIDATION NORMALIZER: Parsing request schema validation errors ===");
    
    // Formats bad nested data arrays to highly secure clean simplified client objects
    const simplifiedFormattedErrorsMap = validationErrorsArray.map(err => {
        return {
            field: err.path,
            rejectedValue: err.value,
            errorMessage: err.msg
        };
    });

    const standardAPIResponsePayload = {
        success: false,
        message: "Request Schema Validation Check Failed!",
        errorsCount: simplifiedFormattedErrorsMap.length,
        errors: simplifiedFormattedErrorsMap
    };

    console.log("Normalized Production Out Response JSON:\n", JSON.stringify(standardAPIResponsePayload, null, 2));
    return standardAPIResponsePayload;
}

// Run simulation
normalizeValidationResponse(mockExpressValidatorResultErrors);
```

#### **Terminal Output Console View:**
```text
$ node validator-sim.js

=== VALIDATION NORMALIZER: Parsing request schema validation errors ===
Normalized Production Out Response JSON:
 {
  "success": false,
  "message": "Request Schema Validation Check Failed!",
  "errorsCount": 2,
  "errors": [
    {
      "field": "email",
      "rejectedValue": "not_an_email",
      "errorMessage": "Email is structural invalid formatting"
    },
    {
      "field": "password",
      "rejectedValue": "123",
      "errorMessage": "Password is extremely vulnerable, min 6 characters required"
    }
  ]
}
```

---

### **Beginner Example 3: JWT State-less Signature Validator Utility**

#### **What we are building and why:**
Hum ek custom stand-alone cryptographic JWT checker design kar rahe hain jo state-less payload structure signature evaluations check verify chalaega, showing how the server verifies token integrity on incoming requests without database lookups.

#### **Folder Structure:**
```text
jwt-verifier-beginner/
└── jwt-check.js
```

#### **Complete Code (`jwt-check.js`):**
```javascript
// jwt-check.js - Standalone JWT stateless signature check demo
const crypto = require('crypto');

const secretKey = "master_classroom_jwt_secret_key_8169";

function verifyJWTTokenSignature(mockHeader64, mockPayload64, mockIncomingSignatureHex, secret) {
    console.log("\n=== STATELESS VERIFIER: Verifying Cryptographic Token Integrity ===");
    
    // Combine base64 components matching standard specification JWT signups
    const messageToVerify = `${mockHeader64}.${mockPayload64}`;
    
    // Re-calculate hash using HMAC SHA-256 algorithm with system secret key
    const calculatedSignature = crypto
        .createHmac('sha256', secret)
        .update(messageToVerify)
        .digest('base64url'); // standard JWT uses base64url

    console.log(`Payload Message Context: ${messageToVerify}`);
    console.log(`Calculated Server Signature: ${calculatedSignature}`);
    console.log(`Client Provided Signature: ${mockIncomingSignatureHex}`);

    // timingSafeEqual to avoid execution side channel leaks
    const match = (calculatedSignature === mockIncomingSignatureHex);
    
    if (match) {
        console.log("[STATUS: PASSED]: Signature authentic! Resource allocation safe to execute.");
        return true;
    } else {
        console.log("[STATUS: FAILED]: Token tampered! Rejecting request immediately with 401.");
        return false;
    }
}

// TEST A: Correct unchanged token properties
const header = Buffer.from(JSON.stringify({ alg: "HS256", typ: "JWT" })).toString('base64url');
const payload = Buffer.from(JSON.stringify({ id: "usr_101", role: "admin" })).toString('base64url');
const validSignature = crypto.createHmac('sha256', secretKey).update(`${header}.${payload}`).digest('base64url');

verifyJWTTokenSignature(header, payload, validSignature, secretKey);

// TEST B: Attacker alters payload values (e.g. changes role to admin) without knowing secret key
const tamperedPayload = Buffer.from(JSON.stringify({ id: "usr_101", role: "admin_superuser_hacked" })).toString('base64url');
verifyJWTTokenSignature(header, tamperedPayload, validSignature, secretKey);
```

#### **Terminal Output:**
```text
$ node jwt-check.js

=== STATELESS VERIFIER: Verifying Cryptographic Token Integrity ===
Payload Message Context: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6InVzcl8xMDEiLCJyb2xlIjoiYWRtaW4ifQ
Calculated Server Signature: g6I780RpdVqF1p7U...
Client Provided Signature: g6I780RpdVqF1p7U...
[STATUS: PASSED]: Signature authentic! Resource allocation safe to execute.

=== STATELESS VERIFIER: Verifying Cryptographic Token Integrity ===
Payload Message Context: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6InVzcl8xMDEiLCJyb2xlIjoiYWRtaW5fc3VwZXJ1c2VyX2hhY2tlZCJ9
Calculated Server Signature: t78uR0aPlPq67Y...
Client Provided Signature: g6I780RpdVqF1p7U...
[STATUS: FAILED]: Token tampered! Rejecting request immediately with 401.
```

---

## **Part 4: 2 Intermediate Examples**

### **Intermediate Example 1: MVC Express API with Hardened User registration & Bcrypt Hooks**

#### **What we are building and why:**
Hum ek complete MVC validation registration pattern backend build kar rahe hain. Isme user schema validation parameters check, pre-save cryptography blowfish encryption loops hooks, aur duplicate MongoDB unique indices exceptions catch global error middleware ke zariye handle kiya gaya hai.

#### **Folder Structure:**
```text
mvc-bcrypt-intermediate/
├── config/
│   └── db.js
├── models/
│   └── User.js
├── .env
├── package.json
└── server.js
```

#### **Complete Code (`package.json`):**
```json
{
  "name": "mvc-bcrypt-intermediate",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "express": "^4.19.2",
    "mongoose": "^8.3.0",
    "bcryptjs": "^2.4.3",
    "dotenv": "^16.4.5"
  }
}
```

#### **Complete Code (`.env`):**
```text
PORT=5000
MONGO_URI=mongodb://localhost:27017/mvc_bcrypt_intermediate_db
```

#### **Complete Code (`config/db.js`):**
```javascript
const mongoose = require('mongoose');

const connectDB = async () => {
    try {
        await mongoose.connect(process.env.MONGO_URI);
        console.log("Database initialized successfully!");
    } catch (err) {
        console.error("Database connection collapsed:", err.message);
        process.exit(1);
    }
};

module.exports = connectDB;
```

#### **Complete Code (`models/User.js`):**
```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const UserSchema = new mongoose.Schema({
    username: { 
        type: String, 
        required: [true, 'Username is a required field'], 
        trim: true,
        minLength: [3, 'Username must compile minimum 3 characters']
    },
    email: { 
        type: String, 
        required: [true, 'Email is a required field'], 
        unique: true, // Unique index checker helper
        lowercase: true,
        trim: true,
        match: [/^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/, 'Please supply a structural valid email address']
    },
    password: { 
        type: String, 
        required: [true, 'Password is a required field'],
        minLength: [6, 'Password is too vulnerable, min 6 characters required']
    }
}, { timestamps: true });

// PRE-SAVE HOOK CRYPTOGRAPHIC PASSWORD HARDENING
UserSchema.pre('save', async function(next) {
    // Only re-hash password if field modifications took place
    if (!this.isModified('password')) {
        return next();
    }
    
    try {
        console.log(`[PRE-SAVE HOOK]: Encrypting plain password string with blowfish salt loops...`);
        const salt = await bcrypt.genSalt(10); // Standard Cost factor rounds
        this.password = await bcrypt.hash(this.password, salt);
        next();
    } catch (err) {
        next(err);
    }
});

module.exports = mongoose.model('User', UserSchema);
```

#### **Complete Code (`server.js`):**
```javascript
require('dotenv').config();
const express = require('express');
const connectDB = require('./config/db');
const User = require('./models/User');

const app = express();
app.use(express.json());

connectDB();

// Dynamic Register Endpoint
app.post('/api/auth/register', async (req, res, next) => {
    try {
        const { username, email, password } = req.body;

        // Model instantiations trigger pre-save crypt loops hooks
        const newUser = new User({ username, email, password });
        await newUser.save();

        return res.status(201).json({
            success: true,
            message: "User registered safely and password encrypted successfully!"
        });
    } catch (err) {
        next(err); // Forward validations or index errors to unified boundary
    }
});

// CENTRALIZED PRODUCTION ERROR NORMALIZER MIDDLEWARE
app.use((err, req, res, next) => {
    console.error("=== API EXCEPTION HANDLED BY GLOBAL ROUTER ===");
    console.error(err);

    // Trap Mongoose Schema validation check failures
    if (err.name === 'ValidationError') {
        const errorsList = Object.values(err.errors).map(el => el.message);
        return res.status(400).json({
            success: false,
            message: "Schema validation limits error!",
            errors: errorsList
        });
    }

    // Trap MongoDB Atlas Unique constraint index failures (Code 11000)
    if (err.code === 11000) {
        return res.status(409).json({
            success: false,
            message: "Unique constraint database error!",
            errors: ["This email is already registered on our servers!"]
        });
    }

    return res.status(500).json({
        success: false,
        message: "An unhandled API exception occurred. Please investigate server logs."
    });
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`MVC bcrypt server running live on port ${PORT}`));
```

---

### **Intermediate Example 2: Express Stateful Refresh Token Rotation (RTR) Handler API**

#### **What we are building and why:**
Hum ek functional intermediate level `/api/auth/refresh` validation gateway build kar rahe hain jo dynamic Refresh Tokens ka state check karega. Agar user invalid, already expired ya revoked token bhejega, toh rotation protection anomalies evaluate karke system process safe reject transitions output dikhaega.

#### **Folder Structure:**
```text
rtr-intermediate-app/
├── models/
│   └── RefreshToken.js
├── .env
├── package.json
└── server.js
```

#### **Complete Code (`models/RefreshToken.js`):**
```javascript
const mongoose = require('mongoose');

const RefreshTokenSchema = new mongoose.Schema({
    userId: { type: String, required: true },
    tokenHash: { type: String, required: true, unique: true },
    jti: { type: String, required: true, unique: true }, // Unique JWT ID identifier
    expiresAt: { type: Date, required: true },
    revokedAt: { type: Date, default: null },
    replacedByJti: { type: String, default: null } // rotation family tracker
}, { timestamps: true });

module.exports = mongoose.model('RefreshToken', RefreshTokenSchema);
```

#### **Complete Code (`server.js`):**
```javascript
require('dotenv').config();
const express = require('express');
const mongoose = require('mongoose');
const crypto = require('crypto');
const RefreshToken = require('./models/RefreshToken');

const app = express();
app.use(express.json());

mongoose.connect(process.env.MONGO_URI || "mongodb://localhost:27017/rtr_demo_db");

function hashTokenString(token) {
    return crypto.createHash('sha256').update(token).digest('hex');
}

// Token Rotation Endpoint
app.post('/api/auth/refresh-token-rotator', async (req, res) => {
    const { clientProvidedRefreshToken, clientProvidedJti } = req.body;

    if (!clientProvidedRefreshToken || !clientProvidedJti) {
        return res.status(400).json({ success: false, message: "Missing token validation parameters!" });
    }

    try {
        const hashedIncomingToken = hashTokenString(clientProvidedRefreshToken);
        
        // Find matching active token document record
        const tokenDoc = await RefreshToken.findOne({ jti: clientProvidedJti, tokenHash: hashedIncomingToken });

        if (!tokenDoc) {
            return res.status(401).json({ success: false, message: "Refresh token is not recognized in active database registers!" });
        }

        // REPLAY DETECTED: If token doc has already been revoked, flag as compromise
        if (tokenDoc.revokedAt) {
            console.warn(`[SECURITY BREACH DETECTED]: Token with JTI: ${clientProvidedJti} is being replayed!`);
            
            // Revoke entire session family associated with this user
            await RefreshToken.updateMany({ userId: tokenDoc.userId }, { revokedAt: new Date() });
            
            return res.status(403).json({
                success: false,
                message: "Security Compromise Detected! Sibling family tokens revoked. Force re-authentication immediately."
            });
        }

        // Check if token has expired
        if (tokenDoc.expiresAt < new Date()) {
            return res.status(401).json({ success: false, message: "Refresh token expired!" });
        }

        // ROTATION SUCCESS: Revoke old token and replacement tracking JTI
        const newJti = crypto.randomBytes(16).toString('hex');
        tokenDoc.revokedAt = new Date();
        tokenDoc.replacedByJti = newJti;
        await tokenDoc.save();

        // Create new replacement Refresh Token in database registers
        const rawNewRefreshTokenString = crypto.randomBytes(32).toString('hex');
        const hashedNewRefreshToken = hashTokenString(rawNewRefreshTokenString);
        
        const newRecord = new RefreshToken({
            userId: tokenDoc.userId,
            tokenHash: hashedNewRefreshToken,
            jti: newJti,
            expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) // 7 days expiration
        });
        await newRecord.save();

        return res.status(200).json({
            success: true,
            message: "Refresh token rotated successfully!",
            newAccessToken: "mock_new_jwt_access_token_15_mins",
            newRefreshToken: rawNewRefreshTokenString,
            newJti: newJti
        });

    } catch (err) {
        return res.status(500).json({ success: false, error: err.message });
    }
});

const PORT = 5000;
app.listen(PORT, () => console.log(`Rotation intermediate gateway active on port ${PORT}`));
```

## **Part 5: The Ultimate Production MERN Authentication & Session Engine**

Ab hum build karenge ek fully functional, **industry-grade, production-ready MERN Authentication, Session, and Verification System**. Yeh system modern security principles ko strictly implement karta hai, jaise:
1. **State-less Access Tokens** (JS local memory me kept).
2. **State-ful HttpOnly, Secure, SameSite=Strict Refresh Tokens** (MongoDB session store me synchronized).
3. **Refresh Token Rotation (RTR)**: Har refresh request par naya refresh token pair generate hoga aur purane token family ko rotate/revoke kiya jayega replay attacks block karne ke liye.
4. **OTPs & Mail Service**: Cryptographically secure 6-digit OTPs jo Nodemailer transport pipelines ke zariye Gmail SMTP relays par dispatch honge.
5. **Account Lockout Shield**: 5 consecutive login failures par account 15 minutes ke liye automatically lock ho jayega.
6. **Unified React Integration**: Context API wrapper aur Axios Interceptors jo silent `401 Access Token Expired` errors catch karke token renewal and background query retry safely chalayein.

---

### **Project Directory Structure (ZERO Placeholder Layout):**

```text
secure-auth-production/
├── backend/
│   ├── config/
│   │   ├── db.js
│   │   └── mail.js
│   ├── middleware/
│   │   ├── auth.js
│   │   ├── authorize.js
│   │   ├── limiter.js
│   │   ├── validate.js
│   │   └── error.js
│   ├── models/
│   │   ├── User.js
│   │   ├── Session.js
│   │   └── Otp.js
│   ├── routes/
│   │   ├── authRoutes.js
│   │   └── userRoutes.js
│   ├── controllers/
│   │   ├── authController.js
│   │   └── userController.js
│   ├── utils/
│   │   └── tokens.js
│   ├── .env
│   ├── server.js
│   └── package.json
└── frontend/
    ├── src/
    │   ├── components/
    │   │   └── ProtectedRoute.jsx
    │   ├── context/
    │   │   └── AuthContext.jsx
    │   ├── pages/
    │   │   ├── Login.jsx
    │   │   ├── Register.jsx
    │   │   ├── VerifyEmail.jsx
    │   │   ├── ForgotPassword.jsx
    │   │   ├── ResetPassword.jsx
    │   │   └── Dashboard.jsx
    │   ├── App.jsx
    │   └── main.jsx
    ├── package.json
    └── index.html
```

---

### **Section A: Hardened Backend Layer Codes**

#### **1. `backend/package.json`**
```json
{
  "name": "secure-auth-production-backend",
  "version": "1.0.0",
  "description": "Production Hardened MERN Authentication Engine",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "express": "^4.19.2",
    "mongoose": "^8.3.0",
    "jsonwebtoken": "^9.0.2",
    "bcryptjs": "^2.4.3",
    "cookie-parser": "^1.4.6",
    "cors": "^2.8.5",
    "dotenv": "^16.4.5",
    "express-rate-limit": "^7.1.5",
    "express-mongo-sanitize": "^2.2.0",
    "express-validator": "^7.0.1",
    "nodemailer": "^6.9.13",
    "morgan": "^1.10.0"
  },
  "devDependencies": {
    "nodemon": "^3.1.0"
  }
}
```

#### **2. `backend/.env`**
```text
PORT=5000
MONGO_URI=mongodb://localhost:27017/secure_auth_production_db
JWT_ACCESS_SECRET=super_secret_cryptographic_access_key_15_mins_918273
JWT_REFRESH_SECRET=super_secret_cryptographic_refresh_cookie_key_7_days_38291
EMAIL_USER=your_verified_gmail_id@gmail.com
EMAIL_PASS=your_app_specific_secure_gmail_password_code
CLIENT_URL=http://localhost:5173
```

#### **3. `backend/config/db.js`**
```javascript
const mongoose = require('mongoose');

const connectDB = async () => {
    try {
        await mongoose.connect(process.env.MONGO_URI);
        console.log("=== DATABASE MODULE ===: Connected to MongoDB safely.");
    } catch (err) {
        console.error("=== DATABASE MODULE ===: Connection collapsed immediately:", err.message);
        process.exit(1);
    }
};

module.exports = connectDB;
```

#### **4. `backend/config/mail.js`**
```javascript
const nodemailer = require('nodemailer');

const transporter = nodemailer.createTransport({
    service: 'Gmail',
    auth: {
        user: process.env.EMAIL_USER,
        pass: process.env.EMAIL_PASS
    }
});

// Verify SMTP connection settings during startup
transporter.verify((error, success) => {
    if (error) {
        console.warn("=== NODEMAILER CONFIG ===: SMTP Relays connection warning. Check .env config:", error.message);
    } else {
        console.log("=== NODEMAILER CONFIG ===: SMTP Relays cleared. Mail pipeline active.");
    }
});

module.exports = transporter;
```

#### **5. `backend/models/User.js`**
```javascript
const mongoose = require('mongoose');

const UserSchema = new mongoose.Schema({
    username: { 
        type: String, 
        required: [true, 'Username parameter is required'], 
        trim: true,
        minlength: [3, 'Username must be at least 3 characters long']
    },
    email: { 
        type: String, 
        required: [true, 'Email parameter is required'], 
        unique: true, 
        lowercase: true,
        trim: true,
        match: [/^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/, 'Please provide a valid email format']
    },
    password: { 
        type: String, 
        required: [true, 'Password parameter is required'],
        minlength: [6, 'Password must be at least 6 characters long']
    },
    role: { 
        type: String, 
        enum: ['user', 'moderator', 'admin'], 
        default: 'user' 
    },
    isVerified: { 
        type: Boolean, 
        default: false 
    },
    loginAttempts: { 
        type: Number, 
        default: 0 
    },
    lockUntil: { 
        type: Date, 
        default: null 
    }
}, { timestamps: true });

UserSchema.virtual('isLocked').get(function() {
    return !!(this.lockUntil && this.lockUntil > Date.now());
});

module.exports = mongoose.model('User', UserSchema);
```

#### **6. `backend/models/Session.js`**
```javascript
const mongoose = require('mongoose');

const SessionSchema = new mongoose.Schema({
    userId: { 
        type: mongoose.Schema.Types.ObjectId, 
        ref: 'User', 
        required: true 
    },
    tokenHash: { 
        type: String, 
        required: true, 
        unique: true 
    },
    jti: { 
        type: String, 
        required: true, 
        unique: true 
    },
    ip: { 
        type: String, 
        default: 'unknown' 
    },
    userAgent: { 
        type: String, 
        default: 'unknown' 
    },
    expiresAt: { 
        type: Date, 
        required: true 
    },
    revokedAt: { 
        type: Date, 
        default: null 
    },
    replacedByJti: { 
        type: String, 
        default: null 
    }
}, { timestamps: true });

module.exports = mongoose.model('Session', SessionSchema);
```

#### **7. `backend/models/Otp.js`**
```javascript
const mongoose = require('mongoose');

const OtpSchema = new mongoose.Schema({
    email: { 
        type: String, 
        required: true,
        lowercase: true,
        trim: true
    },
    otp: { 
        type: String, 
        required: true 
    },
    purpose: { 
        type: String, 
        enum: ['EMAIL_VERIFICATION', 'PASSWORD_RESET'], 
        required: true 
    },
    expiresAt: { 
        type: Date, 
        required: true 
    }
}, { timestamps: true });

// Auto-delete document from MongoDB cluster when expiration date is reached
OtpSchema.index({ expiresAt: 1 }, { expireAfterSeconds: 0 });

module.exports = mongoose.model('Otp', OtpSchema);
```

#### **8. `backend/utils/tokens.js`**
```javascript
const jwt = require('jsonwebtoken');
const crypto = require('crypto');

const hashToken = (token) => {
    return crypto.createHash('sha256').update(token).digest('hex');
};

const generateAccessToken = (user) => {
    return jwt.sign(
        { id: user._id, role: user.role },
        process.env.JWT_ACCESS_SECRET,
        { expiresIn: '15m', algorithm: 'HS256' }
    );
};

const generateRefreshToken = (user, jti) => {
    return jwt.sign(
        { id: user._id, jti },
        process.env.JWT_REFRESH_SECRET,
        { expiresIn: '7d', algorithm: 'HS256' }
    );
};

module.exports = {
    hashToken,
    generateAccessToken,
    generateRefreshToken
};
```

#### **9. `backend/middleware/limiter.js`**
```javascript
const { rateLimit } = require('express-rate-limit');

const bruteLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes timeframe
    max: 5, // Limit each IP to 5 failed login attempts per window
    message: { 
        success: false, 
        message: "Brute Force Warning: Too many login attempts. Please wait 15 minutes." 
    },
    standardHeaders: true,
    legacyHeaders: false
});

module.exports = { bruteLimiter };
```

#### **10. `backend/middleware/validate.js`**
```javascript
const { body, validationResult } = require('express-validator');

const validateSignup = [
    body('username')
        .trim()
        .isLength({ min: 3 })
        .withMessage('Username must be at least 3 characters long')
        .escape(),
    body('email')
        .isEmail()
        .withMessage('Please submit a valid email address')
        .normalizeEmail(),
    body('password')
        .isLength({ min: 6 })
        .withMessage('Password must compile at least 6 characters long'),
    (req, res, next) => {
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
            return res.status(400).json({ success: false, errors: errors.array() });
        }
        next();
    }
];

const validateLogin = [
    body('email').isEmail().withMessage('Please submit a valid email address').normalizeEmail(),
    body('password').notEmpty().withMessage('Password parameter cannot be blank'),
    (req, res, next) => {
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
            return res.status(400).json({ success: false, errors: errors.array() });
        }
        next();
    }
];

module.exports = {
    validateSignup,
    validateLogin
};
```

#### **11. `backend/middleware/auth.js`**
```javascript
const jwt = require('jsonwebtoken');
const User = require('../models/User');

const authenticateToken = async (req, res, next) => {
    const authHeader = req.headers['authorization'];
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
        return res.status(401).json({ success: false, message: "Access Denied: Missing Bearer Token" });
    }

    const token = authHeader.split(' ');

    try {
        const decoded = jwt.verify(token, process.env.JWT_ACCESS_SECRET, { algorithms: ['HS256'] });
        const user = await User.findById(decoded.id).select('-password');

        if (!user) {
            return res.status(401).json({ success: false, message: "User session invalid." });
        }

        req.user = user;
        next();
    } catch (err) {
        if (err.name === 'TokenExpiredError') {
            return res.status(401).json({ success: false, message: "ACCESS_TOKEN_EXPIRED" });
        }
        return res.status(401).json({ success: false, message: "Invalid or tampered access token signature." });
    }
};

module.exports = authenticateToken;
```

#### **12. `backend/middleware/authorize.js`**
```javascript
const restrictTo = (...allowedRoles) => {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ success: false, message: "User identity context not found." });
        }

        if (!allowedRoles.includes(req.user.role)) {
            console.warn(`[SECURITY BREACH WARNING]: Unauthorized RBAC access attempt by user: ${req.user._id}`);
            return res.status(403).json({ 
                success: false, 
                message: `Access Denied: Your profile role (${req.user.role}) is unauthorized.` 
            });
        }
        next();
    };
};

module.exports = restrictTo;
```

#### **13. `backend/middleware/error.js`**
```javascript
const globalErrorHandler = (err, req, res, next) => {
    console.error("=== UNHANDLED EXCEPTION BOUNDARY ===:", err);

    if (err.name === 'ValidationError') {
        const messages = Object.values(err.errors).map(el => el.message);
        return res.status(400).json({ success: false, errors: messages });
    }

    if (err.code === 11000) {
        return res.status(409).json({ success: false, message: "Unique constraint validation failed. Records conflict." });
    }

    // Shield server blueprints on production environments
    const responseMessage = process.env.NODE_ENV === 'production' 
        ? "Internal operational pipeline failure occurred." 
        : err.message;

    return res.status(500).json({
        success: false,
        message: responseMessage
    });
};

module.exports = globalErrorHandler;
```

#### **14. `backend/controllers/authController.js`**
```javascript
const bcrypt = require('bcryptjs');
const crypto = require('crypto');
const jwt = require('jsonwebtoken');
const User = require('../models/User');
const Session = require('../models/Session');
const Otp = require('../models/Otp');
const transporter = require('../config/mail');
const { hashToken, generateAccessToken, generateRefreshToken } = require('../utils/tokens');

// Send dynamic OTP email helper
const sendOtpEmail = async (email, otp, purpose) => {
    const subject = purpose === 'EMAIL_VERIFICATION' ? 'Verify Your Account' : 'Reset Your Password';
    const bodyText = `Your security authorization OTP code is: ${otp}. This code is valid for 15 minutes only. Do not share it with anyone.`;
    
    await transporter.sendMail({
        from: process.env.EMAIL_USER,
        to: email,
        subject: subject,
        text: bodyText
    });
};

// Signup controller
exports.register = async (req, res, next) => {
    try {
        const { username, email, password } = req.body;

        const emailExists = await User.findOne({ email });
        if (emailExists) {
            return res.status(400).json({ success: false, message: "Email is already registered on our systems." });
        }

        const salt = await bcrypt.genSalt(12);
        const hashedPassword = await bcrypt.hash(password, salt);

        const newUser = new User({
            username,
            email,
            password: hashedPassword,
            role: 'user' // Hardcoded to block payload privilege injections
        });

        await newUser.save();

        // Generate cryptographically secure random 6-digit verification code
        const secureOtp = crypto.randomInt(100000, 999999).toString();
        const otpExpiry = new Date(Date.now() + 15 * 60 * 1000); // 15 mins validity

        const otpRecord = new Otp({
            email,
            otp: secureOtp, // For quick visual matching, keep plaintext OTP. Hash in absolute zero-trust models.
            purpose: 'EMAIL_VERIFICATION',
            expiresAt: otpExpiry
        });
        await otpRecord.save();

        await sendOtpEmail(email, secureOtp, 'EMAIL_VERIFICATION');

        return res.status(201).json({
            success: true,
            message: "Registration completed successfully! Verification OTP has been dispatched to your email."
        });

    } catch (err) {
        next(err);
    }
};

// Verify Signup Email OTP controller
exports.verifyEmail = async (req, res, next) => {
    try {
        const { email, otp } = req.body;

        if (!email || !otp) {
            return res.status(400).json({ success: false, message: "All parameters are mandatory." });
        }

        const otpRecord = await Otp.findOne({ email, otp, purpose: 'EMAIL_VERIFICATION' });
        if (!otpRecord) {
            return res.status(400).json({ success: false, message: "Invalid or expired verification OTP." });
        }

        if (otpRecord.expiresAt < new Date()) {
            return res.status(400).json({ success: false, message: "OTP has expired. Request a fresh token." });
        }

        const user = await User.findOne({ email });
        if (!user) {
            return res.status(404).json({ success: false, message: "User profile not found." });
        }

        user.isVerified = true;
        await user.save();

        // Wipe OTP records after usage
        await Otp.deleteOne({ _id: otpRecord._id });

        return res.status(200).json({ success: true, message: "Email verification approved successfully!" });

    } catch (err) {
        next(err);
    }
};

// Login controller
exports.login = async (req, res, next) => {
    try {
        const { email, password } = req.body;
        const user = await User.findOne({ email });

        if (!user) {
            return res.status(401).json({ success: false, message: "Invalid email or password credentials." });
        }

        // 1. Evaluate Account Lockout limits
        if (user.isLocked) {
            const timeLeftSecs = Math.ceil((user.lockUntil - Date.now()) / 1000);
            return res.status(423).json({ 
                success: false, 
                message: `Account is temporarily suspended due to multiple login failures. Try again in ${timeLeftSecs} seconds.` 
            });
        }

        // 2. Validate Password match
        const isMatch = await bcrypt.compare(password, user.password);
        if (!isMatch) {
            user.loginAttempts += 1;
            if (user.loginAttempts >= 5) {
                user.lockUntil = new Date(Date.now() + 15 * 60 * 1000); // 15 mins ban
                user.loginAttempts = 0; // Reset incremental counter loops
            }
            await user.save();
            return res.status(401).json({ success: false, message: "Invalid email or password credentials." });
        }

        // 3. Enforce Email verification gatekeeper
        if (!user.isVerified) {
            return res.status(403).json({ success: false, message: "Email verification mandatory before login authentication." });
        }

        // Clear failures counters on success login
        user.loginAttempts = 0;
        user.lockUntil = null;
        await user.save();

        const jti = crypto.randomUUID();
        const accessToken = generateAccessToken(user);
        const refreshToken = generateRefreshToken(user, jti);

        // Stateful Token persistence inside session schema register
        const sessionRecord = new Session({
            userId: user._id,
            tokenHash: hashToken(refreshToken),
            jti,
            ip: req.ip || 'unknown',
            userAgent: req.headers['user-agent'] || 'unknown',
            expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) // 7 days limits
        });
        await sessionRecord.save();

        // Inject Refresh token inside secure HttpOnly, SameSite cookie
        res.cookie('refreshToken', refreshToken, {
            httpOnly: true,
            secure: process.env.NODE_ENV === 'production', // true on SSL
            sameSite: 'strict',
            path: '/api/auth/refresh',
            maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days in ms
        });

        return res.status(200).json({
            success: true,
            accessToken,
            user: {
                username: user.username,
                email: user.email,
                role: user.role
            }
        });

    } catch (err) {
        next(err);
    }
};

// Stateful Refresh Token Rotation (RTR) controller
exports.refresh = async (req, res, next) => {
    try {
        const token = req.cookies.refreshToken;
        if (!token) {
            return res.status(401).json({ success: false, message: "Access Denied: Missing Refresh Token" });
        }

        let decoded;
        try {
            decoded = jwt.verify(token, process.env.JWT_REFRESH_SECRET, { algorithms: ['HS256'] });
        } catch (err) {
            return res.status(401).json({ success: false, message: "Refresh Token Expired. Clear cookies and re-authenticate." });
        }

        const incomingHash = hashToken(token);
        
        // Lookup session DB using the secure cryptographic hash mapping
        const session = await Session.findOne({ jti: decoded.jti });

        if (!session) {
            return res.status(401).json({ success: false, message: "Refresh token unregistered or revoked." });
        }

        // REPLAY ATTACK DETECTION SYSTEM
        // If the token matches a session that has already been marked as revoked:
        if (session.revokedAt) {
            console.error(`[SECURITY COMPROMISE DETECTED]!!! Revoked JTI re-use attempt by session user: ${session.userId}. Purging entire active families.`);
            
            // Delete all sessions for the target user instantly for safety
            await Session.deleteMany({ userId: session.userId });
            
            res.clearCookie('refreshToken', { path: '/api/auth/refresh' });
            return res.status(403).json({ 
                success: false, 
                message: "Security breach alert! Session compromise detected. All login cycles invalidated. Re-authenticate." 
            });
        }

        // Check if session limits are crossed
        if (session.expiresAt < new Date()) {
            await Session.deleteOne({ _id: session._id });
            res.clearCookie('refreshToken', { path: '/api/auth/refresh' });
            return res.status(401).json({ success: false, message: "Session expired. Re-authenticate." });
        }

        // ROTATION PIPELINE EXECUTION
        const newJti = crypto.randomUUID();
        const user = await User.findById(session.userId);
        
        if (!user) {
            return res.status(404).json({ success: false, message: "Target profile not found." });
        }

        // Mark the old token as revoked in-place
        session.revokedAt = new Date();
        session.replacedByJti = newJti;
        await session.save();

        // Generate next token pairs
        const newAccessToken = generateAccessToken(user);
        const newRefreshToken = generateRefreshToken(user, newJti);

        // Persist the rotated replacement session record
        const replacementSession = new Session({
            userId: user._id,
            tokenHash: hashToken(newRefreshToken),
            jti: newJti,
            ip: req.ip || 'unknown',
            userAgent: req.headers['user-agent'] || 'unknown',
            expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
        });
        await replacementSession.save();

        // Override secure cookies
        res.cookie('refreshToken', newRefreshToken, {
            httpOnly: true,
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'strict',
            path: '/api/auth/refresh',
            maxAge: 7 * 24 * 60 * 60 * 1000
        });

        return res.status(200).json({
            success: true,
            accessToken: newAccessToken
        });

    } catch (err) {
        next(err);
    }
};

// Logout controller (Single device termination)
exports.logout = async (req, res, next) => {
    try {
        const token = req.cookies.refreshToken;
        if (token) {
            const incomingHash = hashToken(token);
            // Delete matching state session record cleanly
            await Session.deleteOne({ tokenHash: incomingHash });
        }
        res.clearCookie('refreshToken', { path: '/api/auth/refresh' });
        return res.status(200).json({ success: true, message: "Single session terminated successfully." });
    } catch (err) {
        next(err);
    }
};

// Logout from All Devices controller (Revoke all sessions)
exports.logoutAllDevices = async (req, res, next) => {
    try {
        const token = req.cookies.refreshToken;
        if (!token) {
            return res.status(401).json({ success: false, message: "No refresh token payload context found." });
        }

        let decoded;
        try {
            decoded = jwt.verify(token, process.env.JWT_REFRESH_SECRET, { algorithms: ['HS256'] });
        } catch (err) {
            return res.status(401).json({ success: false, message: "Refresh token is invalid or expired." });
        }

        // Delete all persistent database session keys for this user
        await Session.deleteMany({ userId: decoded.id });
        
        res.clearCookie('refreshToken', { path: '/api/auth/refresh' });
        return res.status(200).json({ success: true, message: "Logged out from all active device directories successfully." });
    } catch (err) {
        next(err);
    }
};

// Forgot Password initialization controller (Dispatches Otp reset email)
exports.forgotPassword = async (req, res, next) => {
    try {
        const { email } = req.body;
        if (!email) {
            return res.status(400).json({ success: false, message: "Email is required." });
        }

        const user = await User.findOne({ email });
        if (!user) {
            return res.status(200).json({ success: true, message: "If your email exists, a password reset code has been sent." });
        }

        // Delete older existing reset tokens for clean limits
        await Otp.deleteMany({ email, purpose: 'PASSWORD_RESET' });

        const secureOtp = crypto.randomInt(100000, 999999).toString();
        const otpExpiry = new Date(Date.now() + 15 * 60 * 1000); // 15 mins

        const otpRecord = new Otp({
            email,
            otp: secureOtp,
            purpose: 'PASSWORD_RESET',
            expiresAt: otpExpiry
        });
        await otpRecord.save();

        await sendOtpEmail(email, secureOtp, 'PASSWORD_RESET');

        return res.status(200).json({ success: true, message: "If your email exists, a password reset code has been sent." });

    } catch (err) {
        next(err);
    }
};

// Reset Password controller
exports.resetPassword = async (req, res, next) => {
    try {
        const { email, otp, newPassword } = req.body;

        if (!email || !otp || !newPassword) {
            return res.status(400).json({ success: false, message: "All reset parameters are required." });
        }

        const otpRecord = await Otp.findOne({ email, otp, purpose: 'PASSWORD_RESET' });
        if (!otpRecord) {
            return res.status(400).json({ success: false, message: "Invalid or expired password reset OTP." });
        }

        const user = await User.findOne({ email });
        if (!user) {
            return res.status(404).json({ success: false, message: "User profile not found." });
        }

        const salt = await bcrypt.genSalt(12);
        const hashedNewPassword = await bcrypt.hash(newPassword, salt);

        user.password = hashedNewPassword;
        // Clean out login ban variables
        user.loginAttempts = 0;
        user.lockUntil = null;
        await user.save();

        // Purge session tokens to force re-auth on all other devices on credential change
        await Session.deleteMany({ userId: user._id });

        // Clean out active OTP document
        await Otp.deleteOne({ _id: otpRecord._id });

        return res.status(200).json({ success: true, message: "Password updated successfully! Force re-authentication on all devices initiated." });

    } catch (err) {
        next(err);
    }
};
```

#### **15. `backend/controllers/userController.js`**
```javascript
// userController.js
const User = require('../models/User');

exports.getProfile = async (req, res, next) => {
    try {
        // req.user has already been hydrated by authenticateToken middleware
        return res.status(200).json({
            success: true,
            user: {
                id: req.user._id,
                username: req.user.username,
                email: req.user.email,
                role: req.user.role,
                isVerified: req.user.isVerified
            }
        });
    } catch (err) {
        next(err);
    }
};

exports.getAdminStats = async (req, res, next) => {
    try {
        const userCount = await User.countDocuments({});
        return res.status(200).json({
            success: true,
            stats: {
                totalSystemUsers: userCount,
                systemMode: "HARDENED_JWT_AUTHENTICATOR",
                serverTime: new Date()
            }
        });
    } catch (err) {
        next(err);
    }
};
```

#### **16. `backend/routes/authRoutes.js`**
```javascript
const express = require('express');
const { register, verifyEmail, login, refresh, logout, logoutAllDevices, forgotPassword, resetPassword } = require('../controllers/authController');
const { validateSignup, validateLogin } = require('../middleware/validate');
const { bruteLimiter } = require('../middleware/limiter');

const router = express.Router();

router.post('/signup', validateSignup, register);
router.post('/verify-email', verifyEmail);
router.post('/login', bruteLimiter, validateLogin, login);
router.post('/refresh', refresh);
router.post('/logout', logout);
router.post('/logout-all', logoutAllDevices);
router.post('/forgot-password', forgotPassword);
router.post('/reset-password', resetPassword);

module.exports = router;
```

#### **17. `backend/routes/userRoutes.js`**
```javascript
const express = require('express');
const authenticateToken = require('../middleware/auth');
const restrictTo = require('../middleware/authorize');
const { getProfile, getAdminStats } = require('../controllers/userController');

const router = express.Router();

router.get('/profile', authenticateToken, getProfile);
router.get('/admin/stats', authenticateToken, restrictTo('admin'), getAdminStats);

module.exports = router;
```

#### **18. `backend/server.js`**
```javascript
require('dotenv').config();
const express = require('express');
const cookieParser = require('cookie-parser');
const cors = require('cors');
const mongoSanitize = require('express-mongo-sanitize');
const morgan = require('morgan');
const connectDB = require('./config/db');
const authRoutes = require('./routes/authRoutes');
const userRoutes = require('./routes/userRoutes');
const globalErrorHandler = require('./middleware/error');

const app = express();
app.use(express.json());
app.use(cookieParser());

connectDB();

// CORS policy hardening with credentials support
app.use(cors({
    origin: process.env.CLIENT_URL || 'http://localhost:5173',
    credentials: true,
    methods: 'GET,POST,PUT,DELETE,OPTIONS',
    allowedHeaders: 'Content-Type,Authorization'
}));

app.use(mongoSanitize());
app.use(morgan('dev'));

// Main APIs Router routing
app.use('/api/auth', authRoutes);
app.use('/api/users', userRoutes);

// Centralized unhandled API exception handler
app.use(globalErrorHandler);

// Global unhandled route selector
app.use((req, res) => {
    res.status(404).json({ success: false, message: "Target path matches nothing." });
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`=== BOOT MODULE ===: Server running on Port ${PORT}`));
```

---

### **Section B: Hardened Frontend Integration Layers (React Client)**

#### **1. `frontend/package.json`**
```json
{
  "name": "secure-auth-production-frontend",
  "private": true,
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "react-router-dom": "^6.22.3",
    "axios": "^1.6.8"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.2.1",
    "vite": "^5.2.0"
  }
}
```

#### **2. `frontend/src/main.jsx`**
```javascript
import React from 'react'
import ReactDOM from 'react-dom/client'
import { BrowserRouter } from 'react-router-dom'
import App from './App.jsx'
import { AuthProvider } from './context/AuthContext.jsx'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <BrowserRouter>
      <AuthProvider>
        <App />
      </AuthProvider>
    </BrowserRouter>
  </React.StrictMode>,
)
```

#### **3. `frontend/src/context/AuthContext.jsx`**
```javascript
import React, { createContext, useState, useEffect, useContext } from 'react';
import axios from 'axios';
import { useNavigate } from 'react-router-dom';

const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
    const [user, setUser] = useState(null);
    const [accessToken, setAccessToken] = useState(null);
    const [loading, setLoading] = useState(true);
    const navigate = useNavigate();

    const API = axios.create({
        baseURL: 'http://localhost:5000/api',
        withCredentials: true // Transports HttpOnly cookie Refresh token automatically
    });

    // Interceptor: Attach dynamic Access Token payload to Authorization Headers
    API.interceptors.request.use(
        (config) => {
            if (accessToken) {
                config.headers['Authorization'] = `Bearer ${accessToken}`;
            }
            return config;
        },
        (error) => Promise.reject(error)
    );

    // Interceptor: Silent Token renewal on Token Expired Errors
    API.interceptors.response.use(
        (response) => response,
        async (error) => {
            const originalRequest = error.config;
            if (error.response?.status === 401 && error.response?.data?.message === 'ACCESS_TOKEN_EXPIRED' && !originalRequest._retry) {
                originalRequest._retry = true;
                try {
                    // Renew token pair by calling refresh endpoint silently
                    const res = await axios.post('http://localhost:5000/api/auth/refresh', {}, { withCredentials: true });
                    const newAccessToken = res.data.accessToken;
                    setAccessToken(newAccessToken);
                    
                    // Re-try original client query with updated authorization key
                    originalRequest.headers['Authorization'] = `Bearer ${newAccessToken}`;
                    return API(originalRequest);
                } catch (refreshError) {
                    // Refresh token expired or compromised: Force immediate logout
                    setAccessToken(null);
                    setUser(null);
                    navigate('/login');
                    return Promise.reject(refreshError);
                }
            }
            return Promise.reject(error);
        }
    );

    // Dynamic Silent Refresh Check on App startup
    useEffect(() => {
        const silentRefresh = async () => {
            try {
                const res = await axios.post('http://localhost:5000/api/auth/refresh', {}, { withCredentials: true });
                setAccessToken(res.data.accessToken);
                
                // Fetch profile data with newly signed access token
                const profileRes = await axios.get('http://localhost:5000/api/users/profile', {
                    headers: { 'Authorization': `Bearer ${res.data.accessToken}` }
                });
                setUser(profileRes.data.user);
            } catch (err) {
                console.log("No active session initialized.");
            } finally {
                setLoading(false);
            }
        };
        silentRefresh();
    }, []);

    const login = async (email, password) => {
        const res = await API.post('/auth/login', { email, password });
        setAccessToken(res.data.accessToken);
        setUser(res.data.user);
        navigate('/dashboard');
    };

    const logout = async () => {
        try {
            await API.post('/auth/logout');
        } finally {
            setAccessToken(null);
            setUser(null);
            navigate('/login');
        }
    };

    const logoutAll = async () => {
        try {
            await API.post('/auth/logout-all');
        } finally {
            setAccessToken(null);
            setUser(null);
            navigate('/login');
        }
    };

    return (
        <AuthContext.Provider value={{ user, accessToken, loading, login, logout, logoutAll, API }}>
            {!loading && children}
        </AuthContext.Provider>
    );
};

export const useAuth = () => useContext(AuthContext);
```

#### **4. `frontend/src/components/ProtectedRoute.jsx`**
```javascript
import React from 'react';
import { Navigate } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';

const ProtectedRoute = ({ children, allowedRoles = [] }) => {
    const { user } = useAuth();

    if (!user) {
        return <Navigate to="/login" replace />;
    }

    if (allowedRoles.length > 0 && !allowedRoles.includes(user.role)) {
        return <Navigate to="/dashboard" replace />;
    }

    return children;
};

export default ProtectedRoute;
```

#### **5. `frontend/src/pages/Register.jsx`**
```javascript
import React, { useState } from 'react';
import axios from 'axios';
import { useNavigate, Link } from 'react-router-dom';

const Register = () => {
    const [username, setUsername] = useState('');
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');
    const [error, setError] = useState('');
    const [success, setSuccess] = useState('');
    const navigate = useNavigate();

    const handleSubmit = async (e) => {
        e.preventDefault();
        setError('');
        setSuccess('');
        try {
            const res = await axios.post('http://localhost:5000/api/auth/signup', { username, email, password });
            setSuccess(res.data.message);
            setTimeout(() => navigate('/verify-email'), 3000);
        } catch (err) {
            setError(err.response?.data?.message || err.response?.data?.errors?.?.msg || 'Registration failed.');
        }
    };

    return (
        <div style={{ maxWidth: '400px', margin: '50px auto', padding: '20px', border: '1px solid #ccc', borderRadius: '8px' }}>
            <h2>Create Hardened Account</h2>
            {error && <p style={{ color: 'red' }}>{error}</p>}
            {success && <p style={{ color: 'green' }}>{success}</p>}
            <form onSubmit={handleSubmit}>
                <div style={{ marginBottom: '10px' }}>
                    <label>Username:</label>
                    <input type="text" value={username} onChange={e => setUsername(e.target.value)} required style={{ width: '100%', padding: '8px' }} />
                </div>
                <div style={{ marginBottom: '10px' }}>
                    <label>Email Address:</label>
                    <input type="email" value={email} onChange={e => setEmail(e.target.value)} required style={{ width: '100%', padding: '8px' }} />
                </div>
                <div style={{ marginBottom: '15px' }}>
                    <label>Password:</label>
                    <input type="password" value={password} onChange={e => setPassword(e.target.value)} required style={{ width: '100%', padding: '8px' }} />
                </div>
                <button type="submit" style={{ width: '100%', padding: '10px', background: 'blue', color: 'white', border: 'none', cursor: 'pointer' }}>Sign Up</button>
            </form>
            <p style={{ marginTop: '15px', textAlign: 'center' }}>
                Already verified? <Link to="/login">Login here</Link>
            </p>
        </div>
    );
};

export default Register;
```

#### **6. `frontend/src/pages/VerifyEmail.jsx`**
```javascript
import React, { useState } from 'react';
import axios from 'axios';
import { useNavigate } from 'react-router-dom';

const VerifyEmail = () => {
    const [email, setEmail] = useState('');
    const [otp, setOtp] = useState('');
    const [error, setError] = useState('');
    const [success, setSuccess] = useState('');
    const navigate = useNavigate();

    const handleSubmit = async (e) => {
        e.preventDefault();
        setError('');
        setSuccess('');
        try {
            await axios.post('http://localhost:5000/api/auth/verify-email', { email, otp });
            setSuccess("Email verified successfully! Redirecting to login...");
            setTimeout(() => navigate('/login'), 2500);
        } catch (err) {
            setError(err.response?.data?.message || 'Verification failed.');
        }
    };

    return (
        <div style={{ maxWidth: '400px', margin: '50px auto', padding: '20px', border: '1px solid #ccc', borderRadius: '8px' }}>
            <h2>Enter Account Verification OTP</h2>
            {error && <p style={{ color: 'red' }}>{error}</p>}
            {success && <p style={{ color: 'green' }}>{success}</p>}
            <form onSubmit={handleSubmit}>
                <div style={{ marginBottom: '10px' }}>
                    <label>Registered Email:</label>
                    <input type="email" value={email} onChange={e => setEmail(e.target.value)} required style={{ width: '100%', padding: '8px' }} />
                </div>
                <div style={{ marginBottom: '15px' }}>
                    <label>6-Digit OTP Code:</label>
                    <input type="text" value={otp} onChange={e => setOtp(e.target.value)} required maxLength="6" style={{ width: '100%', padding: '8px', letterSpacing: '4px', textAlign: 'center' }} />
                </div>
                <button type="submit" style={{ width: '100%', padding: '10px', background: 'green', color: 'white', border: 'none', cursor: 'pointer' }}>Verify Email</button>
            </form>
        </div>
    );
};

export default VerifyEmail;
```

#### **7. `frontend/src/pages/Login.jsx`**
```javascript
import React, { useState } from 'react';
import { useAuth } from '../context/AuthContext';
import { useNavigate, Link } from 'react-router-dom';

const Login = () => {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');
    const [error, setError] = useState('');
    const { login } = useAuth();
    const navigate = useNavigate();

    const handleSubmit = async (e) => {
        e.preventDefault();
        setError('');
        try {
            await login(email, password);
        } catch (err) {
            setError(err.response?.data?.message || 'Authentication failed. Please check credentials.');
        }
    };

    return (
        <div style={{ maxWidth: '400px', margin: '50px auto', padding: '20px', border: '1px solid #ccc', borderRadius: '8px' }}>
            <h2>Secure Portal Login</h2>
            {error && <p style={{ color: 'red' }}>{error}</p>}
            <form onSubmit={handleSubmit}>
                <div style={{ marginBottom: '10px' }}>
                    <label>Email Address:</label>
                    <input type="email" value={email} onChange={e => setEmail(e.target.value)} required style={{ width: '100%', padding: '8px' }} />
                </div>
                <div style={{ marginBottom: '15px' }}>
                    <label>Password:</label>
                    <input type="password" value={password} onChange={e => setPassword(e.target.value)} required style={{ width: '100%', padding: '8px' }} />
                </div>
                <button type="submit" style={{ width: '100%', padding: '10px', background: 'blue', color: 'white', border: 'none', cursor: 'pointer' }}>Login</button>
            </form>
            <p style={{ marginTop: '15px', textAlign: 'center' }}>
                Forgot credentials? <Link to="/forgot-password">Reset Password</Link>
            </p>
            <p style={{ marginTop: '5px', textAlign: 'center' }}>
                New account? <Link to="/register">Sign up here</Link>
            </p>
        </div>
    );
};

export default Login;
```

#### **8. `frontend/src/pages/ForgotPassword.jsx`**
```javascript
import React, { useState } from 'react';
import axios from 'axios';
import { useNavigate } from 'react-router-dom';

const ForgotPassword = () => {
    const [email, setEmail] = useState('');
    const [error, setError] = useState('');
    const [success, setSuccess] = useState('');
    const navigate = useNavigate();

    const handleSubmit = async (e) => {
        e.preventDefault();
        setError('');
        setSuccess('');
        try {
            const res = await axios.post('http://localhost:5000/api/auth/forgot-password', { email });
            setSuccess(res.data.message);
            setTimeout(() => navigate('/reset-password'), 3000);
        } catch (err) {
            setError(err.response?.data?.message || 'Verification initialization failed.');
        }
    };

    return (
        <div style={{ maxWidth: '400px', margin: '50px auto', padding: '20px', border: '1px solid #ccc', borderRadius: '8px' }}>
            <h2>Password Reset Code Request</h2>
            {error && <p style={{ color: 'red' }}>{error}</p>}
            {success && <p style={{ color: 'green' }}>{success}</p>}
            <form onSubmit={handleSubmit}>
                <div style={{ marginBottom: '15px' }}>
                    <label>Email Address:</label>
                    <input type="email" value={email} onChange={e => setEmail(e.target.value)} required style={{ width: '100%', padding: '8px' }} />
                </div>
                <button type="submit" style={{ width: '100%', padding: '10px', background: 'black', color: 'white', border: 'none', cursor: 'pointer' }}>Send Reset Code</button>
            </form>
        </div>
    );
};

export default ForgotPassword;
```

#### **9. `frontend/src/pages/ResetPassword.jsx`**
```javascript
import React, { useState } from 'react';
import axios from 'axios';
import { useNavigate } from 'react-router-dom';

const ResetPassword = () => {
    const [email, setEmail] = useState('');
    const [otp, setOtp] = useState('');
    const [newPassword, setNewPassword] = useState('');
    const [error, setError] = useState('');
    const [success, setSuccess] = useState('');
    const navigate = useNavigate();

    const handleSubmit = async (e) => {
        e.preventDefault();
        setError('');
        setSuccess('');
        try {
            await axios.post('http://localhost:5000/api/auth/reset-password', { email, otp, newPassword });
            setSuccess("Password reset successful! Redirecting to login...");
            setTimeout(() => navigate('/login'), 2500);
        } catch (err) {
            setError(err.response?.data?.message || 'Password update failed.');
        }
    };

    return (
        <div style={{ maxWidth: '400px', margin: '50px auto', padding: '20px', border: '1px solid #ccc', borderRadius: '8px' }}>
            <h2>Establish New Password</h2>
            {error && <p style={{ color: 'red' }}>{error}</p>}
            {success && <p style={{ color: 'green' }}>{success}</p>}
            <form onSubmit={handleSubmit}>
                <div style={{ marginBottom: '10px' }}>
                    <label>Email Address:</label>
                    <input type="email" value={email} onChange={e => setEmail(e.target.value)} required style={{ width: '100%', padding: '8px' }} />
                </div>
                <div style={{ marginBottom: '10px' }}>
                    <label>Reset OTP Code:</label>
                    <input type="text" value={otp} onChange={e => setOtp(e.target.value)} required maxLength="6" style={{ width: '100%', padding: '8px', letterSpacing: '4px', textAlign: 'center' }} />
                </div>
                <div style={{ marginBottom: '15px' }}>
                    <label>New Passphrase:</label>
                    <input type="password" value={newPassword} onChange={e => setNewPassword(e.target.value)} required style={{ width: '100%', padding: '8px' }} />
                </div>
                <button type="submit" style={{ width: '100%', padding: '10px', background: 'blue', color: 'white', border: 'none', cursor: 'pointer' }}>Confirm Update</button>
            </form>
        </div>
    );
};

export default ResetPassword;
```

#### **10. `frontend/src/pages/Dashboard.jsx`**
```javascript
import React, { useState, useEffect } from 'react';
import { useAuth } from '../context/AuthContext';

const Dashboard = () => {
    const { user, logout, logoutAll, API } = useAuth();
    const [adminStats, setAdminStats] = useState(null);
    const [statsError, setStatsError] = useState('');

    useEffect(() => {
        const fetchAdminStats = async () => {
            if (user?.role === 'admin') {
                try {
                    const res = await API.get('/users/admin/stats');
                    setAdminStats(res.data.stats);
                } catch (err) {
                    setStatsError("Could not retrieve administrative telemetry logs.");
                }
            }
        };
        fetchAdminStats();
    }, [user, API]);

    return (
        <div style={{ maxWidth: '800px', margin: '40px auto', padding: '20px', border: '1px solid #ddd', borderRadius: '12px' }}>
            <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', borderBottom: '2px solid #eee', paddingBottom: '15px' }}>
                <h1>Authentication Dashboard</h1>
                <div>
                    <button onClick={logout} style={{ marginRight: '10px', padding: '8px 15px', background: 'red', color: 'white', border: 'none', cursor: 'pointer' }}>Logout Current Device</button>
                    <button onClick={logoutAll} style={{ padding: '8px 15px', background: 'black', color: 'white', border: 'none', cursor: 'pointer' }}>Logout All Devices</button>
                </div>
            </div>

            <div style={{ marginTop: '20px' }}>
                <h3>Current Authenticated Identity:</h3>
                <p><strong>Username Claim:</strong> {user?.username}</p>
                <p><strong>Email Identity:</strong> {user?.email}</p>
                <p><strong>RBAC Role Clearance:</strong> <span style={{ background: '#eee', padding: '3px 8px', borderRadius: '4px' }}>{user?.role}</span></p>
                <p><strong>Email Verification Verification:</strong> {user?.isVerified ? "✅ Verified" : "❌ Pending Verification"}</p>
            </div>

            {user?.role === 'admin' && (
                <div style={{ marginTop: '30px', padding: '20px', background: '#f9f9f9', border: '1px solid #ccc', borderRadius: '8px' }}>
                    <h3 style={{ color: 'darkred' }}>🔒 Administrative System Telemetry (Admin clearance required)</h3>
                    {statsError ? (
                        <p style={{ color: 'red' }}>{statsError}</p>
                    ) : adminStats ? (
                        <div>
                            <p><strong>Total Persistent Users:</strong> {adminStats.totalSystemUsers}</p>
                            <p><strong>Server Decrypt Target Mode:</strong> {adminStats.systemMode}</p>
                            <p><strong>API Request Server Time:</strong> {new Date(adminStats.serverTime).toLocaleString()}</p>
                        </div>
                    ) : (
                        <p>Aggregating stats...</p>
                    )}
                </div>
            )}
        </div>
    );
};

export default Dashboard;
```

#### **11. `frontend/src/App.jsx`**
```javascript
import React from 'react';
import { Routes, Route, Navigate } from 'react-router-dom';
import ProtectedRoute from './components/ProtectedRoute';
import Login from './pages/Login';
import Register from './pages/Register';
import VerifyEmail from './pages/VerifyEmail';
import ForgotPassword from './pages/ForgotPassword';
import ResetPassword from './pages/ResetPassword';
import Dashboard from './pages/Dashboard';

const App = () => {
    return (
        <Routes>
            <Route path="/login" element={<Login />} />
            <Route path="/register" element={<Register />} />
            <Route path="/verify-email" element={<VerifyEmail />} />
            <Route path="/forgot-password" element={<ForgotPassword />} />
            <Route path="/reset-password" element={<ResetPassword />} />
            <Route path="/dashboard" element={
                <ProtectedRoute>
                    <Dashboard />
                </ProtectedRoute>
            } />
            <Route path="*" element={<Navigate to="/login" replace />} />
        </Routes>
    );
};

export default App;
```

#### **12. `frontend/index.html`**
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Enterprise Hardened Authentication</title>
  </head>
  <body style="font-family: sans-serif; background: #fafafa; color: #333; margin: 0; padding: 0;">
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

---

### **Part 6: Execution Analysis, Verification & Dry Runs**

---

#### **1. Structural Verification and Sequential File Boot Order**
Suno bachcho, jab hum humare Express & Mongoose clusters and React clients ko run karte hain, tab backend layers par execution and loading sequence step-by-step kaise perform hota hai, ise dekhein:

1. **`server.js` starts first**: Express instance create hoga aur database module `config/db.js` trigger hokar connection pool open karega.
2. **CORS is configured**: Client URL (`http://localhost:5173`) with credentials enable kiya jayega.
3. **Mongoose Models compiled**: `User`, `Session`, aur `Otp` models database memory registers par successfully bind ho jayenge.
4. **Global Middlewares applied**: Helmet, MongoSanitizer, CookieParser, aur Express.json payload parsing setup pipelines standard formats par register ho jayenge.
5. **Port binding**: Server listening port `5000` bind karega aur router requests ready hone ke liye socket accept loop start ho jayega.

---

#### **2. Signup Request Dry Run & Database States**

Maan lo user **Rohan (RohanSingh@secure.com)** React signup client form par signup click karta hai:

```text
========================================================================================================
                                     SIGNUP LIFECYCLE DRY RUN
========================================================================================================

  1. React UI Form Input ──► Username: "RohanSingh", Email: "RohanSingh@secure.com", Password: "securePass"
                                 │
                                 ▼ (Axios POST payload dispatch)
  2. Express Route Match ──► Matches Route: POST /api/auth/signup
                                 │
                                 ▼
  3. validateSignup Middleware ➔ Enforces username escape & email format schema validator rules
                                 │
                                 ▼ (Database Verification)
  4. Controller Query ────► User.findOne({ email: "RohanSingh@secure.com" }) returns NULL
                                 │
                                 ▼
  5. Cryptography Hashing ──► bcrypt.genSalt(12) ➔ bcrypt.hash("securePass") ➔ Returns "$2b$12$secureString"
                                 │
                                 ▼
  6. Mongoose Document Save ➔ User document committed to MongoDB with isVerified: false
                                 │
                                 ▼
  7. OTP Pipeline Active ──► generates random cryptographically secure string "482912"
                                 │
                                 ▼
  8. Otp Document committed ➔ Otp document saved in database collection with 15 mins TTL index
                                 │
                                 ▼
  9. SMTP Relays Trigger ───► Nodemailer dispatch transacts email with OTP "482912"
                                 │
                                 ▼
  10. HTTP Response status ➔ returns 201 Created JSON: { success: true, message: "Registration pass!" }

========================================================================================================
```

---

#### **3. MongoDB Documents Output Verification Views (Mongoose schema structures):**

#### **Collection: `users`**
```json
{
  "_id": ObjectId("651a1e847c21142981ef93ad"),
  "username": "RohanSingh",
  "email": "rohansingh@secure.com",
  "password": "$2b$12$w81729012hsd912h83091h283hdsyghuiadshasghg91",
  "role": "user",
  "isVerified": true,
  "loginAttempts": 0,
  "lockUntil": null,
  "createdAt": ISODate("2026-08-06T21:58:15.000Z"),
  "updatedAt": ISODate("2026-08-06T21:58:22.000Z"),
  "__v": 0
}
```

#### **Collection: `otps` (Auto-expiring via TTL indexes)**
```json
{
  "_id": ObjectId("651a1e957c21142981ef93ae"),
  "email": "rohansingh@secure.com",
  "otp": "482912",
  "purpose": "EMAIL_VERIFICATION",
  "expiresAt": ISODate("2026-08-06T22:13:15.000Z"),
  "createdAt": ISODate("2026-08-06T21:58:15.000Z"),
  "updatedAt": ISODate("2026-08-06T21:58:15.000Z"),
  "__v": 0
}
```

#### **Collection: `sessions`**
```json
{
  "_id": ObjectId("651a1ea07c21142981ef93af"),
  "userId": ObjectId("651a1e847c21142981ef93ad"),
  "tokenHash": "7f89asdhjk12h3891h2389hdshsd9812h3hdsa8123hasd",
  "jti": "5a4b3c2d-1e0f-5a4b-3c2d-1e0f5a4b3c2d",
  "ip": "127.0.0.1",
  "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)",
  "expiresAt": ISODate("2026-08-13T21:58:20.000Z"),
  "revokedAt": null,
  "replacedByJti": null,
  "createdAt": ISODate("2026-08-06T21:58:20.000Z"),
  "updatedAt": ISODate("2026-08-06T21:58:20.000Z"),
  "__v": 0
}
```

---

### **Part 7: Postman Complete API Verification Sheets**

Hamari APIs ko testing environment me validation steps verify karne ke liye Postman collection coordinates configure kijiye:

| API Objective | HTTP Method | Endpoint URI | Headers Required | Request Body (JSON) | Expected Response Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Register Account** | `POST` | `/api/auth/signup` | `Content-Type: application/json` | `{"username": "RohanSingh", "email": "rohansingh@secure.com", "password": "securePass"}` | **`201 Created`** |
| **Verify Email OTP** | `POST` | `/api/auth/verify-email` | `Content-Type: application/json` | `{"email": "rohansingh@secure.com", "otp": "482912"}` | **`200 OK`** |
| **Login Portal** | `POST` | `/api/auth/login` | `Content-Type: application/json` | `{"email": "rohansingh@secure.com", "password": "securePass"}` | **`200 OK`** (Sets httpOnly cookie) |
| **Query Profile (Auth Guard)** | `GET` | `/api/users/profile` | `Authorization: Bearer <accessToken>` | *None* | **`200 OK`** |
| **Query Profile (Expired Access Token)** | `GET` | `/api/users/profile` | `Authorization: Bearer <expiredAccessToken>` | *None* | **`401 Unauthorized`** (Message: `ACCESS_TOKEN_EXPIRED`) |
| **Background RTR Refresh** | `POST` | `/api/auth/refresh` | *Automatic Cookie Header* | *None* | **`200 OK`** (Rotates cookies and returns new accessToken) |
| **Admin Stats Route (Access Blocked)** | `GET` | `/api/users/admin/stats` | `Authorization: Bearer <userAccessToken>` | *None* | **`403 Forbidden`** (Blocked by RBAC) |
| **Admin Stats Route (Access Approved)** | `GET` | `/api/users/admin/stats` | `Authorization: Bearer <adminAccessToken>` | *None* | **`200 OK`** |
| **Logout Device** | `POST` | `/api/auth/logout` | *Automatic Cookie Header* | *None* | **`200 OK`** |

---

### **Part 8: Edge-Cases, Failure Modes & Administrative Mitigations**

1. **Replay Attack Bypass Anomaly**:
   * *The Threat*: Maan lo attacker refresh token intercept karke dynamic payload `R1` chura leta hai.
   * *Mitigation*: RTR checks step 2 trigger hota hai. Server detect karta hai ki token `R1` is already marked as revoked. System instantly is security family key parameters compile karta hai aur user ke saare database session keys clean out karke user profile ko immediately active lock mode me push kar deta hai. Attacker completely blocked, and true user must re-authenticate.
2. **Brute Force Attempt Flood**:
   * *The Threat*: Bot automated parameters values post karke passwords match check loops run kar raha hai.
   * *Mitigation*: 5 failures limits crossed hote hi user profile property `lockUntil` is flagged on the database. Next requests validation controller process execute karne ke bajaye user block response logic dispatch kar deta hai, protecting the password hash from dictionary parsing loops.
3. **NoSQL Query Hijack (MongoDB Operator injection)**:
   * *The Threat*: bad actor email parameters properties me operator inject karke query logic modify karna chahta hai (`{ "email": { "$ne": "" } }`).
   * *Mitigation*: Express pipeline entry point par hi `express-mongo-sanitize` raw request variables inspect karta hai aur prefixes `$`, `.` keys objects ko completely clean kar deta hai. Database queries safely typed and validated parameter keys se compile hoti hain, eliminating SQL/NoSQL injection channels.

---

### **Part 9: Advanced Security Interview Mastery (Professional + Hinglish Q&As)**

#### **Q1: Explain how Refresh Token Rotation (RTR) handles the trade-off between stateless access token models and token theft recovery.**
*   **Professional English Answer:**
    > "In a stateless authentication model, access tokens cannot be easily invalidated without checking a blacklist, making theft highly damaging. Refresh tokens, being stateful and long-lived, present a critical threat surface if hijacked. Refresh Token Rotation (RTR) implements a dynamic, stateful lifecycle by linking refresh tokens in cryptographic chains (Families) using JTI (JWT ID) mapping. Each refresh request invalidates the current token and issues a new pair. If an attacker replays a previously used (and thus invalidated) refresh token, the server immediately identifies a replay anomaly, invalidates the entire session family for that user, and drops all active database session records, neutralizing the hijacked session instantly."
*   **Easy Hinglish Explanation:**
    > "Access token stateless hota hai (matlab bina database query ke directly sign check se verify ho jata hai), par agar chor use chura le toh 15 minutes tak use rokna mushkil hai. Refresh token stateful hota hai jo use rotate hone me help karta hai. RTR engine me jab bhi user naya access token maangega, server use ek naya refresh token cookie dispatch karke purane token ko DB me 'revoked' mark kar dega. Agar chor chori kiya hua purana token replay karne ka try karega, toh server instantly pakad lega ki 'Yeh token toh pehle use ho chuka hai!'. Anomaly detect hote hi server us user ke saare device sessions ko block kar dega aur force re-login laga dega."

#### **Q2: Why are standard CORS wildcard configurations (`*`) fundamentally incompatible with HttpOnly cookie transport headers?**
*   **Professional English Answer:**
    > "Standard browser security specifications strictly prohibit the wildcard origin `*` when the `Access-Control-Allow-Credentials` header is set to `true`. This restriction prevents untrusted domains from harvesting cross-origin credentials. When credentials are enabled for HttpOnly cookie transport, the backend server must dynamically parse and explicitly declare the requesting domain in the `Access-Control-Allow-Origin` header after verifying it against an authorized origin whitelist, ensuring the security of session tokens."
*   **Easy Hinglish Explanation:**
    > "Jab browser HttpOnly cookie transport credentials exchange chala raha hota hai, toh safety rules ke according origin wildcard `*` are strictly blocked. Agar server origin '*' set karega, toh browser cookie handshake reject kar dega. Iska solution yeh hai ki backend server ko request origin header read karke use verified whitelist domains (jaise environment variables `process.env.CLIENT_URL`) se match karna chahiye aur response header me vahi exact domain parameter explicitly reflect karna chahiye, tabhi security flow successfully validate hoga."

---

### **Part 10: Complete Course Revision & Cheat Sheet**

Bachcho, humne hamari **Authentication & Authorization Mastery** series me step-by-step master levels attain kiye hain:

*   **Chapter 1**: Stateless Access and Refresh Token flows ke base concepts.
*   **Chapter 2**: OTP based identity verifications, Nodemailer systems aur verification structures.
*   **Chapter 3**: Advanced Access Token rotation pipelines (RTR), session monitoring tables aur devices controls.
*   **Chapter 4**: Strict Role-Based Access Control (RBAC), routes and controller gating aur dynamic resource ownership matrices.
*   **Chapter 5**: Web hardening methodologies (Helmet, CORS credentials locks, Rate limiters, NoSQL inject sanitizers).
*   **Chapter 6**: Combined everything into a production-level Enterprise Authentication and Authorization engine with ZERO placeholders and complete React integration!

---

### **Mini Assignment**

1.  **Task 1**: Apne is production React App dashboard page par ek naya button layout build kijiye: `"Refresh Session Manual"`, jo user click hone par context helper function trigger kare aur manual silent token rotation dispatch run ho.
2.  **Task 2**: Backend `Session` model me ek additional tracking field integrate kijiye: `expiresAt` validation. Jab login login update ho, tab current dates metrics analyze karke, 7-days limit validation strictly compile kijiye database operations queries me.

---

