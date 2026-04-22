# Frontend Agent Rules

## Stack
- **Core:** Vite, React (TS), TailwindCSS, shadcn/ui (use CLI to add), lucide-react.
- **Routing:** Tanstack Router (Code-based for apps, file-based for sites).
- **State:** React Query (remote), Zustand (global - only if strictly needed), Tanstack searchParams (URL).
- **Forms:** React Hook Form + Zod.
- **Testing:** Vitest, RTL, Playwright.
- **Linting:** ESLint.

## Architecture: Feature-Sliced Design (FSD)
- **Layers (Top down):** `app` -> `processes` (optional) -> `pages` -> `widgets` -> `features` -> `entities` -> `shared`.
- **Rules:** Unidirectional dependency (only import from layers below). Cross-slice imports ONLY via `index.ts` Public API.
- **Slice Internals:** `ui/`, `model/` (state/hooks/schemas), `api/` (requests), `lib/` (utils).

## Accessibility (WCAG 2.2 AA)
- Semantic HTML > ARIA.
- 100% keyboard navigable (visible focus, no traps).
- Meaningful `alt` for images, `.sr-only`/`aria-label` for icons, `aria-live` for dynamic content.
- Contrast: 4.5:1 (text), 3:1 (large text/UI).
- Min touch target: 24x24px.
- Respect `prefers-reduced-motion`.

## Testing Strategy
- **Unit/Integration:** Test user behavior. Query by a11y roles (`getByRole`). Use `@testing-library/user-event`. Assert with `jest-axe`.
- **E2E:** Cover critical flows. Automate a11y with `@axe-core/playwright`.

## Strict Guardrails & Best Practices
- NEVER use `any` or `@ts-ignore`.
- ALWAYS use named exports (`export const Component = ...`).
- NEVER install npm packages without asking.
- AVOID duplication: Check `shared/ui` (or shadcn) before creating UI components.
- KEEP files under ~200 lines. If a component grows too large, extract complex state/logic into custom hooks (model/) and isolate sub-components into smaller files (ui/) to maintain readability.
- PREFER functional components with explicit TS prop interfaces.
- NO `useEffect` for data fetching. Use React Query.
- HANDLE loading/errors: Use Tanstack Router `loader` and/or `<Suspense>` + Error Boundaries.
- USE absolute path aliases (e.g., `@/features/auth`) instead of deep relative imports. Ensure `tsconfig.json` (`paths`) and `vite.config.ts` (`resolve.alias` or `vite-tsconfig-paths`) are configured to support the `@/` prefix mapping to `./src/`.
- BEFORE creating a new feature slice, check if the required logic already exists. If it exists in shared or entities, reuse it. If it exists in a sibling slice (like another feature), DO NOT cross-import. Instead, extract the shared logic down into entities or shared first.
