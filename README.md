# Haikuverse

A rich, multi-sensory platform for creative expression, designed for humans.

**Status:** 🚀 **Closed Alpha Now Live!**

* The Haikuverse Closed Alpha test for the Android app began on **October 17, 2025**. Haikuverse requires a valid subscription from the Google Play Store. 
* The Haikuverse Public Gallery went live on Firebase hosting as of **October 22, 2025.** Only subscribers have access to the personalized dashboard.

To join the Closed Alpha, please contact: **chesterwizzle@fancyland.net**

---
## Key Links

| Resource | URL |
| :--- | :--- |
| **Get the App (Android)** | [https://play.google.com/store/apps/details?id=net.fancyland.haikuverse](https://play.google.com/store/apps/details?id=net.fancyland.haikuverse&pcampaignid=web_share) |
| **Haikuverse Public Gallery** | [https://haikuverse-gallery.web.app/](https://haikuverse-gallery.web.app/) |
| **Official Company Site** | [https://www.fancyland.net/](https://www.fancyland.net/) |
| **Haikuverse Support** | [support@fancyland.net](mailto:support@fancyland.net) |

---
## Project Overview

Haikuverse is a creative ecosystem built on a **"Designed for Humans"** philosophy, empowering users to co-create techno-poetic art. It combines powerful generative AI with a non-toxic community by design, allowing users to augment their artistry rather than replace it.

The ecosystem consists of two primary components:

1.  **The Haikuverse Application (Flutter):** The core multi-sensory experience. This Flutter app (for Android and web) is where users co-create with AI partners (**Gemini**, **Imagen 3**, and **Text-to-Speech**). They can publish their work and explore the community through a unique, procedurally generated 2D "constellation" graph.
2.  **The Public Gallery (Next.js):** The public-facing, SEO-optimized "front door" for discovery. This Next.js web portal uses Static Site Generation (SSG) to create fast, indexable poet portfolios and constellation galleries, serving as the main user acquisition funnel and a personalized dashboard for members.

## Detailed Documentation

This repository contains the detailed technical documentation for the entire Haikuverse ecosystem, split into its core components.

### 1. The Haikuverse Application (Flutter)

* **[View Full Technical Documentation: Haikuverse.md](./Haikuverse.md)**

A comprehensive deep-dive into the main **Flutter application**. This document details the complete architecture, from the client-side state management and procedural graph generation to the complex serverless backend on Google Cloud (Functions, Vertex AI, Vector Search) and the non-toxic community features.

### 2. The Haikuverse Public Gallery (Next.js)

* **[View Full Technical Documentation: Haikuverse Public Gallery.md](./Haikuverse%20Public%20Gallery.md)**

The official documentation for the **public-facing Next.js web portal**. This file explains the hybrid rendering model (SSG/SSR) used for SEO, the server-side architecture (Cloud Run, Admin SDK, Secret Manager), and the secure authentication flow for the member dashboard.

---
## License

Copyright © 2025 Fancyland, LLC
