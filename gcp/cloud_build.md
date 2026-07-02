# Cloud Build Cost Optimization & Research

Google Cloud Build is a serverless CI/CD platform that imports source code, executes builds, runs tests, and produces artifacts (like container images or software packages). Because Cloud Build is billed on a **per-build-minute** basis, slow build execution times and un-optimized trigger structures can lead to high monthly build costs.

---

## 1. Cloud Build Billing Components

Cloud Build billing is determined by two main factors:
1. **Machine Type Hourly Rate (per build-minute):** The rate scales with the size of the build worker:
   * **g1-standard (1 vCPU, 3.75 GB RAM):** Billed at $0.003 per minute.
   * **e2-highcpu-8 (8 vCPUs, 8 GB RAM):** Billed at $0.016 per minute.
   * **e2-highcpu-32 (32 vCPUs, 32 GB RAM):** Billed at $0.064 per minute.
2. **Free Tier:** The first **120 build-minutes per day per billing account are free** when using the default `g1-standard` machine type.

---

## 2. Core Cost-Optimization Levers

### A. Implement Docker Layer Caching (Kaniko / `--cache-from`)
* **The Waste:** Re-downloading and compiling package dependencies (like `npm install` or `pip install`) on every single build run.
* **The Solution:** Use Docker layer caching.
* **Action:**
  1. For standard Docker builds, use the **Kaniko cache** runner in your `cloudbuild.yaml` file:
     ```yaml
     steps:
     - name: 'gcr.io/kaniko-project/executor:latest'
       args:
       - --destination=gcr.io/$PROJECT_ID/my-image
       - --cache=true
       - --cache-ttl=24h
     ```
  2. Alternatively, if using standard `docker build`, append `--cache-from` targeting the latest image in Artifact Registry.
* **The Benefit:** Reduces build execution time by **50% to 80%**, saving significant build-minutes daily.

### B. Match Machine Types to Compilation Profiles
* **The Waste:** Running a simple linting, formatting, or script deployment step on an expensive `e2-highcpu-32` machine.
* **Action:**
  * Use the default `g1-standard` machine (which counts towards the free 120 minutes/day) for script execution, terraform runs, and basic file transfers.
  * Reserve high-compute machines (`e2-highcpu-8` or higher) strictly for large-scale source code compilations (e.g. Java, C++, Go) or heavy Docker image bundling where the compile time reduction offsets the higher per-minute rate.

### C. Restrict Trigger Scopes (Ignored Files)
* **The Waste:** Triggering a full CI/CD test and build run when a developer commits a change to a documentation folder or edits a `README.md` file.
* **Action:** In your Cloud Build trigger configurations, specify **Ignored Files** (e.g. `docs/**` or `**.md`) to prevent commits on non-code files from launching builds.

---

## 3. Cloud Build Audit Checklist

1. [ ] **Build Caching Verification:** Confirm that Docker build pipelines utilize Kaniko cache or `--cache-from` configurations.
2. [ ] **Machine Type Mapping:** Audit active `cloudbuild.yaml` configurations. Ensure high-spec machines are not used for lightweight tasks.
3. [ ] **Trigger Filter Implementation:** Review repository triggers. Apply file path filters (ignored files) to prevent builds on documentation updates.
4. [ ] **Build Timeout Limits:** Enforce maximum build timeouts (e.g., limit builds to max 15 minutes) to prevent hung scripts from billing indefinitely.
