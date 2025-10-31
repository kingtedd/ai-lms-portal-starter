# Project Requirements Document (PRD)

## 1. Project Overview

The AI-LMS Portal is a modern web application built as a starter kit to accelerate development of a full-featured Learning Management System (LMS) powered by AI and Google Apps Script (GAS). It provides a pre-configured architecture using Next.js for the frontend, `shadcn/ui` and Tailwind CSS for styling, and NextAuth.js with Google OAuth for authentication. Instead of a traditional database, it connects to GAS endpoints that read and write data in Google Sheets, enabling multi-tenant support and role-based access out of the box.

We’re building this starter kit so schools and educators can quickly launch a customized LMS without reinventing foundational features like sign-in flows, protected dashboards, data tables, and charts. The key success criteria are: 1) Seamless Google sign-in with automatic role assignment (IT_ADMIN, GURU, SISWA); 2) Dynamic dashboards that fetch live data from GAS; 3) A clean, responsive UI that adapts to desktop and mobile; and 4) A clear code structure that new developers can easily extend or replace with their own business logic.

## 2. In-Scope vs. Out-of-Scope

**In-Scope (Version 1.0)**
- NextAuth.js integration with Google Provider for authentication and role management.
- Protected routes for three roles: IT_ADMIN, GURU (teacher), SISWA (student).
- Page templates: `/` (login), `/dashboard` (teacher overview), `/materials` (teacher material management), `/my-feedback` (student feedback), `/admin/settings` (admin portal).
- Data fetch calls from GAS web apps using a lightweight API client in `/lib` and SWR or React Query for state management.
- Pre-built UI components (data tables, charts) from `shadcn/ui`, wired to live data.
- File upload form on `/materials` that sends base64-encoded files to GAS.
- Print stylesheet (`@media print`) for the teacher’s dashboard report.
- Docker configuration for optional local environment consistency.
- Deployment ready for Vercel with environment variables for GAS URLs.

**Out-of-Scope (Version 1.0)**
- Traditional relational database setup (Drizzle ORM, PostgreSQL).
- Mobile-specific native apps (React Native, SwiftUI).
- Advanced analytics beyond basic charts (machine-learning recommendations).
- Integration with external LMS platforms (Canvas, Moodle).
- Multi-language (i18n) support.
- Offline mode or PWA features.

## 3. User Flow

A new user visits the portal’s home page (`/`) and clicks “Sign in with Google.” NextAuth.js directs them through Google’s OAuth flow, then determines their role by examining the authenticated email against a predefined list in GAS. Once authenticated, the user is redirected to their role-specific landing page: teachers go to `/dashboard`, students to `/my-feedback`, and admins to `/admin/settings`. All subsequent requests include a session token; if a user tries to access a page they’re not authorized for, they’re redirected back to the login page with an error message.

On their landing page, users see a top navigation bar and a left sidebar (teachers and admins only). Teachers view class statistics and student summaries in interactive charts and tables, then navigate to `/materials` to upload or edit course materials. Students visit `/my-feedback` to read personalized comments and grades. Admins can adjust system settings, manage user-role mappings, and view global usage reports in `/admin/settings`. Each page fetches data from GAS, handles loading and error states gracefully, and displays user-friendly notifications.

## 4. Core Features

- **Authentication & Role Management**: Sign-up/sign-in with Google OAuth via NextAuth.js, automatic assignment of IT_ADMIN, GURU, SISWA roles, protected server-side routes.
- **Dynamic Dashboards**: Interactive data tables and charts showing class performance, student feedback, and admin metrics using `shadcn/ui` components.
- **API Client for GAS**: Lightweight fetch wrapper in `/lib/api.ts` with typed TypeScript interfaces for endpoints like `GET /materials`, `POST /materials`, `GET /my-feedback`, `GET/POST /settings`.
- **Material Upload**: File input form on `/materials`; files converted to base64 and sent to GAS endpoint with metadata.
- **Role-Based Navigation**: Conditional rendering of menus and pages based on `session.user.role`.
- **Print-Ready Reports**: Dedicated CSS rules under `@media print` to format teacher dashboards for printing via `window.print()`.
- **State Management**: SWR (Stale-While-Revalidate) or React Query to fetch, cache, and update data.
- **Error Handling & Notifications**: Global error boundary, toast notifications for success/failure, retries for transient network errors.

## 5. Tech Stack & Tools

- **Frontend Framework**: Next.js (App Router) with React Server Components.
- **Language**: TypeScript for end-to-end type safety.
- **Authentication**: NextAuth.js with Google Provider (OAuth2-based).
- **UI Library**: `shadcn/ui` for reusable components; Tailwind CSS for utility-first styling.
- **Data Fetching**: SWR or React Query.
- **Backend**: Google Apps Script (GAS) Web Apps acting as REST-like endpoints; data stored in Google Sheets.
- **Containerization**: Docker (optional) for local dev environment.
- **Hosting & CI/CD**: Vercel for automatic builds and deployments.
- **VS Code Plugins**: ESLint, Prettier, Tailwind CSS IntelliSense.

## 6. Non-Functional Requirements

- **Performance**: Initial page load under 2 seconds on 4G; subsequent API calls under 500ms.
- **Security**: OAuth2 flows secured by NextAuth.js; CSRF protection; server-side session checks; sanitize all user inputs to prevent XSS.
- **Usability & Accessibility**: WCAG 2.1 AA compliance; responsive design for mobile and desktop; keyboard navigation; ARIA labels on form controls.
- **Reliability**: 99.9% uptime on Vercel; retry logic for transient failures; fallback UIs for network errors.
- **Scalability**: Support hundreds of concurrent users per school; GAS quotas respected via caching and batching where possible.

## 7. Constraints & Assumptions

- Google Apps Script APIs must be deployed with web app URLs exposed and properly set up for cross-origin requests.
- Users must have Google Workspace or personal Google accounts.
- No relational database in version 1.0; all data lives in Google Sheets via GAS.
- Vercel hosting environment; environment variables (`NEXTAUTH_URL`, `GAS_API_BASE_URL`, etc.) handled via `.env.local`.
- Docker is optional and not required for deployment.

## 8. Known Issues & Potential Pitfalls

- **GAS Rate Limits**: Google Apps Script enforces per-minute quotas. Mitigate by implementing client-side caching and debouncing repeated requests.
- **CORS Errors**: If the GAS web app isn’t configured with `doGet(e) { return ContentService.create... }.setAccessControlAllowOrigin('*')`, fetch calls will fail. Always set proper HTTP headers.
- **Large File Uploads**: Base64 encoding inflates file size by ~33%. Consider chunked uploads or size limits (e.g., 5 MB max).
- **Stale Data**: SWR/React Query default stale times may not match real-time needs. Configure revalidation intervals or manual refetch on key actions.
- **Print Layout Variations**: Browser differences in print CSS may cause layout shifts. Test on Chrome, Firefox, and Edge, and use explicit page-break rules.


---

This PRD contains all necessary details for the AI model to generate technical specifications, frontend guidelines, backend structure, and deployment instructions without ambiguity. Feel free to use it as the single source of truth for the project’s next phases.