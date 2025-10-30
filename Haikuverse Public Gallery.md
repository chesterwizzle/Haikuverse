# Haikuverse Public Gallery 🦑

---
## 1. Introduction

The Haikuverse Public Gallery is a lightweight, read-only web portal designed to serve as the public, discoverable "front door" to the Haikuverse ecosystem while also providing a personalized hub for authenticated members.

Built with **Next.js**, it is deployed on **Firebase Hosting** using the `frameworksBackend` feature, which intelligently runs all server-side logic (API Routes, SSR) on a scalable **Google Cloud Run** service. It strategically employs a **hybrid rendering model**:
* **Static Site Generation with ISR:** Used for all core public pages (Home, Poet, Constellation) to combine maximum performance with automatic data-freshness.
* **Dynamic Rendering:** Used for individual Star pages and all secure API routes.
* **Client-Side Rendering:** Used for the fully interactive user Dashboard.

This mission-critical app attracts new users organically and offers a convenient, high-performance web touchpoint for the existing community.

### Key Features:

* **Blazing Fast & Always Fresh:** Leverages Incremental Static Regeneration (ISR) for core content (`/poet/[nickname]`, `/constellation/[name]`). This combines the instant load times of static generation with the ability to automatically refresh data from Firebase in the background, ensuring content like new stars and follower counts stays up-to-date without needing a redeploy.
* **Interactive Homepage Hub:** Features a dynamic **Zeitgeist Map** highlighting trending community themes and a **Poet Search** bar for easy discovery.
* **Clear User Acquisition Funnel:** A prominent call-to-action on the main page directs new visitors to the Google Play Store, creating a direct path from discovery to download.
* **Fully Responsive & Mobile-First:** The entire gallery is built with a mobile-first philosophy to align with the Haikuverse ecosystem. All pages, from the interactive homepage to the feature-rich Print Studio, are fully responsive, using a modern CSS architecture that defaults to a clean, single-column layout on phones and scales up gracefully for tablets and desktops.
* **Dynamic Poet Portfolios:** Every poet gets a unique, shareable, statically-generated page that automatically showcases their bio, high-resolution avatar, owned constellations, and contributions to other constellations.
* **Immersive Constellation Galleries:** Each constellation has its own dedicated, statically-generated page featuring its fable, lore image, and a gallery of all the stars within it.
* **Detailed Star Experience:** A server-rendered star page showcases each creation with a high-resolution, optimized image slideshow timed to the cadence of the audio preview and a display for community like counts.
* **Personalized Member Dashboard:** Logged-in users access a dashboard that calls serverless API routes to display their Haikuverse **nickname** and real-time lists of poets they **follow** and who **follow them**.
* **Gated Creator Access:** Provides a convenient link to the **Haikuverse Web Creator** and the **Print Studio** for authenticated subscribers.
* **High-Resolution Haikucard Generation:** A dedicated Print Studio provides a rich, interactive experience for creators. The studio features a comprehensive grid displaying all three generated images for each star, sorted alphabetically for easy browsing. Users can individually select any of the artworks to load it into the editor, allowing for precise customization. The server-side API then dynamically composites the chosen artwork and text onto various backgrounds (paper, foil, solid colors), automatically adjusting font size and centering to create a perfectly balanced, print-ready PNG with true transparency.
* **Secure User Authentication:** Supports secure sign-in via **Google Sign-In (GSI)**, managed globally with React Context. The production environment is secured by authorizing the live domain in both the **GCP OAuth Client ID** and the **reCAPTCHA Enterprise key** settings.
* **Robust & Secure Backend:**
    * Employs a server-only **Firebase Admin SDK** initialized using a **singleton pattern** (`src/lib/firebaseAdmin.ts`) to prevent redundant initializations and improve server performance.
    * All server-side credentials (`PROJECT_ID`, `CLIENT_EMAIL`, `PRIVATE_KEY`) are securely injected into the **Google Cloud Run** environment at deploy time using **Google Secret Manager**, configured via `firebase.json`.
    * All API routes (`/api/*`) are refactored to import and use the single, centralized `auth` and `db` instances, ensuring consistent, authenticated access to Firebase services.
* **Optimized for Production:**
    * The deployed Cloud Run service is configured with **1GiB of memory** in `firebase.json` to handle Next.js server-side rendering loads without crashing.
    * Uses the Next.js `<Image>` component with correctly configured `remotePatterns` (for `storage.googleapis.com` and `firebasestorage.googleapis.com`) and `sizes` props (e.g., `sizes="16rem"` for avatars) to ensure high-resolution, optimized images are served.

---
## 2. Functional Block Diagram

```mermaid
---
config:
  layout: elk
  elk:
    nodePlacementStrategy: NETWORK_SIMPLEX
  theme: base
  themeVariables:
    lineColor: '#555'
    textColor: '#333'
---
flowchart LR
  subgraph Browser["User's Browser"]
    direction TB
    User["User Interaction"]
    subgraph ClientComponents ["Client Components (React)"]
        HomePageClient["Home Page<br>(ZeitgeistClient, PoetSearch)"]
        DashboardPageClient["Dashboard Page<br>(HaikucardEditor)"]
        HeaderClient["Header (Auth Aware)"]
        SSGPages["SSG Pages (Hydrated)<br>(Poet, Constellation)"]
    end
    AuthContext["AuthContext (useAuth)"]
    FirebaseClientSDK[("Firebase Client SDK<br>(Auth, App Check)")]
    GSI["Google Sign-In (GSI) Library"]
  end

  subgraph ProductionEnvironment["Production Environment<br>(GCP)"]
    direction TB
    subgraph Hosting["Firebase Hosting (Frontend)"]
        direction TB
        StaticAssets["Static HTML, CSS, JS<br>(SSG Pages, Client Bundles)"]
    end
    
    subgraph CloudRun["Google Cloud Run<br>(Backend)<br>firebase-frameworks<br>-haikuverse-gallery"]
        direction TB
        FirebaseAdminModule["**src/lib/firebaseAdmin.ts**<br>(Singleton Pattern for<br>Admin SDK)"]
        
        subgraph ServerBuild ["Build Process"]
            NextServerBuild["Next.js Build"]
            SSGPageGen["SSG Page Generation<br>(getStaticParams)"]
        end

        subgraph ServerRuntime ["Runtime"]
            NextServerRuntime["Next.js Server"]
            ApiNickname["/api/profile/nickname"]
            ApiFollowing["/api/network/following"]
            ApiFollowers["/api/network/followers"]
            ApiSearch["/api/search-poets"]
            ApiGeneratePng["/api/print/generate-png"]
            SSRPageGen["SSR Page Generation<br>(/star/[id])"]
        end
    end
  end

  subgraph BackendServices["Backend Services (GCP)"]
    direction TB
    SecretManager[("Google Secret Manager<br>GALLERY_FIREBASE_PRIVATE_KEY, ...")]
    FirestoreDB[("Cloud Firestore<br>/users, /public_profiles, ...")]
    CloudFunctions[("Haikuverse v1 Cloud Functions<br>getFollowingDetails,<br>getFollowersDetails, ...")]
    subgraph GoogleAuthServices["Google Auth Services (GCP)"]
        FirebaseAuthBackend[("Firebase Auth<br>(Backend)")]
        FirebaseAppCheck[("Firebase App Check")]
        GCP_OAuth["GCP OAuth Client ID<br>(Authorized JavaScript<br>origins)"]
        GCP_reCAPTCHA["GCP reCAPTCHA Key<br>(Authorized Domains)"]
    end
  end

  %% --- Flows ---

  %% DEPLOYMENT & INITIALIZATION
  SecretManager -- "Injects Env Vars (PRIVATE_KEY, ...)" --> CloudRun
  %% CloudRun -- "On Start" --> FirebaseAdminModule

  %% Static Page Load (SSG) - Build Time
  NextServerBuild --> SSGPageGen
  SSGPageGen -- "Imports & Uses" --> FirebaseAdminModule
  FirebaseAdminModule -- "Reads" --> FirestoreDB
  SSGPageGen -- "Generates" --> StaticAssets
  User -- "Visits /poet/Wiz" --> Hosting
  %% Hosting -- "Serves" --> StaticAssets
  StaticAssets -- "Loads in" --> Browser
  StaticAssets -- "Hydrates" --> SSGPages

  %% SSR Page Load (Star Page) - Runtime
  User -- "Visits /star/123" --> Hosting
  Hosting -- "Rewrites SSR/API to" --> NextServerRuntime
  NextServerRuntime -- "Renders Page" --> SSRPageGen
  SSRPageGen -- "Imports & Uses" --> FirebaseAdminModule
  FirebaseAdminModule -- "Reads" --> FirestoreDB
  SSRPageGen -- "Returns HTML" --> NextServerRuntime
  NextServerRuntime -- "Sends HTML" --> Browser

  %% Home Page Client Interaction (Search)
  User -- "Searches on Home" --> HomePageClient
  HomePageClient -- "Calls API" --> ApiSearch
  ApiSearch -- "Imports & Uses" --> FirebaseAdminModule
  FirebaseAdminModule -- "Reads Nicknames" --> FirestoreDB
  ApiSearch -- "Returns JSON" --> HomePageClient
  HomePageClient -- "Updates UI" --> User

  %% Authentication Flow (GSI & App Check)
  User -- "Clicks Sign In" --> HeaderClient
  HeaderClient -- "Triggers GSI" --> GSI
  GSI -- "Checks Origin vs" --> GCP_OAuth
  GSI -- "Callback with Credential" --> FirebaseClientSDK
  FirebaseClientSDK -- "Verifies via" --> FirebaseAppCheck
  FirebaseAppCheck -- "Checks Domain vs" --> GCP_reCAPTCHA
  FirebaseClientSDK -- "signInWithCredential" --> FirebaseAuthBackend
  FirebaseAuthBackend -- "Returns User" --> FirebaseClientSDK
  FirebaseClientSDK -- "Updates" --> AuthContext
  AuthContext -- "Notifies" --> HeaderClient & DashboardPageClient

  %% Haikucard Generation Flow
  User -- "Clicks Generate PNG" --> DashboardPageClient
  DashboardPageClient -- "Calls API with Token & Options" --> ApiGeneratePng
  ApiGeneratePng -- "Imports & Uses" --> FirebaseAdminModule
  FirebaseAdminModule -- "Verifies Token & Reads Haiku" --> FirebaseAuthBackend & FirestoreDB
  ApiGeneratePng -- "Returns PNG Blob" --> DashboardPageClient
  DashboardPageClient -- "Updates UI (shows download link)" --> User
  
  %% Dashboard Data Fetch Flow
  User -- "Navigates to /dashboard" --> DashboardPageClient
  DashboardPageClient -- "Gets ID Token from" --> FirebaseClientSDK
  DashboardPageClient -- "Calls APIs" --> ApiNickname & ApiFollowing & ApiFollowers

  %% API Route Flow (Nickname Example)
  ApiNickname -- "Imports & Uses" --> FirebaseAdminModule
  FirebaseAdminModule -- "Verifies Token & Reads DB" --> FirebaseAuthBackend & FirestoreDB
  ApiNickname -- "Returns data" --> DashboardPageClient
  
  %% API Route Flow (Following Example - Proxy)
  ApiFollowing -- "Imports & Uses" --> FirebaseAdminModule
  ApiFollowing -- "Calls (HTTP POST)" --> CloudFunctions
  CloudFunctions -- "Verifies Token & Reads DB" --> FirebaseAuthBackend & FirestoreDB
  CloudFunctions -- "Returns results" --> ApiFollowing

  %% Final UI Update
  DashboardPageClient -- "Updates UI" --> User

  %% Styles
  style Browser fill:#E8F0FE,stroke:#4285F4,stroke-width:2px
  style ProductionEnvironment fill:#f8f8ff,stroke:#4b0082,stroke-width:2px
  style BackendServices fill:#FFF7E6,stroke:#FFA900,stroke-width:2px
  style GoogleAuthServices fill:#FFEFE6,stroke:#FF5733,stroke-width:1px
  style ClientComponents fill:#fff
  style ServerBuild fill:#eee
  style ServerRuntime fill:#ddd
  style FirebaseAdminModule fill:#E6F4EA,stroke:#34A853,stroke-width:2px
```

---
## 3. Core Architecture & Services

The Haikuverse Public Gallery is built on a modern, SEO-first technology stack, leveraging the **Next.js** framework and deploying to **Firebase Hosting**. The `frameworksBackend` feature is enabled, which deploys all server-side logic (API Routes, SSR) to a scalable **Google Cloud Run** service. The table below provides a high-level overview of the key technologies employed.

| Layer                | Technology / Service                             | Purpose                                       |
| :------------------- | :----------------------------------------------- | :-------------------------------------------- |
| **Web Framework**    | Next.js (App Router), React, TypeScript          | UI, Routing, SSG/SSR, API Routes              |
| **Styling**          | CSS Modules                                      | Component-scoped styling                      |
| **Image Composition**| **sharp** (Node.js)                              | High-performance server-side image processing |
| **Client Auth**      | Firebase Auth SDK, Google Sign-In (GSI), Context | User login, session management                |
| **Client Security**  | Firebase App Check (reCAPTCHA Enterprise)        | Verifying client integrity                    |
| **Server Backend**   | **Google Cloud Run** (via Firebase)              | Hosts all server-side Next.js logic           |
| **Server Auth/Data** | **Firebase Admin SDK** (Node.js)                 | Secure data fetching (SSG/SSR/API)            |
| **Database**         | Cloud Firestore                                  | Storing all core data                         |
| **Server Secrets**   | **Google Secret Manager**                        | Securely injects credentials into Cloud Run   |
| **API Layer**        | Next.js API Routes                               | Secure endpoints for client data fetching     |
| **Hosting**          | Firebase Hosting                                 | Serves static assets, routes backend requests |

The following sections provide a detailed breakdown of the application's core components.

### 3.1 Next.js App Router (`src/app/`)

The application is built using the **Next.js App Router**, which uses a file-based routing system. Folders within the `src/app/` directory define the URL structure of the site.

* **Static Routes:** The root homepage is defined by `src/app/page.tsx`.
* **Dynamic Routes:** Pages for individual poets and constellations are created using folders with square brackets, such as `src/app/poet/[nickname]/page.tsx`.
* **Layouts:** The root layout at `src/app/layout.tsx` defines the shell `<html>` and `<body>` tags that wrap every page, ensuring a consistent base structure and styling.
* **Client Routes:** The user dashboard (`/dashboard`) is implemented as a client component, dynamically rendering based on authentication state.
* **API Routes:** Backend logic accessible to the client is defined in `/api/` folders (e.g.,`/api/profile/nickname/route.ts`). These handle secure data fetching for interactive components.

### 3.2 Hybrid Rendering Strategy

The gallery employs a hybrid rendering approach, optimized for SEO, performance, and interactivity.

* **Static Site Generation (SSG):** Core discovery pages like poet portfolios (`/poet/[nickname]`) and constellation galleries (`/constellation/[name]`) use `generateStaticParams`. This pre-renders static HTML at build time using the **Firebase Admin SDK** for secure data fetching, ensuring fast initial loads and excellent SEO.
* **Incremental Static Regeneration (ISR):** To prevent content from becoming stale, all statically-generated and server-rendered pages employ ISR with a revalidate period of one hour (3600s). This strategy ensures that while users always receive an instant, cached response, the page is automatically rebuilt in the background if new data (e.g., new stars, updated follower counts) is available. This provides the perfect balance between static performance and data freshness, minimizing server load and Firebase reads.
* **Server-Side Rendering (SSR):** Individual star detail pages (`/star/[id]`) are rendered on-demand (dynamic) to ensure the latest like counts and details are always shown.
* **Client-Side Rendering (CSR):** Pages requiring user interaction and browser-side state (like the Homepage and Dashboard) are designated as Client Components using the **`"use client";`** directive.
* **API Routes:** Serverless functions co-located with the app (`/api/*`) act as a secure backend gateway for Client Components.

### 3.3 Client Components & State Management (`"use client";`)

Client Components handle user interaction, browser-specific APIs, and state management within the user's browser.

* **Interactivity & Data Fetching:** Components like the `PoetSearch` on the homepage, the `DashboardPage`, and the site `Header` are Client Components. They use React Hooks (`useState`, `useEffect`) to manage local UI state and fetch dynamic data.
* **Authentication State:** A global **`AuthContext`** (`src/context/AuthContext.tsx`) provides application-wide access to the current user's authentication state (`user`, `loading`) via the `useAuth` hook.
* **Client-Side Firebase:** The **Firebase Client SDK** (`src/lib/firebaseClient.ts`) is initialized for client-side operations:
    * **Firebase Auth:** Manages the sign-in state.
    * **Firebase App Check:** Secures client-side requests using reCAPTCHA Enterprise.
* **Google Sign-In (GSI):** GSI is integrated via `gsiHelper.ts`. For production, this required authorizing the deployed domain (`haikuverse-gallery.web.app`) in both the **GCP OAuth Client ID's** "Authorized JavaScript origins" and the **reCAPTCHA Enterprise key's** "Domains" list.
* **API Interaction:** Client Components fetch data securely by calling the Next.js **API Routes** using `fetch` within `useEffect` hooks, passing the user's Firebase ID token for authorization.

### 3.4 Styling with CSS Modules

The gallery's styling is implemented using **CSS Modules**, a system that scopes CSS locally to each component. This approach was chosen to ensure style encapsulation and prevent class name conflicts.

* **Scoped & Responsive by Default:** Each page or component has its own *.module.css file (e.g., `PoetPage.module.css`), built with a mobile-first philosophy. The build process automatically generates unique class names, guaranteeing that styles from one component cannot accidentally affect another, while media queries are used to progressively enhance the layout for larger screens.
* **Maintainability:** This architecture creates a clean separation between a component's structure (in the `.tsx` file) and its appearance (in the `.module.css` file), making the codebase easier to read and maintain.

### 3.5 Backend Integration & Security

This layer manages the connection to backend services and enforces security, running entirely within the **Google Cloud Run** environment on the server.

* **Centralized Admin SDK (`src/lib/firebaseAdmin.ts`):** This is the core of the backend. It initializes the **Firebase Admin SDK** using a **singleton pattern** to ensure only one instance of the app exists on the server. This prevents redundant initializations, improves performance, and exports the shared `db` (Firestore) and `auth` (Auth) instances for use across the entire server.
* **Secure Credentials:** In production, server-side credentials (`PROJECT_ID`, `CLIENT_EMAIL`, `PRIVATE_KEY`) are **injected securely from Google Secret Manager** into the Cloud Run environment at deploy time. This is configured in `firebase.json` using the `secretEnvironmentVariables` block.
* **API Routes (`src/app/api/`):** All API routes have been refactored to **import the centralized `db` and `auth` instances** from `firebaseAdmin.ts`. This ensures they all use the single, correctly authenticated Admin SDK instance.
* **Data Fetching Service (`src/lib/firebaseService.ts`):** Centralizes reusable data-fetching functions (e.g., `getAllPoets`, `getStarById`) used by Server Components during SSG.
 * **Server-Side Image Generation:** A key API route, `/api/print/generate-png`, leverages the **`sharp`** library for high-performance image processing and **`opentype.js`** for precise text measurement. It composites artwork, dynamically-sized haiku text with embedded fonts, and user-selected backgrounds onto a transparent canvas, producing a print-ready 300 DPI PNG with true transparency.

---
## 4. Security, Testing, and Setup

This section details the gallery's security model, outlines the testing strategy, and provides a clear guide for setting up the development and production environments.

### 4.1 Multi-Layered Security Model

The gallery employs a multi-layered security approach, combining server-side safeguards with client-side verification.

* **Server-Side Security (Firebase Admin SDK):**
    * The entire Next.js server (SSG, SSR, API Routes) runs in a **Google Cloud Run** container managed by Firebase Hosting.
    * A **centralized Firebase Admin SDK** (`src/lib/firebaseAdmin.ts`) is initialized using a **singleton pattern** to ensure a single, shared connection to Firebase services.
    * This Admin SDK is used by all server-side code (SSG, API Routes) for data fetching and token verification.
    * **In Production:** Credentials (`PROJECT_ID`, `CLIENT_EMAIL`, `PRIVATE_KEY`) are securely injected into the Cloud Run environment from **Google Secret Manager** at deploy time, as configured in `firebase.json`.
    * **In Development:** Credentials are read from local `.env.local` variables.

* **Client-Side Security (Client SDK & App Check):**
    * User authentication is handled client-side via the **Firebase Authentication Client SDK** and **Google Sign-In (GSI)**.
    * **Firebase App Check** (with reCAPTCHA Enterprise) is integrated to verify that requests originate from a legitimate instance of the gallery website.
    * **Production GSI/App Check:** This requires authorizing the production URL (`https://haikuverse-gallery.web.app`) in two places:
        1.  The **GCP OAuth Client ID's** "Authorized JavaScript origins".
        2.  The **reCAPTCHA Enterprise Key's** "Domains" list.
    * **Local GSI/App Check:** This requires setting the `self.FIREBASE_APPCHECK_DEBUG_TOKEN` in the browser console.

* **Secure Cloud Function Interaction:**
    * Next.js API Routes act as a secure proxy to the original Haikuverse v1 Cloud Functions (`getFollowingDetails`, `getFollowersDetails`).
    * The API Route verifies the client's ID token using the Admin SDK, then makes a server-to-server `fetch` call to the Cloud Function URL, forwarding the ID token for the v1 function to perform its own verification.

### 4.2 Testing Strategy

A comprehensive testing strategy is essential for ensuring the gallery is robust, performant, and maintainable. The plan includes:

* **Unit Testing:** The service layer functions in `src/lib/firebaseService.ts` will be unit-tested using a framework like **Jest**. This involves mocking the Firebase Admin SDK to verify that our data-fetching functions correctly handle various database responses without making actual database calls.
* **Component Testing:** Interactive Client Components, such as the `StarSlideshow` and `StarDetailClient`, will be tested with **React Testing Library**. These tests will simulate user interactions (like clicking a play button) and assert that the component's state and appearance update as expected.
* **End-to-End (E2E) Testing:** A tool like **Cypress** or **Playwright** will be used to run automated tests on the complete, running application. These tests will simulate a full user journey—navigating from the homepage to a poet page, then to a constellation, and finally to a star—verifying that all pages load correctly and links work as intended.
* **Context Testing:** The `AuthContext` provider will be tested to ensure it correctly reflects user login/logout states and provides the user object as expected.
* **API Route Testing:** Next.js API Routes will be tested, potentially using tools like `next-test-api-route-handler` or similar, mocking Firebase Admin functions (`verifyIdToken`, Firestore reads) and fetch calls to Cloud Functions.

### 4.3 Setup and Configuration

To set up and run this project locally, follow these steps:

**Prerequisites:**

* **Node.js** (v20 required for production compatibility)
* **Project Structure:** All static assets required by the server (like fonts) **must be placed in the `public` directory**.
* **Visual Studio Code** or another code editor
* Access to the Haikuverse Firebase project
* Google Cloud Project with Billing enabled
* Enabled Firebase Services: Authentication (Google Sign-In), Firestore, App Check (Web provider)
* Enabled Google Cloud APIs: reCAPTCHA Enterprise, Secret Manager, Cloud Run

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
    * In the project root, create a file named `.env.local` (this file is git-ignored).
    * **Server-Side Variables (Firebase Admin SDK):**
        * Generate a new **service account JSON key** from Google Cloud (IAM & Admin > Service Accounts > Your Admin SDK Account > Keys > Add Key).
        * Copy the values into the `.env.local` file. **IMPORTANT:** The `PRIVATE_KEY` value must be a single-line string with `\n` characters replacing the newlines.
```bash
        # Server-Side (Admin SDK)
        PROJECT_ID="haiku-bot-advanced"
        CLIENT_EMAIL="firebase-adminsdk-fbsvc@haiku-bot-advanced.iam.gserviceaccount.com"
        PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nMIIEvg...Your...Key...Here...\n...More...Key...\n-----END PRIVATE KEY-----\n"
```
    * **Server-Side Variables (Cloud Function URLs):**
```bash
        GET_FOLLOWING_DETAILS_URL="[https://us-central1-haiku-bot-advanced.cloudfunctions.net/getFollowingDetails](https://us-central1-haiku-bot-advanced.cloudfunctions.net/getFollowingDetails)"
        GET_FOLLOWERS_DETAILS_URL="[https://us-central1-haiku-bot-advanced.cloudfunctions.net/getFollowersDetails](https://us-central1-haiku-bot-advanced.cloudfunctions.net/getFollowersDetails)"
```
    * **Client-Side Variables (Firebase Client SDK & App Check):**
        * Get these values from your **Firebase Project Settings > Your Web App** ("Haikuverse Gallery (web)").
        * Get the **reCAPTCHA Site Key** from the **Google Cloud Console > reCAPTCHA Enterprise**.
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
    * Open `http://localhost:3000` in your browser.
    * Open the Developer Console (F12).
    * Run `self.FIREBASE_APPCHECK_DEBUG_TOKEN = true;` and refresh the page.
    * Look for the log `App Check debug token: '...'`. Copy the token string.
    * Run `self.FIREBASE_APPCHECK_DEBUG_TOKEN = "PASTE_YOUR_TOKEN_HERE";` in the console.
    * Refresh again. Google Sign-In should now work on localhost.

### 4.4 Production Deployment (Firebase Hosting)

Deploying to production requires one-time setup for credentials and authorized domains.

1. **Configure Server Secrets (Google Secret Manager):**
    * Go to **Google Cloud Console > Security > Secret Manager**.
    * Create three secrets: `GALLERY_FIREBASE_PROJECT_ID`, `GALLERY_FIREBASE_CLIENT_EMAIL`, and `GALLERY_FIREBASE_PRIVATE_KEY`.
    * **CRITICAL:** For the `PRIVATE_KEY` secret, paste the **raw, multi-line** key value directly from the **downloaded service account JSON file**.
    * Grant the project's **Cloud Run service account** the **"Secret Manager Secret Accessor"** role for each of these secrets.

2. **Configure `firebase.json`:**
    * Update your `firebase.json` to link the secrets to the Cloud Run environment and set the memory:
```json
    "frameworksBackend": {
      "memory": "1GiB",
      "secretEnvironmentVariables": [
        { "key": "PRIVATE_KEY", "secret": "GALLERY_FIREBASE_PRIVATE_KEY" },
        { "key": "CLIENT_EMAIL", "secret": "GALLERY_FIREBASE_CLIENT_EMAIL" },
        { "key": "PROJECT_ID", "secret": "GALLERY_FIREBASE_PROJECT_ID" }
      ]
    }
```

3. **Configure GSI & App Check Domains:**
    * **GCP OAuth Client ID:** In **APIs & Services > Credentials**, add `https://haikuverse-gallery.web.app` to the "Authorized JavaScript origins" of your Web client OAuth ID.
    * **reCAPTCHA Enterprise:** Edit your key and add `haikuverse-gallery.web.app` to the "Domains" list.
    * **Firebase App Check:** In the Firebase Console, register your web app with the reCAPTCHA key.

4. **Deploy:**
```bash
    firebase deploy --only hosting:gallery
```

---
## 5. Design Philosophy & Production Architecture

The Haikuverse Public Gallery embodies a dual mission, extending the core application's **"Designed for Humans"** philosophy. It serves as both a public, discoverable archive optimized for SEO and a personalized web portal for authenticated members. Architectural decisions balance the need for organic user acquisition via search engines with providing a seamless, low-friction authenticated experience for the existing community.

### 5.1 An Open Ecosystem for Creators

A core tenet of the Haikuverse is to create an open ecosystem that empowers and delights creators by giving them multiple channels to share their work. While the native Android app is optimized for quick, social sharing of "haikucards," the Public Gallery extends this philosophy by providing high-fidelity, permanent artifacts.

The **Haikucard Print Studio** is the flagship of this strategy. By giving subscribers the ability to generate magnificent, high-resolution PNGs, we are giving them true ownership of their digital masterpieces. These files are not just for viewing; they are print-ready assets designed for personal use and, eventually, to be seamlessly funneled into a cost-effective **print-on-demand service** for physical products like postcards and mugs.

This multi-channel approach—from ephemeral social shares in-app to permanent digital artifacts and physical products on the web—is designed to create delight and foster a vibrant ecosystem. Public content draws new users in, while authenticated members are rewarded with powerful tools that transform their creations into tangible art, ensuring that there are always new and exciting ways to share their voice.

### 5.2 Hybrid Architecture: Balancing SSG, SSR, and Client Interactivity

A key architectural choice was building the gallery with **Next.js** and deploying it to **Firebase Hosting** with the `frameworksBackend` feature. This runs the entire Next.js application (including server-side logic) on a **Google Cloud Run** service, allowing for a sophisticated hybrid rendering model. Core discovery pages like poet portfolios (`/poet/[nickname]`) and constellation galleries (`/constellation/[name]`) use **Static Site Generation (SSG)**. This pre-renders static HTML at build time, providing semantic content that is perfectly indexable by search engines and loads instantly. Individual star pages (`/star/[id]`) are rendered on-demand (dynamic) to ensure data like like counts is always fresh, while pages requiring user interaction (like the Homepage and Dashboard) are rendered as Client Components using **`"use client";`** to provide a fast, app-like interactive experience.

### 5.3 A Secure & Resilient Backend Infrastructure

The production environment runs reliably on a **Google Cloud Run** service managed by Firebase. The core of the backend is a centralized **Firebase Admin SDK** (`src/lib/firebaseAdmin.ts`), initialized using a **singleton pattern** to ensure a single, performant connection to Firebase services. In production, server-side credentials are securely injected into the Cloud Run environment from **Google Secret Manager**, as configured in `firebase.json`, which also provisions the service with **`"memory": "1GiB"`** to ensure the Next.js server has enough resources for server-side rendering.

### 5.4 Robust Authentication (GSI & App Check)

Integrating Google Sign-In on a custom domain with App Check required solving a complex chain of security errors. The final, secure solution required authorizing the production URL (`https://haikuverse-gallery.web.app`) in three distinct places: first, in the **Firebase Console** by registering the reCAPTCHA key to the web app; second, in the **GCP OAuth Client ID's** "Authorized JavaScript origins" (to fix `origin_mismatch`); and third, in the **reCAPTCHA Enterprise Key's** "Domains" list (to fix `appCheck/recaptcha-error`). This multi-pronged configuration ensures both Google's sign-in service and Firebase's App Check service recognize the deployed domain as legitimate.

### 5.5 Performance & Optimization

The site uses the Next.js `<Image>` component for all content, with `remotePatterns` configured for Firebase Storage (`storage.googleapis.com` and `firebasestorage.googleapis.com`). Low-resolution, pixelated avatars on poet pages were fixed by adding a `sizes="16rem"` prop to the avatar's `<Image>` component, correctly informing Next.js to fetch a high-quality image. The entire project was downgraded to the stable **Next.js 14** and **React 18** stack to resolve a critical, persistent TypeScript build error, and the codebase now passes all linting and type checks without workarounds.

### 5.6 Data Freshness & Caching Strategy
A core challenge was solving the "stale data" problem inherent with static site generation. Content pre-rendered at build time would not reflect new stars or community activity in Firebase. The solution was to implement **Incremental Static Regeneration (ISR)** across all public-facing server pages. By exporting revalidate = 3600 from each page, we instruct the Next.js server to serve the fast, cached version while automatically regenerating the page in the background if it's more than an hour old. This one-hour period was a deliberate choice, balancing the need for fresh content against the cost of server resources and Firebase reads, perfectly aligning with the gallery's role as a public archive rather than a real-time feed.

### 5.7 Design Principles (Maintainability & User Experience)

A primary design principle is that the Public Gallery must be a first-class citizen in the **mobile-first Haikuverse ecosystem**. To that end, the entire application has been architected with a responsive, mobile-first styling approach. Layouts are built by default for narrow viewports, ensuring a fast and intuitive experience on phones, and then progressively enhanced for tablets and desktops using media queries. This philosophy is grounded in a modern CSS foundation, including a global `box-sizing` reset in `globals.css`, which guarantees that components are predictable, maintainable, and free from common layout issues like overflow.

The styling architecture is built on **CSS Modules** to guarantee stability. Each component or page is paired with its own `*.module.css` file, generating unique class names that make it impossible for styles to leak and break other components. All database query logic is consolidated into a dedicated **service layer** (`/lib/firebaseService.ts`), allowing UI components to remain clean and making database logic easy to maintain. This design allows the Public Gallery to serve its dual role: for the public, it's a fast, read-only "showroom" designed for SEO and discovery. For authenticated members, it transforms into a personalized web portal, with the **Dashboard** providing a familiar hub and quick access to their community and creator tools—including a revamped **Print Studio** that empowers creators to browse and select from all of their generated masterpieces for any given star.

---
## License

Copyright © 2025 Fancyland, LLC
