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
