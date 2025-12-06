# 📖 README: Enterprise Multi-Tenant Monorepo

This repository hosts the Next.js applications (`client`, `admin`) and shared packages (`ui`, `types`, `api-client`) for our scalable multi-tenant platform.

It is managed via **pnpm Workspaces** under the principle of **"One Build, Dynamic Configuration"**—meaning the core `client` app binary is identical for all clients, with differences managed by API-driven configuration.

---

## 1. 🚀 Getting Started

### Prerequisites

* **Node.js** (LTS version)
* **pnpm** (Installed globally: `npm install -g pnpm`)

### Local Setup

1.  **Clone the repository:**
    ```bash
    git clone [your-repo-url] enterprise-monorepo
    cd enterprise-monorepo
    ```
2.  **Install dependencies and link workspaces:**
    ```bash
    pnpm install
    ```
3.  **Run Development Servers:**

    We use the root `package.json` scripts to target specific applications:

    ```bash
    # Run the client application (Tenant Facing)
    pnpm run dev:client 

    # Run the admin dashboard
    pnpm run dev:admin 
    ```

> 💡 **Tip:** `pnpm install` automatically links all shared packages (`@repo/*`) into the `node_modules` of both apps, thanks to the `workspace:*` protocol.

---

## 2. 🏛️ Architecture & Structure

The repository is divided into two primary sections: `apps/` (independent applications) and `packages/` (shared libraries).

| Folder | Name | Description & Usage |
| :--- | :--- | :--- |
| `apps/client` | Client Application | The single, Next.js App Router application. Uses the **ClientConfigProvider** to fetch configuration based on the domain. |
| `apps/admin` | Admin Dashboard | Internal Next.js application for managing client provisioning and reports. Shares code but is a distinct product. |
| `packages/ui` | `@repo/ui` | Houses **all shared UI components** (e.g., Button, Card). Components must be generic and styled using Tailwind CSS classes. **Must not contain client-specific business logic.** |
| `packages/types` | `@repo/types` | **Single Source of Truth** for all TypeScript interfaces, DTOs (Data Transfer Objects), and Feature Flags. |
| `packages/api-client` | `@repo/api-client` | Centralized service layer for **all data fetching** logic. Contains `fetchClientConfig` and handles API fallbacks. |

---

## 3. 🧩 Development Conventions

### A. Code Sharing Philosophy

Any code, type, or component needed by **more than one app** must be extracted into a `packages/` library. **Do not** import code directly between `apps/client` and `apps/admin`.

### B. Adding a New Feature

1.  **Define Types First:** Create necessary interfaces (e.g., `InvoiceResponse`) in `packages/types/src/`.
2.  **Implement Fetching:** Add the corresponding function (e.g., `fetchInvoices`) in `packages/api-client/src/`.
3.  **Implement Component:** Create the component in `packages/ui/src/` if it's generic, or directly inside the target app's `src/` folder if it's application-specific.

### C. Type-Safe Imports

All imports from shared libraries must use the `@repo/` alias defined in the root `tsconfig.json`:

```typescript
// Correct
import { UserProfile } from '@repo/types';
import { Button } from '@repo/ui';

// Incorrect (Avoid relative paths for shared code)
// import { Button } from '../../../../packages/ui';