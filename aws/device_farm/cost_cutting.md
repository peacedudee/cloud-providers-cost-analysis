# Cost-Cutting Playbook: AWS Device Farm
> **Companion File:** [device_farm.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/device_farm/device_farm.md)
> **Last Updated:** July 2026
---
## Executive Summary
AWS Device Farm provides a scalable environment for automated and manual testing of mobile and web applications on physical devices. While it eliminates the need to maintain an internal device lab, its Pay-As-You-Go ($0.17/minute) model can lead to significant cost overruns if test suites are bloated, timeout limits are missing, or testing is triggered on every minor code commit. This playbook outlines 16 comprehensive strategies for optimizing AWS Device Farm expenditures, focusing on eliminating wasted device-minutes, shifting testing left to local emulators, optimizing device pools, and strategically leveraging Unmetered Slots ($250/mo) and Private Devices ($200+/mo).

## Strategy Categories

### 1. Waste Elimination

#### 1. Enforce Strict Maximum Timeouts for Test Runs
- **What:** Configure hard timeout limits on all automated test runs in Device Farm to prevent runaway or hung tests.
- **Why It Saves Money:** A test script stuck in an infinite loop or waiting for an element that never appears will continue billing at $0.17 per device-minute until the default AWS maximum timeout is reached (which can be hours). A 2-hour hung test on 10 devices costs $204.00.
- **Implementation Steps:**
  1. Review average test suite execution times.
  2. Modify the AWS CLI `schedule-run` command or AWS SDK parameters to include the `--execution-configuration` parameter with a custom `jobTimeoutMinutes`.
  3. Set the timeout to slightly higher than the maximum expected run time (e.g., 20 minutes instead of the default 150 minutes).
- **Estimated Savings:** 10-20%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Understanding of normal test execution duration.

#### 2. Cancel Unutilized Unmetered Slots
- **What:** Identify and cancel Unmetered Slots ($250/month each) that are not being fully utilized across the organization.
- **Why It Saves Money:** Unmetered slots are billed at a flat rate of $250.00/month regardless of usage. If a project ends or testing frequency decreases, maintaining an unused slot wastes $3,000 annually.
- **Implementation Steps:**
  1. Review Device Farm usage metrics in CloudWatch or CUR to identify slot utilization rates.
  2. Identify slots executing fewer than 1,470 minutes of tests per month.
  3. Downgrade those testing pipelines to Pay-As-You-Go pricing.
  4. Cancel the unnecessary slots via the AWS Console or CLI.
- **Estimated Savings:** $250.00 per slot/month
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Procurement/Leadership
- **Prerequisites:** Cost Explorer or CUR data for Device Farm.

#### 3. Audit and Remove Unused Private Devices
- **What:** Terminate Private Device subscriptions that are no longer actively required for specialized testing.
- **Why It Saves Money:** Private Devices start at $200.00/month per device. If the specific hardware model is no longer relevant to the user base or the specialized OS configuration is no longer needed, it represents pure waste.
- **Implementation Steps:**
  1. List all active Private Devices in the account.
  2. Cross-reference device usage with current application testing requirements.
  3. Cancel subscriptions for deprecated devices (note that minimum subscription terms may apply).
- **Estimated Savings:** $200.00+ per device/month
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Inventory of active Private Devices.

#### 4. Skip Device Farm Testing for Backend-Only Changes
- **What:** Configure CI/CD pipelines to bypass AWS Device Farm physical device testing if a pull request only contains backend, documentation, or non-UI code changes.
- **Why It Saves Money:** Running an extensive mobile UI test suite on 20 devices for a typo fix in a README or a database schema update incurs metered device-minute charges with zero testing value.
- **Implementation Steps:**
  1. Implement path-filtering in the CI/CD pipeline (e.g., GitHub Actions `paths` or GitLab `changes` triggers).
  2. Ensure Device Farm triggers only fire when mobile client code (e.g., Android/iOS source, UI assets) is modified.
- **Estimated Savings:** 15-30%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CI/CD pipeline integration.

#### 5. Prune Obscure Device Models from Device Pools
- **What:** Review and update the custom Device Pools used for test runs, removing older or obscure devices that represent a negligible fraction of your user base.
- **Why It Saves Money:** Running a 30-minute test suite on a device that only 0.1% of your users possess costs $5.10 per run. By narrowing the device pool from 30 devices to the top 10 most critical devices, you cut per-run costs by 66%.
- **Implementation Steps:**
  1. Analyze mobile app analytics (e.g., Google Analytics, Mixpanel) to determine the top device models and OS versions used by your customers.
  2. Navigate to AWS Device Farm > Project Settings > Device Pools.
  3. Create or update Device Pools to include only the top 80-90% percentile of real user devices.
- **Estimated Savings:** 40-60% per test run
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Mobile user analytics data.

### 2. Rightsizing

#### 6. Shift-Left UI Testing to Local Emulators
- **What:** Execute unit tests and basic UI smoke tests on local Android emulators or iOS simulators during the initial CI/CD stages before triggering Device Farm.
- **Why It Saves Money:** Discovering a compile error or a crash-on-launch bug during a physical device run wastes $0.17/minute per device. Catching it on a local (or free CI/CD runner) emulator costs nothing in AWS Device Farm fees.
- **Implementation Steps:**
  1. Configure the first stage of the CI pipeline to build and test the app using headless emulators/simulators.
  2. Only advance to the AWS Device Farm deployment stage if the emulator tests pass successfully.
- **Estimated Savings:** 20-30%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Dockerized emulator infrastructure or CI/CD runner support.

#### 7. Optimize Test Script Execution Speed
- **What:** Refactor Appium, Espresso, or XCTest UI automation scripts to execute faster.
- **Why It Saves Money:** Every minute saved in test execution directly reduces Pay-As-You-Go costs by $0.17 per device. Saving 5 minutes on a 20-device run saves $17.00 every single time the pipeline runs.
- **Implementation Steps:**
  1. Replace hardcoded arbitrary wait times (e.g., `Thread.sleep(5000)`) with explicit waits (e.g., wait until an element is visible or clickable).
  2. Optimize test data setup (e.g., inject data via API rather than clicking through the UI to create records).
  3. Profile test runs to identify the slowest steps and refactor them.
- **Estimated Savings:** 10-25%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Test automation engineering expertise.

#### 8. Offload Web Testing to Desktop Browsers/EC2
- **What:** For responsive web applications, run tests on desktop browser grids (Selenium/Playwright) hosted on standard EC2 instances rather than using physical mobile devices in Device Farm.
- **Why It Saves Money:** Physical device testing is premium-priced ($10.20/hour). An EC2 `t3.medium` running a headless browser costs ~$0.04/hour. Unless mobile-specific hardware features are needed, testing responsive web views on EC2 is vastly cheaper.
- **Implementation Steps:**
  1. Separate mobile app (APK/IPA) tests from responsive web browser tests.
  2. Migrate web browser tests to an EC2-backed Selenium Grid, AWS CodeBuild, or a cheaper desktop browser testing alternative.
  3. Reserve Device Farm strictly for native/hybrid mobile application testing.
- **Estimated Savings:** Up to 90% for web-only tests
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Web test framework compatible with desktop execution.

### 3. Commitment Discounts

#### 9. Consolidate Unmetered Slots Across Teams
- **What:** Centralize the purchase and management of Unmetered Slots so multiple teams can share them sequentially, rather than each team buying their own.
- **Why It Saves Money:** If Team A uses a slot for 4 hours a day and Team B uses a slot for 4 hours a day, buying two slots costs $500/month. Combining them into one shared CI/CD queue using a single slot costs $250/month.
- **Implementation Steps:**
  1. Audit Unmetered Slot purchases across all AWS accounts using AWS Organizations / Cost Explorer.
  2. Implement a centralized CI/CD queue that routes Device Farm test requests to a shared AWS account.
  3. Cancel redundant Unmetered Slots in individual team accounts.
- **Estimated Savings:** $250.00+ per month per consolidated slot
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Centralized CI/CD infrastructure.

### 4. Architecture Changes

#### 10. Mock External APIs and Backend Services During Testing
- **What:** Use mock servers (e.g., WireMock) to simulate backend API responses during Device Farm UI tests instead of hitting live staging environments.
- **Why It Saves Money:** Live network calls add latency, increasing total test execution time. Mocking APIs reduces test duration, thereby reducing the billed device-minutes under the Pay-As-You-Go model.
- **Implementation Steps:**
  1. Instrument the mobile app to accept a mock server URL for testing environments.
  2. Deploy a mock server alongside the test suite or bundle mocked responses within the app build.
  3. Execute tests against the mocked data.
- **Estimated Savings:** 10-15%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Mobile app architecture supporting dependency injection / configurable endpoints.

#### 11. Implement Smart Test Impact Analysis (TIA)
- **What:** Use AI or dependency mapping tools to execute only the specific UI tests that cover the modified codebase, rather than running the entire test suite.
- **Why It Saves Money:** Running a 5-minute subset of tests instead of a 60-minute full regression suite reduces the device-minute cost by 91% for that specific CI/CD pipeline run.
- **Implementation Steps:**
  1. Integrate a Test Impact Analysis tool into the CI/CD pipeline.
  2. Configure the pipeline to dynamically generate the list of tests to run based on the git diff.
  3. Pass the filtered test list to AWS Device Farm via the CLI/SDK.
- **Estimated Savings:** 40-70%
- **Risk Level:** High
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Advanced CI/CD maturity and modular test framework.

### 5. Scheduling & Auto-Scaling

#### 12. Limit Full Regression Suites to Nightly Builds
- **What:** Configure CI/CD pipelines to run only a small "Smoke Test" suite on every Pull Request, and defer the full multi-device Regression Suite to a scheduled nightly build.
- **Why It Saves Money:** If a team merges 10 PRs a day, running a $50 regression suite on every PR costs $500/day. Running smoke tests ($5/PR) plus one nightly regression ($50) reduces the daily cost to $100.
- **Implementation Steps:**
  1. Tag UI tests as `@Smoke` or `@Regression`.
  2. Update CI/CD PR triggers to execute only `@Smoke` tests against a small device pool (1-2 devices).
  3. Create a cron job in the CI/CD system to trigger `@Regression` tests nightly against the full device pool.
- **Estimated Savings:** 50-80%
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Ability to filter/tag automated tests.

#### 13. Queue Non-Critical Tests for Sequential Slot Execution
- **What:** Instead of bursting test execution in parallel across the Pay-As-You-Go fleet, queue test runs to execute sequentially on available Unmetered Slots.
- **Why It Saves Money:** Bursting parallel tests utilizes metered billing once Unmetered Slots are fully occupied. Queuing ensures all tests stay within the fixed $250/mo slot capacity, avoiding metered overflow charges entirely.
- **Implementation Steps:**
  1. Purchase an appropriate baseline of Unmetered Slots.
  2. Configure the CI/CD orchestrator (e.g., Jenkins, GitHub Actions) to limit concurrent Device Farm deployment jobs to match the number of purchased slots.
  3. Non-critical jobs wait in the CI queue until a slot is free.
- **Estimated Savings:** Prevents 100% of metered overflow charges
- **Risk Level:** Medium
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** CI/CD concurrency controls.

### 6. Pricing Model Optimization

#### 14. Perform Regular Breakeven Analysis (Metered vs. Unmetered)
- **What:** Continuously monitor test duration metrics to ensure the correct pricing model is applied based on the 1,470-minute breakeven point.
- **Why It Saves Money:** The math is fixed: $250 / $0.17 = 1,470 minutes (24.5 hours). If you consume 2,000 minutes on Pay-As-You-Go, you pay $340. Switching to an Unmetered Slot saves $90. If you consume 500 minutes on an Unmetered Slot, you pay $250. Switching to Pay-As-You-Go saves $165.
- **Implementation Steps:**
  1. Establish a monthly FinOps review of Device Farm CUR data.
  2. Sum the device-minutes per project/team.
  3. Transition projects crossing the 1,470-minute threshold to Unmetered Slots, and downgrade under-utilizing projects to Pay-As-You-Go.
- **Estimated Savings:** 10-30%
- **Risk Level:** Low
- **Implementation Scope:** FinOps Team
- **Prerequisites:** AWS Cost Explorer or CUR.

#### 15. Evaluate Private Devices vs. Public Unmetered Slots
- **What:** Analyze if your testing requirements can be fulfilled by Private Devices ($200/mo) instead of Unmetered Slots ($250/mo) if you only need exclusive access to a single specific hardware model.
- **Why It Saves Money:** If a team purchases an Unmetered Slot simply to get unlimited testing on one specific phone model (e.g., iPhone 13), switching to a Private Device subscription for that exact model saves $50/month, while also providing exclusive, continuous access without waiting in public pool queues.
- **Implementation Steps:**
  1. Review the testing patterns of teams using Unmetered Slots.
  2. If testing is highly concentrated on 1-2 specific device models rather than a diverse pool, price out the Private Device equivalent.
  3. Procure the Private Device and cancel the Unmetered Slot.
- **Estimated Savings:** $50.00 per device/month (20% reduction)
- **Risk Level:** Medium
- **Implementation Scope:** FinOps Team | Engineer/DevOps
- **Prerequisites:** Assessment of device fragmentation requirements.

### 7. Network & Data Transfer Optimization

#### 16. Optimize Application Payload Sizes and Logging
- **What:** Reduce the file size of the uploaded APK/IPA and disable verbose logging or screen recording for stable tests.
- **Why It Saves Money:** While AWS Device Farm does not heavily penalize for storage/transfer, massive payloads take longer to upload and install on the physical device. This installation time is billed as active device-minutes. Faster app installs equal cheaper test runs.
- **Implementation Steps:**
  1. Strip unnecessary assets, debug symbols, or large media files from the test build.
  2. Disable video recording (if using custom Appium environments) for non-essential test runs.
  3. Measure the reduction in device setup/teardown time.
- **Estimated Savings:** 2-5%
- **Risk Level:** Low
- **Implementation Scope:** Engineer/DevOps
- **Prerequisites:** Build pipeline optimization.

---

## Cross-Service Synergies
- **AWS CodeBuild / AWS CodePipeline:** Integrate tightly to orchestrate the CI/CD triggers, enabling the "Shift-Left" and "Smoke Test vs. Regression" scheduling strategies.
- **AWS Cost Explorer / AWS CUR:** Essential for tracking the 1,470-minute breakeven point between Metered and Unmetered billing models.
- **Amazon EC2:** Can be utilized for hosting cheaper desktop browser testing grids (Selenium) to offload web application testing from physical mobile devices.

---

## Required Input Data for Real-World Analysis

### A. AWS CUR 2.0
- `lineItem/ProductCode`: `AWSDeviceFarm`
- `lineItem/UsageType`: Look for `DeviceMinutes`, `UnmeteredSlot`, and `PrivateDevice`.
- `lineItem/UsageAmount`: Crucial for calculating if Pay-As-You-Go minutes exceed the Unmetered breakeven point.

### B. CloudWatch Metrics
- Device Farm project execution metrics (Duration, Success Rate) to identify test bloat and timeout issues.

### C. AWS Config / Trusted Advisor
- Generally less applicable for Device Farm, but useful for ensuring IAM policies restrict who can trigger expensive test runs.

### D. Company Policies
- Mobile device support matrix (determines which devices must be in the test pools).
- QA/Release requirements (determines frequency of required regression testing).

### E. IaC (Optional)
- Terraform/CloudFormation scripts managing AWS CodeBuild/CodePipeline integrations with Device Farm to identify missing path-filtering controls.

---

## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "DEVICEFARM-001",
  "service": "AWS Device Farm",
  "strategy_category": "Pricing Model Optimization",
  "observation": "Project X consumed 3,500 device-minutes under Pay-As-You-Go pricing last month.",
  "financial_impact": {
    "current_monthly_cost": 595.00,
    "optimized_monthly_cost": 250.00,
    "estimated_monthly_savings": 345.00
  },
  "recommendation": "Purchase one Unmetered Slot for $250/month and associate it with Project X.",
  "risk_level": "Low",
  "status": "Open"
}
```

### Summary Report Table
| Finding ID | Strategy Category | Description | Est. Monthly Savings | Risk Level | Implementation Effort |
|------------|-------------------|-------------|----------------------|------------|-----------------------|
| DEVICEFARM-001 | Pricing Model | Switch Project X from Metered to Unmetered Slot | $345.00 | Low | Low |
| DEVICEFARM-002 | Waste Elimination | Implement 20-min timeout on all Device Farm runs | $150.00 | Low | Low |
| DEVICEFARM-003 | Rightsizing | Prune device pools from 30 devices down to top 10 | $400.00 | Medium | Low |
| DEVICEFARM-004 | Scheduling | Move full regression suites to nightly scheduled builds | $800.00 | Medium | Medium |
