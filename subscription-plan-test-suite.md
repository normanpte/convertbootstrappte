# Subscription Plan Module – QC Test Suite

## Overview
- Scope covers manual and automated verification of subscription plan lifecycle: creation, update, cancellation, renewal, and exception handling.
- Designed for QC team handoff; formatted for easy Markdown-to-Excel export.
- Traceability ensured through explicit linkage between business requirements (R1–R10) and each test case.

## Business Requirement Reference
| Requirement ID | Description |
| --- | --- |
| R1 | System allows creation of a new subscription plan with mandatory fields (name, code, billing cycle, price, currency, start date). |
| R2 | Plan name/code must be unique across active plans; duplicates blocked with descriptive errors. |
| R3 | Billing cycle, duration, price tiers, free-trial length, and currency follow validation rules and guardrails. |
| R4 | Plans have lifecycle statuses (Draft, Active, Suspended, Retired) with role-based transitions. |
| R5 | Users can update plan metadata and pricing with audit tracking and optional approval flow. |
| R6 | Plans can be canceled/soft-deleted; system enforces dependency checks on existing subscribers. |
| R7 | Hard deletion only allowed when no subscriptions reference the plan; otherwise action denied. |
| R8 | Plans support manual renewal/extension and scheduled auto-renewal with proration rules. |
| R9 | Payment processor failures during renewal are retried and escalated via notifications. |
| R10 | Notifications and audit trails fire on create/update/status change events. |

## Test Cases
| Test Case ID | Test Scenario | Test Steps | Test Data | Expected Result | Actual Result | Status | Remarks |
| --- | --- | --- | --- | --- | --- | --- | --- |
| SP-TC-001 | R1 – Create subscription plan with valid data | 1. Login as Product Admin.<br>2. Navigate to Subscription Plans → New Plan.<br>3. Populate all mandatory fields with valid values.<br>4. Set status to Active and click Save. | Name: `Premium Monthly`<br>Code: `PREM-MONTH`<br>Billing cycle: Monthly<br>Price: 19.99 USD<br>Start date: Today | Plan is created with unique ID, status Active, visible in list view; audit entry recorded (R1,R4,R10). |  | Not Run | Baseline happy path for plan creation. |
| SP-TC-002 | R1,R2 – Mandatory field validation | 1. Repeat creation flow but leave `Plan Name` blank.<br>2. Attempt Save. | Missing Plan Name; other fields valid. | Inline validation message, save blocked; no record created. |  | Not Run | Confirms required field enforcement. |
| SP-TC-003 | R2 – Duplicate plan code check | 1. Create plan with existing code `PREM-MONTH` while plan still active.<br>2. Save. | Name: `Premium Monthly Copy`<br>Code: `PREM-MONTH` | System rejects save, shows “Plan code already in use” error; no duplicate inserted. |  | Not Run | Ensures uniqueness constraint. |
| SP-TC-004 | R3 – Invalid billing cycle and price guardrails | 1. Enter billing cycle `0 days` and negative price.<br>2. Save. | Billing cycle: 0 Days<br>Price: -5 | Validation errors displayed for cycle and price; form not submitted. |  | Not Run | Covers numeric validation edge case. |
| SP-TC-005 | R4 – Draft to Active transition rules | 1. Create plan in Draft.<br>2. Attempt to activate as non-admin role.<br>3. Retry as Product Admin. | Draft plan `Starter Trial`. | Non-admin blocked with permission error; admin succeeds and status changes to Active; audit trail logs both attempts. |  | Not Run | Verifies role-based status transitions. |
| SP-TC-006 | R5 – Update metadata and pricing | 1. Open existing plan `Premium Monthly`.<br>2. Edit description and increase price.<br>3. Save.<br>4. Review version history. | New description text; new price 21.99 USD. | Plan updates persist; effective date/time logged; history shows previous and new values. |  | Not Run | Confirms update flow plus audit trail. |
| SP-TC-007 | R5 – Approval workflow for major price change | 1. Configure rule requiring approval when price increase >10%.<br>2. Increase price 25%.<br>3. Submit for approval.<br>4. Approver approves. | Old price 10 USD → New 12.50 USD. | Plan update enters Pending Approval state; after approval, status returns to Active and notification sent. |  | Not Run | Tests conditional workflow branch. |
| SP-TC-008 | R5,R6 – Update blocked when active subscribers exist | 1. Identify plan with 100 active subscribers.<br>2. Attempt to reduce billing cycle mid-term.<br>3. Observe response. | Plan `Enterprise Annual`. | System rejects change with message referencing active subscriptions; instructs to schedule for next cycle. |  | Not Run | Safeguards subscriber impact. |
| SP-TC-009 | R6 – Cancel plan without subscribers | 1. Choose plan with zero subscribers.<br>2. Click Cancel Plan.<br>3. Confirm action. | Plan `Legacy Beta`. | Plan moves to Retired/Cancelled, hidden from catalog; audit + notification triggered. |  | Not Run | Simple cancel path. |
| SP-TC-010 | R6 – Cancel plan with active subscribers | 1. Select plan with active users.<br>2. Attempt immediate cancel.<br>3. Choose “Cancel at next renewal”. | Plan `Premium Monthly`. | System prevents immediate cancel, offers scheduled cancel; upon confirmation sets end date to next cycle and notifies impacted customers. |  | Not Run | Ensures graceful handling when dependencies exist. |
| SP-TC-011 | R7 – Hard delete blocked when dependencies exist | 1. Attempt to delete plan referenced by archived subscriptions.<br>2. Confirm. | Plan `Starter Free`. | System blocks hard delete, displays dependency list; action aborted. |  | Not Run | Covers R7 restriction. |
| SP-TC-012 | R7 – Hard delete allowed with no references | 1. Locate plan never used.<br>2. Invoke Delete and confirm. | Plan `Test Sandbox`. | Plan removed from database, no catalog entry; audit entry notes deletion. |  | Not Run | Positive hard delete verification. |
| SP-TC-013 | R8 – Manual renewal before expiry | 1. Open subscription for plan nearing end date.<br>2. Click Renew Now.<br>3. Confirm payment. | Subscription `SUB-1001` on `Premium Monthly`. | End date extends by 1 cycle; invoice generated; plan status unchanged. |  | Not Run | Tests manual extension flow. |
| SP-TC-014 | R8 – Auto-renew executes on schedule | 1. Set plan with auto-renew flag true.<br>2. Fast-forward cron/scheduler to renewal date.<br>3. Observe job execution logs. | Plan `Enterprise Annual`. | Renewal job charges account, updates subscription end date, logs success; notification email sent. |  | Not Run | Validates scheduled automation. |
| SP-TC-015 | R9 – Renewal payment failure handling | 1. Simulate payment gateway decline during auto-renew.<br>2. Monitor retries and alerts. | Injected gateway response `Do Not Honor`. | System retries per policy (e.g., 3 attempts), marks subscription Pending Payment, sends failure notification to customer + finance. |  | Not Run | Ensures failure path coverage. |
| SP-TC-016 | R3 – Date validation (end before start) | 1. Set plan start date 01/10 and end date 01/05 same year.<br>2. Save. | Start: 10 Jan<br>End: 05 Jan | Validation error: end date must be after start; save blocked. |  | Not Run | Edge case for temporal validation. |
| SP-TC-017 | R4 – Suspend/Resume lifecycle | 1. Suspend an active plan.<br>2. Confirm plan hidden from new sales.<br>3. Resume plan. | Plan `Premium Monthly`. | Status moves to Suspended; creation of new subscriptions blocked. Resume returns to Active; events logged. |  | Not Run | Lifecycle coverage beyond cancel. |
| SP-TC-018 | Security – Unauthorized update attempt | 1. Login as Support role (read-only).<br>2. Try to edit plan price. | Support user credentials. | System denies access with 403/permission error; no audit change recorded. |  | Not Run | Ensures RBAC enforcement (ties to R4/R5). |
| SP-TC-019 | R10 – Notification dispatch on plan update | 1. Update plan description.<br>2. Check notification queue/email.<br>3. Verify payload correctness. | Plan `Enterprise Annual`. | Notification/event emitted with correct metadata (plan ID, editor, timestamp); audit log matches. |  | Not Run | Validates eventing. |
| SP-TC-020 | R10 – Audit log completeness | 1. Perform create, update, cancel in sequence on same plan.<br>2. Navigate to audit log view.<br>3. Export log. | Actions from TC001, TC006, TC010. | Audit trail shows chronological entries with actor, action, payload; export matches UI. |  | Not Run | Ensures traceability for external stakeholders. |

