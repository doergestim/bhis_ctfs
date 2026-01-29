**PKI Foundations Webcast**

**Symmetric Encryption & Shared Keys**

**Q1:**  Your team encrypts backups using AES-256 with the same static key stored in a script on every server for “ease of automation.” What’s the primary weakness in this design, and how should it be fixed?

**Answer:**\
` `The static key is the single point of failure — compromise of one system exposes all backups. Use a central key management service (KMS) or encrypt backups with unique data keys protected by a master key in a secure vault (e.g., AWS KMS, HashiCorp Vault). Rotate keys periodically.

**Q2:** An engineer claims “AES-256 is unbreakable, so we don’t need HTTPS.” Why is this incorrect?

**Answer:**\
` `Encryption strength ≠ secure key exchange. AES is symmetric; without a secure channel to share the key, an attacker can intercept it. HTTPS provides both encryption **and** secure key negotiation via TLS, which uses asymmetric cryptography to safely exchange session keys.

**Q3:** You must choose between AES-CBC and AES-GCM for encrypting API secrets. Which should you pick and why?

**Answer:**\
` `AES-GCM (Galois/Counter Mode) — it provides both confidentiality and integrity (authenticated encryption). AES-CBC only offers confidentiality and requires separate MAC handling. GCM is faster and less error-prone when implemented correctly.

**Asymmetric Encryption: Public & Private Keys**

**Q4:** A developer uses their private key to encrypt sensitive data and gives others the corresponding public key to decrypt it. What’s wrong with this approach?

Answer:\
They’ve reversed the roles. Encrypting with a private key exposes confidentiality — anyone with the public key can decrypt. Private keys should sign; public keys should verify. To protect confidentiality, encrypt with the recipient’s public key and decrypt with your private key.

**Q5:** Your company wants faster TLS performance and smaller key sizes while maintaining strong security. Should you choose RSA or ECC and why?

**Answer:**\
` `Choose ECC (Elliptic Curve Cryptography) — it achieves equivalent security with smaller keys and better performance. Example: a 256-bit ECC key ≈ 3072-bit RSA key in strength, reducing CPU and bandwidth overhead.

` `**Hashing & Data Integrity**

**Q6:** A web app stores user passwords using MD5. Why is this insecure, and what’s the proper alternative?

**Answer:** MD5 is fast, outdated, and prone to collisions. Attackers can brute-force or use rainbow tables easily. Fix: Use slow, adaptive password hashing functions like bcrypt, scrypt, or Argon2 with a unique salt per password.

**Q7:** Two files have identical SHA-1 hashes but different contents. What does this demonstrate, and what’s the impact?

**Answer**: This demonstrates a hash collision — SHA-1’s weakness. Impact: an attacker could substitute malicious files without changing the hash, undermining digital signatures or software verification. Use SHA-256 or stronger algorithms instead.

**Digital Signatures & Certificates**

**Q8:** Your team uses internal code signing with self-signed certs. After a developer leaves, you need to revoke their cert. How can you enforce revocation?

**Answer:** Maintain a Certificate Revocation List (CRL) or use Online Certificate Status Protocol (OCSP) in your internal PKI. Update endpoints to check revocation before trusting signatures.

**Public Key Infrastructure (PKI) in Practice**

**Q9:** A company’s web app cert expired over the weekend, causing downtime. What control can prevent this in the future?

**Answer:** Implement certificate monitoring and automation — use tooling like Let’s Encrypt, Certbot, or AWS ACM to auto-renew certs and alert on expirations. Expired certs break trust and availability; automation reduces human error.

**Q10:** A phishing domain uses a valid TLS cert from Let’s Encrypt. Why doesn’t HTTPS alone mean “safe”?

**Answer:** TLS only ensures encryption and domain control, not legitimacy.\
` `Phishers can obtain valid certs for fake domains (e.g., paypa1.com).\
` `Lesson: PKI secures the *connection*, not the *intent* — users must validate the domain name and source.

