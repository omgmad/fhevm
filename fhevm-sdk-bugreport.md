
# 🧩 Bug Report: Critical & High Severity Issues in `fhevm-sdk` Package

**Repository:** [zama-ai/fhevm-react-template](https://github.com/zama-ai/fhevm-react-template)  
**Package:** `packages/fhevm-sdk`  
**Scanned by:** Static analysis (TypeScript + pattern scan)  
**Date:** 2025-10-23

---

## 🧠 Summary

The SDK exposes sensitive data, uses insecure patterns (`any`, unhandled errors, console logging), and swallows exceptions in decryption routines.  
These behaviors may lead to information leakage, runtime instability, and reduced type safety in production builds.

---

## 🚨 Critical Issue 1 — Sensitive Key Exposure

**Files:**

- `src/FhevmDecryptionSignature.ts`  
- `src/fhevmTypes.ts`  
- `src/react/useFHEDecrypt.ts`  

**Severity:** 🔴 High / Security  

**Description:**  
Variables or constants referencing `privateKey` or `mnemonic` appear in code and are not clearly isolated in secure storage.  
Exposing cryptographic key material within SDK layers could lead to memory disclosure or key reuse vulnerabilities.

**Recommendation:**

- Remove or mask sensitive key variables.  
- Use environment variables or secure in-memory handling.  
- Add a static analyzer rule to block committing secrets.

---

## ⚠️ Issue 2 — Silent Error Handling in `useFHEDecrypt`

**File:** `src/react/useFHEDecrypt.ts`  
**Severity:** 🟠 Medium / Logic  

**Description:**  
A `try/catch` block captures an error but never rethrows or reports it.  
This causes silent failure during decryption, leading to undefined states for applications using this hook.

**Recommendation:**
```ts
try {
  // ...
} catch (err) {
  console.error("FHE decrypt failed", err);
  throw err; // rethrow to allow app-level error handling
}
```

---

## ⚠️ Issue 3 — Excessive Console Logging (Info Leak Risk)

**Files:**

- `src/internal/RelayerSDKLoader.ts`  
- `src/internal/fhevm.ts`  
- `src/react/useFhevm.tsx`  

**Severity:** 🟠 Medium / Information Disclosure  

**Description:**  
`console.log` statements are present throughout internal modules.  
These may print FHE-encrypted payloads, API endpoints, or intermediate state objects in production builds.

**Recommendation:**

- Remove `console.log` calls in release builds.  
- Replace with a proper logger (e.g., debug flag or conditional logging).  
- Add ESLint rule `no-console` to enforce.

---

## ⚙️ Issue 4 — Widespread `any` Type Usage

**Files (sample):**

- `src/FhevmDecryptionSignature.ts`  
- `src/fhevmTypes.ts`  
- `src/internal/RelayerSDKLoader.ts`  
- `src/internal/fhevmTypes.ts`  
- `src/react/useFHEDecrypt.ts`  
- `src/react/useFHEEncryption.ts`  
- `src/react/useFhevm.tsx`  

**Severity:** 🟡 Low / Maintainability  

**Description:**  
The SDK heavily relies on `any` types, disabling static safety.  
Potential runtime type errors may occur during encryption/decryption or contract interactions.

**Recommendation:**

- Enable `noImplicitAny` and `strict` in `tsconfig.json`.  
- Define explicit FHE type interfaces (e.g., `FHEEncrypted<T>`, `FHESigner`).

---

## ⚙️ Issue 5 — Catch Block Without Rethrow (Swallowed Error)

**File:** `src/react/useFHEDecrypt.ts`  
**Severity:** 🟡 Low / Logic  

**Description:**  
Catching without rethrow or logging hides runtime issues, breaking error propagation chains.

**Recommendation:**  
Always log or rethrow errors.

---

## 📊 Summary Table

| Category                     | Files Affected | Severity | Description                     |
|-------------------------------|----------------|----------|---------------------------------|
| Sensitive key variables       | 3              | 🔴 High  | Possible key disclosure         |
| Silent catch (no rethrow)     | 1              | 🟠 Medium| Hidden runtime failures         |
| Console logging in SDK        | 3              | 🟠 Medium| Information leak risk           |
| `any` type usage              | 8              | 🟡 Low   | Type safety loss                |

---

## 🧾 Suggested Mitigation Plan

- Enable strict TypeScript mode (`"strict": true`, `"noImplicitAny": true`).  
- Add ESLint rule `no-console` and CI enforcement.  
- Centralize key management (move to secure provider / env variables).  
- Audit all error handling to ensure errors are logged or surfaced.  
- Add `pnpm` test cases for encrypt/decrypt error propagation.



---

## ✅ Environment

- Node: >=20  
- pnpm: 8.x  
- Packages analyzed: `fhevm-sdk`  
- Detected ~14 issues during static scan  

**Reporter:** Automated analysis (GPT-5 assisted audit)  
**Date:** 2025-10-23  
**Category:** Security / Quality / Maintainability
