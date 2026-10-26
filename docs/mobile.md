# Frontend Mobile Documentation

## 1. Overview

The ZIBO Mobile application (`zibo-mobile`) is a cross-platform mobile frontend developed using the Flutter framework inside the `mbegu_link` repository layer. It acts as the direct user-facing interface for the **Begulink** digital platform, empowering users to register seed assets, submit micro-loan queries, track processing states across an enterprise dashboard timeline, and consume localized agricultural training packages.

The mobile frontend is architected as a decoupled client application that communicates with the remote ZIBO backend API via secure HTTP operations. State operations, credential storage checks, data transformations, and profile adjustments are isolated inside local validation managers and service wrappers, completely removing direct on-device dataset mutations.

### Main Functional Areas

The mobile workspace manages five primary feature clusters:
* **Gateway & Identity Lifecycle:** Manages splash state validations, onboarding tokens, primary authentication matching, secure user signups, and password restoration processes.
* **Seed Deposit & Registry:** Directs physical deposit configurations, transaction logs, inventory registrations, and asset validation checks.
* **Loan Operations Subsystem:** Provides a robust financial dashboard mapping lines of credit, verifying request histories, and processing ledger details.
* **Knowledge Hub & Forums:** Pulls decentralized tutorial articles, topic categorizations, forum creation pipelines, and community interaction logs.
* **Client Configurations:** Orchestrates local terminal setting adjustments, dynamic validation tracking rules, and hardware preferences.

---

# 2. Technology Stack

| Technology | Purpose | Implementation Target |
| :--- | :--- | :--- |
| **Flutter / Dart** | Multi-platform application compiler | All layout screens, components, and target test cases |
| **HTTP Interceptor Client** | REST endpoint networking boundary | `session_service.dart`, `auth_api.dart` |
| **Shared Preferences** | Persistent disk cache synchronization | Authentication tokens, control flags, local preferences |
| **ChangeNotifier** | State mutation framework & layout broadcasts | Architecture ViewModels (`view_models/`) |
| **Material 3 / Seed** | Seed-color interface layout baseline builder | Global branding definitions, theme fields |
| **Inter & Roboto** | Global design system typography layers | Typeface asset loading configurations via `pubspec.yaml` |

## 2.1 Theme Subsystem Specs
The workspace establishes high interface uniformity using layout injectors maintained directly inside `theme.dart`:
* **Typography Anchors:** `Inter` handles primary workspace labels and high-priority copy blocks, supported by `Roboto` across localized secondary readouts.
* **Layout Scaffolding:** Scaffold backdrops implement a smooth canvas color profile built from dynamic branding seeds (`AppColors.green` and `AppColors.mint`), keeping color spaces clean and scannable.
* **Input Box Foundations:** Fields leverage filled, borderless containers with white backdrops, symmetrical vertical inner margins, and pre-registered validation errors tracking.

## 2.2 Lifecycle Status Mappings
System entities track transactional processing states across layout wrappers by utilizing fixed color design tokens established in `loan_status.dart`:

* **Pending Validation:** Warm Gold Variant (`Bg: 0xFFFFF3D6`, `Text: 0xFF8A6800`) applied to verification logs and pending queues.
* **In Progress / Active:** Light Emerald Variant (`Bg: 0xFFE7F5E7`, `Text: 0xFF1D6B4B`) for open storage allocations or ongoing workflows.
* **Finalized / Completed:** Mint Variant (`Bg: 0xFFE2F7EB`, `Text: 0xFF126B48`) confirming successful transactional steps.
* **Rejected / Declined:** Soft Crimson Variant (`Bg: 0xFFFDECEC`, `Text: 0xFFB42318`) communicating systemic or administrative drop.

---

# 3. Project Structure

The project tree decouples data structures, interface components, automation unit tests, and persistent system controllers:

```text
mbegu_link/
├── android/                             # Android wrapper project configuration layout
├── assets/
│   ├── fonts/
│   │   ├── Inter-VariableFont.ttf       # Primary typography font resource
│   │   ├── Inter_18pt-Bold.ttf          # Bold variant style parameter resource
│   │   ├── Roboto-Bold.ttf              # Secondary typeface bold variant
│   │   └── Roboto-Regular.ttf           # Secondary typeface regular weight parameter
│   └── images/
│       └── logo.png                     # Primary Begulink corporate green graphic asset
├── ios/                                 # iOS wrapper project configuration layout
├── lib/
│   ├── app.dart                         # Global MaterialApp route mapping configuration
│   ├── main.dart                        # Initial client execution orchestration entry point
│   │
│   ├── core/
│   │   ├── constants/
│   │   │   ├── app_routes.dart          # Virtual navigation string constants definitions
│   │   │   └── loan_status.dart         # Status badge capsule design hex parameters
│   │   ├── exceptions/
│   │   │   └── auth_exception.dart      # Client session failure exception mapping wrappers
│   │   ├── services/
│   │   │   ├── auth_api.dart            # Remote login authentication endpoint operations proxy
│   │   │   └── session_service.dart     # Interceptor client network layer implementation
│   │   ├── utils/
│   │   │   └── validators.dart          # Input layout parsing syntax processing validation rules
│   │   └── widgets/
│   │       ├── theme.dart               # Explicit Material 3 asset styling properties
│   │       └── widgets.dart             # Atomic interface reuse components directory
│   │
│   ├── models/
│   │   ├── activity_item.dart           # Dashboard timeline list representation parsing model
│   │   ├── knowledge.dart               # Content hub training information object attributes
│   │   ├── loan_details.dart            # Financial request ledger details parsing schema
│   │   ├── login_response.dart          # Remote credential token parsing transformation rules
│   │   ├── registry_activity.dart       # Asset registry logs payload data parsing model
│   │   ├── registry_details.dart        # Deposit configuration properties validation rules
│   │   ├── registry_item.dart           # Immutable seed asset catalog item field data model
│   │   ├── seed_deposit.dart            # Inventory storage allocation item schema rules
│   │   ├── settings.dart                # Local hardware device configuration settings model
│   │   ├── token_response.dart          # Active lifecycle secure token attribute mapping
│   │   ├── user_profile.dart            # Detailed complex individual profile attribute parsing
│   │   └── user.dart                    # Baseline user profile identity parsing objects
│   │
│   ├── screens/
│   │   ├── all_activity_screen.dart     # Aggregated global operational ledger logs grid
│   │   ├── auth_screens.dart            # Parent authentication presentation layout container
│   │   ├── deposit_seed_screen.dart     # Material seed collection input submission layout
│   │   ├── forgot_password_screen.dart  # Credential reset sequence initiation input field
│   │   ├── form_page.dart               # Generic nested dynamic data input field screen
│   │   ├── homescreen.dart              # Begulink central active system dashboard homepage
│   │   ├── knowledge_create_screen.dart # Document publication content creator template layout
│   │   ├── knowledge_detail_screen.dart # Hub article reading screen display layout
│   │   ├── knowledge_discussion_screen.dart # Interactive contextual forum feed interaction deck
│   │   ├── knowledge_screen.dart        # Grouped training documentation asset base overview
│   │   ├── loan_details_screen.dart     # Micro-loan balance verification dashboard layout
│   │   ├── login_screen.dart            # Identity verification entry module layout
│   │   ├── redirect_alert.dart          # Session interruption navigation security alert container
│   │   ├── registry_screen.dart         # Seed asset ledger inventory registry home interface
│   │   ├── registry_success_screen.dart # Completed registration step confirmation splash view
│   │   ├── request_seed_screen.dart     # External loan provisioning collection data input sheet
│   │   ├── reset_password_screen.dart   # Secure credential mutation transformation layout field
│   │   ├── settings_screen.dart         # Local platform feature control configuration terminal
│   │   ├── shell.dart                   # Global multi-module bottom navigation framework node
│   │   └── signup_screen.dart           # New user account generation attributes capture module
│   │
│   └── view_models/
│       ├── all_activity_view_model.dart # Timeline list mutation processing pipeline coordinator
│       ├── auth_view_model.dart         # Gateway token verification state tracking proxy
│       ├── deposit_seed_view_model.dart # Physical deposit submission collection controller
│       ├── edit_profile_view_model.dart # Personal profile account transformation execution loop
│       ├── knowledge_view_model.dart    # Documentation reading collections notification model
│       ├── loan_details_view_model.dart # Financial balance evaluation logic change notifier
│       ├── login_view_model.dart        # Sign-in pipeline mutation state tracking notifier
│       ├── registry_view_model.dart     # Inventory list registry parameters management context
│       ├── settings_view_model.dart     # Local operational parameter modification manager
│       ├── signup_view_model.dart       # Account creation layout event verification block
│       └── splash_view_model.dart       # Asynchronous cache memory verification checkpoint
│
└── test/                                # Automated logic test validation unit registry
Use code with caution.
├── all_activity_screen_test.dart
├── all_activity_view_model_test.dart
├── auth_screens_test.dart
├── auth_validation_test.dart
├── deposit_seed_view_model_test.dart
├── knowledge_test.dart
├── loan_details_view_model_test.dart
├── registry_test.dart
├── screens_test.dart
├── seed_deposit_validation_test.dart
├── splash_view_model_test.dart
├── viewmodels_test.dart
└── widget_test.dart
```

4. Architecture Layers

The code implements a unidirectional layer setup separating user event processing steps from immutable remote model transformation logic:
text ┌────────────────────────────────────────────────────────┐ │                   Presentation Layer                   │ │   Screens (homescreen.dart) + Utility Widgets (theme)  │ └───────────────────────────┬────────────────────────────┘ │ ▼ ┌────────────────────────────────────────────────────────┐ │                 State Management Layer                 │ │    ViewModels (registry_view_model.dart - Listeners)   │ └───────────────────────────┬────────────────────────────┘ │ ▼ ┌────────────────────────────────────────────────────────┐ │                     Service Layer                      │ │   session_service.dart (Networking & Interceptors)     │ └───────────────────────────┬────────────────────────────┘ │ ▼ ┌────────────────────────────────────────────────────────┐ │                    Data/Model Layer                    │ │   Immutable Parsing Classes (seed_deposit.dart Models) │ └────────────────────────────────────────────────────────┘ 
