# Cost-Cutting Playbook: Amazon WorkMail
> **Companion File:** [workmail.md](file:///Users/anjukumari/Desktop/Cloud%20Providers%20Cost%20Analysis/aws/workmail/workmail.md)
> **Last Updated:** July 2026

---

## Executive Summary
Amazon WorkMail is a managed business email and calendar service billed on a predictable, per-user monthly subscription basis ($4.00/user-month). Each user license generously includes 50 GB of storage. Cost overruns in WorkMail rarely stem from complex architectural issues; instead, they are almost exclusively driven by poor user lifecycle management—specifically, paying for mailboxes of terminated employees, inactive users, or automated service accounts. This playbook outlines 18 actionable strategies to eliminate waste, offload unnecessary mailbox usage to more cost-effective services (like Amazon SES or S3), and ensure you are only paying for active human users.

## Strategy Categories

### 1. Waste Elimination
*   **WORKMAIL-WST-001: Delete Inactive Mailboxes:** Identify users who have not logged into their WorkMail accounts for over 90 days. Delete these mailboxes to instantly save $4.00 per user per month.
*   **WORKMAIL-WST-002: Clean Up Abandoned Shared Mailboxes:** Audit shared team mailboxes. If a department or project has concluded and the shared mailbox is no longer monitored, delete it to eliminate the monthly licensing fee.
*   **WORKMAIL-WST-003: Empty Deleted Items & Junk Folders Regularly:** Educate users or enforce organizational policies to routinely clear 'Deleted Items' and 'Junk Email'. This prevents users from exceeding the 50 GB limit and incurring the $0.20/GB-month overage fee.
*   **WORKMAIL-WST-004: Delete Unused Resource Mailboxes:** While equipment and room resources are generally free, actively auditing and removing decommissioned resources maintains a clean Active Directory environment and prevents accidental conversion to paid user licenses.
*   **WORKMAIL-WST-005: Clean Up Old ActiveSync Partnerships:** Periodically remove unused or stale mobile device ActiveSync partnerships. While not directly a billing dimension, it reduces sync loads and lowers IT support overhead costs.

### 2. Rightsizing
*   **WORKMAIL-RGT-001: Replace Role-Based Mailboxes with Aliases:** Do not provision paid mailboxes for role-based addresses (e.g., `info@`, `sales@`, `support@`). Instead, create free email aliases that route to an existing user's primary mailbox.
*   **WORKMAIL-RGT-002: Consolidate Multiple Mailboxes per User:** If a single employee manages multiple brands or domain names, consolidate them into one paid mailbox utilizing multiple domain aliases rather than paying $4.00/month for each separate identity.
*   **WORKMAIL-RGT-003: Enforce Mailbox Retention Policies:** Implement strict email retention policies (e.g., delete emails older than 3 years) across the organization. This proactively keeps mailbox sizes under the included 50 GB threshold, entirely avoiding storage overage charges.
*   **WORKMAIL-RGT-004: Use Email Forwarding for Short-Term Contractors:** For temporary contractors or freelancers, use Amazon SES inbound rules to forward `@yourcompany.com` emails directly to their personal external addresses, bypassing the need for a $4.00/month WorkMail license.

### 3. Commitment Discounts
*   **WORKMAIL-COM-001: Leverage the 30-Day Free Trial:** Amazon WorkMail does not offer Reserved Instances or Savings Plans. However, for new deployments or Proof of Concepts (PoCs), you can utilize the WorkMail Free Tier, which covers up to 25 users for the first 30 days.

### 4. Architecture Changes
*   **WORKMAIL-ARC-001: Archive Former Employee Mailboxes to S3 Glacier:** Instead of maintaining active mailboxes for departed employees for compliance reasons, export their data (PST/EML) and store it in Amazon S3 Glacier Instant Retrieval ($0.004/GB-mo). This reduces the cost of a 50 GB mailbox from $4.00/mo to $0.20/mo—a 95% savings.
*   **WORKMAIL-ARC-002: Automate Lifecycle Management via Directory Service:** Integrate WorkMail with AWS Directory Service (Managed AD or Simple AD). Automate offboarding scripts so that when an employee is disabled in AD, their WorkMail license is immediately revoked.
*   **WORKMAIL-ARC-003: Use Amazon SES for Application Outbound Emails:** Never consume a $4.00/month WorkMail license for applications, scripts, or printers to send automated emails. Use Amazon Simple Email Service (SES) instead, which costs fractions of a penny per thousand emails.
*   **WORKMAIL-ARC-004: Use Amazon SES + S3 for Automated Email Ingestion:** If you have an automated process that parses incoming emails (e.g., a ticketing system), do not use a WorkMail mailbox and IMAP polling. Instead, configure Amazon SES to receive emails and trigger an AWS Lambda function or save them directly to S3.
*   **WORKMAIL-ARC-005: Offload Large File Attachments to S3 or WorkDocs:** Implement a company policy to share large files via secure Amazon S3 presigned URLs or Amazon WorkDocs links rather than sending massive email attachments, drastically slowing the growth of mailbox storage toward the 50 GB limit.

### 5. Scheduling & Auto-Scaling
*   **WORKMAIL-SCH-001: Just-in-Time Provisioning for Seasonal Workers:** Ensure mailboxes for temporary or seasonal staff are provisioned exactly on their start date and aggressively de-provisioned on their last day to avoid paying for idle licenses during off-seasons.

### 6. Pricing Model Optimization
*   **WORKMAIL-PRC-001: Monitor and Alert on Storage Overages:** Set up AWS Budgets and CloudWatch Billing Alarms to trigger immediately if WorkMail costs exceed the expected `(User Count * $4.00)` amount. This rapidly identifies if a user has breached the 50 GB limit and is generating $0.20/GB-month overages.

### 7. Network & Data Transfer Optimization
*   **WORKMAIL-NET-001: Restrict IMAP/SMTP Access for Web-Only Users:** For users who only access email via the WorkMail Web Client, disable IMAP and SMTP access. This prevents unauthorized third-party apps from constantly polling the mailbox, which can improve security and eliminate any fringe data transfer costs associated with excessive syncing.

---

## Cross-Service Synergies
*   **Amazon S3 & Glacier:** Crucial for replacing expensive active mailbox licenses for departed staff with ultra-low-cost cold storage archiving.
*   **Amazon SES (Simple Email Service):** Acts as the cost-effective alternative for programmatic email sending and receiving, reserving WorkMail strictly for human end-users.
*   **AWS Directory Service:** Essential for automating user lifecycles, ensuring billing accurately reflects real-time organizational headcount.
*   **AWS Lambda:** Used in conjunction with SES to process inbound emails in a serverless, pay-per-execution model rather than keeping a WorkMail account active just for parsing.

---

## Required Input Data for Real-World Analysis
### A. AWS CUR 2.0
*   `lineItem/ProductCode` = `AmazonWorkMail`
*   Filter by `lineItem/Operation` (e.g., `CreateUser`, `MailboxStorage`) to differentiate between standard user license fees and storage overage fees.

### B. CloudWatch Metrics
*   *(Note: WorkMail provides limited CloudWatch metrics. Focus must be placed on Active Directory login events or internal script logs to determine user inactivity.)*

### C. AWS Config / Trusted Advisor
*   Use AWS Config rules to monitor changes in AWS Directory Service that might indicate lingering users without active AD accounts.

### D. Company Policies
*   **Offboarding Policy:** Timelines for exporting and deleting former employee mailboxes.
*   **Data Retention Policy:** Maximum time emails must be kept for compliance, dictating inbox cleanup frequency.
*   **Application Email Policy:** Mandate the use of SES for all service accounts.

### E. IaC (Optional)
*   Review Terraform or CloudFormation scripts to ensure that programmatic email requirements are deploying SES identities rather than WorkMail users.

---

## Output Schema

### Finding Record (JSON)
```json
{
  "finding_id": "WORKMAIL-WST-001",
  "category": "Waste Elimination",
  "service": "Amazon WorkMail",
  "title": "Delete Inactive Mailboxes",
  "description": "Identify and delete mailboxes for users who haven't logged in for over 90 days to save license costs.",
  "potential_savings": "$4.00 per inactive user per month",
  "effort": "Low",
  "risk": "Low"
}
```

### Summary Report Table

| Finding ID | Category | Title | Potential Savings | Effort | Risk |
|---|---|---|---|---|---|
| WORKMAIL-WST-001 | Waste Elimination | Delete Inactive Mailboxes | $4.00 / user / month | Low | Low |
| WORKMAIL-WST-002 | Waste Elimination | Clean Up Abandoned Shared Mailboxes | $4.00 / mailbox / month | Low | Low |
| WORKMAIL-WST-003 | Waste Elimination | Empty Deleted Items & Junk Folders | Avoid $0.20 / GB-mo overages | Low | Low |
| WORKMAIL-WST-004 | Waste Elimination | Delete Unused Resource Mailboxes | Avoid licensing errors | Low | Low |
| WORKMAIL-WST-005 | Waste Elimination | Clean Up Old ActiveSync Partnerships | Indirect IT support savings | Low | Low |
| WORKMAIL-RGT-001 | Rightsizing | Replace Role-Based Mailboxes with Aliases | $4.00 / role / month | Low | Low |
| WORKMAIL-RGT-002 | Rightsizing | Consolidate Multiple Mailboxes per User | $4.00 / extra mailbox / month | Medium | Low |
| WORKMAIL-RGT-003 | Rightsizing | Enforce Mailbox Retention Policies | Avoid $0.20 / GB-mo overages | Medium | Medium |
| WORKMAIL-RGT-004 | Rightsizing | Email Forwarding for Contractors | $4.00 / contractor / month | Low | Low |
| WORKMAIL-COM-001 | Commitment Discounts | Leverage the 30-Day Free Trial | Up to $100 for first 30 days | Low | Low |
| WORKMAIL-ARC-001 | Architecture Changes | Archive Former Staff to S3 Glacier | 95% savings on inactive accounts | Medium | Low |
| WORKMAIL-ARC-002 | Architecture Changes | Automate Lifecycle via Directory Service | Prevent paying for ghost users | High | Low |
| WORKMAIL-ARC-003 | Architecture Changes | Use Amazon SES for App Emails | ~$4.00 / service account / month | Medium | Low |
| WORKMAIL-ARC-004 | Architecture Changes | Use Amazon SES + S3 for Ingestion | ~$4.00 / ingest account / month | High | Low |
| WORKMAIL-ARC-005 | Architecture Changes | Offload Large File Attachments to S3 | Avoid $0.20 / GB-mo overages | Medium | Medium |
| WORKMAIL-SCH-001 | Scheduling & Auto-Scaling | Just-in-Time Provisioning for Seasonal | $4.00 / user / idle month | Medium | Low |
| WORKMAIL-PRC-001 | Pricing Model Optimization | Monitor and Alert on Storage Overages | Prevent silent bill bloat | Low | Low |
| WORKMAIL-NET-001 | Network Optimization | Restrict IMAP/SMTP for Web Users | Incidental data transfer savings | Low | Low |
