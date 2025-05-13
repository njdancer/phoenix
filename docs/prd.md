# Phoenix Product Requirements Document (PRD)

_Version: 1.0_
_Date: 2025-05-14_
_Author: PM-V3-BETA (in collaboration with User)_

## 1. Goal, Objective and Context

**Goal:** To develop "Phoenix," a robust, reliable, and maintainable personal finance tool. Initially, Phoenix will serve as a bridge between Up Bank and YNAB, automating transaction synchronization. It is designed with a clean architecture to be an extensible platform for future personal finance hobby projects.

**Objective:**

- Replace an existing rudimentary and unreliable custom solution for syncing Up Bank transactions to YNAB.
- Significantly reduce the manual effort and time spent by the user on reconciling Up Bank transactions within YNAB.
- Provide a reliable foundation, including debugging tools and a cleared balance calculator, to ensure data accuracy and simplify troubleshooting.
- Achieve high accuracy (99.9%) in transaction syncing and reduce debugging time by 90% within the first few months of use.
- Implement basic SMS notifications for critical system events.

**Context:**
The primary user faces a lack of official YNAB integration for Up Bank in their country. Their current custom solution is error-prone and difficult to maintain. This leads to a cumbersome and time-consuming manual process for budgeting, particularly for calculating the "cleared balance" required by YNAB, as Up Bank only provides a "working balance" (available balance). Phoenix aims to solve these issues by providing a streamlined, automated, and reliable solution, ultimately improving the user's engagement with their budgeting process. The project also serves as a personal development platform for exploring future financial tools.

## 2. Success Metrics

The success of the Phoenix MVP will be evaluated based on the following key metrics, aligned with the project's core goals of reliability, improved workflow, and foundational stability for future development. It is a fundamental principle that any tools or techniques employed for the purpose of measuring these metrics must be privacy-preserving by design (e.g., avoiding services like Google Analytics or similar invasive trackers) and should prioritize local or self-hosted analytics solutions where feasible if automated data collection is pursued. Many of these initial metrics will rely on direct user observation and self-reporting, especially for workflow efficiencies.

1.  **System Reliability & Accuracy:**

    - **Transaction Sync Accuracy:** 99.9% of transactions (creations, clearances, cancellations) from Up Bank are accurately reflected in YNAB without manual intervention within 24 hours of the event.
    - **Error Rate:** Less than 0.5% of webhook events result in processing errors requiring manual investigation via the debugging dashboard.
    - **System Uptime (Core Sync Functionality):** The core transaction syncing mechanism maintains 99.5% uptime (excluding planned maintenance or outages of external services like Up Bank or YNAB APIs).

2.  **Workflow Efficiency & User Time Savings:**

    - **Time Spent on YNAB Reconciliation:** Reduce time spent by the user on manually correcting or debugging YNAB transaction data by 90% compared to the previous solution (as measured by user self-reporting over a 1-month period post-MVP deployment).
    - **Time to Resolve Sync Discrepancies:** Any identified sync discrepancies can be diagnosed and resolved using the debugging dashboard and cleared balance calculator within 15 minutes.
    - **Manual "Cleared Balance" Calculation Aided:** The "Balance Reconciliation Calculator" effectively aids the user in understanding and reconciling differences between Up Bank's available balance and YNAB's cleared balance.

3.  **Operational Performance:**

    - **Transaction Latency:** 95% of Up Bank webhook events are processed, and the corresponding transaction is created/updated in YNAB within 5 minutes of webhook receipt under normal operating conditions.
    - **Cost Efficiency:** Operational costs for Cloudflare services (Workers, KV, D1, R2 if used) and SMS provider remain within the targeted soft cap of $25 AUD/month for each service, ideally leveraging free tiers as much as possible.

4.  **Adoption & Usability (for Admin/Debugging Tools):**

    - **Debugging Dashboard Usage:** The debugging dashboard is utilized as the primary tool for investigating any transaction sync issues (measured by user behavior).
    - **System Status Dashboard Confidence:** The system status dashboard provides a clear and trusted overview of the system's health, reducing the need for direct log checking for routine status inquiries.
    - **User Authentication Success Rate:** >99% success rate for logins to the UI dashboards using SMS OTP and Passkey methods.

5.  **Foundation for Future Development:**
    - **Modularity & Extensibility:** The Nx Monorepo structure and defined libraries are successfully established, demonstrating clear separation of concerns that would allow for the addition of a new, distinct feature with an estimated 20% less foundational setup effort than starting from scratch (qualitative assessment by the developer after MVP).
    - **Code Maintainability:** The codebase adheres to defined linting rules and architectural patterns, facilitating easier onboarding for potential future collaborators (if open-sourced) or for the original developer to extend functionality after a period away from the code.

## 3. Functional Requirements (MVP)

This section details the core functionalities that the Phoenix MVP must deliver.

**FR1: Up Bank Transaction Ingestion & Processing**

- FR1.1: The system must securely receive webhook notifications from Up Bank for new transactions (creations, updates including clearances, and potentially cancellations if provided by the API).
- FR1.2: The system must parse and transform the incoming Up Bank transaction data into a format suitable for YNAB. This includes:
  - Mapping Up Bank transaction categories/descriptions to appropriate YNAB payee names or memo fields as per user configuration.
  - Correctly identifying transaction amounts and types (debit/credit).
  - Handling timestamps and converting them appropriately.
- FR1.3: The system must be able to retrieve additional transaction details from Up Bank via its API if the webhook payload is insufficient.
- FR1.4: The system must robustly handle potential errors during webhook reception or Up Bank API interaction, with appropriate logging and retry mechanisms where applicable.

**FR2: YNAB Transaction Creation & Management**

- FR2.1: The system must securely authenticate with the YNAB API using the user's Personal Access Token.
- FR2.2: The system must create new transactions in the specified YNAB budget and account based on the processed Up Bank data.
- FR2.3: The system must update existing transactions in YNAB if Up Bank provides updates (e.g., when a transaction clears). This includes mechanisms to reliably identify the corresponding YNAB transaction.
- FR2.4: The system must accurately set the "cleared" status of transactions in YNAB based on information from Up Bank.
- FR2.5: The system must handle potential errors during YNAB API interactions with appropriate logging.

**FR3: System Configuration & Secrets Management**

- FR3.1: The user must be able to securely configure and store essential credentials via Cloudflare Secrets (set via `wrangler` CLI for MVP), including: Up Bank API token, YNAB Personal Access Token, YNAB Budget ID, YNAB Account ID (for sync), SMS provider API credentials, GitHub Webhook secret.
- FR3.2: The system should allow for basic mapping rules or preferences for how Up Bank transaction data is translated into YNAB transaction fields (MVP: simple rules, potential for future UI config).
- FR3.3: The selected Up Bank Account ID for the Balance Reconciliation Calculator must be configurable via a UI element and stored securely (e.g., in Cloudflare KV).

**FR4: Debugging Dashboard & System Monitoring**

- FR4.1: A web-based debugging dashboard must be available, accessible after user authentication.
- FR4.2: The dashboard must display a log of recent webhook events received from Up Bank and the corresponding actions taken by Phoenix.
- FR4.3: The dashboard must allow the user to view the payload of incoming webhooks and the transformed data sent to YNAB for specific events.
- FR4.4: The dashboard must provide a clear indication of processing errors and any relevant error messages or context.
- FR4.5: A system status overview page/section within the dashboard must display the general health of integrations.
- FR4.6: The dashboard must also display critical events received from GitHub (CI/CD failures, specified security alerts).

**FR5: Balance Reconciliation Calculator (Account HUD)**

- FR5.1: A web-based tool, accessible after user authentication, must be available to help reconcile Up Bank's available balance with YNAB's cleared balance.
- FR5.2: The calculator must fetch the necessary balances (Up Bank available balance, YNAB cleared balance) for the user-configured accounts.
- FR5.3: The calculator must display these balances, their difference, and a list of recent Up Bank transactions with a best-effort indication of their status in YNAB to aid reconciliation.

**FR6: SMS Notifications**

- FR6.1: The system must send SMS notifications for: critical transaction syncing errors, and critical alerts from GitHub (CI/CD `main` branch failures, specified security alerts).
- FR6.2: SMS content should be concise, informative, and include deep links to relevant content (e.g., dashboard, GitHub workflow).

**FR7: User Authentication for UI Components**

- FR7.1: Secure user login to web-based UI components (Dashboard, Calculator, Settings).
- FR7.2: Support SMS-based One-Time Password (OTP) login.
- FR7.3: Support Passkey login (WebAuthn).
- FR7.4: No traditional password-based authentication for MVP.
- FR7.5: Securely manage user authentication factors.

**FR8: GitHub Integration for Operational & Security Alerting**

- FR8.1: Provide a secure webhook endpoint to receive notifications from GitHub Actions.
- FR8.2: Validate incoming webhooks from GitHub.
- FR8.3: Upon `main` branch CI/CD pipeline failure (build, test, or deploy stages), log the event and trigger an SMS alert.
- FR8.4: (Highly Desirable for MVP; Architect to assess complexity) If implemented, upon new critical/high severity Dependabot alert, log and trigger SMS alert.
- FR8.5: (Highly Desirable for MVP; Architect to assess complexity) If implemented, upon new secret scanning alert, log and trigger immediate SMS alert.

## 4. Non-Functional Requirements (MVP)

1.  **Performance:**
    - NFR1.1 (Transaction Processing Latency): 95% of Up Bank webhook events processed, and corresponding YNAB transactions created/updated, within 5 minutes of webhook receipt under normal conditions.
    - NFR1.2 (UI Responsiveness): Web-based UI components load and respond within 3 seconds for common operations.
2.  **Reliability & Availability:**
    - NFR2.1 (Core Sync Uptime): Core transaction syncing functionality achieves 99.5% uptime, excluding external dependencies.
    - NFR2.2 (Error Handling & Recovery): Robust error handling for external APIs, including retries where appropriate. Critical errors trigger SMS alerts.
    - NFR2.3 (Data Integrity): Minimize data duplication/corruption; favor idempotency.
3.  **Security:**
    - NFR3.1 (Credential Management): API keys/sensitive config stored securely via Cloudflare Secrets.
    - NFR3.2 (Authentication for UI): Access to web UIs protected by in-app authentication (SMS OTP, Passkey).
    - NFR3.3 (Transport Security): HTTPS for all external communication.
    - NFR3.4 (Principle of Least Privilege): API tokens configured with minimum necessary scope.
    - NFR3.5 (Webhook Security): GitHub webhook endpoint secured (e.g., signature validation).
4.  **Maintainability & Extensibility:**
    - NFR4.1 (Code Quality): Well-commented code, consistent styling (ESLint/Prettier), adherence to Nx Monorepo patterns, automated checks (pre-commit/CI).
    - NFR4.2 (Modularity): Modular design within Nx Monorepo.
    - NFR4.3 (Configuration Driven): Key operational parameters configurable where practical.
    - NFR4.4 (Logging): Comprehensive, structured logging for key events, errors, processing steps.
    - NFR4.5 (Git Commit Standards): Commit messages adhere to a defined convention (e.g., Conventional Commits), enforced by linting tools.
    - NFR4.6 (Development Workflow Support): Repository/CI setup supports story-centric branching, frequent small commits. Full checks before merging to `main`.
5.  **Usability (for Admin/Debugging Tools):**
    - NFR5.1 (Clarity & Intuitiveness): Dashboards/tools are clear, intuitive for the technical user.
    - NFR5.2 (Actionable Information): Information in dashboards is actionable for diagnosis.
6.  **Cost-Efficiency:**
    - NFR6.1 (Operational Costs): Aim to stay within free tiers or soft budget caps ($25 AUD/month per major service like Cloudflare, AWS SNS).
7.  **Privacy:**
    - NFR7.1 (Data Handling): Only necessary data processed/temporarily stored. No long-term storage of full transaction history by Phoenix beyond what's essential for MVP.
    - NFR7.2 (Analytics & Monitoring): Any measurement tools must be privacy-preserving.
8.  **Deployability:**
    - NFR8.1 (CI/CD Automation): CI/CD via GitHub Actions.
    - NFR8.2 (Automated Deployments & Environments):
      - Pushes to `main` trigger CI. `main` branch not protected against direct pushes.
      - Deployment to `dev.phoenix.dncr.me` from `main` _only_ if all automated CI checks pass.
      - Manual promotion to production (`phoenix.dncr.me`) from validated `dev` environment.
      - Other branches auto-deploy to ephemeral preview environments (e.g., `<branch-name>.dev.phoenix.dncr.me`).
    - NFR8.3 (CI Pipeline Stages & Quality Gates):
      - `main` branch pipeline: mandatory linting, formatting, all tests (unit, integration), successful build.
      - Feature branch pipelines: optimized for speed (e.g., focus on linting, unit tests, build).
      - Pipeline optimized for rapid feedback.
9.  **Testability:**
    - NFR9.1 (Unit & Integration Tests): Key business logic, data transformations, API interaction modules covered.
    - NFR9.2 (Local Development & Testing): Support local execution/testing, simulation of webhooks, API interaction testing. CLI commands for local testing.

## 5. User Interaction and Design Goals

This section outlines the high-level vision for Phoenix's web-based UI components (Login Screen, Debugging Dashboard, Balance Reconciliation Calculator, System Status Dashboard, Configuration/Settings Page).

1.  **Overall Vision & Experience:**
    - **Aesthetic:** Clean, modern, information-dense without overusing white space. Efficient, leveraging space effectively. Hint of "neon underglow" for a polished, professional developer-tool feel.
    - **Analogy:** UniFi dashboards (loose analogue for professional, data-rich feel).
    - **User Experience:** Efficient, powerful, enabling quick task accomplishment with confidence.
2.  **Key Interaction Paradigms:**
    - **Data-Driven Interactions:** Tailor interaction types to data for natural feel.
    - **Copy to Clipboard:** Available for useful data (error messages, IDs, balances).
    - **Debugging Dashboard:** Clear logs, event details, filtering/searching.
    - **Balance Reconciliation Calculator:** Straightforward balance display, minimal input.
    - **Configuration Page:** Secure, clear interface for API keys (view only if set via CLI for some), Up Bank account selection for calculator, YNAB budget/account IDs.
3.  **Core Screens/Views (Conceptual):**
    - Login Screen (SMS OTP & Passkey)
    - Debugging Dashboard
    - Balance Reconciliation Calculator (Account HUD)
    - System Status Dashboard
    - Configuration/Settings Page
4.  **Accessibility Aspirations:**
    - **Target Standard:** Aspirational WCAG 2.2 Level AA. Pragmatic approach if technically prohibitive for MVP.
    - **Font Sizing:** Use correct CSS units (e.g., `rem`, `em`) for browser font size modification viability.
5.  **Responsiveness & Mobile Experience:**
    - **Baseline Mobile Usability:** All UI components functional/usable on mobile.
    - **Optimized vs. Utilitarian:** Data-heavy screens may offer a sub-optimal (but functional) experience on small viewports.
    - **Progressive Disclosure/Slimmed-Down Mobile Views:** Consider for data-heavy screens.
    - **Access to Full Functionality:** Mobile users must always be able to access full desktop functionality, even if it's via an "escape hatch" to an unoptimized view.

## 6. Technical Assumptions

- **Platform:** Cloudflare Workers (for compute, frontend serving via Vite+React Router app, and backend logic), Cloudflare KV (for user configuration like selected Up Bank account, sync state, short-term caching), Cloudflare D1 (potential for structured data like detailed logs if needed, with Drizzle ORM preference).
- **Repository & Service Architecture:** Nx Monorepo. Primary application `phoenix-app` (Vite + React Router in "framework mode" with data routers/Vite plugin) capable of full-stack functionality, deployed as a single Cloudflare Worker. The Architect will determine if any specific backend logic warrants separation into distinct worker services later.
- **Integration Design Philosophy:** Evaluate a pluggable adapter pattern for Up Bank and YNAB integrations to facilitate future extensibility.
- **Frontend Stack:** Vite, React, React Router (framework mode with data routers and Vite plugin), Tailwind CSS v4, Shadcn/UI (configured with CSS variables), Storybook v9 (beta target for component development).
- **Code Hosting & CI/CD:** GitHub repository, GitHub Actions for CI/CD.
- **Security:**
  - API keys (Up Bank, YNAB, SMS provider, GitHub Webhook secret) managed via Cloudflare Secrets (set using `wrangler secret put` for MVP).
  - In-application authentication for UI components: SMS OTP and Passkey login (no traditional passwords for MVP).
  - GitHub Webhook endpoint secured via signature validation.
- **SMS Integration:** Third-party SMS API (user preference for AWS SNS if feasible and cost-effective).
- **Configuration:**
  - Core secrets via Cloudflare Secrets.
  - Target YNAB Budget ID and Account ID for sync via Cloudflare Secrets (MVP).
  - Target Up Bank Account ID for the Balance Reconciliation Calculator configured via a UI dropdown (Story 4.1) and stored in Cloudflare KV.
- **Testing Requirements:**
  - Vitest as the primary testing framework.
  - Unit tests for modules/functions/components.
  - Integration tests for interactions between parts of the system.
  - Consideration for E2E testing of UI components using Vitest's browser mode.
  - Testing scope: backend logic (Workers), data transformations, API interactions (mocked), frontend components.
  - Support for local development and testing, including CLI commands where appropriate.

## 7. Epic Overview

This section outlines the high-level Epics and their User Stories for the MVP.

---

**Epic 1: Foundational Setup & Initial Deployment Pipeline**

- **Description:** Establish the core project structure, version control, basic Vite + React Router application shell (`phoenix-app`) capable of full-stack functionality, and a CI/CD pipeline capable of deploying this application as a Cloudflare Worker to the `dev.phoenix.dncr.me` environment.
  - **Story 1.1: Initialize Project Repository & Nx Monorepo for a Full-Stack Capable Vite/React Router Application**
    - As a Developer, I want a new Git repository initialized with an Nx Monorepo structure, configured to support a Vite + React Router application (e.g., `phoenix-app`) capable of full-stack functionality (UI serving and server-side logic via React Router handlers), deployable to Cloudflare Workers. The monorepo should also permit the future addition of distinct backend worker services if deemed necessary by the Architect.
    - **Acceptance Criteria (ACs):**
      1.  A new Git repository is created (e.g., on GitHub).
      2.  The repository is initialized as an Nx Monorepo.
      3.  The Nx Monorepo includes an application generated for a Vite + React Router frontend (e.g., `phoenix-app`), configured with the `@cloudflare/vite-plugin` (or equivalent) for Cloudflare Worker deployment and local development simulation.
      4.  A `README.md` file is present with basic project information and setup instructions for the `phoenix-app`.
      5.  Linting (ESLint) and formatting (Prettier) tools are configured and operational across the monorepo.
  - **Story 1.2: Develop Minimal Vite + React Router Shell Application with Basic Server-Side Handler**
    - As a Developer, I want a minimal `phoenix-app` (Vite + React Router) shell running within the Nx Monorepo, which includes a basic client-side route and a simple server-side request handler (e.g., for a health check or echo endpoint) leveraging React Router's server-side capabilities, so that I have a deployable full-stack-capable application target.
    - **ACs:**
      1.  The `phoenix-app` can be built successfully within the Nx Monorepo (e.g., `nx build phoenix-app`).
      2.  The `phoenix-app` can be run locally (e.g., `nx serve phoenix-app`), with the Vite development server accurately simulating the Cloudflare Workers environment (including execution of server-side route logic).
      3.  React Router is configured with at least one basic client-side route (e.g., `/`) displaying a simple placeholder page.
      4.  A simple server-side handler is implemented within the `phoenix-app` structure (e.g., at `/api/health`) that returns a basic success response (e.g., JSON `{"status": "ok"}`).
      5.  Tailwind CSS and shadcn/ui are integrated into the `phoenix-app`, and basic styling can be applied to the placeholder page.
  - **Story 1.3: Configure Basic CI/CD Pipeline for Development Environment (Deploying `phoenix-app` to Worker)**
    - As a Developer, I want a basic CI/CD pipeline (using GitHub Actions) configured for the `main` branch, so that code changes to `phoenix-app` are automatically built, linted, tested (with placeholder tests initially), and deployed as a Cloudflare Worker to the development environment.
    - **ACs:**
      1.  A GitHub Actions workflow is created that triggers on pushes to the `main` branch.
      2.  The workflow checks out the code, sets up Node.js, and installs dependencies.
      3.  The workflow runs linting checks for the `phoenix-app` and any other relevant parts of the monorepo; the pipeline fails if linting errors occur.
      4.  The workflow runs the build command for `phoenix-app` (packaging it for Worker deployment); the pipeline fails if the build fails.
      5.  (Placeholder for tests) The workflow includes a step for running tests (e.g., `nx test phoenix-app`); initially, these tests can be placeholder/passing tests. The pipeline fails if tests fail.
      6.  If all previous steps pass, the `phoenix-app` (including its client-side assets and server-side logic) is deployed to a Cloudflare Worker service at `dev.phoenix.dncr.me`.
      7.  The deployed `phoenix-app` shell (from Story 1.2) is accessible and functional via `dev.phoenix.dncr.me`, serving the client-side UI.
      8.  The integrated server-side handler (e.g., `/api/health` from Story 1.2) in the deployed `phoenix-app` is accessible and returns the expected response.

---

**Epic 2: Secure Credential Management**

- **Description:** Implement a secure way to store and manage API tokens for Up Bank and YNAB (and other services like SMS provider, GitHub webhook secret), ensuring they are encrypted at rest and handled safely. For MVP, these will be set by the user via the `wrangler` CLI.
  - **Story 2.1: Securely Store and Access Up Bank API Token**
    - As the System (Phoenix App), I need to securely store and access the Up Bank API token, so that I can make authenticated calls to the Up Bank API on behalf of the user.
    - **ACs:**
      1.  A Cloudflare Worker secret is designated for storing the Up Bank API token (e.g., `UP_BANK_API_TOKEN`).
      2.  Clear instructions are documented for the user on how to set this secret using `wrangler secret put UP_BANK_API_TOKEN`.
      3.  Server-side logic within `phoenix-app` can securely access the value of `UP_BANK_API_TOKEN`.
      4.  The token's value is not exposed to client-side code or logged insecurely.
      5.  A basic server-side function within `phoenix-app` uses the token to attempt a simple read-only Up Bank API call (e.g., list accounts) to confirm validity and log the outcome securely.
  - **Story 2.2: Securely Store and Access YNAB API Token**
    - As the System (Phoenix App), I need to securely store and access the YNAB API token, so that I can make authenticated calls to the YNAB API on behalf of the user.
    - **ACs:**
      1.  A Cloudflare Worker secret is designated for storing the YNAB API token (e.g., `YNAB_API_TOKEN`).
      2.  Clear instructions are documented for setting this secret using `wrangler secret put YNAB_API_TOKEN`.
      3.  Server-side logic within `phoenix-app` can securely access the value of `YNAB_API_TOKEN`.
      4.  The token's value is not exposed to client-side code or logged insecurely.
      5.  A basic server-side function within `phoenix-app` uses the token to attempt a simple read-only YNAB API call (e.g., list budgets) to confirm validity and log the outcome securely.
  - **(Implicit Story within this Epic): Securely Store and Access Other Necessary Secrets**
    - Similar secure storage and access mechanisms (Cloudflare Secrets set via `wrangler` CLI for MVP) will be implemented for:
      - YNAB Budget ID (e.g., `YNAB_BUDGET_ID`)
      - YNAB Account ID for sync (e.g., `YNAB_SYNC_ACCOUNT_ID`)
      - SMS Provider API Key (e.g., `SMS_PROVIDER_API_KEY`)
      - GitHub Webhook Secret (e.g., `GITHUB_WEBHOOK_SECRET`)
    - Documentation for setting each of these secrets will be provided.
    - Server-side logic in `phoenix-app` will access these as needed.

---

**Epic 3: Automated Up Bank to YNAB Transaction Syncing**

- **Description:** Develop the core functionality for ingesting transaction data from Up Bank, processing it, and accurately creating corresponding transactions in YNAB. This includes handling different transaction types, ensuring data integrity, and implementing robust error handling.
  - **Story 3.1: Fetch New Transactions from Up Bank API**
    - As the System (Phoenix App), I need to fetch new (unprocessed) transactions from the Up Bank API using the stored credentials, so that they can be prepared for syncing to YNAB.
    - **ACs:**
      1.  Uses `UP_BANK_API_TOKEN`.
      2.  Retrieves transactions since the last successful sync point (state management in Story 3.5). If no prior state, fetches from a configurable initial lookback period (e.g., 7 days default).
      3.  Correctly handles Up Bank API pagination.
      4.  Logs success/failure and number of transactions fetched.
      5.  Fetched transactions are available for transformation.
  - **Story 3.2: Configure Target YNAB Budget and Account for Syncing**
    - As the User, I need to be able to specify my target YNAB Budget ID and YNAB Account ID for transaction syncing, so that transactions from Up Bank are sent to the correct destination within my YNAB.
    - **ACs:** (Covered by "Implicit Story" in Epic 2 - `YNAB_BUDGET_ID` and `YNAB_SYNC_ACCOUNT_ID` are set as Cloudflare Secrets).
      1.  Server-side logic within `phoenix-app` securely accesses these configured IDs.
      2.  These IDs are used when pushing transactions to YNAB.
  - **Story 3.3: Transform Up Bank Transactions to YNAB API Format**
    - As the System, I need to transform the fetched Up Bank transactions into the data format required by the YNAB API, including generating unique import IDs, so that they can be correctly imported into YNAB.
    - **ACs:**
      1.  Maps Up Bank fields (Date, Description/Payee, Memo, Amount) to YNAB fields.
      2.  Amounts converted to YNAB's milliunits.
      3.  Unique, deterministic `import_id` generated for each YNAB transaction (e.g., `UP:${up_bank_transaction_id}`).
      4.  Handles settled transactions. Basic memo enrichment applied (e.g., prefix "[Up]").
  - **Story 3.4: Push Transformed Transactions to YNAB API**
    - As the System, I need to push the transformed transactions to the specified YNAB budget and account using stored credentials and import IDs, so my YNAB register is updated.
    - **ACs:**
      1.  Uses `YNAB_API_TOKEN`, `YNAB_BUDGET_ID`, `YNAB_SYNC_ACCOUNT_ID`.
      2.  Sends transactions to YNAB API (e.g., `POST /budgets/{budget_id}/transactions`).
      3.  Handles YNAB API responses (success, duplicates, errors), logging outcomes.
      4.  Notes successfully imported transactions/IDs for state update.
  - **Story 3.5: Manage Sync State (Last Synced Point)**
    - As the System, I need to persist the timestamp/ID of the last successfully processed Up Bank transaction (or sync time), so subsequent syncs only fetch newer transactions.
    - **ACs:**
      1.  Cloudflare KV store used for sync state (e.g., `last_successful_up_bank_sync_timestamp`).
      2.  State updated only after transactions are fetched, transformed, and confirmed by YNAB.
      3.  Fetching logic (Story 3.1) uses this state. If no state, uses initial lookback.
  - **Story 3.6: Implement Core Sync Orchestration Logic**
    - As the System (Phoenix App), I need a primary server-side function/handler that orchestrates the end-to-end transaction sync process, so it can be reliably triggered.
    - **ACs:**
      1.  Single, invokable server-side function/handler created in `phoenix-app`.
      2.  Orchestrates fetch (3.1), transform (3.3), push (3.4), and state update (3.5) using configured YNAB targets (3.2).
      3.  Robust error handling; does not update sync state on failure of a critical step.
      4.  Logs overall sync process (start, end, count, errors).
      5.  Can be triggered manually (e.g., via a protected URL endpoint for MVP).

---

**Epic 4: Balance Reconciliation Calculator (Account HUD)**

- **Description:** Develop a tool within Phoenix to help reconcile Up Bank's available balance with YNAB's cleared balance by displaying both, their difference, and relevant recent Up Bank transactions.
  - **Story 4.1: User Configuration of Target Up Bank Account for Calculator**
    - As a User, I want to select which of my Up Bank accounts is used by the Balance Reconciliation Calculator from a list of my available accounts, so I can ensure the calculations are performed against the correct account.
    - **ACs:**
      1.  Settings area in `phoenix-app` fetches all Up Bank accounts via API.
      2.  UI presents a dropdown of `TRANSACTIONAL` accounts (displaying name/type).
      3.  Previously selected account (if any from KV) is pre-selected. Otherwise, a heuristic (e.g., first transactional) may suggest an initial selection.
      4.  User selection of Up Bank Account ID is stored in Cloudflare KV (e.g., `calculator_selected_up_bank_account_id`).
      5.  Calculator uses the Account ID from KV.
      6.  UI provides feedback if account fetching/selection fails or if no account is configured.
  - **Story 4.2: Fetch Up Bank Available Balance and YNAB Cleared Balance (with Caching)**
    - As a User, I want the system to fetch the current _available_ balance from my configured Up Bank account and the current _cleared_ balance from my linked YNAB account, utilizing caching for the Up Bank balance.
    - **ACs:**
      1.  Retrieves `calculator_selected_up_bank_account_id` from KV. Fails gracefully if not configured.
      2.  On "Refresh Balances" trigger: Calls Up Bank API for live `availableBalanceInBaseUnits`. If successful, stores in KV (e.g., `cache_up_balance_{accountId}`) with TTL (e.g., 5-10 min) and uses for display. If API call fails, checks KV for cached balance; if found, displays with "stale/error" indicator. Else, shows error.
      3.  Fetches live `cleared_balance` from YNAB API for the linked YNAB account.
      4.  Balances converted to comparable currency format.
      5.  Handles API errors gracefully.
  - **Story 4.3: Display Comparative Balances (Up Available vs. YNAB Cleared) and Difference**
    - As a User, I want a UI section to display my Up Bank _available_ balance, my YNAB _cleared_ balance, and their calculated difference, so I can quickly assess reconciliation status.
    - **ACs:**
      1.  Dedicated "Balance Reconciliation Calculator" UI view in `phoenix-app`.
      2.  "Refresh Balances" button triggers Story 4.2 logic.
      3.  Displays: "Up Bank Available Balance: $X.XX" (with stale indicator if applicable), "YNAB Cleared Balance: $Y.YY", "Difference: $Z.ZZ" (indicating which is higher).
      4.  Displays error if configuration missing or fetching fails and no cache is usable.
  - **Story 4.4: List Recent Up Bank Transactions to Aid Reconciliation of Balance Difference**
    - As a User, to help reconcile the difference between my Up Bank _available_ balance and YNAB _cleared_ balance, I want the calculator UI to display recent Up Bank transactions, with a best-effort indication of their status in YNAB.
    - **ACs:**
      1.  When balances displayed, fetches recent settled transactions from configured Up Bank account (e.g., last 30 days).
      2.  Fetches recent transactions from linked YNAB account (same period), noting `cleared` status.
      3.  UI displays list of recent Up Bank settled transactions (Date, Payee/Description, Amount).
      4.  For each Up Bank transaction, a best-effort indicator shows YNAB status (e.g., "Matched, YNAB Cleared", "Matched, YNAB Uncleared", "No Match in YNAB / Needs Syncing").
      5.  Helps user identify discrepancies.

---

**Epic 5: Basic Debugging/Status Dashboard**

- **Description:** Create a UI within `phoenix-app` for monitoring system operations, viewing alerts, manually triggering syncs, accessing the calculator, and managing basic settings.
  - **Story 5.1: Implement Dashboard UI Shell and Navigation**
    - As the User (system administrator), I want a basic dashboard shell within `phoenix-app` with clear navigation, so I can easily access different monitoring, utility, and settings sections.
    - **ACs:**
      1.  Main dashboard route (e.g., `/dashboard`) created, accessible post-login.
      2.  Consistent layout (header, nav: "System Status," "Event Log," "Balance Calculator," "Settings").
      3.  Navigable sections initially show placeholders.
  - **Story 5.2: Display System Event Log on Dashboard**
    - As the User, I want to view a chronological log of key system events, errors, and alerts on the dashboard, so I can monitor activity, understand processing flow, and diagnose issues.
    - **ACs:**
      1.  "Event Log" section in dashboard.
      2.  Displays: Up Bank webhooks (summary, payload on demand), processing status/outcome, details of YNAB syncs (data, response), system errors, GitHub alerts (CI/CD failures, security alerts as per FR8.x) with links.
      3.  Events timestamped, reverse chronological.
      4.  Basic filtering (event type, date range).
      5.  Pagination for logs.
  - **Story 5.3: Display System Health Status Overview**
    - As the User, I want a "System Status" section on the dashboard for an at-a-glance overview of system health and key metrics, so I can quickly verify operational readiness.
    - **ACs:**
      1.  "System Status" section/widget.
      2.  Displays connectivity status to Up Bank API & YNAB API (Connected/Error, last check time).
      3.  Timestamp of last successful Up Bank to YNAB sync.
      4.  Basic stats (e.g., transactions synced/errors in last 24h).
      5.  Status refreshed on load or manual refresh.
  - **Story 5.4: Provide Manual Sync Trigger via Dashboard**
    - As the User, I want a button on the dashboard to manually trigger the Up Bank to YNAB transaction synchronization process, so I can initiate a sync on demand.
    - **ACs:**
      1.  "Trigger Sync Now" button available.
      2.  Clicking invokes core sync orchestration logic (Story 3.6).
      3.  UI feedback on trigger (e.g., "Sync initiated...").
      4.  Outcome reflected in Event Log and System Status.
  - **Story 5.5: Integrate YNAB Cleared Balance Calculator Access**
    - As the User, I want to easily navigate to and use the YNAB Cleared Balance Calculator from the main dashboard.
    - **ACs:**
      1.  Navigation link/tab ("Balance Calculator") in dashboard nav.
      2.  Links to/embeds the YNAB Cleared Balance Calculator UI (Epic 4).
  - **Story 5.6: Integrate Up Bank Account Configuration UI Access**
    - As the User, I want to access the settings page from the dashboard to configure my Up Bank account for the calculator and manage other future settings.
    - **ACs:**
      1.  Navigation link/tab ("Settings") in dashboard nav.
      2.  Links to settings UI, including Up Bank account selection dropdown (Story 4.1).
      3.  Settings page structured for potential future settings.

---

## 8. Key Reference Documents

- Project Brief: `project-brief.md` (as provided by user)
- This PRD document.
- (Future: Architect Design Document, UI/UX Specifications if detailed separately)

## 9. Out of Scope for MVP

To maintain focus and ensure a lean, achievable Minimum Viable Product, the following features and functionalities are explicitly out of scope for the initial release of Phoenix:

1.  **Support for Financial Institutions Other Than Up Bank.**
2.  **Advanced Transaction Management Rules & AI Categorization.**
3.  **User Interface for Transaction Review/Approval Before YNAB Sync.**
4.  **Direct YNAB Budget/Goal Management.**
5.  **Advanced Reporting & Custom Analytics Dashboards.**
6.  **Multi-User Support & SaaS Capabilities.**
7.  **Extensive Configuration UI Beyond Essential Settings defined for MVP** (e.g., complex sync mapping rules).
8.  **Offline Functionality or PWA Features for UI Components.**
9.  **Automated Handling of All Obscure Edge Cases for Up Bank/YNAB APIs** (MVP focuses on common cases).
10. **Strict WCAG 2.2 AA Compliance if Overly Complex** (aspiration with pragmatic approach).
11. **Complex GitHub Alert Processing Beyond Critical Notifications defined.**
12. **Automated scheduling of transaction sync via Cron Triggers** (manual trigger is MVP; Cron can be fast follow-on).

## 10. Open Questions / Assumptions to Validate

1.  **External API Behaviors & Reliability:**
    - **Up Bank API:** Webhook event completeness for all transaction scenarios; precise nature of "transaction cancelled" events (if distinct); real-world rate limits and error handling nuances.
    - **YNAB API:** Idempotency confirmation for all relevant update scenarios; API performance for fetching transactions.
    - **SMS Provider API:** Reliability and deliverability.
2.  **Cloudflare Service Performance & Limitations:** Confirmation that KV/D1 performance, queue reliability, and Worker execution limits are well within needs.
3.  **Complexity of "Highly Desirable" GitHub Alert Integrations:** Architect to assess FR8.4 (Dependabot) & FR8.5 (Secret Scanning) for MVP feasibility.
4.  **Passkey Implementation Nuances:** Cross-browser/device compatibility of chosen libraries; ease of integration in Cloudflare Workers.
5.  **Long-Term Cost Management:** Confirmation that free tiers will largely cover MVP operational needs.
6.  **Data Mapping Complexity:** Sufficiency of initial basic mapping rules for Up Bank to YNAB.
7.  **Exact structure of the `phoenix-app` for serving UI and handling server-side React Router logic on a single Worker.** (Architect to detail).

## 11. Change Log

| Version | Date       | Author                               | Description                            |
| :------ | :--------- | :----------------------------------- | :------------------------------------- |
| 1.0     | 2025-05-14 | PM-V3-BETA (with User collaboration) | Initial Product Requirements Document. |

---

This concludes the PRD content. Next is the Initial Architect Prompt.

---

## Initial Architect Prompt

Based on our detailed discussions and the Product Requirements Document (PRD) for the **Phoenix** project, the following technical guidance and requirements have been established to kick off the Architecture Creation Mode. Your primary task is to design a robust, maintainable, and secure architecture that meets these MVP requirements, leveraging the Cloudflare ecosystem.

### Technical Infrastructure & Stack

- **Repository & Service Architecture Decision:**
  - An **Nx Monorepo** is required.
  - The primary application (`phoenix-app`) will be built using **Vite + React + React Router (in "framework mode" with data routers and the Vite plugin)**. This application must be capable of full-stack functionality, serving its own UI and handling server-side logic (e.g., API-like request handlers, data transformations) via React Router's capabilities.
  - The `phoenix-app` will be deployed as a **Cloudflare Worker**.
  - You (the Architect) are to determine if any specific backend logic (e.g., particularly intensive processing, or for clearer separation of concerns for future features) warrants the creation of distinct, separate Cloudflare Worker services within the monorepo, or if all MVP backend logic can be efficiently integrated within the server-side capabilities of the primary `phoenix-app` worker.
- **Starter Project/Template Guidance:**
  - The user has indicated that `yarn create cloudflare my-react-router-app --framework=react-router` (or equivalent `npx create-cloudflare@latest`) might provide a good bootstrap for the `phoenix-app`. Evaluate the output of this and determine the best way to integrate this pattern (Vite + React Router "framework mode" targeting Cloudflare Workers) within the Nx Monorepo structure.
- **Hosting/Cloud Provider:** **Cloudflare**
  - Compute: Cloudflare Workers (for `phoenix-app` serving UI and backend logic, and any additional distinct workers if designed).
  - Storage:
    - Cloudflare KV: For user configuration (e.g., selected Up Bank account for calculator), sync state, short-term caching (e.g., Up Bank balance).
    - Cloudflare D1: Consider for more structured, queryable data if needed (e.g., persistent detailed event/audit logs for the dashboard, though simpler logging or KV might suffice for MVP). If D1 is used, there's a preference for **Drizzle ORM**.
    - Cloudflare R2: Evaluate if needed for efficient storage and serving of larger static assets for the `phoenix-app` if they are not optimally bundled/served directly by the Worker/KV.
  - Queues: Cloudflare Queues may be considered for resilient message handling if complex asynchronous processing is identified (e.g., for webhook processing if it becomes very involved, though direct processing in worker handlers might be sufficient for MVP).
- **Frontend Platform:** Vite, React, React Router (framework mode with data routers, Vite plugin), Tailwind CSS v4, Shadcn/UI (CSS variables), Storybook v9 (beta target).
- **Backend Platform:** Cloudflare Workers (Node.js runtime environment).
- **Database Requirements:** See "Storage" under Hosting/Cloud Provider.

### Technical Constraints

- **Operational Costs:** Must aim to utilize free tiers of Cloudflare services and the chosen SMS provider (e.g., AWS SNS) as much as possible. Soft budget caps of $25 AUD/month per major service apply if costs are incurred (NFR6.1).
- **Security:** High priority. Adherence to all security NFRs (NFR3.x) and Functional Requirements (FR3.x, FR7.x, FR8.x) regarding secret management, user authentication (SMS OTP & Passkey), and webhook security is paramount.
- **Platform Lock-in:** Strong preference for the Cloudflare ecosystem for all core components.

### Deployment Considerations

- **CI/CD:** GitHub Actions.
- **Environments:**
  - Local development environment (simulating Cloudflare Workers via Vite plugin and/or `wrangler dev`).
  - Development/Preview: `dev.phoenix.dncr.me` (automatically deployed from `main` branch; deployment conditional on all CI checks passing). The `main` branch will not have push protections to facilitate AI agent workflow.
  - Production: `phoenix.dncr.me` (manually promoted from a validated `dev` environment state via CI/CD trigger).
  - Ephemeral Preview Environments: For feature branches (e.g., `<branch-name>.dev.phoenix.dncr.me`).
- **Quality Gates:** CI pipeline for `main` must include mandatory stages for linting, formatting, all automated tests (unit, integration), and successful build before deployment to `dev.phoenix.dncr.me`.

### Local Development & Testing Requirements

- **Framework:** Vitest for unit, integration, and potentially E2E tests (using browser mode for UI).
- **Environment:** Nx Monorepo setup. Local development server (Vite with `@cloudflare/vite-plugin` or `wrangler dev`) must accurately simulate the Cloudflare Workers runtime.
- **Testability:** Adherence to Testability NFRs (NFR9.x), including local test execution and considerations for simulating external dependencies/events.

### Other Technical Considerations

- **Maintainability:** Adherence to Maintainability NFRs (NFR4.x), including code quality standards, Nx conventions, modular design, Git commit standards, and automated checks.
- **Integration Design:** Evaluate the pluggable adapter pattern for Up Bank and YNAB API integrations.
- **Privacy:** Ensure all designs adhere to Privacy NFRs (NFR7.x), especially regarding data handling and any monitoring/analytics.
- **Open Questions:** Review the "Open Questions / Assumptions to Validate" section of the PRD and incorporate investigation or resolution of these into your design where appropriate.
- **Optional Features:** Assess the implementation complexity of FR8.4 (Dependabot Alerts) and FR8.5 (Secret Scanning Alerts) and advise on their feasibility for inclusion in the initial MVP build versus deferral.
- **User Experience:** Ensure the architecture supports the "User Interaction and Design Goals" outlined in the PRD, including responsiveness and accessibility aspirations.

Your deliverables will be an Architecture Design Document that outlines the proposed system architecture, technology choices (where not already specified), data models (if using D1), service interactions, and deployment strategy, addressing all functional and non-functional requirements from the PRD.
