# Haikuverse Public Gallery 🦑

---

## 1. Introduction

The Haikuverse Public Gallery is a high-performance Next.js web application designed to empower two complementary audiences: the curious public and the creative community. Acting as both an SEO-optimized "front door" to the ecosystem and a secure, automated **Print-on-Demand E-Commerce Engine**, the application bridges the gap between ephemeral digital content and permanent physical artifacts. Built on **Firebase Hosting** and backed by **Google Cloud Run**, the architecture strategically employs a hybrid rendering model—combining **Static Site Generation (SSG)** for lightning-fast public access with **Client-Side Rendering (CSR)** for rich, interactive dashboards. This mission-critical platform was engineered to meet users exactly where they are, removing the friction of "walled gardens" to foster inclusivity while maintaining the rigorous security standards required for global manufacturing and financial compliance.

### Key Features:

- **Blazing Fast & Always Fresh:** Leverages Incremental Static Regeneration (ISR) for core content (`/poet/[nickname]`, `/constellation/[name]`). This combines the instant load times of static generation with the ability to automatically refresh data from Firebase in the background, ensuring content like new stars and follower counts stays up-to-date without needing a redeploy.
- **Interactive Homepage Hub:** Features a dynamic **Zeitgeist Map** highlighting trending community themes and a **Poet Search** bar for easy discovery.
- **Smart User Acquisition Funnel:** A context-aware call-to-action system intelligently routes users based on authentication status. Subscribers that click on the Print Studio Call to Action are sent directly to their Print Studio, while guests are directed to a dedicated Print Studio Promo page (`/print-studio-promo`). The promo page leverages high-fidelity physical product imagery ("Forge Artifacts") to drive conversion by focusing on the tangible utility of the platform.
- **Fully Responsive & Mobile-First:** The entire gallery is built with a mobile-first philosophy to align with the Haikuverse ecosystem. All pages, from the interactive homepage to the feature-rich Print Studio, are fully responsive, using a modern CSS architecture that defaults to a clean, single-column layout on phones and scales up gracefully for tablets and desktops.
- **Dynamic Poet Portfolios:** Every poet gets a unique, shareable, statically-generated page that automatically showcases their bio, high-resolution avatar, owned constellations, and contributions to other constellations.
- **Immersive Constellation Galleries:** Each constellation has its own dedicated, statically-generated page featuring its fable, lore image, and a gallery of all the stars within it.
- **Detailed Star Experience:** A server-rendered star page showcases each creation with a high-resolution, optimized image slideshow timed to the cadence of the audio preview and a display for community like counts.
- **Personalized Member Dashboard:** Logged-in users access a dashboard that calls serverless API routes to display their Haikuverse **nickname** and real-time lists of poets they **follow** and who **follow them**.
- **Value-Driven Gating:** The application architecture employs a psychological "soft gate" for monetization. Instead of a generic paywall, a comprehensive **Benefit Comparison Table** is revealed within the Print Studio flow, clearly differentiating "Guest" vs. "Creator License" tiers. This frames the subscription as a value-add across a broad spectrum of services, including manufacturing premium physical goods rather than just software access.
- **High-Resolution Print Studio:** A dedicated Print Studio empowers creators to browse their generated images and purchase physical Haikucards.
  - **Server-Side Composition:** Uses the `sharp` library to generate two distinct 300 DPI assets per order: a full-bleed **Blueprint** for the printer and a rounded **Preview** for the user.
  - **Immutable Orders:** Implements a "Clump Factory" pattern (`/api/orders/create`) that snapshot-freezes pricing, shipping data, and print specifications into a `pending_payment` document _before_ the transaction begins, ensuring total data integrity.
- **Stripe E-Commerce & Tax Compliance:** Features a full **Stripe** integration with **Payment Elements** (supporting Apple Pay/Google Pay) and robust webhook signature verification.
  - **Automated Tax:** Utilizes `stripe.tax.calculations.create` on the server to calculate sales tax in real-time based on the user's shipping address.
  - **Tax Ledger:** A webhook listener (`/api/stripe/webhook`) atomically records every transaction into a dedicated, anonymized **Firestore Tax Ledger**, separating tax liability from net revenue for GAAP compliance.
- **Automated Fulfillment Pipeline:** Paid orders are automatically handed off to the **Gelato API v4** via a secure Cloud Function (`onOrderPaid`). The system handles double-sided printing logic (injecting a static "Back Template"), processes "Shipped" and "Delivered" webhooks, and triggers branded emails via the **Firebase Email Extension**.
- **Privacy by Design (The "Janitor"):** Aligned with the Prohumanist philosophy, the system minimizes PII retention.
  - **Zero-PII Friction:** Shipping forms cache data in `localStorage` rather than the database.
  - **The PII Janitor:** A scheduled backend function (`purgeDeliveredOrders`) automatically hard-deletes order documents, email logs, and print assets 180 days after delivery, leaving behind only the anonymized tax ledger.
- **Secure User Authentication:** Supports secure sign-in via **Google Sign-In (GSI)**, managed globally with React Context. The production environment is secured by authorizing the live domain in both the **GCP OAuth Client ID** and the **reCAPTCHA Enterprise key** settings.
- **Robust & Secure Backend:**
  - Employs a server-only **Firebase Admin SDK** initialized using a **singleton pattern** (`src/lib/firebaseAdmin.ts`) to prevent redundant initializations and improve server performance.
  - All server-side credentials (`PROJECT_ID`, `CLIENT_EMAIL`, `PRIVATE_KEY`) are securely injected into the **Google Cloud Run** environment at deploy time using **Google Secret Manager**, configured via `firebase.json`.
  - All API routes (`/api/*`) are refactored to import and use the single, centralized `auth` and `db` instances, ensuring consistent, authenticated access to Firebase services.
- **Optimized for Production:**
  - The deployed Cloud Run service is configured with **1GiB of memory** in `firebase.json` to handle Next.js server-side rendering loads without crashing.
  - **Environment Parity:** The build process is stabilized by enforcing **Node.js v20** parity between local and production environments. This configuration is required to prevent npm ci failures and includes an explicit dependency on `@opentelemetry/api` (`npm install @opentelemetry/api`) to resolve `MODULE_NOT_FOUND` crashes when converting to Node.js v20.
  - Uses the Next.js `<Image>` component with correctly configured `remotePatterns` (for `storage.googleapis.com` and `firebasestorage.googleapis.com`) and `sizes` props (e.g., `sizes="16rem"` for avatars) to ensure high-resolution, optimized images are served.

---

## 2. Functional Block Diagram

```mermaid
---
config:
  layout: elk
  elk:
    nodePlacementStrategy: BRANDES_KOEPF
    spacing:
      nodeNode: 60
      component: 40
  theme: base
  themeVariables:
    lineColor: '#666'
    textColor: '#222'
    fontSize: '14px'
---
flowchart LR
  subgraph Browser["User's Browser"]
    direction TB
    User["User Interaction"]
    subgraph ClientComponents ["Client Components (React)"]
        HomePageClient["Home Page<br>(ZeitgeistClient, PoetSearch,<br>Smart CTA Logic)"]
        DashboardPageClient["Dashboard Page<br>(HaikucardEditor)"]
        HeaderClient["Header (Auth Aware)"]
        SSGPages["SSG Pages (Hydrated)<br>(Poet, Constellation)"]
        PrintStudioPromo["Print Studio Promo<br>(Static Landing Page)"]
    end
    AuthContext["AuthContext"]
    FirebaseClientSDK[("Firebase Client SDK")]
    GSI["Google Sign-In Lib"]
  end

  subgraph ProductionEnvironment["Production Environment (GCP)"]
    direction TB
    subgraph Hosting["Firebase Hosting"]
        StaticAssets["Static Assets (HTML/JS)"]
    end

    subgraph CloudRun["Google Cloud Run (Next.js Server)"]
        direction TB
        FirebaseAdminModule["**src/lib/firebaseAdmin.ts**<br>(Singleton Admin SDK)"]

        subgraph ServerRuntime ["Runtime API Routes"]
            %% Data Fetching
            ApiSearch["/api/search-poets"]
            ApiNickname["/api/profile/nickname"]

            %% Network Proxy (Cloud Functions)
            ApiFollowing["/api/network/following"]
            ApiFollowers["/api/network/followers"]

            %% E-Commerce & Print
            ApiGeneratePng["/api/print/generate-png"]
            ApiCreateOrder["/api/orders/create"]
            ApiCreatePayment["/api/stripe/create-payment-intent"]

            %% Webhooks
            ApiStripeWebhook["/api/stripe/webhook"]
            ApiGelatoWebhook["/api/gelato/webhook"]

            SSRPageGen["SSR Page Generation<br>(/star/[id])"]
        end
    end
  end

  subgraph BackendServices["Backend Services"]
    direction TB
    FirestoreDB[("Cloud Firestore")]
    StorageBucket[("Firebase Storage")]

    subgraph CloudFunctionsV1["v1 Cloud Functions"]
        ProxyFunctions["getFollowing / getFollowers"]
        OnOrderPaid["onOrderPaid"]
    end

    EmailExt["Email Extension"]
    FirebaseAppCheck["Firebase App Check"]
    FirebaseAuthBackend["Firebase Auth"]
  end

  subgraph ExternalServices["External APIs"]
    StripeAPI["Stripe API"]
    GelatoAPI["Gelato API"]
    GoogleWorkspace["Google Workspace (SMTP)"]
    GCP_OAuth["GCP OAuth"]
    SecretManager[("Google Secret Manager")]
  end

  %% ================= CONNECTIONS =================

  %% 1. INITIALIZATION
  SecretManager -- "Injects Secrets" --> CloudRun

  %% 2. USER & BROWSER INTERACTION (Auth Loop)
  User -- "Clicks Sign In" --> HeaderClient
  HeaderClient -- "Triggers" --> GSI
  GSI -- "Validates Origin" --> GCP_OAuth
  GSI -- "Returns Credential" --> FirebaseClientSDK
  FirebaseClientSDK -- "Verifies Client" --> FirebaseAppCheck
  FirebaseClientSDK -- "Signs In" --> FirebaseAuthBackend
  FirebaseClientSDK -- "Updates State" --> AuthContext
  AuthContext -- "Provides User" --> HeaderClient
  AuthContext -- "Protects Route" --> DashboardPageClient
  AuthContext -- "Smart Route" --> HomePageClient

  %% 3. PAGE RENDERING FLOWS
  User -- "Visits /poet/..." --> StaticAssets
  StaticAssets -- "Hydrates" --> SSGPages
  SSGPages -- "Displays Content" --> User
  User -- "Visits /print-studio-promo" --> StaticAssets
  StaticAssets -- "Hydrates" --> PrintStudioPromo
  PrintStudioPromo -- "Displays Artifacts" --> User

  User -- "Visits /star/..." --> Hosting
  Hosting -- "Rewrites to" --> SSRPageGen
  SSRPageGen -- "Reads Data" --> FirebaseAdminModule
  SSRPageGen -- "Returns HTML" --> User

  %% 4. CLIENT COMPONENT ACTIONS
  User -- "Searches" --> HomePageClient
  HomePageClient -- "Fetch" --> ApiSearch
  ApiSearch -- "Query" --> FirebaseAdminModule

  User -- "Views Dashboard" --> DashboardPageClient
  DashboardPageClient -- "Fetch Profile" --> ApiNickname
  ApiNickname -- "Read Profile" --> FirebaseAdminModule

  DashboardPageClient -- "Fetch Network" --> ApiFollowing & ApiFollowers
  ApiFollowing & ApiFollowers -- "Proxy Auth" --> FirebaseAdminModule
  ApiFollowing & ApiFollowers -- "Proxy Request" --> ProxyFunctions
  ProxyFunctions -- "Read DB" --> FirestoreDB

  %% SMART CTA LOGIC
  HomePageClient -- "Subscriber CTA" --> DashboardPageClient
  HomePageClient -- "Guest CTA" --> PrintStudioPromo

  %% 5. PRINT STUDIO & ORDER CREATION
  User -- "Customizes Card" --> DashboardPageClient
  DashboardPageClient -- "Generate Preview" --> ApiGeneratePng
  ApiGeneratePng -- "Upload Asset" --> FirebaseAdminModule

  DashboardPageClient -- "Create Order" --> ApiCreateOrder
  ApiCreateOrder -- "Generate & Upload" --> FirebaseAdminModule
  ApiCreateOrder -- "Write Order" --> FirebaseAdminModule

  %% 6. PAYMENT FLOW
  DashboardPageClient -- "Pay" --> ApiCreatePayment
  ApiCreatePayment -- "Calculate Tax" --> StripeAPI
  ApiCreatePayment -- "Update Order" --> FirebaseAdminModule

  %% 7. WEBHOOKS (INBOUND)
  StripeAPI -- "Payment Succeeded" --> ApiStripeWebhook
  ApiStripeWebhook -- "Verify & Update" --> FirebaseAdminModule

  GelatoAPI -- "Status Update" --> ApiGelatoWebhook
  ApiGelatoWebhook -- "Verify & Update" --> FirebaseAdminModule

  %% 8. FULFILLMENT LOOP (ASYNC)
  FirebaseAdminModule -- "Writes/Updates" --> FirestoreDB
  FirebaseAdminModule -- "Uploads/Reads" --> StorageBucket

  FirestoreDB -- "Trigger (Paid)" --> OnOrderPaid
  OnOrderPaid -- "Create Order (v4)" --> GelatoAPI

  FirestoreDB -- "Trigger (Mail Doc)" --> EmailExt
  EmailExt -- "Send SMTP" --> GoogleWorkspace
  GoogleWorkspace -- "Email Notification" --> User

  %% Styles
  style Browser fill:#E8F0FE,stroke:#4285F4,stroke-width:2px
  style ProductionEnvironment fill:#f8f8ff,stroke:#4b0082,stroke-width:2px
  style BackendServices fill:#FFF7E6,stroke:#FFA900,stroke-width:2px
  style ExternalServices fill:#E0F2F1,stroke:#009688,stroke-width:2px

  style ApiGeneratePng fill:#ffecb3,stroke:#ffa000
  style ApiCreateOrder fill:#ffecb3,stroke:#ffa000
  style ApiCreatePayment fill:#ffecb3,stroke:#ffa000
  style ApiStripeWebhook fill:#dcedc8,stroke:#689f38
  style ApiGelatoWebhook fill:#dcedc8,stroke:#689f38
```

---

## 3. Core Architecture & Services

| Layer               | Technology / Service                      | Purpose                                            |
| :------------------ | :---------------------------------------- | :------------------------------------------------- |
| **Web Framework**   | Next.js (App Router), React, TypeScript   | UI, Routing, SSG/SSR, API Routes                   |
| **Styling**         | CSS Modules                               | Component-scoped styling                           |
| **Image Engine**    | `sharp` & `opentype.js`                   | Server-side generation of 300 DPI print assets     |
| **E-Commerce**      | Stripe (Payment Elements & Tax)           | Payments, Apple/Google Pay, & Auto-Tax calculation |
| **Fulfillment**     | Gelato API v4                             | Automated print-on-demand manufacturing            |
| **Notifications**   | Firebase Extension (Trigger Email)        | Transactional emails via Google Workspace SMTP     |
| **Client Auth**     | Firebase Auth SDK, Google Sign-In (GSI)   | User login & session management                    |
| **Client Security** | Firebase App Check (reCAPTCHA Enterprise) | Verifying client integrity                         |
| **Server Backend**  | Google Cloud Run (via Firebase)           | Hosts all server-side Next.js logic                |
| **Database**        | Cloud Firestore                           | Core data, Order history, & Tax Ledger             |
| **Object Storage**  | Google Cloud Storage                      | Hosting full-bleed blueprints & email previews     |
| **Server Secrets**  | Google Secret Manager                     | Secure injection of Stripe/Gelato keys             |
| **Hosting**         | Firebase Hosting                          | Serves static assets, routes backend requests      |

The following sections provide a detailed breakdown of the application's core components.

### 3.1 Next.js App Router (`src/app/`)

The application is built using the **Next.js App Router**, which uses a file-based routing system. Folders within the `src/app/` directory define the URL structure of the site.

- **Static Routes:** The root homepage (`src/app/page.tsx`) and the promotional landing page (`src/app/print-studio-promo/page.tsx`) define the primary static entry points.
- **Dynamic Routes:** Pages for individual poets and constellations are created using folders with square brackets, such as `src/app/poet/[nickname]/page.tsx`.
- **Layouts:** The root layout at `src/app/layout.tsx` defines the shell `<html>` and `<body>` tags that wrap every page, ensuring a consistent base structure and styling.
- **Client Routes:** The user dashboard (`/dashboard`) is implemented as a client component, dynamically rendering based on authentication state.
- **API Routes:** Backend logic accessible to the client is defined in `/api/` folders (e.g.,`/api/profile/nickname/route.ts`). These handle secure data fetching for interactive components.

### 3.2 Hybrid Rendering Strategy

The gallery employs a hybrid rendering approach, optimized for SEO, performance, and interactivity.

- **Static Site Generation (SSG):** Core discovery pages like poet portfolios (`/poet/[nickname]`) and constellation galleries (`/constellation/[name]`) use `generateStaticParams`. This pre-renders static HTML at build time using the **Firebase Admin SDK** for secure data fetching, ensuring fast initial loads and excellent SEO.
- **Incremental Static Regeneration (ISR):** To prevent content from becoming stale, all statically-generated and server-rendered pages employ ISR with a revalidate period of one hour (3600s). This strategy ensures that while users always receive an instant, cached response, the page is automatically rebuilt in the background if new data (e.g., new stars, updated follower counts) is available. This provides the perfect balance between static performance and data freshness, minimizing server load and Firebase reads.
- **Server-Side Rendering (SSR):** Individual star detail pages (`/star/[id]`) are rendered on-demand (dynamic) to ensure the latest like counts and details are always shown.
- **Client-Side Rendering (CSR):** Pages requiring user interaction and browser-side state (like the Homepage and Dashboard) are designated as Client Components using the **`"use client";`** directive.
- **API Routes:** Serverless functions co-located with the app (`/api/*`) act as a secure backend gateway for Client Components.

### 3.3 Client Components & State Management (`"use client";`)

Client Components handle user interaction, browser-specific APIs, and state management within the user's browser.

- **Interactivity & Data Fetching:** Components like the `PoetSearch`, the `PrintStudioCTA` (Smart Routing Logic), `DashboardPage`, and the site `Header` are Client Components. They use React Hooks (`useState`, `useEffect`) to manage local UI state and fetch dynamic data.
- **Authentication State:** A global **`AuthContext`** (`src/context/AuthContext.tsx`) provides application-wide access to the current user's authentication state (`user`, `loading`) via the `useAuth` hook.
- **Client-Side Firebase:** The **Firebase Client SDK** (`src/lib/firebaseClient.ts`) is initialized for client-side operations:
  - **Firebase Auth:** Manages the sign-in state.
  - **Firebase App Check:** Secures client-side requests using reCAPTCHA Enterprise.
- **Google Sign-In (GSI):** GSI is integrated via `gsiHelper.ts`. For production, this required authorizing the deployed domain (`haikuverse-gallery.web.app`) in both the **GCP OAuth Client ID's** "Authorized JavaScript origins" and the **reCAPTCHA Enterprise key's** "Domains" list.
- **API Interaction:** Client Components fetch data securely by calling the Next.js **API Routes** using `fetch` within `useEffect` hooks, passing the user's Firebase ID token for authorization.

### 3.4 Styling with CSS Modules

The gallery's styling is implemented using **CSS Modules**, a system that scopes CSS locally to each component. This approach was chosen to ensure style encapsulation and prevent class name conflicts.

- **Scoped & Responsive by Default:** Each page (`PoetPage.module.css`, `PrintStudioPromo.module.css`) and component (`PrintStudioCTA.module.css`) has its own unique style sheet built with a mobile-first philosophy. The build process automatically generates unique class names, guaranteeing that styles from one component cannot accidentally affect another, while media queries are used to progressively enhance the layout for larger screens.
- **Maintainability:** This architecture creates a clean separation between a component's structure (in the `.tsx` file) and its appearance (in the `.module.css` file), making the codebase easier to read and maintain.

### 3.5 Backend Integration & Security

This layer manages the connection to backend services and enforces security, running entirely within the **Google Cloud Run** environment on the server.

- **Centralized Admin SDK (`src/lib/firebaseAdmin.ts`):** This is the core of the backend. It initializes the **Firebase Admin SDK** using a **singleton pattern** to ensure only one instance of the app exists on the server. This prevents redundant initializations, improves performance, and exports the shared `db` (Firestore) and `auth` (Auth) instances for use across the entire server.
- **Secure Credentials:** In production, server-side credentials (`PROJECT_ID`, `CLIENT_EMAIL`, `PRIVATE_KEY`) are **injected securely from Google Secret Manager** into the Cloud Run environment at deploy time. This is configured in `firebase.json` using the `secretEnvironmentVariables` block.
- **API Routes (`src/app/api/`):** All API routes have been refactored to **import the centralized `db` and `auth` instances** from `firebaseAdmin.ts`. This ensures they all use the single, correctly authenticated Admin SDK instance.
- **Data Fetching Service (`src/lib/firebaseService.ts`):** Centralizes reusable data-fetching functions (e.g., `getAllPoets`, `getStarById`) used by Server Components during SSG.
- **Server-Side Image Generation:** A key API route, `/api/print/generate-png`, leverages the **`sharp`** library for high-performance image processing and **`opentype.js`** for precise text measurement. It composites artwork, dynamically-sized haiku text with embedded fonts, and user-selected backgrounds onto a transparent canvas, producing a print-ready 300 DPI PNG with true transparency.

---

## 4. Security, Testing, and Setup

This section details the gallery's security model, outlines the testing strategy, and provides a clear guide for setting up the development and production environments.

### 4.1 Multi-Layered Security Model

The gallery employs a multi-layered security approach, combining server-side safeguards with client-side verification.

- **Server-Side Security (Firebase Admin SDK):**
  - The entire Next.js server (SSG, SSR, API Routes) runs in a **Google Cloud Run** container managed by Firebase Hosting.
  - A **centralized Firebase Admin SDK** (`src/lib/firebaseAdmin.ts`) is initialized using a **singleton pattern** to ensure a single, shared connection to Firebase services.
  - This Admin SDK is used by all server-side code (SSG, API Routes) for data fetching and token verification.
  - **In Production:** Credentials (`PROJECT_ID`, `CLIENT_EMAIL`, `PRIVATE_KEY`) are securely injected into the Cloud Run environment from **Google Secret Manager** at deploy time, as configured in `firebase.json`.
  - **In Development:** Credentials are read from local `.env.local` variables.

- **Webhook Integrity:**
  - Incoming webhooks from **Stripe** and **Gelato** are cryptographically verified using a shared secret (`STRIPE_WEBHOOK_SECRET`, `GELATO_WEBHOOK_SECRET`) to ensure requests originate from the legitimate payment and fulfillment providers.

- **Client-Side Security (Client SDK & App Check):**
  - User authentication is handled client-side via the **Firebase Authentication Client SDK** and **Google Sign-In (GSI)**.
  - **Firebase App Check** (with reCAPTCHA Enterprise) is integrated to verify that requests originate from a legitimate instance of the gallery website.
  - **Production GSI/App Check:** This requires authorizing the production URL (`https://haikuverse-gallery.web.app`) in two places:
    1.  The **GCP OAuth Client ID's** "Authorized JavaScript origins".
    2.  The **reCAPTCHA Enterprise Key's** "Domains" list.
  - **Local GSI/App Check:** This requires setting the `self.FIREBASE_APPCHECK_DEBUG_TOKEN` in the browser console.

- **Secure Cloud Function Interaction:**
  - Next.js API Routes act as a secure proxy to the original Haikuverse v1 Cloud Functions (`getFollowingDetails`, `getFollowersDetails`).
  - The API Route verifies the client's ID token using the Admin SDK, then makes a server-to-server `fetch` call to the Cloud Function URL, forwarding the ID token for the v1 function to perform its own verification.

- **API Authorization Strategy (IDOR Prevention):**
  - To neutralize **Insecure Direct Object Reference (IDOR)** vulnerabilities, the backend employs a strict "Anchor Pattern" for data access.
  - Although the Admin SDK operates with elevated privileges ("God Mode"), all sensitive endpoints (such as `/api/orders/create`) forcefully restrict queries to the authenticated user's specific subcollections (e.g., `db.collection('users').doc(uid)...`).
  - This ensures that possession of a valid resource ID is insufficient for access; the requestor must mathematically prove ownership within the database hierarchy before any action is taken.

### 4.2 Testing Strategy

A comprehensive testing strategy is essential for ensuring the gallery is robust, performant, and maintainable. The plan includes:

- **Unit Testing:** The service layer functions in `src/lib/firebaseService.ts` will be unit-tested using a framework like **Jest**. This involves mocking the Firebase Admin SDK to verify that our data-fetching functions correctly handle various database responses without making actual database calls.
- **Component Testing:** Interactive Client Components, such as the `StarSlideshow` and `StarDetailClient`, will be tested with **React Testing Library**. These tests will simulate user interactions (like clicking a play button) and assert that the component's state and appearance update as expected.
- **End-to-End (E2E) Testing:** A tool like **Cypress** or **Playwright** will be used to run automated tests on the complete, running application. These tests will simulate a full user journey—navigating from the homepage to a poet page, then to a constellation, and finally to a star—verifying that all pages load correctly and links work as intended.
- **Context Testing:** The `AuthContext` provider will be tested to ensure it correctly reflects user login/logout states and provides the user object as expected.
- **API Route Testing:** Next.js API Routes will be tested, potentially using tools like `next-test-api-route-handler` or similar, mocking Firebase Admin functions (`verifyIdToken`, Firestore reads) and fetch calls to Cloud Functions.
- **Routing Logic Testing:** Verify the `PrintStudioCTA` component correctly routes unauthenticated users to `/print-studio-promo` and authenticated subscribers to `/dashboard`, ensuring the acquisition funnel behaves as architected.

### 4.3 Setup and Configuration

To set up and run this project locally, follow these steps:

**Prerequisites:**

- **Node.js (v20):** It is critical to use Node.js v20.x.x (e.g., via `nvm use 20`) because the production Firebase server runs v20.
- **Project Structure:** All static assets required by the server (like fonts) **must be placed in the `public` directory**.
- **Visual Studio Code** or another code editor
- Access to the Haikuverse Firebase project
- Google Cloud Project with Billing enabled
- Enabled Firebase Services: Authentication (Google Sign-In), Firestore, App Check (Web provider)
- Enabled Google Cloud APIs: reCAPTCHA Enterprise, Secret Manager, Cloud Run

**Steps:**

1.  **Clone the Repository:**

```bash
    git clone [https://github.com/chesterwizzle/haikuverse_gallery.git](https://github.com/chesterwizzle/haikuverse_gallery.git)
    cd haikuverse_gallery
```

2.  **Install Dependencies:**

```bash
    npm install
```

3.  **Configure Local Environment Variables (`.env.local`):**
    - In the project root, create a file named `.env.local` (this file is git-ignored).
    - **Server-Side Variables (Firebase Admin SDK):**
      - Generate a new **service account JSON key** from Google Cloud (IAM & Admin > Service Accounts > Your Admin SDK Account > Keys > Add Key).
      - Copy the values into the `.env.local` file. **IMPORTANT:** The `PRIVATE_KEY` value must be a single-line string with `\n` characters replacing the newlines.

```bash
    # Server-Side (Admin SDK)
    PROJECT_ID="haiku-bot-advanced"
    CLIENT_EMAIL="firebase-adminsdk-fbsvc@haiku-bot-advanced.iam.gserviceaccount.com"
    PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nMIIEvg...Your...Key...Here...\n...More...Key...\n-----END PRIVATE KEY-----\n"
```

- **Server-Side Variables (Cloud Function URLs):**

```bash
    GET_FOLLOWING_DETAILS_URL="[https://us-central1-haiku-bot-advanced.cloudfunctions.net/getFollowingDetails](https://us-central1-haiku-bot-advanced.cloudfunctions.net/getFollowingDetails)"
    GET_FOLLOWERS_DETAILS_URL="[https://us-central1-haiku-bot-advanced.cloudfunctions.net/getFollowersDetails](https://us-central1-haiku-bot-advanced.cloudfunctions.net/getFollowersDetails)"
```

- **Server-Side Variables (E-Commerce Secrets):**
  - Add your **Test Mode** keys for local development.

```Bash
    STRIPE_SECRET_KEY="sk_test_..."
    STRIPE_WEBHOOK_SECRET="whsec_..."
    GELATO_WEBHOOK_SECRET="your_gelato_webhook_secret"
```

- **Client-Side Variables (Firebase Client SDK & App Check):**
  - Get these values from your **Firebase Project Settings > Your Web App** ("Haikuverse Gallery (web)").
  - Get the **reCAPTCHA Site Key** from the **Google Cloud Console > reCAPTCHA Enterprise**.

```bash
    # Client-Side (MUST start with NEXT_PUBLIC_)
    NEXT_PUBLIC_FIREBASE_API_KEY="AIzaSy..."
    NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN="haiku-bot-advanced.firebaseapp.com"
    NEXT_PUBLIC_FIREBASE_PROJECT_ID="haiku-bot-advanced"
    NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET="haiku-bot-advanced.firebasestorage.app"
    NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID="..."
    NEXT_PUBLIC_FIREBASE_APP_ID="1:...:web:..."
    NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID="G-..."
    NEXT_PUBLIC_RECAPTCHA_SITE_KEY="6Le0Lowr...YourSiteKey..."
    NEXT_PUBLIC_GOOGLE_CLIENT_ID="9450815...apps.googleusercontent.com"
```

4.  **Run the Development Server:**

```bash
    npm run dev
```

    The site will be available at `http://localhost:3000`.

5.  **Localhost GSI/App Check Setup:**
    - Open `http://localhost:3000` in your browser.
    - Open the Developer Console (F12).
    - Run `self.FIREBASE_APPCHECK_DEBUG_TOKEN = true;` and refresh the page.
    - Look for the log `App Check debug token: '...'`. Copy the token string.
    - Run `self.FIREBASE_APPCHECK_DEBUG_TOKEN = "PASTE_YOUR_TOKEN_HERE";` in the console.
    - Refresh again. Google Sign-In should now work on localhost.

### 4.4 Production Deployment (Firebase Hosting)

Deploying to production requires one-time setup for credentials and authorized domains.

1. **Configure Server Secrets (Google Secret Manager):**
   - Go to **Google Cloud Console > Security > Secret Manager**.
   - Create the following secrets (ensure you use Live Mode keys for Stripe):
     - `GALLERY_FIREBASE_PROJECT_ID`
     - `GALLERY_FIREBASE_CLIENT_EMAIL`
     - `GALLERY_FIREBASE_PRIVATE_KEY` (Raw, multi-line key)
     - `STRIPE_SECRET_KEY`
     - `STRIPE_WEBHOOK_SECRET`
     - `STRIPE_PUBLISHABLE_KEY`
     - `GELATO_WEBHOOK_SECRET`
   - Grant the project's Cloud Run service account the **"Secret Manager Secret Accessor"** role for each of these secrets.

2. **Configure `firebase.json`:**
   - Update your `firebase.json` to link the secrets to the Cloud Run environment and set the memory:

```json
    "frameworksBackend": {
      "memory": "1GiB",
      "secretEnvironmentVariables": [
        { "key": "PRIVATE_KEY", "secret": "GALLERY_FIREBASE_PRIVATE_KEY" },
        { "key": "CLIENT_EMAIL", "secret": "GALLERY_FIREBASE_CLIENT_EMAIL" },
        { "key": "PROJECT_ID", "secret": "GALLERY_FIREBASE_PROJECT_ID" },
        { "key": "STRIPE_SECRET_KEY", "secret": "STRIPE_SECRET_KEY" },
        { "key": "STRIPE_WEBHOOK_SECRET", "secret": "STRIPE_WEBHOOK_SECRET" },
        { "key": "NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY", "secret": "STRIPE_PUBLISHABLE_KEY" },
        { "key": "GELATO_WEBHOOK_SECRET", "secret": "GELATO_WEBHOOK_SECRET" }
      ]
    }
```

3. **Configure GSI & App Check Domains:**
   - **GCP OAuth Client ID:** In **APIs & Services > Credentials**, add `https://haikuverse-gallery.web.app` to the "Authorized JavaScript origins" of your Web client OAuth ID.
   - **reCAPTCHA Enterprise:** Edit your key and add `haikuverse-gallery.web.app` to the "Domains" list.
   - **Firebase App Check:** In the Firebase Console, register your web app with the reCAPTCHA key.

4. **Deploy:**

```bash
    firebase deploy --only hosting:gallery
```

---

## 5. Design Philosophy & Production Architecture

The Haikuverse Public Gallery is not just a website; it is an extension of the **"Designed for Humans"** philosophy that governs the entire Fancyland ecosystem. It serves two distinct masters: the **public web** (which demands speed and SEO) and the **authenticated subscriber** (who demands privacy, security, and tangible value).

### 5.1 An Open Ecosystem for Creators

A core tenet of the Haikuverse is to create an ecosystem that empowers creators by giving them multiple channels to share their work. While the native Android app is optimized for ephemeral social sharing, the Public Gallery provides high-fidelity, permanent artifacts.

The **Haikucard Print Studio** is the flagship of this strategy. By giving subscribers the ability to generate magnificent, high-resolution PNGs, we are giving them true ownership of their digital masterpieces. We do not lock these assets behind a walled garden. Users can download them for free or use our integrated service to manufacture them. This multi-channel approach—from social shares to physical products—is designed to foster a vibrant ecosystem where digital art becomes tangible reality.

### 5.2 The "Clump Factory" Pattern: Financial Integrity

In building the e-commerce engine, we rejected the standard "shopping cart" model in favor of a **"Clump Factory"** architecture (`/api/orders/create`).

In a standard cart, prices and product details are often re-calculated at checkout, leading to potential race conditions or discrepancies between what the user sees and what is charged. Our "Clump Factory" creates an **immutable snapshot** of the order—including the specific shipping address, the calculated tax, the exact price, and the generated artwork URLs—_before_ the payment intent is even created.

This "Clump" (the `pending_payment` document) acts as a single source of truth. The payment processor (Stripe) and the fulfillment provider (Gelato) both read from this frozen record. This ensures that **what the user agrees to pay is exactly what is charged**, and **what the user sees on screen is exactly what is printed**, eliminating "bait-and-switch" bugs entirely.

### 5.3 The "Architecture Tax": Dual-Asset Manufacturing

We made a deliberate choice to pay an "Architecture Tax" to improve the user experience. Instead of generating a single image file to serve both the user and the printer, we implemented a **Dual-Asset Pipeline**.

- **The Blueprint (For the Machine):** We generate a raw, 300 DPI, full-bleed CMYK-compatible PNG with square corners. This file is ugly to look at but perfect for Gelato’s cutting machines, ensuring no white edges appear on the final card.
- **The Preview (For the Human):** We simultaneously generate a cropped, RGB-optimized PNG with rounded corners that mimics the physical die-cut card.

By separating these concerns in `imageService.ts`, we ensure the customer sees a beautiful "product shot" in their email, while the manufacturer gets a technically perfect file. This reduces print errors and increases customer trust.

### 5.4 Privacy by Design: The Janitor Protocol

Aligned with our **Prohumanist** philosophy, we treat user data as a liability, not an asset. We strictly limit the lifespan of Personally Identifiable Information (PII).

- **Zero-PII Friction:** We use `localStorage` on the client to cache shipping details for the user's convenience. This data never touches our database until the user explicitly clicks "Pay".
- **The PII Janitor:** We implemented a scheduled Cloud Function (`purgeDeliveredOrders`) that acts as a relentless privacy enforcer. 180 days after an order is delivered—just enough time to handle chargebacks or returns—the Janitor creates an atomic batch operation. It hard-deletes the order record, wipes the email logs, and destroys the print assets from Cloud Storage.

This architecture ensures that Fancyland LLC does not accumulate a toxic "data lake" of user addresses. We keep only what is required for tax compliance (anonymized in the `accounting_ledger`) and nothing more.

### 5.5 Hybrid Rendering: Balancing Speed and Freshness

A key architectural choice was building the gallery with **Next.js** and deploying it to **Firebase Hosting** with the `frameworksBackend` feature. This runs the entire application on a **Google Cloud Run** service, allowing for a sophisticated hybrid rendering model:

- **SSG (Static Site Generation):** Core discovery pages (`/poet`, `/constellation`) are pre-rendered at build time. This provides semantic HTML that is instantly indexable by search engines.
- **ISR (Incremental Static Regeneration):** To solve the "stale data" problem, we use ISR with a 1-hour revalidation window. This ensures that the gallery reflects new stars and community activity without requiring a full site redeploy, balancing freshness against server costs.
- **CSR (Client-Side Rendering):** The Dashboard and Print Studio are purely client-side. This allows for a rich, app-like interactive experience where the UI reacts instantly to user input (like changing a card background) without server round-trips.

### 5.6 Robust Security & Compliance

Security is not an addon; it is baked into the infrastructure.

- **Singleton Admin SDK:** We use a singleton pattern (`firebaseAdmin.ts`) to manage server-side connections, preventing memory leaks and ensuring consistent authentication.
- **Secret Injection:** All sensitive keys (Stripe, Gelato, Firebase Admin) are injected directly from **Google Secret Manager** into the Cloud Run environment at runtime. They never exist in the codebase or the build artifacts.
- **Active Awareness Compliance:** We integrated legal compliance directly into the UX. The checkout flow uses embedded accordions for Terms and Privacy policies, ensuring "Active Awareness"—users must interact with the agreement layer to proceed, ensuring informed consent without disrupting the purchase flow.
- **The "Trust but Verify" Gatekeeper:** We recognized that the transition from client-side Firestore rules to server-side Admin SDK logic creates a potential security gap. We closed this by implementing manual ownership verification logic in every write-capable API route, effectively re-implementing RLS (Row-Level Security) at the application layer to prevent unauthorized resource manipulation via IDOR.
- **Asset Integrity (Explicit Authority):** To prevent "bait-and-switch" discrepancies between the UI and the backend, we enforce an **Explicit Authority** pattern. The client must explicitly request a specific asset URL, and the server validates that this specific URL belongs to the authenticated user's underlying Firestore document before processing any generation or order requests.

### 5.7 Mobile-First Styling Architecture

Finally, the Public Gallery is a first-class citizen in the mobile-first Haikuverse ecosystem. We utilize **CSS Modules** to scope styles locally to each component, preventing cascade regressions. The layout uses a "mobile-first" media query strategy: the default CSS defines the experience for narrow phone screens (single columns, large touch targets), while `min-width` queries progressively enhance the layout for tablets and desktops (grid views, sticky sidebars). This guarantees that the site is performant and accessible on the devices our users actually use.

### 5.8 Artifact-First Marketing Strategy

The application’s user funnel is architected to prioritize the result over the tool. We recognize that users are often fatigued by abstract software subscriptions but value tangible utility. Therefore, the marketing strategy leverages the Public Gallery to showcase high-fidelity physical goods ("Forge Artifacts") as the primary hook. The architecture purposefully delays the full feature breakdown until the specific moment of intent. The clear value proposition—the distinction between a Guest and a Creator—is made discoverable only after the user has mentally committed to the utility of the Print Studio. This psychological framing positions the subscription not as a gate to a "app," but as _a necessary "manufacturing license"_ for the physical artifacts the user has already decided they want.

### 5.9 Designed for Humans

Ultimately, the architecture of the Haikuverse Public Gallery serves as a manifesto for Human-Centric Engineering, a rejection of the industry's obsession with efficiency at the expense of the user's emotional experience. We deliberately chose to embrace architectural complexity—increasing our own maintenance burden—specifically to ensure simplicity, honesty, and sovereignty for the human on the other side of the screen.

This philosophy is most visible in our decision to pay the "Architecture Tax" of the Dual-Asset Pipeline. A purely logical system would generate a single image file to satisfy the printer, forcing the user to decipher a raw, bleed-edged CMYK schematic filled with crop marks and strange margins. We rejected that path. Instead, we engineered a parallel rendering pipeline solely to generate delight. When a creator opens their "Order Shipped" notification, they are not greeted by a robotic blueprint, but by a beautiful, rounded, high-fidelity visualization of their physical art as it exists in the real world. We accepted the complexity of managing two distinct assets for every order because we believe the software’s job is to bridge the gap between digital imagination and physical arrival, transforming a standard transactional update into a genuine spark of joy and anticipation.

Similarly, the "Clump Factory" was not built merely to solve the technical problem of database race conditions; it was built to solve the human problem of anxiety. In an e-commerce landscape rife with dynamic pricing and hidden fees, the Clump Factory acts as a guarantee of financial integrity. By freezing the asset state, tax calculation, and final price into an immutable snapshot before the transaction begins, we ensure that the price the user sees is the price they pay—transparent, locked, and honest.

Furthermore, the very existence of this Public Gallery is a structural commitment to inclusivity over exclusivity. We rejected the "walled garden" model that demands an app download or a login credential just to experience a moment of beauty. By architecting a high-performance, read-only web layer, we meet curious newcomers exactly where they are—in the browser, on a link shared by a friend, without friction or commitment. This decision transforms the Haikuverse from a closed club into an open agora, ensuring that the poetry experience remains accessible to all, while reserving the deep tools of creation for those ready to take the next step.

Most critically, the "Janitor" protocol embodies our commitment to data hygiene and respect. By treating a user's physical address as a temporary necessity rather than a permanent asset, we operationalized the belief that personal data is something we are privileged to borrow only for the specific moment of fulfillment. We do not build vast data lakes; we build clean rooms. In the Haikuverse ecosystem, technology steps back from being a central authority to become a humble, efficient steward, ensuring that from the first spark of creativity to the final delivery of the physical artifact, the user’s sovereignty and delight remain the undisputed center of gravity.

---

## License

Copyright © 2025 Fancyland, LLC
