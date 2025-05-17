## Phoenix Project: Master AI Frontend Generation Prompt (for Dev Agent in Cursor AI)

**Project:** Phoenix - Admin & Debug UI for Up Bank to YNAB Sync Service

**Objective of this Prompt:** Provide comprehensive context to an AI development agent for generating frontend code (React 19, TypeScript, React Router v7, Tailwind CSS v4, Shadcn/UI, Zustand, Vitest) for the Phoenix application, an SSR React Router app running on Cloudflare Workers. The agent will be working within the existing project structure.

**Key Reference Documents (Assumed to be accessible or contextually understood by the agent):**

- `UI/UX Specification (phoenix-front-end-spec.md)`: Details user personas, UX goals, IA, user flows, conceptual wireframes (ASCII art), and visual/AX guidelines.
- `Frontend Architecture Document (phoenix-front-end-architecture.md)`: Details directory structure, component strategy, state management, API interaction, routing, build, testing, security, and performance guidelines.
- `Main System Architecture (architecture.md)`: Provides overall system context, backend API details (though frontend mostly interacts with its own worker's API routes or actions).
- `Product Requirements Document (prd.md)`: Outlines functional and non-functional requirements.

**I. Core Technology Stack & Architectural Principles:**

- **Framework:** React 19, React Router v7 (SSR on Cloudflare Workers).
- **Language:** TypeScript.
- **Styling:** Tailwind CSS v4. Adhere to utility-first principles. Use `tailwind.config.js` for theme extensions. Global styles in `apps/phoenix-app/frontend/app/styles/app.css`.
- **UI Components:**
  - Utilize **Shadcn/UI** components as the primary toolkit. Assume these are added via the Shadcn CLI (e.g., `npx shadcn-ui@latest add button card ...`) and reside in `apps/phoenix-app/frontend/app/lib/components/ui/`.
  - Customize Shadcn/UI components using Tailwind CSS utility classes directly.
  - Create custom components in `apps/phoenix-app/frontend/app/components/` (organized into `ui/`, `common/`, `features/`).
  - Follow naming conventions (PascalCase for components/files, lowercase for feature directories).
- **State Management:**
  - **React Router Loaders & Actions:** Primary mechanism for fetching server state and handling data mutations.
  - **Zustand:** Use for minimal global client-side UI state (e.g., notifications, theme, global UI toggles in `apps/phoenix-app/frontend/app/store/uiStore.ts`) and, if necessary, for complex local/feature state on a case-by-case basis. Avoid using Zustand for data already managed by React Router loaders/actions.
- **Routing:**
  - SSR React Router v7. Routes defined as JavaScript objects, likely composed from feature-specific route configurations (e.g., `apps/phoenix-app/frontend/app/features/{featureName}/{featureName}.routes.ts`) into a main router config (`apps/phoenix-app/frontend/app/router.config.ts`).
  - The root layout is `apps/phoenix-app/frontend/app/root.tsx`.
  - Route components/views likely in `apps/phoenix-app/frontend/app/features/{featureName}/views/`.
  - Authentication guards implemented server-side (in worker) via loaders/actions, potentially exploring React Router's experimental middleware.
- **API Interaction:**
  - Primarily via React Router `loader` and `action` functions using `Workspace` to interact with the Cloudflare Worker's API routes or action handlers.
  - For non-router client-side calls, use the `apps/phoenix-app/frontend/app/services/apiClient.ts`.
  - Error handling via React Router `errorElement` and `uiStore` for global notifications.
- **Directory Structure:** Adhere strictly to the defined structure in `Frontend Architecture Document` (NX monorepo with `apps/phoenix-app/frontend/` and `apps/phoenix-app/worker/`).
- **Testing:** Vitest with React Testing Library. Co-locate tests (`*.test.tsx` or `*.spec.tsx`). Storybook (`apps/phoenix-app/frontend/storybook/`) for component development and visual/AX testing (use `@storybook/addon-a11y`).

**II. Visual Design & User Experience (Reference UI/UX Specification):**

- **Aesthetic:** Clean, modern, functional, information-dense but not cluttered. Primarily black or white theme with configurable accent colors (Shadcn/UI default initially). Consider subtle "neon underglow" hints for accents if feasible with Tailwind.
- **Layout:** Responsive. Refer to ASCII art conceptual layouts for Desktop/Wide (100 chars) and Mobile/Narrow (approx. 48 chars) for key screens.
- **Iconography:** Use **Iconify** with the **Lucide** icon set. Implement via Iconify Web Components (`<iconify-icon icon="lucide:...">`).
- **Typography:** Default sans-serif stack from Shadcn/UI (e.g., Inter). Maintain clear hierarchy.
- **Accessibility:** Target WCAG 2.1 AA. Use semantic HTML, ARIA (leveraging Shadcn/UI), ensure keyboard navigation, visible focus, and good color contrast. Test with Storybook Axe addon.
- **Progressive Enhancement:** SSR ensures base content is available. Client-side JS enhances interactivity. Forms should be submittable without client JS if possible (via server actions). Use `<noscript>` where JS is critical.

**III. Key Screens & Components to Consider (Examples for common tasks):**

- **`apps/phoenix-app/frontend/app/root.tsx`:** Defines the HTML shell, global context providers, and `<Outlet />`.
- **`apps/phoenix-app/frontend/app/layouts/DashboardLayout.tsx` (or similar):** Main authenticated layout with sidebar navigation and header.
- **Login Page (`apps/phoenix-app/frontend/app/features/auth/views/LoginPage.tsx` or similar):** Handles Passkey and SMS OTP login flows via React Router actions.
- **Transactions List & Balances Overview (Default Authenticated View):**
  - Displays Up Bank Available Balance, YNAB Cleared Balance, and difference. "Copy YNAB Cleared Balance" button.
  - Filterable/sortable list of transactions. Columns: Date, Description, Amount, Up Status, YNAB Status, Actions (link to detail view).
  - Refer to ASCII art layouts for structure.
- **Transaction Detail View (Slide-out Panel):** Shows comprehensive details of a transaction, links to originating webhooks.
- **Webhook Events List & Detail View:** Similar to transactions.
- **Settings Page:** For configuring Up Bank account ID used for balance display.
- **Component Primitives (Examples - ensure these are built robustly if custom or used frequently from Shadcn):**
  - `Button` (various variants, with icon support)
  - `Table`/`DataGrid` (for transaction/webhook lists, with sorting, filtering integration)
  - `Input`, `Select`, `Checkbox` (for forms and filters)
  - `Card` (for mobile list items or dashboard widgets)
  - `Modal`/`Dialog`/`Sheet` (for slide-out detail views)
  - `NavigationMenu`/`Sidebar`

**IV. Specific Instructions for the Dev Agent (Example Task):**

- **(This is where you would prepend your specific task, e.g., "Implement the Transactions List & Balances Overview page as defined in the UI/UX spec and based on the conceptual ASCII wireframes. Create necessary components, route definitions, and loader functions...")**

**V. Important Considerations & Constraints:**

- **Adherence to Specifications:** Strictly follow the file paths, naming conventions, and architectural patterns outlined in the `Frontend Architecture Document`.
- **SSR Compliance:** All components and data fetching must be compatible with server-side rendering via React Router on Cloudflare Workers. Loaders are server-side, actions handle mutations.
- **Shadcn/UI Usage:** Prefer using and styling Shadcn/UI components. Create custom components only when necessary. Install Shadcn components using its CLI as needed.
- **TypeScript:** Use strong typing for all props, state, and function signatures.
- **Accessibility:** Build with accessibility in mind from the start.
- **Error Handling:** Implement robust error handling using React Router `errorElement` for route-level errors and clear feedback for API/action errors.
- **No Direct DOM Manipulation:** Use React's declarative approach.
- **Security:** Be mindful of XSS (rely on React's escaping) and other frontend security best practices from the architecture doc.

**VI. Output Expectations:**

- Clean, readable, well-commented TypeScript/TSX code.
- Components that adhere to the visual and functional specifications.
- Unit/integration tests (Vitest) for new components and logic.
- Storybook stories for new UI components.
- Updates to routing configuration, state stores, or service layers as necessary.
- All code should lint and format correctly according to project setup.
