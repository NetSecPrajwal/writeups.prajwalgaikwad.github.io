---
title: "JWT Attacks: What They Are and Why They Still Work"
description: "A practical breakdown of common JWT vulnerabilities — algorithm confusion, weak secrets, and the none algorithm bypass."
category: Research
tags: [jwt, web-security, authentication, burp-suite]
---

JWTs are everywhere. And so are JWT vulnerabilities. Here's a practical breakdown of the attacks that still land in 2026.

## What is a JWT, Quickly

A JSON Web Token has three parts separated by dots:

```
header.payload.signature
```

Each part is base64url-encoded. The header says which algorithm was used to sign it. The payload carries the claims (user ID, role, expiry). The signature proves it wasn't tampered with.

The attacks all target one of three things: the algorithm, the secret, or the validation logic.

## 1. The `alg: none` Attack

Some libraries accept `none` as a valid algorithm — meaning no signature is required at all.

**The attack:**
1. Decode the JWT header
2. Change `"alg": "HS256"` to `"alg": "none"`
3. Remove the signature (keep the trailing dot)
4. Send it

```
eyJhbGciOiJub25lIn0.eyJ1c2VyIjoiYWRtaW4ifQ.
```

If the server accepts this — game over. Fortunately, modern libraries patch this. But legacy applications still run legacy libraries.

**Testing in Burp:**
Intercept the request containing the JWT → send to Repeater → modify manually or use the JWT Editor extension.

## 2. Weak Secret Brute Force

HS256 tokens are signed with a secret key. If that secret is weak, you can crack it offline.

```bash
hashcat -a 0 -m 16500 <jwt_token> /usr/share/wordlists/rockyou.txt
```

Or using `jwt_tool`:

```bash
python3 jwt_tool.py <token> -C -d /usr/share/wordlists/rockyou.txt
```

Once you have the secret, you can forge any token you want.

**What counts as a weak secret?**
Anything in rockyou.txt. Anything under 32 characters. Anything that looks like a placeholder (`secret`, `password`, `jwt_secret`). I've seen `secret123` in production more than once.

## 3. Algorithm Confusion (RS256 → HS256)

This one is subtle and powerful.

RS256 uses a **private key** to sign and a **public key** to verify. If an attacker can obtain the public key (often exposed at `/api/.well-known/jwks.json`), they can try switching the algorithm to HS256 and using the public key as the HMAC secret.

The server, if vulnerable, will try to verify an HS256 token using its public key — which the attacker already has.

**Steps:**
1. Grab the public key from the JWKS endpoint
2. Convert it to PEM format
3. Forge a token with `alg: HS256`, signed with the public key as the secret
4. Send it

Burp's JWT Editor extension has a built-in attack for this under "Embedded JWK."

## What Secure JWT Implementation Looks Like

- Explicitly whitelist accepted algorithms — never trust the header's `alg` value alone
- Use strong, randomly generated secrets (256-bit minimum for HS256)
- Set short expiry (`exp`) and validate it
- Use RS256/ES256 for distributed systems; HS256 only if the secret can be tightly controlled

---

*These attacks are well-documented at [PortSwigger Web Security Academy](https://portswigger.net/web-security/jwt) — the JWT labs there are worth doing hands-on.*
