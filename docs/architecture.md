## Phoenix Project: Architecture Design Document Outline (v1.0)

**Version:** 1.0
**Date:** May 15, 2025
**Architect (Collaborator):** Gemini AI (in collaboration with User)

**1. Introduction & Overview**
_ **1.1. Project Goal & Objectives:** (As per PRD Section 1) Develop "Phoenix," a robust, reliable, and maintainable personal finance tool, initially bridging Up Bank and YNAB, built on a clean, extensible architecture.
_ **1.2. Context:** (As per PRD Section 1) Address lack of official Up Bank to YNAB integration, replace unreliable custom solution, simplify cleared balance calculation. \* **1.3. Scope:** MVP focused on Up Bank to YNAB sync, admin/debug UI, balance calculator, GitHub alerts, and core NFRs.

**2. Architectural Goals & Principles**
_ **Clean Architecture:** Emphasize separation of concerns, modularity.
_ **Extensibility:** Design for future personal finance hobby projects.
_ **Reliability & Availability:** High uptime for core sync, robust error handling.
_ **Security:** Paramount, protecting financial data and API keys.
_ **Maintainability:** Readable, well-structured, testable code.
_ **Performance:** Low latency processing and UI responsiveness.
_ **Cost-Efficiency:** Leverage Cloudflare free tiers where possible.
_ **Privacy by Design:** Minimize data handling, use privacy-preserving tools.

**3. High-Level Architecture**
_ **3.1. Component Diagram & Overview:**
_ **External Entities:** User (Admin), Up Bank API, YNAB API, AWS SNS Service, GitHub.
_ **Phoenix Core System (Cloudflare Workers):**
_ `WebhookWorker` (Hono-based): Ingests Up Bank & GitHub webhooks.
_ `UpBankQueueWorker`: Processes Up Bank transaction events from queue.
_ `GitHubQueueWorker`: Processes GitHub alert events from queue.
_ `App Worker`: Serves the server-side rendered React Router application (Phoenix App UI) and handles its server-side `loader`/`action` logic.
_ **Cloudflare Platform Services:** D1 Database, Queues (`UP_BANK_QUEUE`, `GITHUB_QUEUE`), KV Store (config), Worker Secrets (credentials).
_ **Conceptual Adapters:** `UpBankApiAdapter`, `YnabApiAdapter`, `SmsAdapter` (AWS SNS).
_ **3.2. Core Service Interactions & System Flow:**
_ Up Bank Webhook Ingestion & Processing Flow.
_ GitHub Alert Ingestion & Processing Flow.
_ Phoenix App UI (Admin/Debug) Interaction Flow.
_ Authentication Flow (Passkeys, SMS OTP).
_ Critical Alert SMS Flow.
_(Detailed textual description based on our discussion, with Mermaid diagram as a visual aid source).\*

**4. Technology Stack**
_ **Cloud Platform:** Cloudflare (Workers, D1, KV, Queues, Secrets), AWS (SNS for SMS).
_ **Backend:** TypeScript, Cloudflare Workers, Hono (for `WebhookWorker`), Drizzle ORM (for D1), Zod (validation).
_ **Frontend (Phoenix App UI):** React, React Router (with SSR on Workers), Vite, TypeScript. (Styling: Tailwind CSS, UI Components: Shadcn/UI to be confirmed/implemented).
_ **Monorepo & Tooling:** Nx, Yarn, Vitest.
_ **CI/CD:** GitHub Actions.
_ **Authentication:** Passkeys (WebAuthn), SMS OTP, JWTs in encrypted HTTP-only cookies.

**5. Data Model (Cloudflare D1)**

_ **5.1. `WebhookUpEvents` Table:** Stores raw Up Bank webhook events, validation status, and generated columns for `up_event_type_detected`, `up_transaction_id_ref`.
_ **5.2. `WebhookGitHubEvents` Table:** Stores raw GitHub webhook events, validation status, and generated columns for key event details.
_ **5.3. `Transactions` Table:** Central Phoenix transaction entity. Links to `WebhookUpEvents` via FKs (`created_event_fk`, `settled_event_fk`, `deleted_event_fk`). Contains YNAB sync state (`sync_status`, `ynab_transaction_id`, `ynab_account_id_target`, `ynab_budget_id_target`), internal Phoenix notes (`notes`), and Phoenix timestamps (`created_at`, `updated_at`). Includes generated columns for Up Bank data (`up_description`, `up_amount_value`, `up_currency_code`, `up_status_enum`, `up_created_at`, `up_settled_at`) derived from linked `WebhookUpEvents` (without a `latest_relevant_event_fk`, definitions will use COALESCE/CASE logic). Includes a generated boolean `cleared_status` based on `settled_event_fk`.
_ **5.4. `QueueActivityLog` Table:** Logs activities from queue consumers (`UpBankQueueWorker`, `GitHubQueueWorker`), linked via `webhook_up_event_fk` (or `webhook_github_event_fk` if we make it polymorphic, or have separate processing logs - current design links to `webhook_up_event_fk` or `webhook_github_event_fk`). Includes `timestamp`, `source_queue_name`, `activity_type`, `status`, `details_json`, `duration_ms`.

**6. Configuration Management**
_ **Cloudflare Worker Secrets:** For primary sensitive credentials (API keys, webhook secrets, admin bootstrap phone number, cookie encryption key).
_ **Cloudflare KV Store:** For application-level configuration (default YNAB budget/account IDs, SMS recipient phone number). Values considered sensitive will be encrypted/decrypted by the `App Worker` using an encryption key from Worker Secrets. \* **Secrets Population (from 1Password):** Local script using 1Password CLI (`op`) and Wrangler CLI (`wrangler secret put`) for manually updating Worker Secrets, followed by a Worker redeploy.

**7. API Design (High-Level)**
_ **7.1. External Inbound Webhook APIs (on `WebhookWorker`):**
_ `POST /webhooks/upbank` (Up Bank events).
_ `POST /webhooks/github` (GitHub events).
_(Both include signature validation, schema validation with Zod, logging to D1, and enqueueing).\*
_ **7.2. Phoenix App UI Server-Side Logic (React Router `loader`s & `action`s on `App Worker`):**
_ Authenticated endpoints for dashboard data, transaction lists/details, webhook event logs, queue activity logs, settings management, cleared balance calculation, manual retry, manual status updates, sending Up Bank PING.

**8. Integration Design (Adapter Pattern)**
_ **`UpBankApiAdapter`:** For fetching full transaction details from Up Bank, sending PINGs.
_ **`YnabApiAdapter`:** For creating/updating transactions in YNAB, handling YNAB-specific formatting and idempotency. **Note:** The YNAB API enforces a rate limit of 200 requests per hour per access token. The adapter and sync logic must be designed to respect this limit, implementing appropriate request pacing and error handling for rate limit responses.
_ **`SmsAdapter` (AwsSnsAdapter):** For sending SMS via AWS SNS for critical alerts and authentication OTPs.
_(Adapters encapsulate external API complexities and are managed as libraries within the Nx Monorepo).\*

**9. Security Design**
_ **Authentication:** Passkeys and SMS OTP for Admin UI, JWTs in encrypted HTTP-only cookies for session management.
_ **Authorization:** Single administrator role for MVP.
_ **Secret Management:** As per section 6.
_ **Webhook Security:** Mandatory signature validation.
_ **Data Security:** Encryption at rest (D1 default, application-level for sensitive KV), HTTPS in transit, careful handling/masking of PII in logs/UI.
_ **Rate Limiting:** Rely on Cloudflare platform protection for webhook endpoints; strong signature validation is primary.
_ **CSP & Security Headers:** For the `App Worker` serving the UI.
_ **Dependency Management:** Regular updates using Yarn, leverage Dependabot alerts.

**10. Error Handling and Logging Strategy**
_ **Structured Logging:** To D1 tables (`WebhookUpEvents`, `WebhookGitHubEvents`, `QueueActivityLog`) with consistent error structures in JSON details.
_ **Error Categorization & Handling:** Defined approaches for transient vs. permanent errors within Workers (retries via Queues, DLQs, specific error states).
_ **Dead Letter Queues (DLQs):** Configured for `UP_BANK_QUEUE` and `GITHUB_QUEUE`, with a defined process for monitoring (manual review for MVP).
_ **Correlation IDs:** Using `WebhookUpEvents.id` / `WebhookGitHubEvents.id` for tracing.
_ **Surfacing Errors:** To Admin UI (via `Transactions.sync_status` and `QueueActivityLog`) and critical SMS alerts.
_ **Client-Side Error Logging:** Browser console for MVP; basic forwarding to backend log as low-priority future item.

**11. Deployment Strategy**
_ **Tooling:** Wrangler CLI, Nx Monorepo, Drizzle ORM (`drizzle-kit`), GitHub Actions.
_ **Environments:** Local, `dev.phoenix.dncr.me`, `phoenix.dncr.me` (production), Ephemeral Previews.
_ **CI/CD Pipeline (GitHub Actions):** Linting, testing, building, deployment to `dev` on `main` merge, manual promotion to `prod`. Feature branch deployments to ephemeral environments.
_ **D1 Migrations:** Generated by `drizzle-kit`, applied via `wrangler d1 migrations apply`. Destructive/cleanup migrations are manually triggered pipeline tasks, independent of app code releases, following a deprecation period. Non-destructive additive migrations can deploy with app code to `dev`.
_ **Configuration Management per Environment:** Using `wrangler.toml` environment sections, distinct Cloudflare resources.
_ **Rollback Strategy:** Worker versioning for Worker scripts; D1 via Time Travel for recovery (forward-only migrations preferred).

The Phoenix application does not require pre-defined seed data in the production database for its core functionality. Any data necessary for specific testing scenarios will be introduced through dedicated test setup procedures or scripts and is not part of the standard deployment process. Configuration will be managed via environment variables.

**12. Addressing Non-Functional Requirements** \* (This section would summarize how the choices in sections 3-11 directly address each NFR category from the PRD, referencing specific architectural decisions).

**13. Addressing PRD Open Questions & Assumptions** \* (This section contains our detailed review of PRD Section 10, outlining mitigation/validation plans).

**14. Assessment of Optional Features (GitHub Alerts for MVP)** \* **FR8.4 (Dependabot) & FR8.5 (Secret Scanning):** Assessed as low-to-medium complexity for MVP to implement logging and SMS notification. Recommended for inclusion in MVP.

**15. Future Considerations (Optional)**
_ Multi-user support.
_ More advanced GitHub alert processing/remediation.
_ Client-side error logging to a backend endpoint.
_ OIDC for AWS authentication from Workers. \* More sophisticated D1 migration safety tooling.

**16. Technical Debt Management**
_ Proactive management of technical debt is essential to maintain long-term code quality and project velocity. During MVP development, any known shortcuts, workarounds, or deferred improvements will be explicitly documented as TODOs in code comments and/or as issues in the project tracker. Regular technical debt review will be incorporated into sprint planning or periodic project reviews, with prioritization based on impact and risk. Where feasible, debt items will be linked to architectural decisions or trade-offs made for MVP delivery. Post-MVP, a dedicated effort will be made to address high-priority technical debt, with progress tracked transparently alongside feature development._
