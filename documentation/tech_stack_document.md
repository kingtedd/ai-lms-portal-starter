# AI-LMS Portal Tech Stack Document

This document explains the technology choices behind the AI-LMS Portal starter kit in a straightforward way. You don’t need a technical background to understand why each piece was chosen and how they work together.

## 1. Frontend Technologies

We’ve built the user interface using modern, widely adopted tools that make the app look great, stay responsive, and scale with new features.

- **Next.js (App Router)**
  - Provides the overall structure for pages, server-side rendering, and protected routes.  
  - Lets us split code automatically, improving load times.
- **TypeScript**
  - Adds type checks to catch errors early, making the code more reliable and easier to maintain.  
- **React**
  - Powers our components and hooks, enabling a dynamic single-page experience.
- **shadcn/ui**
  - A library of pre-built, accessible UI components (tables, forms, buttons) that follow consistent design patterns.  
- **Tailwind CSS**
  - A utility-first styling framework that makes it fast to build responsive layouts and themes.  
  - Supports dark/light mode through simple CSS variables.
- **SWR** *(or React Query)*
  - Handles data fetching, caching, and background updates from our backend APIs without writing complex code.  
- **CSS Variables & Print Styles**
  - Custom properties to control colors and theming across the app.  
  - Dedicated `@media print` rules format reports for printing.

These choices combine to give students, teachers, and administrators a smooth, consistent experience on any device.

## 2. Backend Technologies

Instead of a traditional database server, the AI-LMS Portal relies on Google’s cloud-based scripting and storage, keeping our architecture lightweight and easy to manage.

- **Google Apps Script (GAS) Web Apps**
  - Hosts server-side code that talks to Google Sheets and other Google services.  
  - Manages multi-tenant filtering by checking `Session.getActiveUser().getEmail()`.
- **Google Sheets**
  - Acts as our data store for materials, feedback entries, and settings.  
  - No separate database to configure—just familiar spreadsheets.
- **API Client in `/lib`**
  - A set of `fetch` functions (`GET /materials`, `POST /feedback`, `PATCH /settings`, etc.) that talk to the deployed GAS endpoints.  
  - Uses TypeScript interfaces to keep request and response shapes consistent.
- **NextAuth.js with Google Provider**
  - Handles user sign-in and sign-up flows via Google OAuth.  
  - Assigns roles (IT_ADMIN, GURU, SISWA) based on email and protects pages on the server.
- **Protected Server Routes**
  - Next.js server components check sessions before rendering pages to ensure only authorized users see sensitive data.

This setup keeps your infrastructure simple, leverages Google’s security, and ensures your data is always in sync.

## 3. Infrastructure and Deployment

Our choices here make it effortless to develop, test, and deploy the portal with confidence.

- **Vercel**
  - Hosts the Next.js application with automatic global CDN, HTTPS, and serverless function support.  
  - Built-in CI/CD: every push to the main branch triggers a new deployment.
- **Git & GitHub**
  - Version control your source code, review changes in pull requests, and track issues.  
- **Docker (optional)**
  - Provides a repeatable local development environment if you want to containerize dependencies.  
- **Environment Variables (`.env.local`)**
  - Store secrets like your Google Apps Script URL and OAuth credentials outside the codebase.  

Together, these tools ensure your team can collaborate smoothly and ship updates quickly.

## 4. Third-Party Integrations

We’ve integrated a few key services to handle authentication, data storage, and user management without building everything from scratch.

- **Google OAuth (via NextAuth.js)**
  - Lets users sign in with their existing Google accounts securely.  
- **Google Apps Script**
  - Runs custom backend logic on Google’s servers, connecting directly to Sheets.  
- **Google Sheets**
  - Serves as the data layer, offering a familiar interface for simple configuration and exports.  

These integrations minimize overhead and tap into Google’s secure, scalable ecosystem.

## 5. Security and Performance Considerations

We’ve taken steps to protect user data and keep the app running smoothly.

- **Authentication & Session Management**
  - NextAuth.js stores encrypted sessions and enforces role-based access on server routes.  
- **Data Protection**
  - API calls are made over HTTPS.  
  - Sensitive credentials live in environment variables only.  
- **Error Handling & User Feedback**
  - Centralized error handling for network or API failures, displaying friendly messages instead of raw errors.  
- **Caching & Revalidation (SWR/React Query)**
  - Minimizes redundant network requests and keeps data fresh in the background.  
- **Code Splitting & Lazy Loading**
  - Next.js automatically splits code by route and component to speed up initial page loads.

These practices help safeguard data and deliver fast interactions, even on slower connections.

## 6. Conclusion and Overall Tech Stack Summary

The AI-LMS Portal starter kit brings together best-in-class, easy-to-use technologies that match the project goals of rapid development, clear role-based access, and a polished user interface:

- **Next.js + TypeScript**: A rock-solid foundation for pages, APIs, and type safety.
- **shadcn/ui + Tailwind CSS**: Consistent, responsive UI components and styling.
- **NextAuth.js (Google Provider)**: Secure, out-of-the-box Google sign-in and role management.
- **Google Apps Script + Sheets**: Simple, serverless backend and data store without managing servers.
- **Vercel & GitHub**: Zero-config deployments and version control for fast, reliable delivery.

This stack empowers your team to focus on building the AI-LMS features—materials management, feedback flows, and admin settings—while relying on proven tools for security, performance, and developer productivity.