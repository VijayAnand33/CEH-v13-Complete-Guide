# Module 20: Cryptography

## Tier 1: Core Concepts & Principles

### Cryptographic Security Triad & Foundations
* **Core Objectives:**
  * **Confidentiality:** Ensuring data is unreadable to unauthorized entities (Symmetric / Asymmetric Encryption).
  * **Integrity:** Ensuring data has not been altered or tampered with in transit or storage (Cryptographic Hash Functions, HMAC).
  * **Authentication:** Verifying the true identity of the sender, device, or system (Digital Signatures, Certificates).
  * **Non-Repudiation:** Preventing an entity from denying the authenticity of a sent message or executed transaction (Asymmetric Cryptography / PKI).
* **Encryption Types & Classification:**
  * **Symmetric Encryption (Secret Key):** A single shared secret key is used for both encryption and decryption ($C = E_k(P), P = D_k(C)$). Highly efficient for bulk data encryption (e.g., AES, DES, 3DES, RC4, Blowfish, Twofish, ChaCha20).
  * **Asymmetric Encryption (Public Key):** Uses mathematically linked key pairs—a Public Key for encryption/verification and a Private Key for decryption/signing (e.g., RSA, ECC, Diffie-Hellman, DSA, ElGamal).
  * **Cryptographic Hashing:** One-way mathematical algorithms converting arbitrary-length input into a fixed-length digest. Properties: Deterministic, preimage resistant, second preimage resistant, collision resistant (e.g., SHA-2, SHA-3, MD5, SHA-1).

---

## Tier 2: Technical Analysis & Mechanics

### Cryptographic Algorithms & Protocol Workflows

#### 1. Symmetric vs. Asymmetric Cryptography Mechanics
* **Advanced Encryption Standard (AES):** Symmetric block cipher operating on 128-bit blocks using substitution-permutation networks. Key sizes: 128 bits (10 rounds), 192 bits (12 rounds), 256 bits (14 rounds).
* **RSA (Rivest-Shamir-Adleman):** Asymmetric algorithm based on the mathematical difficulty of factoring the product of two large prime numbers ($n = p \times q$). Used for key exchange and digital signatures.
* **Diffie-Hellman (DH) / Elliptic Curve Diffie-Hellman (ECDH):** Asymmetric key-agreement protocol enabling two parties to establish a shared symmetric secret over an untrusted channel without transmitting the key itself.

#### 2. Message Integrity & Hash Mechanics
* **HMAC (Hash-Based Message Authentication Code):** Combines a cryptographic hash function with a secret shared key to verify both data integrity and authentication:
  $$\text{HMAC}(K, m) = H\Big((K' \oplus \text{opad}) \parallel H\big((K' \oplus \text{ipad}) \parallel m\big)\Big)$$
* **Collision Vulnerabilities:** MD5 (128-bit digest) and SHA-1 (160-bit digest) are cryptographically broken due to practical collision attacks and must not be used for security-sensitive integrity verification. Current standards mandate SHA-256 / SHA-512 (SHA-2) or SHA-3.

#### 3. Public Key Infrastructure (PKI) & Digital Certificates
* **X.509 Standard:** Defines the format of public key digital certificates containing Subject Name, Public Key, Issuer (CA) Signature, Validity Period, and Key Usage constraints.
* **Self-Signed Certificates vs. CA-Signed Certificates:**
  * *CA-Signed:* Validated against pre-installed root trust stores in browsers/OS.
  * *Self-Signed:* Signed by the creator's own private key; encrypts transport traffic (TLS) identically to CA-signed certificates, but triggers untrusted root warnings in client browsers due to lack of a trusted chain of custody.

#### 4. Disk & Data-at-Rest Encryption Mechanics
* **Full Disk / Volume Encryption (FDE):** Operates below the filesystem layer (e.g., VeraCrypt, BitLocker, LUKS) utilizing XTS-AES mode to encrypt sectors, file names, directory structures, and metadata transparently.
* **Dismount Mechanics:** Dismounting unloads the volume master key from active RAM and severs the virtual drive mount point, preventing unauthorized cold-boot memory dumps or plaintext retrieval.

---

## Tier 3: CLI, Tools & Framework Syntax

### Core Cryptographic Utilities & Frameworks
* **CyberChef:** Web-based cryptographic swiss-army knife for chaining encoding, decoding, multi-layer hashing, and cipher operations.
* **CryptoForge:** Encryption utility suite providing symmetric file and text encryption via passphrases.
* **VeraCrypt:** Open-source disk encryption software creating virtual encrypted containers and system partitions.
* **ShellGPT / OpenSSL:** Command-line tooling for executing terminal-based cryptographic functions, certificate signing, and checksum verification.

### Key Commands & Execution Syntax

```bash
# OpenSSL: Generate a 2048-bit RSA Private Key
openssl genrsa -out private.key 2048

# OpenSSL: Generate a Self-Signed X.509 Certificate (Valid 365 Days)
openssl req -new -x509 -key private.key -out certificate.crt -days 365

# OpenSSL: Symmetric AES-256-CBC File Encryption
openssl enc -aes-256-cbc -salt -in confidential.txt -out confidential.enc -k SecretPassphrase123

# OpenSSL: Symmetric AES-256-CBC File Decryption
openssl enc -d -aes-256-cbc -in confidential.enc -out decrypted.txt -k SecretPassphrase123

# Checksum & Hash Generation CLI
# Compute SHA-256 Hash of a File
sha256sum target_file.iso

# Compute MD5 Hash of a File
md5sum target_file.iso

# Compute HMAC using SHA256 and Secret Key
echo -n "Confidential Message" | openssl dgst -sha256 -hmac "SharedSecretKey"

# Base64 Encoding and Decoding
echo -n "Confidential Data" | base64
echo -n "Q29uZmlkZW50aWFsIERhdGE=" | base64 --decode

# VeraCrypt CLI: Mount Encrypted Container (Linux)
veracrypt --text --mount /path/to/volume.hc /media/secure_drive --password "StrongPass123!" --pim 0 --keyfiles "" --protect-hidden no

# VeraCrypt CLI: Dismount All Encrypted Volumes
veracrypt --text --dismount
```

---

## Tier 4: Real-World Scenarios & Countermeasures

### Enterprise Cryptographic Hardening Controls
* **Cipher Suite & Protocol Standards:**
  * Enforce TLS 1.3 (or modern TLS 1.2 configurations) across all web applications and APIs; deprecate SSL v2/v3, TLS 1.0, and TLS 1.1.
  * Disallow insecure cipher modes: Disable Electronic Codebook (ECB) mode due to pattern leakage; mandate Galois/Counter Mode (GCM) or Cipher Block Chaining (CBC) with HMAC.
* **Key Management & Storage:**
  * Implement Hardware Security Modules (HSMs) or cloud Key Management Services (AWS KMS, Azure Key Vault) for master key storage and cryptographic operations.
  * Enforce regular key rotation schedules and separate key-encrypting keys (KEKs) from data-encrypting keys (DEKs).
* **Certificate Authority & PKI Governance:**
  * Enforce Certificate Revocation Lists (CRLs) and Online Certificate Status Protocol (OCSP) Stapling to monitor revoked certificates in real time.
  * Implement Certificate Transparency (CT) logging and HTTP Strict Transport Security (HSTS) with preloading.
* **Data-at-Rest Protection:**
  * Deploy full-disk encryption (FDE) using AES-XTS on all portable endpoints and servers.
  * Enforce strong key derivation functions (PBKDF2, Argon2, bcrypt, scrypt) for password hashing and disk volume passphrases.

---

## Tier 5: Exam Key Hooks & Rapid Triggers

| Concept / Mechanism | High-Yield Exam Trigger | Critical Distinction |
| :--- | :--- | :--- |
| **Symmetric vs Asymmetric** | Symmetric = Single shared key (Fast, Bulk); Asymmetric = Public/Private key pair | Asymmetric solves key distribution problem; Symmetric handles fast bulk encryption |
| **AES (Advanced Encryption Standard)** | Symmetric block cipher; 128-bit block size; Key sizes: 128, 192, 256 bits | Replaced DES/3DES; standard for enterprise data-at-rest and in-transit |
| **Diffie-Hellman (DH)** | Key-exchange protocol allowing secret derivation over untrusted channel | Does not encrypt data directly; solely generates a shared symmetric secret |
| **Digital Signature Operation** | Sender encrypts hash with their **Private Key**; Receiver verifies with sender's **Public Key** | Provides Authentication, Integrity, and Non-Repudiation |
| **Digital Encryption Operation** | Sender encrypts data with receiver's **Public Key**; Receiver decrypts with their **Private Key** | Provides Confidentiality |
| **MD5 vs SHA-1 vs SHA-256** | MD5 = 128-bit digest; SHA-1 = 160-bit digest; SHA-256 = 256-bit digest | MD5/SHA-1 are vulnerable to collisions; SHA-2/SHA-3 are current secure standards |
| **HMAC** | Hash-Based Message Authentication Code combining hash function + secret key | Provides both Data Integrity and Data Origin Authentication |
| **ECB Mode (Electronic Codebook)** | Insecure symmetric block cipher mode; identical plaintext blocks yield identical ciphertext | Fails to obscure data patterns (e.g., ECB Penguin); never use in production |
| **Self-Signed Certificate Risk** | Triggers untrusted CA browser warnings; provides TLS encryption without identity trust | Lacks trust chain from commercial Root Certificate Authority |
| **VeraCrypt / FDE** | Volume-level encryption using XTS-AES mode | Encrypts full disk including free space, filenames, and file headers |
| **Base64 vs Encryption** | Base64 = Reversible encoding scheme (No key); Encryption = Secret key required | Encoding changes data format; Encryption ensures confidentiality |
| **Forward Secrecy (PFS)** | Compromise of long-term server private key does not compromise past session keys | Achieved via Ephemeral Diffie-Hellman (DHE / ECDHE) |