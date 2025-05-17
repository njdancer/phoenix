# Project Brief: Phoenix

## Introduction / Problem Statement

Core Idea: To create a robust, reliable, and maintainable bridge between Up Bank and YNAB, replacing a rudimentary and difficult-to-debug existing solution, and to serve as an extensible platform for future personal finance tools.

Problem Being Solved:

- Lack of official YNAB integration for Up Bank in the user's country.
- The current custom solution is unreliable, prone to errors, difficult to debug, and has poor code quality.
- The manual and cumbersome process of calculating the "cleared balance" for YNAB reconciliation, as Up Bank only provides the "working balance."

Opportunity:

- To build a personal tool that significantly streamlines and improves the user's budgeting workflow and reliability.
- To develop a well-architected system that can be a foundation for future personal finance hobby projects.
- To potentially open-source the core Up Bank to YNAB bridging solution for others facing similar challenges.
- To expand the tool's functionality beyond basic transaction syncing, beginning with an integrated debugging dashboard and utilities like a cleared balance calculator.

## Vision & Goals

- **Vision:** To develop "Phoenix," a reliable and extensible personal finance tool, initially focused on seamlessly bridging Up Bank and YNAB. It will feature a clean architecture, enabling it to serve as a robust platform for future personal finance hobby projects. The long-term aspiration is to potentially evolve this into a more comprehensive custom budgeting workflow, building upon foundational features like reliable transaction listing and robust bank integration.
- **Primary Goals:**
  - Goal 1 (Accuracy): Achieve 99.9% accuracy in syncing transactions (creations, clearances, cancellations) from Up Bank to YNAB within 5 minutes of the webhook event, for the first 3 months post-MVP launch.
  - Goal 2 (Reduced Maintenance): Reduce the time spent manually correcting or debugging YNAB transaction data originating from Up Bank by 90% compared to the current rudimentary solution, within the first month of MVP use.
  - Goal 3 (Calculator & Debugging Effectiveness): Successfully implement and utilize the cleared balance calculator and the debugging dashboard to resolve any sync discrepancies within 15 minutes of identification for 100% of occurrences during the first 3 months post-MVP launch.
  - Goal 4 (Reliable Alerting): Implement basic SMS notifications for critical system events (e.g., sync failures, API authentication issues) that are successfully delivered and acknowledged by the user within the first month of MVP use.
- **Success Metrics (Initial Ideas):**
  - Transaction sync success rate (target: 99.9%).
  - Average transaction sync latency (from Up Bank webhook receipt to YNAB API confirmation target: < 5 minutes).
  - Percentage reduction in time spent on manual YNAB data correction for Up Bank transactions (target: 90%).
  - Number of critical sync errors per week/month requiring manual intervention (target: < 1).
  - Average time to identify and resolve sync discrepancies using the dashboard (target: < 15 minutes).
  - Successful usage rate of the cleared balance calculator for monthly YNAB reconciliation.
  - User confirmation (e.g., via logs or a simple feedback mechanism) of receiving and finding SMS alerts useful for critical issues.

## Target Audience / Users

Primary Users: The primary user is the developer of this project, with occasional use by their spouse (a phone repairer).

Key Characteristics & Needs:

- **Technical Proficiency:** High for the primary user (developer). The system's internal workings and debugging tools can be complex. The spouse's interaction would likely be limited to viewing reconciled data or simple status dashboards, if any UI is exposed for such purposes beyond YNAB.
- **Platform Usage:** Both are heavy iPhone users. Any user-facing web interface (e.g., debugging dashboard, calculator) must be responsive and function effectively on modern iPhone screen sizes.
- **Current Tools & Workflow:**
  - Reliant on YNAB for budgeting methodology.
  - Utilize Up Bank for primary banking.
  - Generally satisfied with YNAB's methodology but experience performance issues (especially with virtual scroll lists) and dislike the manual effort required for managing transactions from Up Bank.
- **Key Pain Points:**
  - Significant time and manual effort spent reconciling Up Bank transactions within YNAB.
  - The manual effort leads to reduced engagement with the budgeting process.
  - The existing rudimentary bridge solution is unreliable and difficult to debug when issues arise.
  - Calculating the "cleared balance" for YNAB reconciliation is a manual and error-prone step.
- **Desired Outcomes from "Phoenix":**
  - Comprehensive automation of the Up Bank to YNAB transaction synchronization process.
  - A significant reduction in time spent on budget _maintenance_, allowing for more time dedicated to budget _review and strategic planning_.
  - A highly reliable system that minimizes transaction errors and provides straightforward tools for debugging if discrepancies occur.
  - Improved engagement with the budgeting process due to reduced friction and increased trust in the data.
  - Easy access to a consistently calculated "cleared balance" for YNAB reconciliation.

## Key Features / Scope (High-Level Ideas for MVP)

- **Feature 1: Automated Up Bank to YNAB Transaction Sync:**

  - Reliably receive webhook events from Up Bank for new transactions, cleared/settled transactions, and any updates (e.g., cancellations or changes to existing transactions).
  - Accurately transform Up Bank transaction data into the format required by the YNAB API.
  - Send transformed transaction data to the correct YNAB account and budget (potentially to a default "inbox" category if specific categorization isn't feasible or desired at the MVP stage).
  - Implement robust error handling for API communications (both Up Bank and YNAB) and include retry mechanisms with appropriate backoff strategies.
  - Ensure idempotency in processing webhook events to prevent duplicate transactions in YNAB.

- **Feature 2: Webhook Event Logging & Debugging Dashboard:**

  - Store every incoming webhook event from Up Bank, including its full payload (headers, body) and relevant metadata (e.g., timestamp, source IP if available/relevant).
  - Provide a web-based UI (the "Debugging Dashboard") to list, view the details of, and search these webhook events.
  - Display the processing status for each event (e.g., "Received," "Processing," "Successfully Sent to YNAB," "Failed - YNAB API Error," "Failed - Data Transformation Error," "Retrying").
  - Allow for manual re-triggering of processing for a failed or problematic webhook event, if feasible and safe.

- **Feature 3: Internal Transaction Ledger & Management for Calculator:**

  - Maintain an internal, simplified ledger of transactions derived from processed webhook events, primarily to support the cleared balance calculator.
  - This ledger should store key details like date, description, amount, and Up Bank's cleared/settled status for each transaction within relevant Up Bank accounts.
  - Link transactions in this internal ledger back to their originating webhook event(s) for traceability.
  - The primary purpose of this ledger in the MVP is to feed the "YNAB Cleared Balance Calculator"; extensive transaction management features are not in scope for MVP beyond what's needed for calculator accuracy.
  - Allow manual marking of an internal transaction's status (e.g., "cleared" / "uncleared" for calculator purposes) if a webhook is missed or an automated update fails, to ensure calculator accuracy.

- **Feature 4: YNAB Cleared Balance Calculator:**

  - Based on the data in the internal transaction ledger, accurately calculate the sum of "uncleared" transactions for a selected Up Bank account.
  - Provide a simple UI where the user can input their current "Working Balance" as displayed by Up Bank.
  - Display the calculated "Cleared Balance" (Working Balance - Sum of Uncleared Transactions) for easy reference and copy-pasting into YNAB during reconciliation.

- **Feature 5: Basic System Status/Health Dashboard:**

  - A simple, clear overview page (potentially part of the Debugging Dashboard UI) showing the operational status of the "Phoenix" bridge.
  - Indicate connectivity status to Up Bank (listening for webhooks) and YNAB API (e.g., "Connected and Listening," "Error: YNAB API Unreachable," "Error: Up Bank Webhook Configuration Issue").
  - Display basic statistics, such as "X transactions successfully processed today," "Y events received in the last hour."

- **Feature 6: Basic SMS Notifications for Critical Events:**
  - Implement notifications via a third-party SMS API (e.g., AWS SNS) for critical system events.
  - Critical events include: repeated failures to sync transactions, authentication errors with Up Bank or YNAB APIs, and prolonged system downtime or major errors.

## Post MVP Features / Scope and Ideas

- **Feature Idea 1: Integration with Other Financial Institutions/Accounts:**

  - Explore and potentially implement solutions (e.g., secure screen-scraping if APIs are unavailable, or other aggregation services) to incorporate transaction data from other accounts such as:
    - Commbank
    - Commsec
    - CMC Markets
    - REST Super
    - Vanguard Super
  - This would aim to provide a more holistic view of personal finances within the "Phoenix" ecosystem or to feed into YNAB if desired.

- **Feature Idea 2: Enhanced Financial Dashboarding & Insights:**

  - Beyond the MVP's debugging and status dashboards, develop more user-focused dashboards offering insights into spending patterns, net worth tracking, or other financial metrics.

- **Feature Idea 3: Custom Budgeting Workflow Development:**

  - Evolve "Phoenix" into a more comprehensive, standalone budgeting tool, potentially incorporating methodologies preferred by the user or addressing shortcomings experienced with existing tools like YNAB (e.g., performance, specific workflow needs). This could include features for budget creation, tracking against budget, goal setting, etc.

- **Feature Idea 4: Advanced Transaction Management & Categorization:**

  - Implement more sophisticated automatic categorization rules for transactions.
  - Provide tools for bulk editing, splitting transactions, and managing recurring transactions within Phoenix itself.

- **Feature Idea 5: Open Source Community Version:**
  - If significant interest is shown and the core sync mechanism is stable, explore packaging and documenting a version of the Up Bank to YNAB bridge for open-source release, potentially with a more generalized adapter pattern for other banks or budgeting tools if the architecture supports it.

## Known Technical Constraints or Preferences

- **Constraints:**

  - **Budget (Operational Costs):** Very low. Aim to stay within free tiers for cloud services (Cloudflare, AWS) where possible. A soft cap of approximately $25 AUD/month for Cloudflare services and $25 AUD/month for AWS services (e.g., for SNS if used extensively) is the target if costs are incurred. Development labor is not a monetary cost factor as it's a personal project.
  - **Timeline:** No specific hard deadline. Development can proceed at a flexible, personal pace.
  - **Security:** High Priority. Given the handling of personal financial data, security is paramount.
    - Strategy: Leverage established third-party solutions for authentication and authorization if any direct user login to "Phoenix" itself were ever needed (e.g., Cloudflare Access, or identity providers). This is less critical for the MVP which is primarily backend-focused but important for any web-exposed dashboards.
    - Secure storage and handling of API keys (Up Bank, YNAB, SMS provider) is critical, utilizing Cloudflare's provided secret management.
    - Avoid developing custom security solutions for authentication or encryption.

- **Initial Architectural Preferences:**

  - **Platform:** Cloudflare Workers (utilizing Workers for compute, KV or D1 for storage, and Queues for resilient message handling if deemed necessary).
  - **Repository Structure:** Nx Monorepo. Application code should be organized into individual, well-defined libraries within the monorepo to promote modularity and reusability.
  - **Service Architecture:** Serverless components (via Cloudflare Workers).
  - **Integration Design:** While not a strict MVP requirement if it adds excessive complexity, an ideal approach would be to design integrations (for banks like Up, and budgeting apps like YNAB) using a pluggable adapter pattern. This would facilitate future extensibility (e.g., adding other banks or swapping budgeting tools) more easily.

- **Risks:**

  - **Primary Risk: API Discontinuation/Changes:**
    - **Up Bank API:** The core functionality is highly dependent on the continued availability, stability, and terms of use of the Up Bank API. Any significant changes, rate limiting, or deprecation could break the system or require substantial rework.
    - **YNAB API:** Similarly, reliance on the YNAB API means that changes, rate limits, or deprecation from YNAB's side could impact functionality and require updates.
  - **Scope Creep:** Given the vision for an extensible platform and future hobby projects, there's a risk of the initial MVP scope expanding unintentionally, which could delay the delivery of core functionality if not carefully managed.
  - **Security Vulnerabilities:** Despite intentions to use secure practices and third-party solutions, any system handling financial data carries an inherent risk of security vulnerabilities if new attack vectors are discovered, configurations are mismanaged, or bugs are introduced. Diligent security practices and staying updated on Cloudflare and dependency security advisories are crucial.
  - **Data Integrity Issues:** Incorrectly processing webhooks, data transformation errors, or failures in ensuring idempotency could lead to inaccurate data in YNAB, potentially undermining user trust and requiring manual correction.
  - **Cloudflare Platform Limitations/Changes:** Unexpected changes in Cloudflare Workers, KV/D1 (e.g., consistency models, storage limits, pricing), or other Cloudflare services could necessitate architectural adjustments or impact operational costs.

- **User Preferences (Technical & UX):**
  - **Code Hosting & CI/CD:** GitHub (with GitHub Actions for CI/CD).
  - **Frontend Stack (for Debugging Dashboard / UI components):**
    - React
    - React Router v7
    - TypeScript
    - Tailwind CSS v4
    - Shadcn/UI (configured to use CSS variables for theming)
    - Storybook (targeting v9, even if in beta, for component development and showcasing)
  - **Testing Framework:** Vitest for unit, integration, and potentially end-to-end testing (leveraging its browser mode if suitable for testing UI components).
  - **SMS Integration:** Utilize a third-party API that the user is already familiar with or can easily adopt (e.g., AWS SNS).
  - **User Interface Responsiveness:** Any user-facing web interfaces (dashboards, calculator) must be responsive and optimized for use on modern iPhone screen sizes, reflecting the primary users' device preferences.
  - **Configuration:** User-specific details (e.g., API keys for Up Bank and YNAB, YNAB budget ID) must be securely configurable, likely through environment variables or secrets management within the Cloudflare platform.

## Relevant Research (Optional)

Not applicable for this personal project. The primary 'research' driving this project is the user's direct experience with the problem, familiarity with the existing rudimentary solution, and an understanding of the specific behaviors and APIs of the Up Bank and YNAB systems. No formal external market research or competitive analysis was conducted or deemed necessary for this personal tool.
