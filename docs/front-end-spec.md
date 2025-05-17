# Phoenix UI/UX Specification

## Introduction

This document defines the user experience goals, information architecture, user flows, and visual design specifications for the Phoenix project's user interface. The primary purpose of the UI is to provide administrative, debugging, and balance calculation functionalities for the Up Bank to YNAB synchronization service.

- **Link to Primary Design Files:** Not applicable. Visual design and layout will be developed directly in code, guided by conceptual layouts detailed in the "Wireframes & Mockups" section.
- **Link to Deployed Storybook / Design System:** {URL will be added here once Storybook is deployed}

## Overall UX Goals & Principles

- **Target User Personas:**
  - **Admin User (Primary):** This is the sole user of the system, who is technically proficient (frontend developer). Their goals are:
    - To monitor the health and status of the Up Bank to YNAB synchronization.
    - To easily investigate and debug any issues with webhook processing or transaction syncing.
    - To quickly ascertain current Up Bank and YNAB balances and understand any discrepancies.
    - To configure essential system settings.
- **Usability Goals:**
  - **Efficiency of Use:** Quickly find information and perform necessary actions (e.g., view logs, check balances, trigger sync).
  - **Clarity & Intuitiveness:** The interface should be easy to understand and navigate, even if used infrequently. Information should be presented clearly.
  - **Error Prevention & Recovery:** Minimize potential for user error in configuration; provide clear feedback on system actions.
- **Design Principles:**
  - **Information First:** Prioritize clear presentation of relevant data (statuses, logs, balances).
  - **Functional & Pragmatic:** The design should be straightforward and utilitarian, focusing on task completion rather than elaborate aesthetics.
  - **Consistency:** Maintain consistent layout patterns, terminology, and interaction models across all views.
  - **Responsive & Accessible:** Ensure the UI is usable on preferred devices (iPhone, desktop) and meets accessibility standards.

## Information Architecture (IA)

- **Site Map / Screen Inventory:**

  ```mermaid
  graph TD
      A[Login Screen] --> B(Dashboard);
      B --> BA[Transactions List & Balances Overview: Default View];
      B --> BB[Webhook Events List];
      B --> BC[System Status & Logs];
      B --> BD[Settings];

      BA --> BAA[Transaction Detail View: Slide-out/Modal];
      BB --> BBA[Webhook Event Detail View: Slide-out/Modal];

      %% Links between detail views
      BAA --> BBA;
      BBA --> BAA;
  ```

- **Navigation Structure:**
  - **Primary Navigation:** A simple sidebar (visible on desktop, collapsible to a burger menu on mobile) providing links to:
    - Transactions List & Balances Overview
    - Webhook Events List
    - System Status & Logs
    - Settings
  - **Secondary Navigation:** Not anticipated to be complex. Detail views will be presented as slide-out panels or modals, accessible from items within the lists.
  - **Breadcrumbs:** May not be strictly necessary due to the shallow navigation hierarchy and use of modals/slide-outs for detail views. If implemented, they would show the path from the main list view.

## User Flows

### 1\. User Login

- **Goal:** To securely access the Phoenix application's dashboard.
- **Actors:** User (Admin)
- **Trigger:** User navigates to the application URL.
- **Pre-conditions:** User has been provided with credentials or has set up Passkey/SMS OTP.
- **Post-conditions:** User is authenticated and redirected to the default dashboard view (Transactions List & Balances Overview).
- **Steps / Diagram:**
  ```mermaid
  graph TD
      A[User navigates to App URL] --> B{Is User Authenticated?};
      B -- Yes --> F[Show Transactions List & Balances Overview];
      B -- No --> C[Display Login Options: Passkey / SMS OTP];
      C --> D_Passkey[User selects Passkey];
      D_Passkey --> E_Passkey_Auth[Browser/OS Prompts for Passkey];
      E_Passkey_Auth -- Authenticated --> F;
      E_Passkey_Auth -- Failed/Cancelled --> C;
      C --> D_SMS[User selects SMS OTP];
      D_SMS --> D_SMS_EnterPhone[User enters Phone Number];
      D_SMS_EnterPhone --> D_SMS_SendOTP[System sends OTP via SMS];
      D_SMS_SendOTP --> D_SMS_EnterOTP[User enters received OTP];
      D_SMS_EnterOTP --> E_SMS_ValidateOTP{OTP Valid?};
      E_SMS_ValidateOTP -- Yes --> F;
      E_SMS_ValidateOTP -- No --> D_SMS_EnterOTP_Error[Show Error, allow retry OTP];
      D_SMS_EnterOTP_Error --> D_SMS_EnterOTP;
  ```

### 2\. View Balances, Reconcile, and Investigate Transactions (Default View)

- **Goal:** To check current Up Bank and YNAB balances, understand any discrepancies, and investigate individual transactions.
- **Actors:** User (Admin)
- **Trigger:** User successfully logs in, or navigates to the "Transactions List & Balances Overview" screen.
- **Pre-conditions:** User is authenticated. System has access to Up Bank and YNAB APIs. Target Up Bank account is configured in Settings.
- **Post-conditions:** User has up-to-date balance information and can identify transactions contributing to differences.
- **Steps / Diagram:**
  ```mermaid
  graph TD
      A[User lands on/navigates to Transactions List & Balances Overview] --> B[System fetches/displays Up Bank Available Balance];
      B --> C[System fetches/displays YNAB Cleared Balance];
      C --> D[System displays Difference];
      D --> E[User views balances and difference];
      E --> F[User can click Copy YNAB Cleared Balance];
      F -- Copied --> G[Balance copied to clipboard];
      E --> H[System displays list of Phoenix Transactions];
      H --> I{User wants to investigate further?};
      I -- Yes, Filter/Sort --> J[User applies filters/sorts to transaction list, e.g., to show uncleared];
      J --> H;
      I -- Yes, View Details --> K[User clicks on a specific transaction];
      K --> L[Display Transaction Detail View: slide-out panel];
      L --> M[User reviews details, including linked Webhook];
      M --> I;
      I -- No --> N[User has reconciled/gained understanding];
  ```

### 3\. Investigate a Webhook Event

- **Goal:** To understand the details of a specific incoming webhook (from Up Bank or GitHub) and its processing status.
- **Actors:** User (Admin)
- **Trigger:** User navigates to the "Webhook Events List" screen and selects a specific event.
- **Pre-conditions:** User is authenticated. Webhook events have been received and logged.
- **Post-conditions:** User has a clear understanding of the selected webhook's payload, its processing journey, and any related Phoenix transaction.
- **Steps / Diagram:**
  ```mermaid
  graph TD
      A[User navigates to Webhook Events List screen] --> B[System displays list of webhook events];
      B --> C{User identifies an event of interest};
      C -- Filter/Sort --> D[User applies filters/sorts to find specific events];
      D --> B;
      C -- Select Event --> E[User clicks on a specific webhook event];
      E --> F[Display Webhook Event Detail View: slide-out panel];
      F --> G[User reviews webhook payload, headers, received time, processing status, logs, transformed data: if applicable];
      G --> H{Webhook related to a Phoenix Transaction?};
      H -- Yes --> I[Display link/ID of related Phoenix Transaction];
      I --> J[User can optionally navigate to the Transaction Detail View for that transaction];
      H -- No --> K[User finishes review];
      J --> K;
  ```

### 4\. Investigate a Phoenix Transaction

- **Goal:** To understand the details of a specific Phoenix transaction, its current status, and its link back to original webhook events.
- **Actors:** User (Admin)
- **Trigger:** User navigates to the "Transactions List & Balances Overview" screen and selects a specific transaction (or navigates from a Webhook Event Detail).
- **Pre-conditions:** User is authenticated. Transactions exist in the Phoenix system.
- **Post-conditions:** User has a clear understanding of the transaction's data, its YNAB sync status, and the originating event(s).
- **Steps / Diagram:**
  ```mermaid
  graph TD
      A[User navigates to Transactions List & Balances Overview screen] --> B[System displays list of Phoenix transactions];
      B --> C{User identifies a transaction of interest};
      C -- Filter/Sort --> D[User applies filters/sorts to find specific transactions];
      D --> B;
      C -- Select Transaction --> E[User clicks on a specific transaction];
      E --> F[Display Transaction Detail View: slide-out panel];
      F --> G[User reviews transaction details: amount, date, description, current Up Bank status, YNAB sync status, YNAB transaction ID if synced];
      G --> H{Transaction linked to Webhook Event?};
      H -- Yes --> I[Display link/ID of originating Webhook Event];
      I --> J[User can optionally navigate to the Webhook Event Detail View for that event];
      H -- No --> K[User finishes review];
      J --> K;
  ```

### 5\. Change Settings (e.g., Target Up Bank Account for Balances)

- **Goal:** To modify application configurations, specifically the Up Bank account used for displaying balances on the "Transactions List & Balances Overview" screen.
- **Actors:** User (Admin)
- **Trigger:** User navigates to the "Settings" screen.
- **Pre-conditions:** User is authenticated.
- **Post-conditions:** The selected Up Bank account for balance display is updated in the system.
- **Steps / Diagram:**
  ```mermaid
  graph TD
      A[User navigates to Settings screen] --> B[System fetches and displays current settings, including available Up Bank accounts];
      B --> C[User selects the desired Up Bank account from a dropdown/list for balance display];
      C --> D[User confirms selection: e.g., clicks a Save button or selection is auto-saved];
      D --> E{Save Successful?};
      E -- Yes --> F[System confirms settings updated, stores new selection];
      F --> G[User sees confirmation/updated state on Settings screen];
      E -- No --> H[System displays error message];
      H --> B;
  ```

### 6\. Manually Trigger Sync

- **Goal:** To initiate an on-demand synchronization of transactions from Up Bank to YNAB.
- **Actors:** User (Admin)
- **Trigger:** User clicks a "Trigger Sync Now" button (likely located on the "System Status" screen or as a global action).
- **Pre-conditions:** User is authenticated. System is configured for sync.
- **Post-conditions:** The sync process is initiated. Feedback on initiation is provided. Sync results will be reflected in logs and system status.
- **Steps / Diagram:**
  ```mermaid
  graph TD
      A[User navigates to a screen with the Trigger Sync action: e.g., System Status] --> B[User clicks Trigger Sync Now button];
      B --> C[System initiates the core sync orchestration logic];
      C --> D[UI provides immediate feedback, e.g., Sync initiated...];
      D --> E[User can monitor progress/outcome via Event Log and System Status updates];
  ```

## Wireframes & Mockups

- **Link to Primary Design Files:** Not applicable. Visual design and layout will be
  developed directly in code, guided by conceptual layouts below.
- **Approach:** Conceptual layouts for key screens will be described textually and
  illustrated with ASCII art style diagrams to guide frontend development.
  To ensure responsiveness is considered from the outset, at least two versions
  of each layout will be provided:
  - **Desktop/Wide:** Aiming for a maximum width of 100 characters.
  - **Mobile/Narrow:** Aiming for a width of approximately 48 characters.
    Iteration on design will occur directly during the coding phase.

### Conceptual Layouts:

#### 1\. Transactions List & Balances Overview (Default Screen)

**Desktop/Wide (100c):**

```plaintext
+--------------------------------------------------------------------------------------------------+
| Main Dashboard Layout (e.g., Sidebar Navigation on Left, Header at Top)                            |
| +------------------------------------------------------------------------------------------------+ |
| | [Screen Title: Transactions & Balances]                                                        | |
| +------------------------------------------------------------------------------------------------+ |
| |                                                                                                | |
| | Balances Section:                                                                              | |
| | +------------------------------------------+ +-------------------------------------------------+ | |
| | | Up Bank - Available Balance:             | | YNAB - Cleared Balance:                         | | |
| | |   $XXXX.XX                               | |   $YYYY.YY                [Copy Balance Button] | | |
| | |   (Last checked: YYYY-MM-DD HH:MM:SS)    | |                                                 | | |
| | +------------------------------------------+ +-------------------------------------------------+ | |
| | | Difference: $ZZZ.ZZ (Up - YNAB)          |                                                 | | |
| | +------------------------------------------+                                                 | |
| |                                                                                                | |
| | Transactions Section:                                                                          | |
| | +----------------------------------------------------------------------------------------------+ | |
| | | Filters:                                                                                     | | |
| | | [Date Range Picker Btn] [Status Dropdown v] [Search Input: Description...........] [Apply]    | | |
| | +----------------------------------------------------------------------------------------------+ | |
| |                                                                                                | | |
| | Transactions List (Table):                                                                     | | |
| | |----------------------------------------------------------------------------------------------| | |
| | | Date       | Description                | Amount   | Up Status  | YNAB Status | Actions       | | |
| | |------------|----------------------------|----------|------------|-------------|---------------| | |
| | | 2025-05-15 | Coffee Purchase            |  -$5.00  | Settled    | Uncleared   | [Details >]   | | |
| | | 2025-05-14 | Salary Deposit             | +$500.00 | Settled    | Cleared     | [Details >]   | | |
| | | ...        | ...                        | ...      | ...        | ...         | [Details >]   | | |
| | |----------------------------------------------------------------------------------------------| | |
| |                                                                                                | | |
| | Pagination: [<< Prev]  [Page 1 of X]  [Next >>]          [Items per page: 25 v]                | | |
| | +----------------------------------------------------------------------------------------------+ | |
| |                                                                                                | |
| +------------------------------------------------------------------------------------------------+ |
+--------------------------------------------------------------------------------------------------+
```

**Mobile/Narrow (48c):**

```plaintext
+------------------------------------------------+
| Dashboard (Burger Nav | Header)                |
+------------------------------------------------+
| [Transactions & Balances]                      |
+------------------------------------------------+
| Balances:                                      |
| Up Bank: $XXXX.XX                              |
|   (At: YY-MM-DD HH:MM)                         |
| YNAB Clr: $YYYY.YY [Copy]                      |
| Diff: $ZZZ.ZZ                                  |
|------------------------------------------------|
| Filters:                                       |
| [Dates Btn] [Status v] [Search Btn]            |
|------------------------------------------------|
| Transactions:                                  |
| +----------------------------------------------+ |
| | TXN 1 (e.g., Card View)                      | |
| | Date: 2025-05-15                             | |
| | Desc: Coffee Purchase                        | |
| | Amt: -$5.00                                  | |
| | Up: Settled | YNAB: Uncleared                | |
| |                           [Details >]        | |
| +----------------------------------------------+ |
| | TXN 2                                        | |
| | Date: 2025-05-14                             | |
| | Desc: Salary Deposit                         | |
| | Amt: +$500.00                                | |
| | Up: Settled | YNAB: Cleared                  | |
| |                           [Details >]        | |
| +----------------------------------------------+ |
| | ...                                          | |
|------------------------------------------------|
| Pagination:                                    |
| [<< Prev] [1/X] [Next >>]                      |
+------------------------------------------------+
```

#### 2\. Transaction Detail View (Slide-out Panel)

**Desktop/Wide (100c):**

```plaintext
+--------------------------------------------------------------------------------------------------+
| Main View (e.g., Transactions List) Partially Visible Behind Panel                               |
| +---------------------------------------------------------+--------------------------------------+
| | Dimmed/Inactive Content                                 | [Transaction Detail Panel]           |
| |                                                         |                                      |
| |                                                         | Panel Title: Transaction Details [X] |
| |                                                         |--------------------------------------|
| |                                                         |                                      |
| |                                                         | **Phoenix Transaction ID:** {uuid}     |
| |                                                         |                                      |
| |                                                         | **Up Bank Details:** |
| |                                                         |   ID: {up_bank_txn_id}               |
| |                                                         |   Date: 2025-05-15 10:30:00          |
| |                                                         |   Description: Coffee Purchase       |
| |                                                         |   Message: Optional user message     |
| |                                                         |   Amount: -$5.00                     |
| |                                                         |   Status: SETTLED                    |
| |                                                         |   Foreign Amount: null               |
| |                                                         |   Settled At: 2025-05-15 10:31:00    |
| |                                                         |                                      |
| |                                                         | **YNAB Sync Details:** |
| |                                                         |   Status: UNCLEARED / CLEARED / SYNCED |
| |                                                         |   YNAB Txn ID: {ynab_txn_id_if_any}  |
| |                                                         |   Last Synced: 2025-05-15 10:32:00   |
| |                                                         |   Sync Attempts: 1                   |
| |                                                         |                                      |
| |                                                         | **Originating Webhook(s):** |
| |                                                         |   - ID: {webhook_id_1} [View >]      |
| |                                                         |     Type: up:transaction.created     |
| |                                                         |     Received: 2025-05-15 10:30:05    |
| |                                                         |   - ID: {webhook_id_2} [View >]      |
| |                                                         |     Type: up:transaction.settled     |
| |                                                         |     Received: 2025-05-15 10:31:02    |
| |                                                         |                                      |
| |                                                         | **Raw Data (Collapsible Section):** |
| |                                                         |   [+] Phoenix Internal Data          |
| |                                                         |   [+] Last Mapped YNAB Payload       |
| |                                                         |                                      |
| |                                                         | [Close Button]                       |
| +---------------------------------------------------------+--------------------------------------+
+--------------------------------------------------------------------------------------------------+
```

**Mobile/Narrow (48c):**

```plaintext
+------------------------------------------------+
| Transaction Details                  [Close X] |
+------------------------------------------------+
| PID: {uuid_short}                              |
|------------------------------------------------|
| Up Bank:                                       |
|   ID: {up_bank_txn_id_short}                   |
|   Date: 2025-05-15 10:30                       |
|   Desc: Coffee Purchase                        |
|   Amt: -$5.00      Status: SETTLED             |
|------------------------------------------------|
| YNAB Sync:                                     |
|   Status: UNCLEARED                            |
|   YNAB ID: {ynab_id_short_if_any}              |
|   Synced: 2025-05-15 10:32                     |
|------------------------------------------------|
| Originating Webhook(s):                        |
|   - WH ID: {wh_id_1_short} [>]                 |
|     Type: up:txn.created                       |
|   - WH ID: {wh_id_2_short} [>]                 |
|     Type: up:txn.settled                       |
|------------------------------------------------|
| Raw Data:                                      |
|   [+] Phoenix Internal                         |
|   [+] Mapped YNAB Payload                      |
|------------------------------------------------|
|                                  [Close Btn]   |
+------------------------------------------------+
```

#### 3\. Webhook Event Detail View (Slide-out Panel)

**Desktop/Wide (100c):**

```plaintext
+--------------------------------------------------------------------------------------------------+
| Main View (e.g., Webhook List) Partially Visible Behind Panel                                    |
| +---------------------------------------------------------+--------------------------------------+
| | Dimmed/Inactive Content                                 | [Webhook Event Detail Panel]         |
| |                                                         |                                      |
| |                                                         | Panel Title: Webhook Details   [X]   |
| |                                                         |--------------------------------------|
| |                                                         |                                      |
| |                                                         | **Webhook Event ID:** {uuid}         |
| |                                                         |                                      |
| |                                                         | **Source:** Up Bank / GitHub         |
| |                                                         | **Type:** up:transaction.created     |
| |                                                         | **Received At:** 2025-05-15 10:30:05 |
| |                                                         | **Processing Status:** SUCCESS / FAIL  |
| |                                                         |                                      |
| |                                                         | **Related Phoenix Transaction(s):** |
| |                                                         |   - ID: {phoenix_txn_id_1} [View >]  |
| |                                                         |                                      |
| |                                                         | **Raw Request (Collapsible):** |
| |                                                         |   [+] Headers                        |
| |                                                         |   [+] Payload (JSON Viewer)          |
| |                                                         |                                      |
| |                                                         | **Processing Logs (Collapsible):** |
| |                                                         |   [+] Logs (chronological)           |
| |                                                         |     - Timestamp: Step 1              |
| |                                                         |     - Timestamp: Step 2              |
| |                                                         |                                      |
| |                                                         | **Transformed Data (if applicable):**|
| |                                                         |   [+] Data Sent to YNAB (JSON)       |
| |                                                         |                                      |
| |                                                         | [Close Button]                       |
| +---------------------------------------------------------+--------------------------------------+
+--------------------------------------------------------------------------------------------------+
```

**Mobile/Narrow (48c):**

```plaintext
+------------------------------------------------+
| Webhook Details                      [Close X] |
+------------------------------------------------+
| WH ID: {uuid_short}                            |
|------------------------------------------------|
| Source: Up Bank                                |
| Type: up:transaction.created                   |
| Received: 2025-05-15 10:30                     |
| Status: SUCCESS                                |
|------------------------------------------------|
| Related Phoenix Txn(s):                        |
|   - ID: {phoenix_id_short} [>]                 |
|------------------------------------------------|
| Raw Request:                                   |
|   [+] Headers                                  |
|   [+] Payload                                  |
|------------------------------------------------|
| Processing Logs:                               |
|   [+] View Logs                                |
|------------------------------------------------|
| Transformed Data:                              |
|   [+] YNAB Payload                             |
|------------------------------------------------|
|                                  [Close Btn]   |
+------------------------------------------------+
```

#### 4\. Settings Screen

**Desktop/Wide (100c):**

```plaintext
+--------------------------------------------------------------------------------------------------+
| Main Dashboard Layout (e.g., Sidebar Navigation on Left, Header at Top)                            |
| +------------------------------------------------------------------------------------------------+ |
| | [Screen Title: Settings]                                                                       | |
| +------------------------------------------------------------------------------------------------+ |
| |                                                                                                | |
| |   Application Settings                                                                         | |
| |   --------------------                                                                         | |
| |                                                                                                | |
| |   **Up Bank Account for Balance Display:** | |
| |     Select the Up Bank account you wish to use for the balance display on the                  | |
| |     "Transactions & Balances" overview page.                                                   | |
| |                                                                                                | |
| |     +----------------------------------------------------------------------------------------+ | |
| |     | Current Account: {account_display_name_or_id_if_set}                                   | | |
| |     +----------------------------------------------------------------------------------------+ | |
| |                                                                                                | |
| |     Available Accounts (fetched from Up Bank API):                                             | |
| |     <Dropdown Menu Label: "Select Up Bank Account">                                            | |
| |     [ Account Nickname 1 (Product Name) - $XXXX.XX Available        ▼ ]                        | |
| |       ( Account ID: {up_bank_account_id_1} )                                                 | |
| |                                                                                                | |
| |     [ Save Settings Button ]                                                                   | |
| |                                                                                                | |
| |   ---                                                                                          | |
| |                                                                                                | |
| |   **Notification Settings (Future Placeholder):** | |
| |     [ ] Enable SMS notifications for critical errors                                           | |
| |     Phone Number: [___________________]                                                        | |
| |                                                                                                | |
| |   ---                                                                                          | |
| |                                                                                                | |
| |   **Theme (Future Placeholder):** | |
| |     (o) Light Mode  ( ) Dark Mode                                                              | |
| |                                                                                                | |
| +------------------------------------------------------------------------------------------------+ |
+--------------------------------------------------------------------------------------------------+
```

**Mobile/Narrow (48c):**

```plaintext
+------------------------------------------------+
| Dashboard (Burger Nav | Header)                |
+------------------------------------------------+
| [Screen Title: Settings]                       |
+------------------------------------------------+
| Up Bank Account for Balances:                  |
| Current: {current_acct_name_short}             |
|                                                |
| Select Account:                                |
| [ Acc Nickname 1 ($XXXX.XX)          ▼ ]       |
|   (ID: {up_acct_id_1_short})                  |
|                                                |
| [ Save Settings Button ]                       |
|------------------------------------------------|
| Notifications (Future):                        |
| [ ] SMS for errors                             |
| Phone: [________________]                      |
|------------------------------------------------|
| Theme (Future):                                |
| (o) Light  ( ) Dark                            |
+------------------------------------------------+
```

## Component Library / Design System Reference

- **Primary Source:** Shadcn/UI will serve as the primary collection of re-usable UI components for this project. Components will be integrated into the application as needed by copying them into the codebase, allowing for full customization.
- **Customization:** Components sourced from Shadcn/UI will be styled and customized using Tailwind CSS (v4) and will be configured to use CSS variables for theming, as specified in the Project Brief. This ensures adherence to the project's visual style and allows for flexibility.
- **Development & Showcase:** Storybook (targeting v9) will be implemented for developing, testing, and showcasing all UI components, whether they are customized Shadcn/UI components or bespoke components created for Phoenix. This will serve as the living documentation for the project's UI building blocks.
- **Component Strategy:**
  - While Shadcn/UI provides a comprehensive set of components, initial development will focus on integrating and customizing components essential for the key screens and user flows defined (e.g., buttons, tables, input fields, modals/dialogs for slide-out panels, navigation elements, cards for mobile views).
  - New bespoke components will be created if a suitable Shadcn/UI primitive is not available or if the required customization is too extensive. These will also be documented in Storybook.
- **Goal:** To maintain a consistent, accessible, and maintainable UI by leveraging a well-structured component-based architecture, facilitated by Shadcn/UI and documented in Storybook.

## Branding & Style Guide Basics

- **Overall Aesthetic:** Clean, modern, and functional, aligning with the default styling provided by Shadcn/UI. The primary focus is on clarity, usability, and information density suitable for a debugging and administration tool.
- **Color Palette:**
  - **Default Theme (Light):** Will utilize the default light theme provided by Shadcn/UI, which typically includes a neutral background (e.g., white/light gray), dark text for readability, and a primary accent color (e.g., a muted blue or gray).
  - **Accent Color:** The default Shadcn/UI accent color will be used initially. This can be easily customized project-wide by adjusting CSS variables if a specific preference emerges.
  - **Feedback Colors:** Standard colors for success (green), error (red), warning (yellow/amber), and informational (blue) states will be adopted, likely from Tailwind CSS's default palette or as configured within Shadcn/UI components.
- **Typography:**
  - **Font Family:** Will use the default sans-serif font stack provided by Shadcn/UI (often based on `Inter` or a similar modern sans-serif font).
  - **Hierarchy:** Standard typographic scale will be applied as per Shadcn/UI component defaults and Tailwind CSS utilities.
- **Iconography:**
  - **Delivery System:** Icons will be managed and delivered using the **Iconify** project.
  - **Primary Icon Set:** The **Lucide** icon set will be the primary choice to ensure visual consistency with the design language of Shadcn/UI components. Iconify's tools will facilitate the use of Lucide icons.
  - **Implementation Method:** The primary method for including icons will be Iconify's Web Component approach (`<iconify-icon icon="lucide:home"></iconify-icon>`), as recommended by the Iconify project. This will be tested for any potential impact on perceived load time (e.g., icons appearing after initial content render). The `unplugin-icons` library (for direct SVG embedding at build time) will be considered as a fallback or alternative if the Web Component approach presents challenges for this specific project's needs.
  - **Consistency:** Regardless of the chosen implementation detail, icon usage will be consistent across the application.
- **Spacing & Grid:**
  - Will adhere to the spacing scale and layout utilities provided by Tailwind CSS.
- **Logo:** Not applicable for this internal tool at this stage.

## Accessibility (AX) Requirements

- **Target Compliance:** Aim for WCAG 2.1 AA conformance as a guiding principle. While formal certification is not required for this personal tool, adhering to AA guidelines will ensure a high level of accessibility.
- **Key Principles & Considerations:**
  - **Semantic HTML:** Use HTML5 elements according to their semantic meaning to ensure proper structure and interpretation by assistive technologies.
  - **Keyboard Navigation:** All interactive elements (buttons, links, form fields, custom components) must be fully operable via keyboard. This includes:
    - Logical focus order.
    - Visible focus indicators (Tailwind CSS and Shadcn/UI typically provide good defaults, which will be maintained or enhanced if necessary).
    - Keyboard trap prevention (e.g., in modals/slide-out panels).
  - **ARIA Attributes:** Where standard HTML elements are insufficient for conveying role, state, or properties (especially for custom components derived from Shadcn/UI primitives or built from scratch), appropriate ARIA attributes will be used. Shadcn/UI components generally have good ARIA support out-of-the-box.
  - **Color Contrast:** Ensure text and important UI elements meet WCAG AA contrast ratios (4.5:1 for normal text, 3:1 for large text and graphical objects/UI components). The default Shadcn/UI themes are generally good starting points, but custom color choices will be checked.
  - **Screen Reader Compatibility:** Test with a screen reader (e.g., NVDA, VoiceOver, JAWS) on key views to ensure content is understandable and navigable.
  - **Forms:**
    - All form inputs will have explicitly associated labels.
    - Validation errors will be clearly communicated to assistive technologies.
  - **Responsive Design:** The responsive layouts (as conceptualized in the ASCII wireframes) will ensure content remains accessible and usable across different screen sizes without loss of information or functionality.
  - **Iconography:** Icons used for conveying information or actions will have appropriate text alternatives (e.g., `aria-label`, visually hidden text) if their meaning is not clear from surrounding text. Decorative icons will be hidden from assistive technologies.
- **Testing & Tooling:**
  - **Linters/Static Analysis:** Utilize ESLint plugins like `eslint-plugin-jsx-a11y` during development to catch common accessibility issues.
  - **Browser Developer Tools:** Use built-in accessibility inspectors (e.g., Chrome DevTools Accessibility Pane) for checks.
  - **Component-Level Testing (Storybook):** Leverage the Axe accessibility addon within Storybook to test individual components and verify their conformance during development.
  - **Automated Tools (Occasional Checks):** Tools like Axe DevTools browser extension or Lighthouse accessibility audits can be used for periodic checks on key views.

## Responsiveness

- **Core Principle:** The UI must be responsive and optimized for usability across a range of screen sizes, with a primary focus on modern iPhone screen sizes (as per Project Brief) and standard desktop views.
- **Breakpoints & Target Layouts:**
  - **Mobile/Narrow (Primary Target):**
    - **Target Width:** Approximately 48 characters for ASCII art conceptualization.
    - **Viewport Equivalent:** Aiming for usability on screens around 375px to 480px wide (typical for many mobile devices, including iPhones).
    - **Layout Strategy:** Single-column layouts, stacked elements, card-based views for lists, tab or burger navigation, and modals/full-screen takeovers for detailed views or forms.
  - **Desktop/Wide:**
    - **Target Width:** Approximately 100 characters for ASCII art conceptualization.
    - **Viewport Equivalent:** Aiming for usability on screens \~1024px wide and above.
    - **Layout Strategy:** Multi-column layouts where appropriate (e.g., main content area with sidebar navigation), expanded tables, direct display of filter controls, and slide-out panels or modals for detailed views that don't obscure the entire primary view.
  - **Intermediate Sizes (Tablets):** While not explicitly detailed with a third ASCII art size, layouts should adapt fluidly between the mobile and desktop breakpoints. Components and layouts will be built using Tailwind CSS's responsive modifiers (e.g., `sm:`, `md:`, `lg:`) to ensure graceful adaptation. Content should remain readable and interactive elements easily accessible.
- **Adaptation Strategy:**
  - **Fluid Grids & Flexbox:** Utilize Tailwind CSS utilities for fluid grids, flexbox, and CSS Grid to allow content to reflow and resize naturally.
  - **Image Handling:** (Though not image-heavy) If images are used, they will be responsive (e.g., `max-width: 100%`).
  - **Navigation:** Navigation patterns will adapt (e.g., sidebar visible on desktop, potentially collapsed to a burger menu on mobile).
  - **Touch Targets:** Ensure interactive elements have adequate touch target sizes on mobile.
- **Testing:** Test responsiveness during development using browser developer tools to simulate different viewport sizes. Physical device testing on an iPhone (as per user preference) is recommended for key views before considering them complete.
