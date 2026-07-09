# AWS Service Cost Research: Amazon Q Developer (formerly CodeWhisperer)

> **Status:** ✅ Research Complete
> **Last Updated:** July 2026

---

## 1. Service Overview
Amazon Q Developer is an AI-powered coding, troubleshooting, and cloud architectural companion. It is integrated directly into popular Integrated Development Environments (IDEs like VS Code, JetBrains), command line interfaces (CLIs), and the AWS Console. Q Developer provides real-time code recommendations, translates code languages, conducts security vulnerability scans, and assists in upgrading legacy application frameworks (e.g. Java 8 to Java 17).

---

## 2. Billing Mechanics
Q Developer is billed on a per-user licensing model with two primary tiers:
1.  **Q Developer Free Tier (Individual):** Free of charge for individual developers, linked via an AWS Builder ID.
2.  **Q Developer Pro Tier (Enterprise):** A monthly subscription fee per registered user seat, managed via the AWS Billing console and AWS Organizations.

---

## 3. Key Cost Dimensions

### A. Free Tier vs. Pro Tier (us-east-1 Rates)
*   **Q Developer Free Tier:**
    *   *Cost:* **Free ($0.00)**.
    *   *Features:* Basic code completions, CLI suggestions, general chat support.
*   **Q Developer Pro Tier:**
    *   *Cost:* **$19.00 per user per month**.
    *   *Additional Features:*
        *   Enterprise single sign-on (SSO) integration via **IAM Identity Center**.
        *   Higher quotas for **Q Developer Agents** (auto-writing code, planning tasks).
        *   Access to **Amazon Q Developer Code Transformation** (legacy code upgrades).
        *   Customization (allowing Q to scan your private code repositories to recommend internal library patterns).
        *   Administrative controls (blocking code recommendations that match public open-source code licenses).

### B. Subscription Pro-Rating
*   If you add a user during the month, the subscription fee is prorated for the remaining days of the billing cycle.
*   If you remove a user, the seat remains active until the end of the current billing month, and the subscription is cancelled for the next cycle.

---

## 4. Detailed Pricing Rates (us-east-1)

| Subscription Tier | Monthly Fee (per User) | Target Audience | Primary Features |
|-------------------|------------------------|-----------------|------------------|
| **Individual Tier**| **Free ($0.00)** | Independent devs | Basic IDE completions, CLI, AWS chat |
| **Professional Tier**| **$19.00** | Enterprise teams | SSO, advanced agents, transformation, repo search |

---

## 5. AWS Free Tier Coverage
*   **Individual Tier:** Always free for independent developers. No AWS billing account is required.

---

## 6. Common Cost Hotspots & Pitfalls
*   **Paying for Inactive User Seats:** Purchasing Q Developer Pro licenses for developers who have left the company, transferred roles, or do not use the extension in their IDEs. Each inactive seat bleeds **$19.00/month**.
*   **Duplicate Subscriptions:** Enabling both Q Developer Pro ($19/mo) and another coding assistant (like GitHub Copilot at $19/mo) for the same developer, doubling enterprise tooling costs.
*   **Unmonitored License Proliferation:** Leaving user invitation groups in IAM Identity Center unrestricted, allowing any employee to auto-provision a paid license.

---

## 7. Actionable Cost Optimization Strategies
1.  **Enforce SSO Groups for Q Pro Provisioning:**
    *   Do not allow open enrollment in IAM Identity Center.
    *   Create a specific AD/Identity directory group named `Q-Developer-Pro-Users`.
    *   Only assign Q Developer Pro application access to this group, ensuring that licenses can only be provisioned with management approval.
2.  **Audit Active Usage Monthly (De-provision Inactive Seats):**
    *   Write a script or use AWS IAM Identity Center reports to audit active logins.
    *   Identify any user who has not logged into the Q Developer extension or generated a code suggestion in the last 30 days.
    *   Remove these users from the Q Pro group to immediately cancel the **$19.00/month recurring charge**.
3.  **Standardize on the Free Tier for Non-Corporate Tasks:** For temporary contractors, interns, or developers working on non-proprietary sandbox code, have them sign up using their own **AWS Builder IDs** on the **Free Individual Tier** to bypass enterprise subscription fees.
4.  **Enforce Code Transformation scheduling:** Plan framework upgrade jobs (e.g. Java migrations) carefully. Since Code Transformation is a Pro feature, assign licenses to engineers during active migration sprints and remove them once the codebase has been upgraded.
