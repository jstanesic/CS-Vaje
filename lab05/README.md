# Identifying MITM attacks on GPG keys (Web of Trust)

## 🎯 Exercise goal
The purpose of the exercise is to understand that:
- a public key in itself does not imply trust,
- MITM (Man-in-the-Middle) attacks can occur during key exchange,
- fingerprint verification is crucial,
- Web of Trust helps in detecting such attacks.

---

## 🧠 Short introduction
If an attacker manages to inject his public key instead of the real one, he can:
- read all encrypted messages,
- impersonate someone else,
- GPG cannot detect this without trust verification.

---

## 🧪 Scenario
Three people participate in the exercise:
- **Alice** – sender
- **Bob** – recipient
- **Mallory** – attacker (MITM)

Everything is done on **the same device**.

---

## 🔑 1) Key generation

Generate three GPG keys:

```bash
gpg --full-generate-key
```

Data (example):
- Alice: `alice@example.com`
- Bob: `bob@example.com`
- Mallory: `mallory@example.com`

Verify keys:
```bash
gpg --list-keys
```

---

## 🧾 2) Print fingerprints (very important)

```bash
gpg --fingerprint alice@example.com
gpg --fingerprint bob@example.com
gpg --fingerprint mallory@example.com
```

📌 Fingerprint is the only reliable way to verify a key.

---

## 🕵️ 3) MITM attack – key substitution

Mallory exports **her** public key and names it Bob's:

```bash
gpg --armor --export mallory@example.com > bob_pubkey.asc
```

Alice imports the key:
```bash
gpg --import bob_pubkey.asc
```

➡️ Alice believes she has Bob's key, but in fact she has Mallory's.

---

## 🔐 4) Alice encrypts the message

```bash
echo "Confidential message for Bob" > secret.txt
```

```bash
gpg --encrypt --recipient bob@example.com secret.txt
```

➡️ The message is encrypted with the wrong key.

---

## 👀 5) Mallory decrypts the message

```bash
gpg --decrypt secret.txt.gpg
```

✔ MITM attack is successful.

---

## 🚨 6) Attack detection – fingerprint verification

Bob sends Alice **correct fingerprint via another channel** (in person, phone).

Alice checks:
```bash
gpg --fingerprint bob@example.com
```

❌ Fingerprint does not match → MITM attack detected.

---

## 🛡️ 7) Web of Trust – trust setting

Alice sets trust on the verified key:

```bash
gpg --edit-key bob@example.com
```

In the console:
```text
trust
5
quit
```

---

## 🧠 Reflection
Answer:
1. Why doesn't GPG detect MITM attacks automatically?
2. What is a fingerprint and why is it important?
3. Why is email not a secure channel for exchanging keys?
4. How does the Web of Trust reduce the risk of MITM attacks?

---

## ⭐ Additional challenge

Signing Bob's key with Alice's key:
```bash
gpg --sign-key bob@example.com
```

Explain the difference between:
- trust
- signed key
- ultimate trust

---

## 📌 Summary
- Cryptography works properly as long as we trust the right key.
- Fingerprinting is the foundation of trust.
- Web of Trust helps detect MITM attacks.