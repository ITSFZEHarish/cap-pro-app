# PRO Expenses Platform

## Summary

A SAP CAP 9 (Node.js/TypeScript) enterprise expense management application with **SAP Fiori** user interfaces. The UI layer consists of eight SAP Fiori Applications built with SAPUI5 1.136 and SAP Fiori Elements (OData V4) served from the same CAP origin.

Implement the full PRD in four phases using mock integrations first and separate acceptance gates for live systems. Retain CAP 9, PostgreSQL 2, and TypeScript; verify dependency compatibility rather than copying the PRD's older package versions. Preserve the direct integration architecture without SAP Cloud Integration.

## Architecture and Services

The backend exposes eight OData V4 and REST services implemented in SAP CAP 9 (TypeScript) with server-side authorization and strict company code isolation:

| Service                       | Protocol / Path                            | CDS Definition                | Handler                               | Description                                                                                 |
| ----------------------------- | ------------------------------------------ | ----------------------------- | ------------------------------------- | ------------------------------------------------------------------------------------------- |
| **ExpenseService**            | OData V4: `/odata/v4/expenses`             | `srv/ExpenseManagement.cds`   | `srv/handlers/ExpenseManagement.ts`   | Agent expense capture, draft lifecycle, lines, attachments, calculations, and submission    |
| **OperationsService**         | OData V4: `/odata/v4/operations`           | `srv/OperationManagement.cds` | `srv/handlers/OperationManagement.ts` | Catalog CRUD, outbox operations, error queue, cash float, advances, settlements, recoveries |
| **WorkflowService**           | REST: `/wf`                                | `srv/WorkflowManagement.cds`  | `srv/handlers/WorkflowManagement.ts`  | Multi-step workflow orchestration, task lifecycle, audit trails, and delegation             |
| **AdminService**              | OData V4: `/odata/v4/adm`                  | `srv/AdminManagement.cds`     | `srv/handlers/AdminManagement.ts`     | User provisioning, role assignments, system configuration, master data, security audit      |
| **AccessControlAdminService** | OData V4: `/odata/v4/access-control/admin` | `srv/AccessControlAdmin.cds`  | `srv/handlers/AccessControlAdmin.ts`  | System administration portal and approval rule configuration                                |
| **AccessControlCFOService**   | OData V4: `/odata/v4/access-control/cfo`   | `srv/AccessControlCFO.cds`    | `srv/handlers/AccessControlCFO.ts`    | CFO financial approval queue, high-value expense reviews, recovery and budget oversight     |
| **AccessControlCEOService**   | OData V4: `/odata/v4/access-control/ceo`   | `srv/AccessControlCEO.cds`    | `srv/handlers/AccessControlCEO.ts`    | CEO executive approval queue, expense governance, and company summary                       |
| **AccessControlHRService**    | OData V4: `/odata/v4/access-control/hr`    | `srv/AccessControlHR.cds`     | `srv/handlers/AccessControlHR.ts`     | HR Head approval queue, employee recovery ledger, deductions, and salary cap oversight      |

## SAP Fiori Applications

The presentation tier comprises SAP Fiori Applications built with SAPUI5 and SAP Fiori Elements (OData V4), served directly from the CAP backend runtime:

### 1. Applications Directory & Navigation Intents

| Application Title | Module ID | Inbound FLP Intent | Framework & Pattern | Backend Service | Main Entity | Primary Target Roles | Direct URL Path |
|-------------------|-----------|--------------------|---------------------|-----------------|-------------|----------------------|-----------------|
| **Unified Launchpad** | — | `#Shell-home` | SAPUI5 Custom Portal | All Services | Multiple | All Personas | [`/LaunchPad.html`](file:///d:/github/projects/cap-pro-app/app/LaunchPad.html) |
| **Manage Expenses** | `mexpenses` | `#Expenses-OpenExpenses` | Fiori Elements (LROP) | `ExpenseService` (`/odata/v4/expenses/`) | `Expenses` (Draft) | `PRO_AGENT`, `FINANCE_APPROVER` | [`/mexpenses/index.html`](file:///d:/github/projects/cap-pro-app/app/mexpenses/webapp/index.html) |
| **Manage Expense Types** | `mexpensetypes` | `#ExpenseTypeMaster-OpenManageExpenseType` | Fiori Elements (List Report) | `OperationsService` (`/odata/v4/operations/`) | `ExpenseTypeMaster` | `CATALOG_ADMIN`, `FINANCE_APPROVER` | [`/mexpensetypes/index.html`](file:///d:/github/projects/cap-pro-app/app/mexpensetypes/webapp/index.html) |
| **User Management** | `mappusers` | `#ApplicationUsers-OpenApplicationUsers` | Fiori Elements (LROP) | `AdminService` (`/odata/v4/adm/`) | `ApplicationUsers` (Draft) | `SYSTEM_ADMIN`, `SECURITY_ADMIN` | [`/mappusers/index.html`](file:///d:/github/projects/cap-pro-app/app/mappusers/webapp/index.html) |
| **Monitoring Logs** | `mmonitoringlogs` | `#MessageMonitoringLogs-OpenMessageMonitoringLogs` | Fiori Elements (List Report) | `AdminService` (`/odata/v4/adm/`) | `SecurityAuditEvents` | `INTEGRATION_OPS`, `SYSTEM_ADMIN` | [`/mmonitoringlogs/index.html`](file:///d:/github/projects/cap-pro-app/app/mmonitoringlogs/webapp/index.html) |

### 2. Application Architecture & Functional Modules

#### A. Manage Expenses (`mexpenses`)
- **Floorplan**: List Report Object Page (LROP) with draft orchestration (`@odata.draft.enabled: true`).
- **Core Capabilities**:
  - Personal expense tracking with server-filtered company code scope (`$user.companyCode`).
  - Draft creation and editing with line items composition (`_Items`), calculating company and employee splits using server-side decimal math.
  - Multi-attachment uploads (`_Attachments`) with client-side drag-and-drop and server-side SHA-256 hash calculation, malware status tracking, and private storage.
  - Contextual expense notes (`_Notes`) supporting localized audit communication.
  - Bound transactional actions: `submit()`, `withdraw()`, `copyExpense()`, `requestReversal()`, and `overrideDeductibility()`.

#### B. Manage Expense Types (`mexpensetypes`)
- **Floorplan**: List Report with field-level semantic criticality.
- **Core Capabilities**:
  - Full lifecycle maintenance of EX01–EX31 and CC01–CC08 expense type catalog definitions.
  - Configuration of recoverability flags, high-risk classifications, attachment mandatoriness, and Noqodi payment eligibility.
  - General Ledger mapping rules: GL Expense Account, GL Receivable Account, Tax Code, and Default Cost Center.
  - Salary recovery policy parameters (wage type and salary cap applicability).

#### C. User Management (`mappusers`)
- **Floorplan**: List Report Object Page (LROP) with child entity composition (`roles`).
- **Core Capabilities**:
  - Employee and agent user provisioning, identity details, and company code assignments.
  - Persona role collection assignments (`UserRoleAssignments`) connecting users to BTP role collections.
  - Administrative security operations: `lockUser()`, `unlockUser()`, and `resetPassword()`.

#### D. Monitoring Logs (`mmonitoringlogs`)
- **Floorplan**: List Report with analytical filtering and message parsing.
- **Core Capabilities**:
  - Operational audit logging of all system and security events (`SecurityAuditEvents`).
  - Interface error tracking (`InterfaceErrorQueue`) with diagnostic messages, error codes, and associated outbox operations.
  - Actions for support engineers: `retry()`, `assignOwner()`, and `resolveError()`.

#### E. Persona Approval Cockpits & Governance Views
The platform serves specialized role-filtered cockpits powered by dedicated access control endpoints:
- **CFO Financial Cockpit** (`/odata/v4/access-control/cfo`): Executive approval queue, high-value expense scrutiny (lines $\ge$ AED 10,000, totals $\ge$ AED 25,000), recovery ledger oversight, and budget variance alerts.
- **CEO Executive Cockpit** (`/odata/v4/access-control/ceo`): Executive approval queue, company-wide expense oversight, and high-level audit summaries.
- **HR Head Cockpit** (`/odata/v4/access-control/hr`): Staff expense approvals, deductibility override handling, employee recovery ledger, and payroll deduction execution tracking.
- **Admin Configuration Cockpit** (`/odata/v4/access-control/admin`): System configuration, security audits, and multi-tier approval workflow rule definitions.

#### F. Unified Enterprise Launchpad (`LaunchPad.html`)
- **Technology**: Standalone SAPUI5 launchpad utilizing `sap_horizon` styling and responsive CSS grid.
- **Security & Usability**:
  - Anti-inspect & POS kiosk hardening (context menu and developer shortcut suppression).
  - Dynamic role context switcher and session authentication gate (`/Login.html`).
  - Language toggle supporting English (LTR) and Arabic (RTL) (`sap-language=AR`).
  - Real-time tile metrics and direct intent navigation to all Fiori Elements apps.

### 3. Local Development & Launch Instructions

1. **Start the CAP Server**:
   ```bash
   cds watch
   # or
   npx cds s
   ```
   `cds serve` / `cds watch` load `server.ts` through `tsx`, which applies `tsconfig.json` `paths` at runtime. Keep `@sap/cds` mapped to `./node_modules/@cap-js/cds-types` (the package directory from `cds add typescript`). Mapping it to `dist/cds-types.d.ts` makes Node 22+ fail with `ERR_UNSUPPORTED_NODE_MODULES_TYPE_STRIPPING` while bootstrapping `server.ts`. Full write-up: `doc/known-issues.md`.
2. **Access Applications in Browser**:
   - **Launchpad**: [http://localhost:4004/LaunchPad.html](http://localhost:4004/LaunchPad.html)
   - **Manage Expenses**: [http://localhost:4004/mexpenses/index.html](http://localhost:4004/mexpenses/index.html)
   - **Manage Expense Types**: [http://localhost:4004/mexpensetypes/index.html](http://localhost:4004/mexpensetypes/index.html)
   - **User Management**: [http://localhost:4004/mappusers/index.html](http://localhost:4004/mappusers/index.html)
   - **Monitoring Logs**: [http://localhost:4004/mmonitoringlogs/index.html](http://localhost:4004/mmonitoringlogs/index.html)

## Implementation changes

### 1. Foundation, data, and security

- Model expenses and lines, versioned EX01-EX31 and CC01-CC08 catalogs, attachments, approval tasks and decisions, audit events, recovery schedules and executions, floats and movements, advances, settlement batches and rows, outbox operations, and interface errors.
- Use UUID technical keys with server-generated PRX-YYYY-NNNNNN business identifiers. Store canonical SAP company codes such as 1000; show labels such as CC-1000.
- Configure SQLite for local development and PostgreSQL for production and integration tests. Add database constraints, indexes, migration scripts, and controlled reference-data loading.
- Implement seven role collections through XSUAA with IAS federation. Enforce agent ownership and company scope server-side, including attachments, actions, exports, and operational data.
- Keep audit records immutable and sensitive payloads restricted and redacted. Production authentication must fail closed.

### 2. Expense capture, approvals, and eight Fiori screens

- Deliver the launchpad, expense list, create/edit object page, approval cockpit, errors cockpit, reconciliation workspace, expense catalog maintenance, and recovery overview. Use Fiori Elements with custom sections where required.
- Expose OData V4 projections and explicit actions for submit, copy, withdraw, approve, return, reject, deductibility override, request reversal, retry, owner assignment, and exception resolution.
- Calculate totals and employee/company splits server-side using decimal arithmetic. Validate employee assignments, effective catalog versions, accounting mappings, consent, payment source, and attachment requirements at submission.
- Snapshot catalog and routing parameters at submission. Apply escalation for any line >= AED 10,000, request total >= AED 25,000, any catalog high-risk flag, or deductibility override.
- Approve entire requests sequentially: Finance for standard requests; HR Head then Finance then CFO for escalated requests. Line comments and flags support review without partial posting.
- Require comments for returns and rejections. Prevent self-approval, repeated decisions, and unauthorized state transitions using transactional concurrency checks.
- Persist task deadlines and reminders. Default calendar: Monday-Friday, 09:00-18:00, Asia/Dubai; support configured holidays. Count 48 working hours per level, without automatic approval.
- Store attachment bytes privately, compute SHA-256 server-side, scan before use, and block submission while scanning or when infected. Identify duplicate documents and normalized receipt references across requests, including concurrent submissions; allow an attachment to support multiple lines within its owning request.

### 3. Accounting, recovery, and reconciliation

- Build typed adapters for S/4 posting, reversal and balances; SuccessFactors employee and compensation reads; payroll instructions and execution read-back; malware scanning; and Noqodi SFTP ingestion.
- Use contract fixtures locally. Require actual API metadata before activating adapters; validate the journal API's protocol and payroll deduction entity instead of assuming the PRD's sample endpoints.
- Commit final approval and its outbound operation together. Dispatch remotely after commit through a PostgreSQL-backed outbox with operation identifiers, worker leases, attempt history, six-hour automatic retry, and authorized manual retry.
- Reconcile ambiguous remote outcomes before resending. Mark posting complete only after verified remote confirmation; never fabricate document numbers or salary values. Keep accounting and payroll integration states separate.
- Generate balanced entries from employee/company shares and configured payment-source accounts. Implement approved reversals with recovery cancellation or compensation and retained document links.
- Treat 20% as an initial configurable company recovery policy. Aggregate available capacity across all PRO debts for each employee/payroll period; incorporate validated eligibility, consent, other deductions, and approved legal limits.
- Allocate recoveries oldest-first and carry arrears forward. Record instructions separately from execution; update recovered balances only from confirmed payroll results.
- Implement persistent cash-float movements, S/4 balance comparisons, cash returns, top-ups, advance aging, and settlement allocations.
- Parse Noqodi files with a proper CSV parser and configurable contract. Verify SFTP host keys, deduplicate files and rows, persist malformed-row errors, and archive files only after durable ingestion.
- Match settlements by company, wallet, currency, and transaction reference. Show unmatched and ambiguous rows, highlight variances above AED 100, and escalate unmatched entries after five calendar days.

### 4. Delivery and operations

- Add build, type-check, test, PostgreSQL verification, and MTA packaging commands, plus CI checks and deployment descriptors.
- Configure Destination Service, authentication, PostgreSQL, private attachment storage, scanner, and SFTP secrets through service bindings or secure configuration.
- Provide health checks, integration correlation identifiers, queue monitoring, backups, recovery procedures, and operator runbooks.
- Replace the starter README and add the maintained PRD, gap matrix, architecture, integration contracts, configuration guide, and acceptance checklist.
- Deliver in order: foundation/security then capture/approval UI then accounting/recovery adapters then reconciliation, hardening, and pilot readiness. Live deployment remains a separate acceptance step.

## Test and acceptance plan

- Verify all 39 catalog codes, effective-date snapshots, common-code behavior, splits, consent, overrides, and routing boundaries.
- Test every role, ownership/company restriction, whole-request transition, concurrent approval, withdrawal, return/resubmission, reversal, and working-calendar deadline.
- Test duplicate uploads and references, shared receipts within a request, malware failures, and unauthorized file access.
- Use PostgreSQL tests for constraints, decimal calculations, transactional outbox creation, concurrent workers, retry recovery, and duplicate prevention.
- Exercise remote timeouts after acceptance, invalid mappings, closed posting periods, payroll rejection, missing compensation, and execution read-back.
- Verify aggregate recovery caps across multiple expenses, arrears allocation, salary changes, and reversal compensation.
- Test quoted CSV fields, malformed rows, repeated files, ambiguous matches, variance thresholds, unmatched aging, and archive failures.
- Run browser acceptance journeys across all eight SAP Fiori screens. Require staging evidence for real posting/reversal, payroll execution, Noqodi ingestion, and S/4 reconciliation before production release.

## Confirmed assumptions and release prerequisites

- Full scope delivered in phases; mock integrations first.
- Whole-expense approvals and posting; no independent line approval.
- AED-only initial release, one organization, multiple company codes.
- Configurable recovery policy with live deductions gated by approved eligibility and legal/payroll validation.
- Production requires confirmed tenant API contracts, account and wage mappings, identity/company assignments, Noqodi sample files, holiday calendars, and provisioned BTP services.
- The PRD's blanket statutory 20% assertion must be corrected: Article 25 distinguishes deduction categories and includes an aggregate limit.
