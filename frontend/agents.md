# Frontend Agent Instructions

You are an expert frontend AI coding assistant. When working on this codebase, you must strictly adhere to the following tech stack, architectural guidelines, accessibility standards, and best practices.

## 1. Tech Stack
- **Build Tool / Framework:** Vite + React (TypeScript)
- **Routing:** Tanstack Router (`@tanstack/react-router`)
  - *Default to Code-based routing for complex webapps, and file-based routing for simpler websites.*
- **State Management & Data Fetching:** 
  - Remote State: React Query (`@tanstack/react-query`)
  - Local/Global State: Zustand (ONLY use if the app grows to the point that a global state is strictly necessary).
  - URL State: Leverage Tanstack Router's type-safe `searchParams` for URL-driven state (pagination, filters, tabs, etc.).
- **Forms & Validation:** React Hook Form + Zod
- **Styling:** TailwindCSS
- **UI Components:** shadcn/ui (radix-ui under the hood)
- **Icons:** `lucide-react`
- **Testing:** 
  - Unit/Integration: Vitest + React Testing Library
  - End-to-End: Playwright
- **Linting:** ESLint

## 2. Agent-Specific Guardrails
- **Avoid Duplication:** Before creating a new UI component, ALWAYS check `shared/ui` (or the shadcn folder) to see if it already exists.
- **Dependency Management:** Do NOT install new npm packages without asking for the user's explicit approval first.
- **shadcn/ui Installation:** When adding new shadcn components, use the CLI (`npx shadcn@latest add <component_name>`) rather than trying to hand-write or guess the component's code.

## 3. Architecture: Feature-Sliced Design (FSD)
The project strictly follows the [Feature-Sliced Design (FSD)](https://fsd.how/docs/get-started/overview/) architectural methodology. 

### Layers
Organize code into the following layers (from top to bottom):
1. **`app/`**: Global settings, global styles, and providers (routing, state, etc.).
2. **`processes/`**: (Optional) Complex cross-page scenarios and flows.
3. **`pages/`**: Compositional layer to construct routes/pages from entities, features, and widgets.
4. **`widgets/`**: Compositional layer combining entities and features into meaningful, standalone blocks (e.g., Header, Sidebar).
5. **`features/`**: User interactions, actions, and operations that bring business value (e.g., AddToCart, UserAuth).
6. **`entities/`**: Core business entities and domains (e.g., User, Product, Order).
7. **`shared/`**: Reusable, decoupled infrastructure code (UI kit components, utilities, API clients).

### Internal Slice Structure
To keep the codebase clean and consistent, complex slices (especially within `features/` and `entities/`) should follow a standard internal structure:
- `ui/`: UI components specific to the slice.
- `model/`: Business logic, state (Zustand stores), hooks, and types/schemas.
- `api/`: API requests and data fetching logic (React Query hooks).
- `lib/`: Slice-specific utilities and helpers.

### Strict FSD Rules
- **Unidirectional Dependency:** A layer can only import from layers *strictly below* it. For example, `features` can import from `entities` and `shared`, but NEVER from `widgets` or `pages`.
- **Public API:** Each slice or segment must expose its contents via an `index.ts` file (its Public API). Modules must ONLY import from other slices using their Public API, never reaching into internal files.

## 4. Accessibility (First-Class Priority - WCAG 2.2)
Accessibility is a primary requirement. All implementations must adhere to [WCAG 2.2 AA standards](https://www.w3.org/TR/WCAG22/).

- **Semantic HTML:** Always use native, semantic HTML elements (`<button>`, `<nav>`, `<main>`, `<dialog>`, `<header>`, etc.) over generic `<div>` wrappers. Only use ARIA roles when native semantics fall short.
- **Keyboard Navigation:** Every interactive element MUST be focusable and operable via keyboard alone. Ensure visible and distinct focus indicators (`focus-visible`). Do not trap focus unless within a modal/dialog.
- **Screen Reader Support:** 
  - Provide meaningful alternative text (`alt`) for all images.
  - Use `.sr-only` (screen-reader only text) or `aria-label` for icon-only buttons and links.
  - Announce dynamic content changes using `aria-live` regions.
- **Color Contrast:** Maintain at least a 4.5:1 contrast ratio for normal text and 3:1 for large text and interactive UI elements.
- **Target Size (WCAG 2.2 addition):** Ensure interactive touch targets are at least 24x24 CSS pixels minimum (44x44 recommended).
- **Reduced Motion:** Respect `prefers-reduced-motion` CSS media queries for animations.

## 5. Testing Strategy
- **Unit / Integration (Vitest + RTL):**
  - Test user behavior, not implementation details.
  - Query elements by accessible roles (`getByRole`, `getByLabelText`) to enforce accessibility at the test level.
  - Use `@testing-library/user-event` instead of `fireEvent`.
  - Integrate `jest-axe` to run basic accessibility assertions in unit tests.
- **End-to-End (Playwright):**
  - Cover critical user flows and paths.
  - Utilize `@axe-core/playwright` to run automated accessibility audits on full pages and states.

## 6. Coding Standards & Best Practices
- **Type Safety:** NEVER use `any` or `@ts-ignore`. Ensure rigorous TypeScript typing throughout the codebase.
- **Exports:** Use **named exports** exclusively (e.g., `export const Component = ...`) rather than `export default`. This improves refactoring, searchability, and consistency across the codebase.
- **Components:** Prefer functional components. Explicitly type component props using TypeScript interfaces.
- **Data Fetching & Loading States:** Rely on React Query for remote state and caching. Avoid using `useEffect` for data fetching. Use Tanstack Router `loader` functions and/or React `<Suspense>` boundaries paired with Error Boundaries to gracefully handle data fetching states.
- **Styling:** Use Tailwind utility classes. Extract highly repetitive utility combinations using `cva` (Class Variance Authority) or standard React components (often handled by shadcn).
- **Error Handling:** Implement Error Boundaries at the widget/page layer to gracefully catch and display errors. Handle API errors consistently.
