# Frontend Guideline Document for AI-LMS Portal

This document outlines the frontend architecture, design principles, styling approach, component structure, state management, routing, performance optimizations, testing strategies, and overall guidelines you need to build and maintain the AI-LMS Portal with clarity and consistency.

## 1. Frontend Architecture

**Framework and Language**
- **Next.js (App Router)**: Our core framework handles both page routing and server-side logic. It gives us fast page loads, built-in code splitting, image optimization, and API routes if needed.
- **TypeScript**: Provides type safety across components, API clients, and hooks, helping catch errors early and improving developer experience.

**UI Library and Styling**
- **shadcn/ui**: A set of prebuilt, customizable React components (tables, forms, buttons, charts) that align with our design system.
- **Tailwind CSS**: A utility-first CSS framework that keeps styles consistent and eliminates naming conflicts.

**Authentication and Roles**
- **NextAuth.js with Google Provider**: Manages sign-in and sign-out flows. After login, we assign user roles (`IT_ADMIN`, `GURU`, `SISWA`) based on email and protect routes accordingly.

**API Layer**
- **Custom API Client in `/lib`**: Contains fetch wrappers (using `fetch`) to communicate with Google Apps Script (GAS) web apps. We replace the traditional ORM layer (Drizzle + PostgreSQL) with lightweight, REST-style calls.
- **SWR or React Query**: Handles data fetching, caching, and synchronization for a smooth, type-safe experience.

**Containerization (optional)**
- **Docker**: Provides consistent local environments. It’s optional if you deploy on Vercel but can be helpful for complex setups.

This architecture is designed for **scalability** (clear module boundaries), **maintainability** (type safety, component reuse), and **performance** (server components, code splitting, caching).

## 2. Design Principles

1. **Usability**: Clear navigation, meaningful labels, and consistent feedback (loading spinners, error messages).
2. **Accessibility**: All interactive elements are keyboard-navigable, use semantic HTML, and meet WCAG color-contrast guidelines.
3. **Responsiveness**: Layouts adapt gracefully from mobile to desktop using Tailwind’s responsive utilities.
4. **Consistency**: Common components (buttons, inputs, tables) follow a unified style and behavior.
5. **Performance-First**: Minimize initial bundle size, lazy-load heavy components, and optimize images.

These principles guide every UI decision—from form layouts to data table interactions—to ensure our portal is easy to use for teachers, students, and administrators.

## 3. Styling and Theming

**Styling Approach**
- **Utility-First (Tailwind CSS)**: We write classes like `bg-indigo-50 px-4 py-2 rounded` to style elements, reducing the need for custom CSS files.
- **CSS Variables for Theming**: Define colors and spacing in `tailwind.config.js`, enabling light and dark mode through the `class` strategy.

**Theming**
- Toggle light/dark mode via a root classname (`<html class="dark">`).
- Use CSS variables (`--color-primary`, `--color-background`) for easy theme adjustments.

**Design Style**
- **Modern & Flat**: Clean surfaces, minimal shadows, intuitive controls.
- **Glassmorphism** (optional for cards): Subtle frosted-glass effect with `backdrop-filter: blur(10px)`, used sparingly for highlights.

**Color Palette**
- Primary: #4F46E5 (Indigo 600)
- Secondary: #6366F1 (Indigo 500)
- Accent: #10B981 (Green 500)
- Background (light): #F9FAFB (Gray 50)
- Background (dark): #1F2937 (Gray 800)
- Text (light): #111827 (Gray 900)
- Text (dark): #F3F4F6 (Gray 100)

**Font**
- **Inter**: A modern, highly legible sans-serif font loaded via Google Fonts. Use `font-sans` in Tailwind for body text and UI elements.

## 4. Component Structure

**Organization**
- `/app`: Next.js App Router pages and layouts.
- `/components`: Reusable UI elements (buttons, forms, tables, charts).
- `/lib`: Utility functions, API client, authentication setup.

**Reusable Components**
- **DataTable**: For tabular data (materials list, feedback entries).
- **ChartAreaInteractive**: Interactive charts for teacher dashboards.
- **AuthButtons**: Google OAuth button, sign-out button, etc.

**Benefits of Component-Based Architecture**
- **Maintainability**: Fix or enhance one component and see changes everywhere it’s used.
- **Testability**: Isolate components for unit tests.
- **Consistency**: Uniform look and behavior across the app.

## 5. State Management

**Server Data**
- **React Query (or SWR)**: Fetches data from GAS, caches it, and keeps UI in sync. Key features:
  - Automatic refetch on focus or reconnect
  - Configurable stale times and retry logic
  - Built-in loading and error states

**Local UI State**
- **React Context**: For global UI concerns (theme toggling, modal states).
- **useState/useReducer**: For local component interactions (form inputs, pagination controls).

This hybrid approach ensures data consistency across components and leverages React’s built-in hooks where appropriate.

## 6. Routing and Navigation

- **Next.js App Router**: Create routes under `/app`:
  - `/login` (sign-in page)
  - `/dashboard` (Guru portal)
  - `/materials` (Guru material management)
  - `/my-feedback` (Siswa portal)
  - `/admin/settings` (IT_ADMIN portal)

- **Protected Routes**: Wrap pages in a `SessionProvider` and check `useSession()`:
  - Redirect unauthenticated users to `/login`.
  - Redirect users without the proper role to their designated portal.

- **Navigation Layout**: A shared sidebar or topbar component in `/app/layout.tsx` that changes links based on `session.user.role`.

## 7. Performance Optimization

- **Code Splitting & Lazy Loading**: Next.js automatically splits by route. For large charts or maps, use `dynamic(() => import(…))`.
- **Image Optimization**: Use Next.js `<Image>` for automatic resizing and lazy loading.
- **Server Components**: Fetch data on the server and stream HTML to the client to reduce bundle size.
- **Caching**: Configure React Query’s `staleTime` and `cacheTime` for API data.
- **Bundle Analysis**: Use `next-bundle-analyzer` to inspect and reduce large dependencies.

## 8. Testing and Quality Assurance

**Unit Tests**
- **Jest + React Testing Library**: Test component rendering, user interactions, and utility functions.

**Integration Tests**
- Test pages with mocked React Query or SWR caches to ensure data-driven components render correctly.

**End-to-End (E2E) Tests**
- **Cypress or Playwright**: Simulate real user flows (login, data fetch, role-based redirects).

**Linters and Formatters**
- **ESLint**: Enforce code style and catch errors.
- **Prettier**: Auto-format code for consistency.

**Continuous Integration**
- On each pull request, run lint, tests, and type checks.
- Deploy preview environments via Vercel for manual QA.

## 9. Conclusion and Overall Frontend Summary

This guideline defines a clear, scalable, and maintainable frontend for the AI-LMS Portal. By leveraging Next.js with TypeScript, utility-first styling (Tailwind CSS), a component-driven approach (shadcn/ui), and robust state management (React Query), we ensure a fast and consistent user experience for students, teachers, and administrators.

Key takeaways:
- **Role-Based Access** with NextAuth.js and server-side protections
- **Custom API Client** connecting to Google Apps Script for data operations
- **Reusable Components** for tables, charts, and forms
- **Performance Best Practices** baked into our build and runtime
- **Comprehensive Testing** to keep the portal reliable and bug-free

Following these guidelines will help maintain a high standard of code quality, deliver a polished UI, and accelerate future feature development.