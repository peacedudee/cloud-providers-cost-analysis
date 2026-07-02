# Cloud KMS Cost Optimization & Research

Google Cloud Key Management Service (KMS) allows you to manage cryptographic keys (symmetric, asymmetric, HSM, and external keys) to protect your cloud resources. While encryption-at-rest is enabled by default across Google Cloud for free using Google-managed keys, using Customer-Managed Encryption Keys (CMEK) via Cloud KMS introduces hourly key hosting fees and per-request operation charges.

---

## 1. Cloud KMS Billing Components

KMS costs are calculated based on three dimensions:
1. **Active Key Versions (per version / month):** Charged for keeping key versions active in your project:
   * **Software Keys:** $0.06 per active key version per month.
   * **HSM (Hardware Security Module) Keys:** $1.00 to $2.50 per active key version per month (depending on algorithm).
   * **External Key Manager (EKM) Keys:** $3.00 per active key version per month.
2. **Cryptographic Operations:** Billed per 10,000 API operations (e.g. encrypt, decrypt, sign, verify). Billed at $0.03 per 10,000 requests.
3. **Key Ring & Key Count:** There is no charge for the Key Ring or Key container itself; you are billed strictly for the active **key versions** within them.

---

## 2. Core Cost-Optimization Levers

### A. Implement Envelope Encryption (Reduce API Calls)
* **The Cost Trap:** Ingesting 10 million telemetry payloads per day and calling Cloud KMS to encrypt each payload individually, triggering 10 million API calls.
* **The Solution:** Use **Envelope Encryption**.
* **Action:**
  1. Generate a local Data Encryption Key (DEK) inside your application memory.
  2. Encrypt your database rows or payloads locally using the DEK (e.g. using AES-GCM).
  3. Send the DEK *once* to Cloud KMS to be encrypted (wrapping). Store the encrypted DEK alongside your encrypted data.
  4. Cache the decrypted DEK in application memory (if secure and compliant).
* **The Benefit:** Reduces KMS API requests from millions to a handful, keeping cryptographic operations billing at near $0.

### B. Destroy Retired/Stale Key Versions
* **The Issue:** When you rotate a key, the older key versions remain active so that historical data encrypted with those versions can still be decrypted. If you delete or re-encrypt that historical data with the new key version, the old key versions are no longer needed but **continue billing indefinitely**.
* **Action:** Identify key versions that are no longer used to encrypt or decrypt active data and programmatically destroy them.
  ```bash
  # Destroy a specific key version (reversible within 24 hours)
  gcloud kms keys versions destroy 3 \
      --key=my-key --keyring=my-keyring --location=global
  ```

### C. Align Rotation Cycles to Compliance Baselines
* **The Waste:** Setting up an automated daily key rotation policy. Within 3 years, a single key will have over 1,000 active versions, costing you $60/month (software) or $1,000/month (HSM) for that single key ring.
* **Action:** Align automated rotation intervals strictly with regulatory requirements. For most security guidelines (like PCI-DSS or SOC2), **annual rotation** or **90-day rotation** is completely sufficient.

---

## 3. Cloud KMS Audit Checklist

1. [ ] **Envelope Encryption Compliance:** Verify that high-throughput applications utilize envelope encryption rather than sending individual records directly to the KMS API.
2. [ ] **Key Version Cleanup:** Audit all key rings. Destroy old, unused key versions where underlying datasets have been deleted or re-encrypted.
3. [ ] **Rotation Period Check:** Confirm that key rotation intervals are set to compliance baselines (e.g. 90 or 365 days) rather than excessively fast schedules.
4. [ ] **Software vs. HSM Selection:** Ensure HSM-backed keys ($1.00/version/month) are reserved strictly for production compliance requirements, using Software keys ($0.06/version/month) in development.
