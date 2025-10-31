# Backend Structure Document

This document outlines the backend architecture, database management, APIs, hosting, infrastructure, and operational practices for the AI-LMS Portal. It is written in clear, everyday language to ensure anyone can understand the setup, from nontechnical team members to developers.

## 1. Backend Architecture

### Overview
- We follow a serverless, client-backend model:
  - The **frontend** is built with Next.js (App Router) and deployed on Vercel.
  - The **backend** consists of Google Apps Script (GAS) web apps that expose RESTful endpoints to read and write data in Google Sheets.
- Communication between the frontend and backend happens via a simple API client in the `/lib` folder of the Next.js project.

### Design Patterns and Frameworks
- **Serverless Functions**: GAS scripts run in Google’s cloud—no server to manage.
- **RESTful APIs**: Each action (fetch materials, submit feedback, update settings) corresponds to one HTTP endpoint.
- **Type Safety**: TypeScript interfaces define the shape of data coming from and going to the GAS endpoints.
- **Client-side Caching**: SWR or React Query caches data in the browser, minimizing network calls and improving perceived performance.

### Scalability, Maintainability, and Performance
- **Scalability**: GAS auto-scales with usage. Vercel automatically distributes traffic across its edge network.
- **Maintainability**: Clear separation of concerns:
  - `/lib/apiClient.ts` handles all fetch calls.
  - `/app` and `/components` focus on UI.
  - Google Sheets act as a familiar data store.
- **Performance**: 
  - Server-side rendering (SSR) and static optimizations in Next.js.
  - Edge CDN from Vercel speeds up asset delivery.
  - Client caching avoids repeated fetches.

## 2. Database Management

### Technologies Used
- **Data Store**: Google Sheets (acts as a simple NoSQL store).
- **Configuration Store**: GAS PropertiesService for application settings and small key-value data.

### Data Structure and Access
- **Sheets as Tables**: Each Google Sheet represents a logical table (e.g., `Materials`, `Feedback`, `Settings`).
- **Rows as Records**: The first row holds column names; subsequent rows hold data.
- **GAS Endpoints**: Scripts use `SpreadsheetApp` to read/write rows and return JSON objects.
- **Multi-tenancy**: Handled in GAS by filtering rows based on `Session.getActiveUser().getEmail()`.

### Data Management Practices
- **Backups**: Regular manual or automated exports of Sheets to CSV.
- **Versioning**: Track Apps Script changes via GitHub integration or clasp (Apps Script CLI).
- **Access Control**: Only the deployed web-app URL can invoke GAS endpoints; roles enforced in NextAuth.js and in script logic.

## 3. Database Schema

The following outlines each sheet and its columns in a human-friendly way. No SQL code is used because Google Sheets is a NoSQL-style store.

1. **Materials Sheet**:
   - `materialId` (unique string)
   - `title` (text)
   - `description` (text)
   - `uploadedBy` (user email)
   - `uploadDate` (ISO date string)
   - `fileUrl` (link to Google Drive)

2. **Feedback Sheet**:
   - `feedbackId` (unique string)
   - `materialId` (reference to Materials)
   - `studentEmail` (user email)
   - `comment` (text)
   - `rating` (number 1–5)
   - `submittedAt` (ISO date string)

3. **Settings Sheet**:
   - `settingId` (unique string)
   - `key` (text)
   - `value` (text)
   - `updatedBy` (user email)
   - `updatedAt` (ISO date string)

4. **Roles & Users (optional)**:
   - `userEmail` (primary key)
   - `role` (IT_ADMIN, GURU, SISWA)
   - `createdAt` (ISO date string)
   - `updatedAt` (ISO date string)

## 4. API Design and Endpoints

We use RESTful HTTP endpoints exposed by Google Apps Script. All responses are JSON.

1. **GET /materials**
   - Purpose: Retrieve a list of teaching materials for the logged-in guru.
   - Query Params: none
   - Response: Array of material objects.

2. **POST /materials**
   - Purpose: Upload a new material.
   - Body: `{ title, description, fileBase64 }`
   - Response: The created material object.

3. **GET /my-feedback**
   - Purpose: Retrieve feedback submitted by the logged-in student.
   - Response: Array of feedback objects.

4. **GET /feedback?materialId={id}**
   - Purpose: Retrieve all feedback for a specific material.
   - Response: Array of feedback objects.

5. **POST /feedback**
   - Purpose: Submit feedback on a material.
   - Body: `{ materialId, comment, rating }`
   - Response: The created feedback object.

6. **GET /settings**
   - Purpose: Administrator fetches application settings.
   - Response: Array of key-value pairs.

7. **POST /settings**
   - Purpose: Administrator updates a setting.
   - Body: `{ key, value }`
   - Response: The updated key-value object.

## 5. Hosting Solutions

### Frontend (Next.js)
- **Provider**: Vercel
- **Model**: Serverless + Edge CDN
- **Benefits**:
  - Zero-config deployments on `git push`
  - Global CDN for fast asset delivery
  - Automatic SSL/TLS
  - Built-in analytics and logs
  - Pay-as-you-go pricing

### Backend (GAS Web Apps)
- **Provider**: Google Cloud via Apps Script
- **Model**: Serverless execution environment
- **Benefits**:
  - Integrated with Google Sheets and Drive
  - Auto-scales with traffic
  - No server maintenance
  - Free tier available

## 6. Infrastructure Components

- **CDN**: Vercel Edge Network caches static assets and SSR pages.
- **Load Balancing**: Handled automatically by Vercel’s platform.
- **Caching**:
  - **Client-side**: SWR/React Query caches API responses in memory.
  - **Edge**: Vercel’s CDN caches unchanged pages and assets.
- **File Storage**: Google Drive stores uploaded files; URLs returned by GAS.
- **Environment Variables**: Managed by Vercel for Next.js and by Apps Script PropertiesService for GAS secrets (e.g., Drive folder IDs).

## 7. Security Measures

- **Authentication**:
  - NextAuth.js with Google OAuth provider handles user sign-in and session management.
  - HTTPS enforced by Vercel and GAS endpoints.
- **Authorization**:
  - Roles (IT_ADMIN, GURU, SISWA) assigned in NextAuth callback and stored in session.
  - Frontend route guards (protected server routes in Next.js) block unauthorized access.
  - GAS scripts re-validate `Session.getActiveUser().getEmail()` and optional custom user store to double-check roles.
- **Data Encryption**:
  - All data in transit is encrypted via TLS.
  - Google Sheets and Drive data is encrypted at rest by Google.
- **Secrets Management**:
  - API URLs and OAuth credentials stored in environment variables on Vercel.
  - GAS script keys and folder IDs stored in PropertiesService (script-level environment storage).
- **CORS**: GAS endpoints allow only requests from your Next.js origin.

## 8. Monitoring and Maintenance

- **Monitoring Tools**:
  - **Vercel Dashboard**: Deploy logs, usage metrics, request analytics.
  - **Google Cloud Logs**: Execution logs for Apps Script triggers and web app calls.
  - **Sentry** (recommended): Capture frontend errors and API failures.
- **Alerts**:
  - Configure email or Slack alerts for error rate spikes via Vercel and Sentry.
- **Maintenance Strategies**:
  - **Automated CI/CD**: Vercel auto-deploys on `main` branch merges.
  - **Dependency Updates**: Use Dependabot or Renovate to keep Next.js and npm packages up to date.
  - **Backups**: Schedule daily exports of Sheets to another Drive folder or GitHub.
  - **Health Checks**: Periodic smoke tests for API endpoints, e.g., via GitHub Actions or external uptime monitors.

## 9. Conclusion and Overall Backend Summary

The AI-LMS Portal backend leverages a serverless, lightweight architecture split between Next.js on Vercel and Google Apps Script with Google Sheets. This setup:

- Meets scalability needs with automatic cloud scaling.
- Ensures maintainability through clear folder structures (`/lib` for API, `/app` for pages, `/components` for UI).
- Delivers fast performance via CDN and client-side caching.
- Secures data with OAuth, role-based guards, and encrypted communications.
- Offers cost-effective hosting with generous free tiers.

Unique aspects include the use of Google Sheets as a familiar, low-maintenance data store and the GAS-based multi-tenant filtering that transparently enforces access rules. Together, these components form a reliable, easy-to-understand backend that enables rapid development and straightforward operation of the AI-LMS Portal.