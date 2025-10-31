# Security Guidelines for ai-lms-portal-starter

This document outlines security best practices tailored to the **ai-lms-portal-starter** codebase. Adhere to these guidelines throughout design, development, testing, and deployment to ensure a robust, secure AI-LMS Portal.

---

## 1. Authentication & Access Control

- **NextAuth.js with Google Provider**
  - Enforce Google OAuth as the single sign-on mechanism.
  - In `[...nextauth].ts`, validate tokens using the recommended algorithms (RS256).
  - Reject any tokens with algorithm `none` or missing signatures.
- **Role-Based Access Control (RBAC)**
  - Map Google email domains or specific addresses to roles (`IT_ADMIN`, `GURU`, `SISWA`) in the `callbacks.signIn` or `callbacks.jwt` hooks.
  - Protect server and API routes via `getServerSession` and middleware. Only allow authorized roles to access `/dashboard`, `/materials`, `/my-feedback`, `/admin/settings`.
  - Fallback: redirect unauthorized requests to a safe error page without revealing internal details.
- **Session Management**
  - Enable secure, HTTP-only cookies with `SameSite=Lax` or `Strict`, and `Secure` flags under HTTPS.
  - Configure short-lived session cookies and rotate JWTs on each sign-in.
  - Provide a logout endpoint that invalidates sessions both client- and server-side.
- **Multi-Factor Authentication (MFA)**
  - Consider optional TOTP or SMS-based MFA for `IT_ADMIN` role to protect administrative functions.
  - Leverage NextAuth.js built-in support or integrate a third-party MFA library.

## 2. Input Handling & Processing

- **Server-Side Validation**
  - Never trust client input. Validate all payloads on API routes in `/app/api/*` using a schema validator (e.g., Zod or Yup).
  - Define TypeScript schemas for `MaterialCreateRequest`, `FeedbackUpdate`, `AdminSettings`.
- **Prevent Injection & XSS**
  - Sanitize any HTML or rich text fields (e.g., feedback comments) before rendering. Use a whitelist sanitizer like DOMPurify.
  - For all database-like operations with Google Sheets/GAS, parameterize values to avoid script injection.
- **File Uploads** (`/materials`)
  - Verify file extensions (`.pdf`, `.docx`, `.png`, etc.) and MIME types on the client and server.
  - Enforce a maximum file size (e.g., 5 MB).
  - Convert files to Base64 client-side, then validate length and content on the GAS backend before writing to Drive/Sheets.
  - Store uploads outside the public webroot or in a dedicated Google Drive folder with restricted permissions.
- **Redirect Validation**
  - If performing any dynamic redirects (e.g., after login), ensure target URLs are on an allow-list (our domain or subpaths).

## 3. Data Protection & Privacy

- **Encryption in Transit & at Rest**
  - Enforce HTTPS (TLS 1.2+) on Vercel. Redirect all HTTP traffic to HTTPS via `next.config.js` or `vercel.json`.
  - For sensitive data stored in Google Sheets or Drive, rely on Google’s encryption at rest.
- **Secrets Management**
  - Store OAuth credentials (`GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`), GAS Web App URL, and any API keys in environment variables (`.env.local`).
  - Do **not** commit `.env` files. Use Vercel’s secret management in production.
- **Password Storage**
  - If adding any custom password fields, use Argon2 or bcrypt with unique salts. Prefer OAuth over custom passwords.
- **PII & Data Minimization**
  - Only request and persist the minimum set of user data (email, name, role).
  - Mask or truncate any PII in logs or error messages.
  - Implement a data retention policy: regularly purge obsolete feedback entries if required by regulation.

## 4. API & Service Security

- **Rate Limiting & Throttling**
  - Protect critical API routes (`/api/materials`, `/api/feedback`) against brute-force or DoS by applying a rate limiter (e.g., `express-rate-limit` or a Vercel edge function).
- **CORS Configuration**
  - Restrict CORS to approved origins (your Vercel domain). Disallow wildcard (`*`) origins.
- **HTTP Methods & Status Codes**
  - Enforce proper verbs (`GET` for reads, `POST` for creates, `PUT/PATCH` for updates, `DELETE` for removals).
  - Return appropriate status codes and avoid leaking stack traces or internal errors.
- **API Versioning**
  - If evolving APIs, prefix routes (e.g., `/api/v1/materials`) to avoid breaking changes.

## 5. Web Application Security Hygiene

- **CSRF Protection**
  - Use built-in NextAuth.js CSRF tokens on all state-changing POST routes.
- **Security Headers** (via `next.config.js` or a custom `middleware.ts`)
  - `Content-Security-Policy`: restrict script sources to self and trusted domains (e.g., Google APIs).
  - `Strict-Transport-Security`: `max-age=63072000; includeSubDomains; preload`.
  - `X-Frame-Options`: `DENY` or `SAMEORIGIN`.
  - `X-Content-Type-Options`: `nosniff`.
  - `Referrer-Policy`: `strict-origin-when-cross-origin`.
- **Cookie Security**
  - All session and CSRF cookies should be flagged `HttpOnly`, `Secure`, and `SameSite=Strict` (or Lax for login flows).
- **Subresource Integrity (SRI)**
  - If loading any third-party scripts/styles, use SRI hashes to ensure content hasn’t been tampered with.

## 6. Infrastructure & Configuration Management

- **Vercel Production Hardening**
  - Disable Next.js dev mode. Ensure `NODE_ENV=production`.
  - Turn off verbose error reporting. Use a custom error page that logs details server-side but shows a generic message client-side.
- **Environment Separation**
  - Maintain separate Vercel projects/environments for dev, staging, production.
  - Use environment-specific secrets for OAuth and GAS URLs.
- **Dependency & Port Hardening**
  - Only expose the Next.js default port internally; rely on Vercel’s edge network for external traffic.

## 7. Dependency Management

- **Secure Dependencies**
  - Audit `package.json` dependencies regularly with `npm audit` or GitHub Dependabot.
  - Upgrade stale or vulnerable packages (Next.js, React, shadcn/ui, Tailwind) promptly.
- **Lockfiles**
  - Commit `package-lock.json` to ensure deterministic builds.
- **Minimal Footprint**
  - Remove unused libraries (e.g., Drizzle ORM, PostgreSQL client) once replaced by the GAS API client.

---

By following these guidelines, you ensure the **ai-lms-portal-starter** remains secure by design, minimizing attack surfaces while protecting user data and privacy. Regularly revisit and update these controls as the codebase and threat landscape evolve.