# Information Security Labwork — Classical & Modern Cryptography Demonstration

**Course:** Information Security
**Type:** Lab Coursework
**Deliverable:** Single-page web application (`index.html`)

## Overview

This lab implements three cryptographic mechanisms in a single, self-contained web page:

1. **Rail Fence Cipher** — a classical transposition cipher
2. **RSA Encryption** — modern public-key (asymmetric) encryption
3. **MD5 / SHA-256 Hashing** — applied to a login form that stores only hashed passwords, with a verification step

The page is a static HTML/CSS/JavaScript file with no server-side component, so it can be hosted on any free static host and opened directly in a browser.

## Live Demo

`https://sabinonweb.github.io/ISProject/`

## How to Run Locally

No installation or build step is required.

1. Download `index.html`
2. Open it directly in any modern browser (Chrome, Firefox, Edge)

All cryptographic work happens client-side in JavaScript.

## 1. Rail Fence Cipher

### What it is
A **transposition cipher** — it does not substitute characters, it only rearranges them. The plaintext is written diagonally across a fixed number of "rails" (rows) in a zig-zag pattern, then read off row by row to produce the ciphertext.

### Example
Plaintext `DEFEND THE EAST WALL` written on 3 rails:

```
D...E.....E....W...
.E.D.T.E.A.T.A.L....
..F...H...S...L.....
```

Reading each row left to right and concatenating gives the ciphertext.

### Implementation notes
- The number of rails is configurable (2–20) in the UI.
- Encryption walks the plaintext, bouncing an index up and down between rail `0` and rail `n-1`, appending each character to its corresponding rail.
- Decryption reconstructs the same zig-zag index pattern first, uses it to work out how many characters belong to each rail, slices the ciphertext into those rail-sized chunks, then reassembles the plaintext by walking the same zig-zag order.
- The page also renders the zig-zag grid visually so the transposition pattern is easy to verify by eye.

### Security relevance
Rail Fence is included to demonstrate a **classical, pre-computer cipher** for comparison against RSA. It is trivially breakable (only `n-1` useful key values, and can be brute-forced or solved via anagramming) and is used here purely for educational contrast, not as a serious security mechanism.

## 2. RSA Encryption

### What it is
An **asymmetric (public-key) cipher**. A public/private keypair is generated; anyone with the public key can encrypt a message, but only the holder of the matching private key can decrypt it.

### Implementation notes
- Uses the [`node-forge`](https://github.com/digitalbazaar/forge) library (loaded from cdnjs) for real RSA key generation and encryption — not a simulated/toy implementation.
- Keypairs can be generated at 512, 1024, or 2048 bits (512/1024 are offered for demo speed; 2048+ is the minimum recommended size in real deployments).
- Encryption uses **RSA-OAEP** padding rather than the outdated and insecure PKCS#1 v1.5 padding.
- Keys are displayed in PEM format directly on the page.
- All key generation and encryption/decryption happens **entirely in the browser** — nothing is sent to a server, and the private key never leaves the client.

### Flow demonstrated
1. Generate keypair → public key + private key (PEM) shown
2. Encrypt a plaintext message with the **public** key → Base64 ciphertext
3. Decrypt the ciphertext with the **private** key → original plaintext recovered

### Security relevance
RSA is the basis of much of modern secure communication (TLS/HTTPS certificate exchange, digital signatures, secure email). This exhibit shows the core public-key property: encryption and decryption use *different* keys, and possession of the public key alone is not sufficient to reverse the operation.

## 3. MD5 / SHA-256 Hashing — Login Form

### What it demonstrates
The lab requirement is:
- Build a login form that asks for a username and password
- Ensure the password is **never stored in plaintext** — only its hash (MD5 or SHA-256) is stored
- Verify a login attempt by re-hashing the supplied password and comparing it against the stored hash

### Implementation notes
- Uses [`crypto-js`](https://github.com/brix/crypto-js) (loaded from cdnjs) for MD5 and SHA-256 hashing.
- A dropdown lets you choose which algorithm is used for a given registration.
- **Registration:** username + password are submitted → the password is hashed immediately in the browser → only `{ username, hash, algorithm }` is written to the "database." The plaintext password is discarded right after hashing and is never stored anywhere.
- **Login verification:** the submitted password is hashed the same way and compared against the stored hash for that username. A match grants access; a mismatch is rejected. At no point is a stored password compared directly — only hash values are compared.
- The "database" table is rendered live on the page so the grader can see exactly what a real `users` table would contain: `username`, `algorithm`, `password_hash` — never a plaintext password.

### Important caveat (by design, for coursework transparency)
This demo intentionally uses **raw MD5/SHA-256** because that is what the assignment specifies. In a real production system, this is **not sufficient** on its own:
- MD5 is cryptographically broken (collision attacks) and should never be used for password storage.
- Even SHA-256, while cryptographically strong, is a *fast* hash — this makes it practical for an attacker to brute-force or precompute (rainbow-table) large batches of possible passwords once a hash is leaked.
- A production system should use a **slow, salted** password hashing algorithm designed for this purpose, such as **bcrypt**, **scrypt**, or **Argon2**, along with a unique random salt per user to defeat rainbow tables.

This tradeoff is called out directly in the UI as a note, so it's clear the simplification was made deliberately for the lab's specified requirements, not out of oversight.

### Data storage note
Because this is a static, no-backend page, the "database" is a JavaScript array held in memory for the duration of the page session (it resets on reload). This stands in for what would be a real database table in a full-stack version of the same login flow — the hashing and verification logic itself is identical to what a server-side implementation would do.

## Technology Stack

| Component | Library / Tool |
|---|---|
| RSA key generation & encryption | [node-forge](https://github.com/digitalbazaar/forge) v1.3.1 |
| MD5 / SHA-256 hashing | [crypto-js](https://github.com/brix/crypto-js) v4.2.0 |
| Rail Fence Cipher | Custom implementation (vanilla JavaScript) |
| UI | Vanilla HTML/CSS/JavaScript — no framework, no build step |

## File Structure

```
index.html   # Everything: markup, styling, and all cryptographic logic
```

## Limitations / Scope

- No backend or persistent database — this is a client-side demonstration of the cryptographic mechanisms, per the lab's scope.
- RSA key sizes below 2048 bits and raw MD5/SHA-256 password hashing are used deliberately for demonstration and speed; they are **not** recommended for production use, as noted above.
- The Rail Fence cipher currently supports uppercase/lowercase text and punctuation as-is (no character filtering) — spaces and symbols are transposed along with letters.

## Author

Coursework submission for Information Security lab.
