# picoCTF — Cookie Monster's Secret Recipe

**Challenge name:** Cookie Monster's Secret Recipe

**Category:** Web / Misc (Login / Cookie decoding)

**Difficulty:** Easy(Very Easy)

**Author:** NIGHTFURY0X01 (Arash)

**Objective:** Find the hidden flag (the "secret recipe") on the web challenge.

---

## Summary

This challenge presents a simple HTML page with a login form that posts to `login.php`. The goal is to locate the hidden flag. The flag is ultimately discovered in a URL-encoded Base64 string that represents the secret recipe.

---


![Step 1](images/1.png)

## Steps to gather more information:

1. Open the page source (Ctrl+U) to check for any comments or hidden data.
2. Use browser DevTools (Network tab) and inspect requests when interacting with the page.
3. Inspect cookies and responses for encoded or otherwise suspicious values.

![Step 2](images/2.png)

During inspection, a suspicious string was observed (likely as a cookie or query parameter):

```
cGljb0NURntjMDBrMWVfbTBuc3Rlcl9sMHZlc19jMDBraWVzXzc4QjRDMzkwfQ%3D%3D
```

This string is URL-encoded and contains characters consistent with Base64 encoding (including `=` padding when decoded).

---

### Exploitation / Analysis :

## Step 1 — URL-decoding

The string includes `%3D` sequences which are URL encoded characters (`%3D` → `=`). First, URL-decode the string to recover the Base64 blob.

Example (Python):

```bash
python3 -c "import urllib.parse; print(urllib.parse.unquote('cGljb0NURntjMDBrMWVfbTBuc3Rlcl9sMHZlc19jMDBraWVzXzc4QjRDMzkwfQ%3D%3D'))"
```

After URL-decoding the original submission, the cleaned Base64 string becomes:

```
cGljb0NURntjMDBrMWVfbTBuc3Rlcl9sMHZlc19jMDBraWVzXzc4QjRDMzkwfQ==
```

## Step 2 — Base64 decoding

Next, Base64-decode the URL-decoded string to reveal the plaintext.

Example (combined one-liner, Python):

```bash
echo "cGljb0NURntjMDBrMWVfbTBuc3Rlcl9sMHZlc19jMDBraWVzXzc4QjRDMzkwfQ%3D%3D" | base64 -d
```

![Step 3](images/3.png)

This outputs the flag in plaintext.

---

## Result

**Flag:** `picoCTF{c00k1e_m0nster_l0ves_c00kies_78B4C390}`

(Exact string as decoded from the Base64 payload.)

---

## Notes & Alternate Approaches

* The challenge is intentionally simple and designed to test familiarity with common encodings and how data is sometimes stored in cookies or URLs.
* If the encoded data had been split across multiple cookies or parameters, you would gather and concatenate before decoding.
* Tools such as `curl`, `burp`, or simple browser DevTools are helpful to capture the encoded value. `base64` and language-builtins (Python, Node.js, etc.) make decoding straightforward.

---

## Mitigation / Lessons Learned

* Sensitive data (flags, secrets, or credentials) should never be stored client-side (e.g., in cookies or query parameters) unless properly signed and encrypted. Simply encoding (URL or Base64) is **not** secure.
* Web applications should avoid exposing secrets in responses, HTML comments, or static resources.
* Use secure, server-side sessions and sign any client-side tokens (e.g., HMAC) if client storage is required.

---

## Conclusion

This challenge rewards careful inspection of client-visible artifacts (cookies, query params, source comments) and knowledge of common encodings. The final flag was recovered by URL-decoding and Base64-decoding a client-side string to reveal the secret recipe.

---

## Appendix: Useful commands

```bash
# URL-decode then Base64-decode (bash + python)
python3 -c "import urllib.parse,base64; s='cGljb0NURntjMDBrMWVfbTBuc3Rlcl9sMHZlc19jMDBraWVzXzc4QjRDMzkwfQ%3D%3D'; print(base64.b64decode(urllib.parse.unquote(s)).decode())"

# Quick base64 decode (if you already have the clean base64 string)
echo 'cGljb0NURntjMDBrMWVfbTBuc3Rlcl9sMHZlc19jMDBraWVzXzc4QjRDMzkwfQ==' | base64 -d
```

---

*Write-up created for learning / CTF write-up practice.*
