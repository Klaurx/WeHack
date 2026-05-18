# F-09: MD5 Used for Payment API Signatures

## What was observed

The signature utility in `fastcpl-web-backstage/src/main/java/cn/fast/web/backstage/utils/Signature.java` implements all API request signing using MD5:

```java
public static String MD5(String data) {
    MessageDigest md = MessageDigest.getInstance("MD5");
    byte[] array = md.digest(data.getBytes("UTF-8"));
    // ...
}

public static String generateSignature(final Map<String, String> data, String key) throws Exception {
    // ... sorts params, builds string ...
    return MD5(sb.toString()).toLowerCase();
}
```

This utility is used across all payment channel integrations and API signature verification. The signing pattern concatenates sorted request parameters with the secret key appended, then takes the MD5 hash of the resulting string.

## Why this matters

MD5 has been cryptographically broken since 2004. It is vulnerable to collision attacks and, in the context of HMAC-style constructions like this one, to length extension attacks depending on implementation.

More practically: the secret keys used in these signatures are now exposed (F-04, F-05). Signature verification based on a known key is only useful if the key remains secret. Once the key is known, the signature provides no authenticity guarantee regardless of the hash algorithm used.

Even after key rotation, moving away from MD5 for payment-related signatures is the correct long-term direction. Most of the channel SDKs integrated in this codebase support HMAC-SHA256 or RSA-based signature schemes.

## What needs to change

- After rotating all keys (F-04), audit each channel integration to determine whether the channel's own SDK or API supports a stronger signature scheme and migrate to it.
- Replace the MD5-based `generateSignature` method with HMAC-SHA256 for any internal API signatures where the algorithm is under this team's control.
- Do not treat signature verification as a security control while the signing keys remain compromised.
