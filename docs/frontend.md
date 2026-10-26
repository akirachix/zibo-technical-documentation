Frontend Web Documentation


1. Overview

The Mbegu Link Frontend web application (zibo-frontend) is an advanced Next.js-based single-page application engine utilizing React and Tailwind CSS. It serves as the primary orchestration center for the Mbegu Link digital agriculture network.
The platform implements an integrated, role-based conditional routing architecture. Upon successful gateway authentication, users are evaluated against system-level permissions and dynamically routed to specialized dashboard shells customized specifically for their operational context:
1. GERRI Specialist Portal (Desktop Layout): A data-dense operational system tailored for desktop viewports. It handles high-throughput crop classification approvals, germination trend chart analytics, deposit evaluations, and core compliance tasks.
2. Community Seed Bank (CSB) Coordinator Portal (Progressive Web Application): A mobile-first, highly responsive web dashboard optimized for native PWA installation on phones and tablet viewports. It allows field coordinators to process registry records, submit seed loans, view live notifications, and share localized knowledge data.
The frontend is a decoupled client layer that communicates asynchronously with the Mbegu Link remote backend service API using secure, stateless REST requests over HTTP. All data grids, analytical charts, workflow streams, and user context records are derived directly from live remote endpoints via background interceptors rather than on-device processing. System-wide components leverage a synchronized reactive model, using global state providers to broadcast transaction details and live status modifications instantly across the presentation space.

Main Functional Areas

The web engine organizes operations across five critical architectural clusters:
• Role-Based Gateway Routing: Security checkpoints handling multi-role logins, automated dashboard routing, session state checks, and a complete multi-step password security lifecycle (Forgot Password, Verification, Reset Password, and Change Password layouts).
• GERRI Specialist Management Suite: Verification management data tables, real-time data charts plotting regional germination trends, asset registration audit tools, and compliance modification actions.
• CSB Field Coordinator PWA Terminal: Fast-loading data logging modules for registering local seed types, submitting physical deposits, orchestrating micro-loan distributions, and monitoring community credit lifecycles.
• Live In-App Notification System: A cross-portal reactive communication system that generates transient alert toasts for system status adjustments and updates badged counters on action headers.
• Localized Context & Configuration Controls: Secure interface nodes managing personal settings profiles, user data modifications, on-demand profile photo uploads, and client appearance overrides (Light Mode, Dark Mode, or System Default).

2. Technology Stack


Technology	Purpose	Implementation Target
Next.js / React	Production framework utilizing automated view routing	Shell layouts, contextual pages, components
HTTP Interceptor Client	REST endpoint networking layer with token injections	src/lib/session.js, src/lib/authApi.js
Shared Preferences / Cache	In-browser client preference data storage persistence	Light/Dark preferences, profile metadata, flags
ChangeNotifier Context	State mutation engine & layout change broadcasting	Global data contexts (src/lib/portalStore.js)
Material 3 UI Theme	Seed-color interface theme builder and shadows map	Global branding configurations, component style rules
Inter & Roboto	Global design system typography layers	Typeface asset loading configurations via next/font


2.1 Visual Appearance & Scope Specifications

The frontend implements systematic UI uniformity using style injection parameters maintained directly within src/lib/theme.js to manage responsive changes seamlessly across all user portals:
• Typography Anchors: Inter handles primary workspace labels and high-priority copy blocks, supported by Roboto across analytical indicators and main headers.
• Light / Dark Mode Adaptation: The platform features universal dark mode synchronization across both the GERRI Desktop Dashboard and the CSB Coordinator mobile layouts, dynamically tracking chart scales, dashboard cards, and the central notification center. Input containers explicitly preserve drop shadows inside dark mode frames to maintain visual structure.
• Input Box Foundations: Controlled fields utilize borderless container architectures with crisp backdrops, custom vertical spacing buffers, and explicit htmlFor / id attribute combinations to ensure accessible control targeting.

2.2 Lifecycle Status & Capsule Mappings

Transactional processing stages, loan request tracking lists, and notification event headers map their current remote state parameters visually using color tokens configured inside src/lib/loanStatus.js:
• Pending Verification: Warm Gold Tint (Bg: 0xFFFFF3D6, Text: 0xFF8A6800) applied to active approval queues, unread loan reminders, and initial registration check steps.
• In Progress / Active: Light Emerald Variant (Bg: 0xFFE7F5E7, Text: 0xFF1D6B4B) applied to open seed records, running deposits, and verified database logs.
• Finalized / Completed: Mint Green Tint (Bg: 0xFFE2F7EB, Text: 0xFF126B48) confirming successfully processed steps, approved loans, and resolved variables.
• Rejected / Declined: Soft Crimson Tint (Bg: 0xFFFDECEC, Text: 0xFFB42318) communicating systemic alerts, missed deadlines, or administrative decline status.

3. Project Structure

The codebase isolates underlying networking definitions, immutable properties parsing, view components, asset libraries, and automated testing suites:
text
zibo-frontend/
├── .husky/                              # Automated pre-commit formatting & linter validation hooks
├── docs/
│   └── openapispec.json                 # Backend OpenAPI specification documentation schema
├── public/
│   ├── brand/
│   │   └── logo.png                     # Primary corporate green Mbegu Link graphic asset
│   ├── icon-192.png                     # Progressive Web App mobile home icon asset
│   └── icon-512.png                     # Progressive Web App splash launcher asset
├── src/
│   ├── app/                             # Next.js App Router root layout architecture folder
│   │   ├── notifications/
│   │   │   └── page.jsx                 # Comprehensive system alert dashboard view layout
│   │   ├── profile/
│   │   │   └── page.jsx                 # Personal user data parameters and action overview
│   │   ├── settings/
│   │   │   └── page.jsx                 # Local client configurations and style preferences terminal
│   │   ├── layout.jsx                   # Root platform viewport wrapper and context provider
│   │   └── page.jsx                     # Primary login portal container and authentication routing view
│   │
│   ├── components/
│   │   ├── auth/
│   │   │   ├── AuthShell.jsx            # Onboarding layout presentation framework container
│   │   │   ├── ForgotPasswordForm.jsx   # Credential recovery sequence entry sheet
│   │   │   ├── LoginForm.jsx            # Core secure user identity entry field module
│   │   │   ├── RoleSelector.jsx         # Post-auth conditional permission router dropdown
│   │   │   └── TextField.jsx            # Standardized input utility containing unified labels
│   │   ├── brand/
│   │   │   └── Logo.jsx                 # Central branding vector layout placeholder
│   │   ├── onboarding/
│   │   │   └── OnboardingSlide.jsx      # Dynamic marketing layout slider utility carousel
│   │   ├── portal/
│   │   │   ├── DashboardLayout.jsx      # GERRI Specialist analytics desktop desktop wrapper
│   │   │   ├── NotificationBell.jsx     # Live header navigation alert indicator component
│   │   │   ├── PwaShell.jsx             # CSB Coordinator adaptive mobile application shell
│   │   │   ├── SignOutPopup.jsx         # Multi-device security sign-out confirmation component
│   │   │   └── StatusPill.jsx           # Unified hex-mapped state badge component caps
│   │   └── splash/
│   │       └── LandingSplash.jsx        # App launch persistence initialization loader view
│   │
│   ├── lib/
│   │   ├── api.js                       # Underlying communication layer endpoint register
│   │   ├── authApi.js                   # Remote access authentication transaction definitions
│   │   ├── config.js                    # Core environment connection configurations
│   │   ├── devAccount.js                # Sandbox profiles mapping dictionary
│   │   ├── directory.js                 # Seed database classification lookup dictionaries
│   │   ├── notifications.js             # Live push notification trigger state event handlers
│   │   ├── password.js                  # Mutation chains processing credential configuration changes
│   │   ├── portalApi.js                 # Shared repository log query operation models
│   │   ├── portalStore.js               # Global state context store for role dashboard layouts
│   │   ├── session.js                   # Axios setup providing authorization headers and status catches
│   │   ├── sessionTools.js              # Hardware session token wipe cache tools
│   │   ├── theme.js                     # Multi-theme layout presentation style injectors
│   │   └── useApi.js                    # Custom React hook coordinating background REST actions
│   │
│   └── _tests_/                         # Comprehensive Jest implementation specification files
│       ├── api.test.js
│       ├── ForgotPasswordForm.test.jsx
│       ├── landing.test.jsx
│       ├── LoginForm.test.jsx
│       ├── onboarding.test.jsx
│       ├── openapi.test.js
│       ├── organisations.test.jsx
│       ├── passwordScreens.test.jsx
│       ├── portal.test.jsx
│       ├── portalNotifications.test.jsx
│       ├── portalSettings.test.jsx
│       ├── RoleSelector.test.jsx
│       ├── session.test.js
│       └── useApi.test.js
│
├── package.json                         # Script automation registries, tasks, and system blocks
└── tailwind.config.js                   # Tailwind utility layout style mapping parameter lines


4. Architecture Layers

The repository adheres to a modular design system designed to encapsulate data tracking logic and insulate presentation elements from underlying processing errors:
text
┌────────────────────────────────────────────────────────┐
│                   Presentation Layer                   │
│  Views (homescreen.dart) + Conditional Shells (PwaShell)│
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│                 State Management Layer                 │
│    Context Providers & Event Emitters (portalStore.js)   │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│                     Service Layer                      │
│   session.js Middleware / Endpoints (portalApi.js)     │
└───────────────────────────┬────────────────────────────┘
│                           │                            │
│                           ▼                            │
│               [Mbegu Link Remote Backend]              │
│               REST API Transaction Boundaries          │
│                           │                            │
│                           ▼                            │
┌────────────────────────────────────────────────────────┐
│                   Data/Model Layer                     │
│  OpenAPI JSON Specifications & Preference Disk Caches  │
└────────────────────────────────────────────────────────┘
