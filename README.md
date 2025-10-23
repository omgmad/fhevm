🧠 Summary

The SDK exposes sensitive data, uses insecure patterns (any, unhandled errors, console logging), and swallows exceptions in decryption routines.
These behaviors may lead to information leakage, runtime instability, and reduced type safety in production builds.

🚨 Critical Issue 1 — Sensitive Key Exposure

Files:

src/FhevmDecryptionSignature.ts

src/fhevmTypes.ts

src/react/useFHEDecrypt.ts

Severity: 🔴 High / Security
Description:
Variables or constants referencing privateKey or mnemonic appear in code and are not clearly isolated in secure storage.
In the context of FHEVM, exposing cryptographic key material within SDK layers could lead to memory disclosure or key reuse vulnerabilities.

Recommendation:

Remove or mask sensitive key variables.

Use environment variables or secure in-memory handling.

Add a static analyzer rule to block committing secrets.

⚠️ Issue 2 — Silent Error Handling in useFHEDecrypt

File: src/react/useFHEDecrypt.ts
Severity: 🟠 Medium / Logic
Description:
A try/catch block captures an error but never rethrows or reports it.
This causes silent failure during decryption, leading to undefined states for applications using this hook.

