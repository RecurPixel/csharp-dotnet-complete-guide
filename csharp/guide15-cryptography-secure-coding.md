# C# Cryptography and Secure Coding Quick Reference

---

## Overview

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    CRYPTOGRAPHY DECISION MAP                             │
│                                                                          │
│  GOAL                   USE                          AVOID               │
│  ─────────────────────────────────────────────────────────────────────  │
│  Hash data (integrity)  SHA-256 / SHA-512            MD5, SHA-1          │
│  Hash passwords         PBKDF2 / Argon2 / bcrypt     SHA-256 alone       │
│  Verify data integrity  HMAC-SHA256                  rolling your own    │
│  Encrypt data           AES-GCM                      AES-ECB, DES        │
│  Sign/verify data       ECDsa / RSA-PSS              RSA-PKCS1 (old)     │
│  Random secrets         RandomNumberGenerator        System.Random       │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 1. Hashing vs Password Hashing

### Hashing — Data Integrity (Fast, Deterministic)

```csharp
using System.Security.Cryptography;
using System.Text;

// SHA-256: one-way fingerprint of data
static byte[] HashSha256(string input)
{
    byte[] bytes = Encoding.UTF8.GetBytes(input);
    return SHA256.HashData(bytes);   // .NET 5+ static helper
}

// SHA-512 for larger output
static byte[] HashSha512(string input)
{
    byte[] bytes = Encoding.UTF8.GetBytes(input);
    return SHA512.HashData(bytes);
}

// Hex string output
static string ToHex(byte[] data)
    => Convert.ToHexString(data);    // .NET 5+ — lowercase: .ToLower()
```

> ⚠️ **Pitfall:** SHA-256 is fast by design. Never use it alone for passwords — attackers can compute billions of hashes per second using GPUs.

### Password Hashing — Authentication (Slow, Salted)

**Decision Table: Password Hashing Options**

| Algorithm | NuGet Required | Resistance | Recommendation |
|-----------|---------------|------------|----------------|
| PBKDF2    | Built-in      | Moderate   | ✅ Good default |
| bcrypt    | BCrypt.Net-Next | High     | ✅ Widely used  |
| Argon2    | Konscious.Security.Cryptography | Highest | ✅ Best modern choice |
| SHA-256 alone | Built-in | None     | ❌ Never use    |

```csharp
using System.Security.Cryptography;

// PBKDF2 — built-in, no NuGet needed
static string HashPassword(string password)
{
    byte[] salt = RandomNumberGenerator.GetBytes(16);
    byte[] hash = Rfc2898DeriveBytes.Pbkdf2(
        password,
        salt,
        iterations: 310_000,          // OWASP 2023 recommendation
        HashAlgorithmName.SHA256,
        outputLength: 32
    );
    // Store: "salt:hash" as base64
    return $"{Convert.ToBase64String(salt)}:{Convert.ToBase64String(hash)}";
}

static bool VerifyPassword(string password, string stored)
{
    var parts = stored.Split(':');
    byte[] salt = Convert.FromBase64String(parts[0]);
    byte[] storedHash = Convert.FromBase64String(parts[1]);

    byte[] inputHash = Rfc2898DeriveBytes.Pbkdf2(
        password, salt, 310_000, HashAlgorithmName.SHA256, 32
    );
    return CryptographicOperations.FixedTimeEquals(inputHash, storedHash);
}
```

```csharp
// bcrypt — install BCrypt.Net-Next
using BCrypt.Net;

string hashed = BCrypt.HashPassword(plaintext, workFactor: 12);
bool valid   = BCrypt.Verify(plaintext, hashed);
```

---

## 2. HMAC — Data Authentication

HMAC combines a secret key with a hash to prove both integrity and authenticity.

```csharp
using System.Security.Cryptography;
using System.Text;

static byte[] ComputeHmac(string data, byte[] secretKey)
{
    byte[] bytes = Encoding.UTF8.GetBytes(data);
    using var hmac = new HMACSHA256(secretKey);
    return hmac.ComputeHash(bytes);
}

static bool VerifyHmac(string data, byte[] secretKey, byte[] expectedMac)
{
    byte[] actual = ComputeHmac(data, secretKey);
    // Constant-time compare — never use == or SequenceEqual for MACs
    return CryptographicOperations.FixedTimeEquals(actual, expectedMac);
}

// Generate a proper HMAC key
byte[] hmacKey = RandomNumberGenerator.GetBytes(32);
```

> ✅ **Use HMAC when:** signing API payloads, webhook signatures, or anti-tamper tokens.

---

## 3. AES-GCM Encryption / Decryption

AES-GCM is authenticated encryption — it provides confidentiality **and** integrity in one operation.

```csharp
using System.Security.Cryptography;

const int KeySize  = 32;   // 256-bit key
const int NonceSize = 12;  // GCM standard nonce
const int TagSize   = 16;  // Authentication tag

// Encrypt
static (byte[] ciphertext, byte[] nonce, byte[] tag) Encrypt(
    byte[] plaintext, byte[] key)
{
    byte[] nonce      = RandomNumberGenerator.GetBytes(NonceSize);
    byte[] ciphertext = new byte[plaintext.Length];
    byte[] tag        = new byte[TagSize];

    using var aes = new AesGcm(key, TagSize);
    aes.Encrypt(nonce, plaintext, ciphertext, tag);

    return (ciphertext, nonce, tag);
}

// Decrypt
static byte[] Decrypt(
    byte[] ciphertext, byte[] key, byte[] nonce, byte[] tag)
{
    byte[] plaintext = new byte[ciphertext.Length];
    using var aes = new AesGcm(key, TagSize);
    aes.Decrypt(nonce, ciphertext, tag, plaintext);  // throws if tag invalid
    return plaintext;
}

// Key generation
byte[] key = RandomNumberGenerator.GetBytes(KeySize);
```

### Packaging for Storage / Transport

```csharp
// Combine nonce + tag + ciphertext into one blob
static byte[] Pack(byte[] nonce, byte[] tag, byte[] ciphertext)
{
    byte[] blob = new byte[nonce.Length + tag.Length + ciphertext.Length];
    nonce.CopyTo(blob, 0);
    tag.CopyTo(blob, nonce.Length);
    ciphertext.CopyTo(blob, nonce.Length + tag.Length);
    return blob;
}

static (byte[] nonce, byte[] tag, byte[] ciphertext) Unpack(byte[] blob)
{
    byte[] nonce      = blob[..NonceSize];
    byte[] tag        = blob[NonceSize..(NonceSize + TagSize)];
    byte[] ciphertext = blob[(NonceSize + TagSize)..];
    return (nonce, tag, ciphertext);
}
```

> ❌ **Pitfall:** Never reuse a nonce with the same key in AES-GCM. Nonce reuse completely breaks confidentiality.

---

## 4. Signing and Verification

### ECDsa (Recommended — Smaller Keys, Faster)

```csharp
using System.Security.Cryptography;
using System.Text;

// Key generation
using ECDsa ecdsa = ECDsa.Create(ECCurve.NamedCurves.nistP256);

// Export keys
byte[] privateKey = ecdsa.ExportPkcs8PrivateKey();
byte[] publicKey  = ecdsa.ExportSubjectPublicKeyInfo();

// Sign
static byte[] Sign(byte[] data, byte[] privateKeyBytes)
{
    using ECDsa key = ECDsa.Create();
    key.ImportPkcs8PrivateKey(privateKeyBytes, out _);
    return key.SignData(data, HashAlgorithmName.SHA256);
}

// Verify
static bool Verify(byte[] data, byte[] signature, byte[] publicKeyBytes)
{
    using ECDsa key = ECDsa.Create();
    key.ImportSubjectPublicKeyInfo(publicKeyBytes, out _);
    return key.VerifyData(data, signature, HashAlgorithmName.SHA256);
}
```

### RSA Signing (when RSA is required)

```csharp
using System.Security.Cryptography;

using RSA rsa = RSA.Create(keySizeInBits: 2048);

// Sign with RSA-PSS (prefer over PKCS1)
static byte[] RsaSign(byte[] data, RSA key)
    => key.SignData(data, HashAlgorithmName.SHA256, RSASignaturePadding.Pss);

static bool RsaVerify(byte[] data, byte[] sig, RSA key)
    => key.VerifyData(data, sig, HashAlgorithmName.SHA256, RSASignaturePadding.Pss);
```

---

## 5. Secure Random Number Generation

```csharp
using System.Security.Cryptography;

// Cryptographically secure random bytes
byte[] randomBytes = RandomNumberGenerator.GetBytes(32);

// Secure random integer in [0, max)
static int SecureRandomInt(int max)
    => RandomNumberGenerator.GetInt32(max);

// Secure random integer in [min, max)
static int SecureRandomRange(int min, int max)
    => RandomNumberGenerator.GetInt32(min, max);

// Generate a URL-safe token (e.g., API key, reset token)
static string GenerateToken(int byteLength = 32)
{
    byte[] bytes = RandomNumberGenerator.GetBytes(byteLength);
    return Convert.ToBase64String(bytes)
        .Replace("+", "-").Replace("/", "_").TrimEnd('=');
}
```

> ❌ **Never use `System.Random` for security.** It is seeded with the clock and is predictable.

---

## 6. Key, Nonce, and IV Handling — Do's and Don'ts

| Topic | ✅ Do | ❌ Don't |
|-------|-------|---------|
| Key generation | `RandomNumberGenerator.GetBytes(32)` | Hard-code keys in source |
| Nonce/IV | Always fresh random per operation | Reuse nonce/IV with same key |
| Key storage | OS keychain / Azure Key Vault / env vars | Commit to Git |
| Key length | AES-256 (32 bytes), HMAC ≥ 32 bytes | AES-128 for sensitive data |
| Nonce size | 12 bytes for AES-GCM | Custom sizes without spec |
| Key rotation | Implement versioned key envelopes | Use same key forever |

```csharp
// ❌ NEVER do this
private static readonly byte[] Key = Encoding.UTF8.GetBytes("mysecretkey12345");

// ✅ Load from environment or secret store
private static readonly byte[] Key =
    Convert.FromBase64String(Environment.GetEnvironmentVariable("APP_ENC_KEY")!);
```

---

## 7. Constant-Time Comparisons

```csharp
// ❌ WRONG — timing side-channel leaks information
bool equal = userMac == storedMac;                          // string ==
bool equal = userBytes.SequenceEqual(storedBytes);          // short-circuits

// ✅ CORRECT — always takes same time regardless of difference point
bool equal = CryptographicOperations.FixedTimeEquals(userBytes, storedBytes);
```

> ⚠️ **Why it matters:** Early-exit comparisons let attackers measure response time byte-by-byte to forge MACs or tokens without the key.

---

## 8. Data Protection API (ASP.NET Core / Windows Secrets)

For protecting short-lived secrets on the same machine:

```csharp
// Install: Microsoft.AspNetCore.DataProtection
using Microsoft.AspNetCore.DataProtection;

IDataProtectionProvider provider = DataProtectionProvider.Create("MyApp");
IDataProtector protector = provider.CreateProtector("UserTokens");

string token    = protector.Protect("sensitive-payload");
string original = protector.Unprotect(token);

// Time-limited tokens
ITimeLimitedDataProtector timedProtector =
    protector.ToTimeLimitedDataProtector();

string timed = timedProtector.Protect("payload", TimeSpan.FromHours(1));
string back  = timedProtector.Unprotect(timed);  // throws after 1 hour
```

---

## 9. Recipes

### Recipe 1: File Integrity Checksum

```csharp
static string FileChecksum(string path)
{
    using var fs = File.OpenRead(path);
    byte[] hash = SHA256.HashData(fs);
    return Convert.ToHexString(hash);
}
```

### Recipe 2: Encrypt a String to Base64

```csharp
static string EncryptString(string plaintext, byte[] key)
{
    byte[] data = Encoding.UTF8.GetBytes(plaintext);
    var (cipher, nonce, tag) = Encrypt(data, key);
    return Convert.ToBase64String(Pack(nonce, tag, cipher));
}

static string DecryptString(string blob, byte[] key)
{
    byte[] packed = Convert.FromBase64String(blob);
    var (nonce, tag, cipher) = Unpack(packed);
    byte[] plain = Decrypt(cipher, key, nonce, tag);
    return Encoding.UTF8.GetString(plain);
}
```

### Recipe 3: Webhook Signature Verification

```csharp
static bool VerifyWebhook(string payload, string signatureHeader, byte[] secret)
{
    // signatureHeader = "sha256=<hex>"
    if (!signatureHeader.StartsWith("sha256=")) return false;
    byte[] expected = Convert.FromHexString(signatureHeader[7..]);
    byte[] actual   = ComputeHmac(payload, secret);
    return CryptographicOperations.FixedTimeEquals(actual, expected);
}
```

### Recipe 4: Derive a Sub-Key from Master Key

```csharp
static byte[] DeriveKey(byte[] masterKey, string purpose, int outputBytes = 32)
{
    byte[] label = Encoding.UTF8.GetBytes(purpose);
    return Rfc2898DeriveBytes.Pbkdf2(masterKey, label, 1, HashAlgorithmName.SHA256, outputBytes);
}

byte[] encKey = DeriveKey(masterKey, "encryption");
byte[] macKey = DeriveKey(masterKey, "authentication");
```

---

## Anti-Patterns / Common Crypto Mistakes Checklist

```
❌ Using MD5 or SHA-1 for any new work
❌ Using SHA-256 alone to hash passwords
❌ Reusing nonces/IVs — catastrophic for AES-GCM
❌ Using ECB mode (AES-ECB reveals patterns in data)
❌ Hard-coding keys or secrets in source code
❌ Using == or SequenceEqual for MAC/signature comparison
❌ Rolling your own crypto algorithm or protocol
❌ Using System.Random for token/key generation
❌ Storing plaintext passwords anywhere
❌ Using RSA-PKCS1v1.5 padding (use PSS or OAEP)
❌ Ignoring authentication tags in AES-GCM decryption
❌ Generating RSA keys < 2048 bits (prefer 4096 for long-lived)
```

---

## Best Practices

```
✅ Use AesGcm for symmetric encryption (authenticated by default)
✅ Use PBKDF2 / bcrypt / Argon2 for passwords — never raw hashes
✅ Always use CryptographicOperations.FixedTimeEquals for secret comparisons
✅ Generate nonces fresh per encrypt call using RandomNumberGenerator
✅ Store keys in environment variables, Key Vault, or OS credential store
✅ Derive separate keys per purpose from a master key
✅ Validate the authentication tag on decrypt — AesGcm throws on failure
✅ Use ECDsa over RSA when possible (smaller, faster, equally secure)
✅ Keep algorithm choices in one place (a crypto constants file)
✅ Log failed decryption / verification attempts as security events
```

---

## Quick Reference Summary

| Need | API / Package | Key Size |
|------|--------------|----------|
| Hash data | `SHA256.HashData()` | N/A |
| Hash password | `Rfc2898DeriveBytes.Pbkdf2` / `BCrypt` | salt ≥ 16 bytes |
| Verify MAC | `HMACSHA256` + `FixedTimeEquals` | 32 bytes |
| Encrypt | `AesGcm` | 32 bytes (AES-256) |
| Sign | `ECDsa` (P-256) | curve-based |
| Random bytes | `RandomNumberGenerator.GetBytes()` | as needed |
| Secure int | `RandomNumberGenerator.GetInt32()` | N/A |
| App secrets | `IDataProtector` | managed |

---

**Guide Complete!** These patterns cover 95% of cryptographic needs in production C# applications. When in doubt, prefer authenticated encryption (AES-GCM) and avoid inventing protocols. 🔐
