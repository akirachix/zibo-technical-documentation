# Frontend Web Documentation

<div style="text-align: center; margin: 2rem auto; max-width: 1050px; width: 100%;">
  <div class="lightbox-wrapper" style="text-align: center;">
    <img src="../assets/brand/pwa.png" alt="PWA Overview" style="width: 100%; height: auto; display: block; margin: 0 auto; cursor: zoom-in;" />
  </div>
</div>

<div class="lightbox" id="pwaLightbox" aria-hidden="true" role="dialog" aria-modal="true">
  <button class="lightbox__close" id="pwaLightboxClose" aria-label="Close">×</button>
  <img class="lightbox__image" id="pwaLightboxImage" alt="PWA Overview" />
</div>

<script>
(function(){
  var img = document.querySelector('.lightbox-wrapper img');
  var lightbox = document.getElementById('pwaLightbox');
  var lightboxImage = document.getElementById('pwaLightboxImage');
  var closeBtn = document.getElementById('pwaLightboxClose');
  if(!img || !lightbox) return;
  function openLightbox(){
    lightboxImage.src = img.src;
    lightboxImage.alt = img.alt;
    lightbox.classList.add('open');
    lightbox.setAttribute('aria-hidden','false');
    document.body.style.overflow = 'hidden';
  }
  function closeLightbox(){
    lightbox.classList.remove('open');
    lightbox.setAttribute('aria-hidden','true');
    document.body.style.overflow = '';
  }
  img.addEventListener('click', openLightbox);
  closeBtn.addEventListener('click', closeLightbox);
  lightbox.addEventListener('click', function(e){
    if(e.target === lightbox) closeLightbox();
  });
  document.addEventListener('keydown', function(e){
    if(e.key === 'Escape' && lightbox.classList.contains('open')) closeLightbox();
  });
})();
</script>

## 01 · Overview

The **Mbegu Link Frontend Web Application (`zibo-frontend`)** is a Next.js application built with React and Tailwind CSS. It provides the web interface for the Mbegu Link digital agriculture network, connecting community seed banks with institutional conservation specialists.

The application supports role-based navigation, directing authenticated users to workspaces aligned with their operational responsibilities. Its responsive interface accommodates institutional desktop workflows and community-facing mobile layouts.

<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(250px,1fr));gap:18px;margin:24px 0;">

<div style="border:1px solid #D7E7DA;border-top:4px solid #237A43;border-radius:12px;padding:24px;background:#FFFFFF;">
  <div style="font-size:12px;color:#237A43;font-weight:700;letter-spacing:1.5px;">01 / INSTITUTIONAL</div>
  <h3 style="color:#185C32;margin:12px 0;">GeRRI Specialist Portal</h3>
  <p style="color:#4B6351;line-height:1.8;">A desktop-oriented workspace for record verification, submission reviews, conservation information, analytical dashboards, and administrative workflows.</p>
</div>

<div style="border:1px solid #D7E7DA;border-top:4px solid #68A678;border-radius:12px;padding:24px;background:#F5FAF5;">
  <div style="font-size:12px;color:#237A43;font-weight:700;letter-spacing:1.5px;">02 / COMMUNITY</div>
  <h3 style="color:#185C32;margin:12px 0;">CSB Coordinator Portal</h3>
  <p style="color:#4B6351;line-height:1.8;">A responsive, mobile-first Progressive Web App experience for managing seed records, submitting loan requests, coordinating operations, and viewing notifications.</p>
</div>

</div>

The frontend communicates with the Mbegu Link backend through REST API requests over HTTP. It renders information returned by backend services and coordinates interface updates through shared application state and workflow-specific components.

### Main Functional Areas

| Functional Area | Core Responsibilities |
|:---|:---|
| **Role-Based Authentication** | Login, role-specific routing, session management, and password recovery workflows. |
| **GeRRI Specialist Workspace** | Record verification, submission reviews, conservation information, and administrative operations. |
| **CSB Coordinator Workspace** | Seed registration, deposit records, seed loans, and community operations. |
| **Notifications** | Notification messages, workflow updates, and unread indicators. |
| **Profile and Settings** | User profile management, appearance preferences, and account settings. |

---

## 02 · Technology Stack

The frontend uses a modular architecture that separates application routing, presentation, shared state, and backend communication.

<div style="border:1px solid #D7E7DA;border-radius:12px;overflow:hidden;margin:20px 0;">
<table style="width:100%;border-collapse:collapse;margin:0;">
<thead>
<tr style="background:#1F6B3B;color:white;">
<th style="padding:14px;text-align:left;">Technology</th>
<th style="padding:14px;text-align:left;">Purpose</th>
<th style="padding:14px;text-align:left;">Implementation</th>
</tr>
</thead>
<tbody>
<tr>
<td style="padding:12px;border-bottom:1px solid #E4EDE5;"><strong>Next.js / React</strong></td>
<td style="padding:12px;border-bottom:1px solid #E4EDE5;">Application framework, routing, and reusable interface components.</td>
<td style="padding:12px;border-bottom:1px solid #E4EDE5;"><code>src/app/</code>, <code>src/components/</code></td>
</tr>
<tr style="background:#F5FAF5;">
<td style="padding:12px;border-bottom:1px solid #E4EDE5;"><strong>HTTP Client</strong></td>
<td style="padding:12px;border-bottom:1px solid #E4EDE5;">REST API communication and request configuration.</td>
<td style="padding:12px;border-bottom:1px solid #E4EDE5;"><code>src/lib/session.js</code>, <code>src/lib/authApi.js</code></td>
</tr>
<tr>
<td style="padding:12px;border-bottom:1px solid #E4EDE5;"><strong>React Context</strong></td>
<td style="padding:12px;border-bottom:1px solid #E4EDE5;">Shared application state and cross-component updates.</td>
<td style="padding:12px;border-bottom:1px solid #E4EDE5;"><code>src/lib/portalStore.js</code></td>
</tr>
<tr style="background:#F5FAF5;">
<td style="padding:12px;border-bottom:1px solid #E4EDE5;"><strong>Tailwind CSS</strong></td>
<td style="padding:12px;border-bottom:1px solid #E4EDE5;">Responsive layouts and utility-based styling.</td>
<td style="padding:12px;border-bottom:1px solid #E4EDE5;"><code>tailwind.config.js</code></td>
</tr>
<tr>
<td style="padding:12px;border-bottom:1px solid #E4EDE5;"><strong>Inter / Roboto</strong></td>
<td style="padding:12px;border-bottom:1px solid #E4EDE5;">Typography and interface hierarchy.</td>
<td style="padding:12px;border-bottom:1px solid #E4EDE5;">Font configuration</td>
</tr>
<tr style="background:#F5FAF5;">
<td style="padding:12px;"><strong>Jest</strong></td>
<td style="padding:12px;">Automated testing of application behaviour and components.</td>
<td style="padding:12px;">Test suite and configuration</td>
</tr>
</tbody>
</table>
</div>

### 2.1 · Visual Appearance and Design System

The interface uses a green-and-white visual identity inspired by Mbegu Link's focus on agriculture, conservation, and community collaboration.

<div style="display:flex;flex-wrap:wrap;gap:12px;margin:22px 0;">

<div style="flex:1;min-width:125px;border-radius:10px;overflow:hidden;border:1px solid #D7E7DA;">
<div style="height:65px;background:#185C32;"></div>
<div style="padding:12px;background:#FFFFFF;"><strong>Forest Green</strong><br><code>#185C32</code></div>
</div>

<div style="flex:1;min-width:125px;border-radius:10px;overflow:hidden;border:1px solid #D7E7DA;">
<div style="height:65px;background:#237A43;"></div>
<div style="padding:12px;background:#FFFFFF;"><strong>Brand Green</strong><br><code>#237A43</code></div>
</div>

<div style="flex:1;min-width:125px;border-radius:10px;overflow:hidden;border:1px solid #D7E7DA;">
<div style="height:65px;background:#E7F2E8;"></div>
<div style="padding:12px;background:#FFFFFF;"><strong>Soft Green</strong><br><code>#E7F2E8</code></div>
</div>

<div style="flex:1;min-width:125px;border-radius:10px;overflow:hidden;border:1px solid #D7E7DA;">
<div style="height:65px;background:#FFFFFF;"></div>
<div style="padding:12px;background:#FFFFFF;"><strong>White</strong><br><code>#FFFFFF</code></div>
</div>

</div>

#### Typography

- **Inter:** Primary typography for workspace labels, page headings, and body text.
- **Roboto:** Supporting typography for selected headings and analytical information.

#### Light and Dark Modes

The appearance system supports theme consistency across dashboard layouts, cards, forms, navigation, and notifications. Light and dark modes are intended to preserve readable text, clear visual hierarchy, and consistent component styling.

The documented theme configuration is located in `src/lib/theme.js`.

#### Input Components and Accessibility

Form controls use consistent labels, spacing, borders, and validation feedback. Associated `htmlFor` and `id` attributes help users identify form controls and support assistive technologies.

### 2.2 · Lifecycle Status and Visual Indicators

Workflow statuses use distinct colors to communicate the state of seed records, loans, deposits, and verification processes.

<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(210px,1fr));gap:14px;margin:20px 0;">

<div style="background:#FFF3D6;border:1px solid #F0DFA9;border-radius:10px;padding:18px;">
<div style="font-size:11px;font-weight:700;letter-spacing:1px;color:#8A6800;">PENDING</div>
<div style="font-size:17px;font-weight:700;color:#8A6800;margin:8px 0;">Pending Verification</div>
<div style="font-size:12px;color:#8A6800;">Background: #FFF3D6</div>
</div>

<div style="background:#E7F5E7;border:1px solid #C8E5CA;border-radius:10px;padding:18px;">
<div style="font-size:11px;font-weight:700;letter-spacing:1px;color:#1D6B4B;">ACTIVE</div>
<div style="font-size:17px;font-weight:700;color:#1D6B4B;margin:8px 0;">In Progress</div>
<div style="font-size:12px;color:#1D6B4B;">Background: #E7F5E7</div>
</div>

<div style="background:#E2F7EB;border:1px solid #B8E7CC;border-radius:10px;padding:18px;">
<div style="font-size:11px;font-weight:700;letter-spacing:1px;color:#126B48;">COMPLETED</div>
<div style="font-size:17px;font-weight:700;color:#126B48;margin:8px 0;">Successfully Completed</div>
<div style="font-size:12px;color:#126B48;">Background: #E2F7EB</div>
</div>

<div style="background:#FDECEC;border:1px solid #F4C7C7;border-radius:10px;padding:18px;">
<div style="font-size:11px;font-weight:700;letter-spacing:1px;color:#B42318;">DECLINED</div>
<div style="font-size:17px;font-weight:700;color:#B42318;margin:8px 0;">Rejected / Declined</div>
<div style="font-size:12px;color:#B42318;">Background: #FDECEC</div>
</div>

</div>

The documented status configuration is `src/lib/loanStatus.js`.

| Status | Background | Text |
|:---|:---|:---|
| Pending Verification | `#FFF3D6` | `#8A6800` |
| In Progress / Active | `#E7F5E7` | `#1D6B4B` |
| Completed | `#E2F7EB` | `#126B48` |
| Rejected / Declined | `#FDECEC` | `#B42318` |

---

## 03 · Project Structure

The `zibo-frontend` repository separates application routes, reusable components, shared utilities, configuration, assets, and automated tests.

```text
zibo-frontend/
├── .husky/
├── docs/
│   └── openapispec.json
├── public/
│   ├── brand/
│   │   └── logo.png
│   ├── icon-192.png
│   └── icon-512.png
├── src/
│   ├── app/
│   │   ├── notifications/
│   │   │   └── page.jsx
│   │   ├── profile/
│   │   │   └── page.jsx
│   │   ├── settings/
│   │   │   └── page.jsx
│   │   ├── layout.jsx
│   │   └── page.jsx
│   ├── components/
│   │   ├── auth/
│   │   │   ├── AuthShell.jsx
│   │   │   ├── ForgotPasswordForm.jsx
│   │   │   ├── LoginForm.jsx
│   │   │   ├── RoleSelector.jsx
│   │   │   └── TextField.jsx
│   │   ├── brand/
│   │   │   └── Logo.jsx
│   │   ├── onboarding/
│   │   │   └── OnboardingSlide.jsx
│   │   ├── portal/
│   │   │   ├── DashboardLayout.jsx
│   │   │   ├── NotificationBell.jsx
│   │   │   ├── PwaShell.jsx
│   │   │   ├── SignOutPopup.jsx
│   │   │   └── StatusPill.jsx
│   │   └── splash/
│   │       └── LandingSplash.jsx
│   ├── lib/
│   │   ├── api.js
│   │   ├── authApi.js
│   │   ├── config.js
│   │   ├── devAccount.js
│   │   ├── directory.js
│   │   ├── notifications.js
│   │   ├── password.js
│   │   ├── portalApi.js
│   │   ├── portalStore.js
│   │   ├── session.js
│   │   ├── sessionTools.js
│   │   ├── theme.js
│   │   └── useApi.js
│   └── __tests__/
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
├── package.json
└── tailwind.config.js
```

*Note: This tree documents the files represented in the provided project structure. Additional files may exist in the repository.*

### Directory Responsibilities

| Directory / File | Responsibility |
|:---|:---|
| `src/app/` | Application routes and shared layouts. |
| `src/components/auth/` | Authentication forms and related components. |
| `src/components/brand/` | Branding and logo components. |
| `src/components/onboarding/` | Onboarding interface components. |
| `src/components/portal/` | Dashboard layouts, notifications, and status indicators. |
| `src/components/splash/` | Application initialization and splash interface. |
| `src/lib/api.js` | Shared API communication. |
| `src/lib/authApi.js` | Authentication API operations. |
| `src/lib/portalApi.js` | Portal API operations. |
| `src/lib/portalStore.js` | Shared portal state. |
| `src/lib/session.js` | HTTP client and session request configuration. |
| `src/lib/theme.js` | Theme and appearance configuration. |
| `src/__tests__/` | Automated application tests. |
| `package.json` | Project dependencies and scripts. |
| `tailwind.config.js` | Tailwind CSS configuration. |

---

## 04 · Architecture Layers

The frontend separates presentation, state management, service communication, and backend responsibilities. This separation helps keep interface components focused on rendering and user interaction while API-related operations are handled through shared service utilities.

<div style="border:1px solid #D7E7DA;border-radius:14px;padding:24px;background:#F8FBF8;margin:24px 0;">

<div style="background:#185C32;color:white;text-align:center;padding:20px;border-radius:10px;">
<div style="font-size:11px;letter-spacing:2px;opacity:.8;">LAYER 01</div>
<div style="font-size:19px;font-weight:700;margin:6px 0;">Presentation Layer</div>
<div style="font-size:13px;color:#E5F5E8;">Next.js · React · Application Views</div>
</div>

<div style="text-align:center;font-size:24px;color:#47845E;padding:8px;">↕</div>

<div style="background:#E7F2E8;border:1px solid #C8DFCC;color:#185C32;text-align:center;padding:20px;border-radius:10px;">
<div style="font-size:11px;letter-spacing:2px;">LAYER 02</div>
<div style="font-size:19px;font-weight:700;margin:6px 0;">State Management</div>
<div style="font-size:13px;">React Context · Hooks · portalStore.js</div>
</div>

<div style="text-align:center;font-size:24px;color:#47845E;padding:8px;">↕</div>

<div style="background:#E7F2E8;border:1px solid #C8DFCC;color:#185C32;text-align:center;padding:20px;border-radius:10px;">
<div style="font-size:11px;letter-spacing:2px;">LAYER 03</div>
<div style="font-size:19px;font-weight:700;margin:6px 0;">Service Layer</div>
<div style="font-size:13px;">HTTP Client · session.js · portalApi.js</div>
</div>

<div style="text-align:center;font-size:24px;color:#47845E;padding:8px;">↕</div>

<div style="background:#FFFFFF;border:2px solid #237A43;color:#185C32;text-align:center;padding:20px;border-radius:10px;">
<div style="font-size:11px;letter-spacing:2px;">LAYER 04</div>
<div style="font-size:19px;font-weight:700;margin:6px 0;">Mbegu Link Backend</div>
<div style="font-size:13px;">REST API · Business Logic · Data Persistence</div>
</div>

</div>

### Presentation Layer

Responsible for rendering pages, dashboard layouts, authentication forms, reusable components, and responsive interfaces.

### State Management Layer

Coordinates shared portal state, interface updates, notification indicators, and application context through React Context and related hooks.

### Service Layer

Handles API requests, authentication headers, response processing, and communication with backend endpoints.

### Backend and Data Layer

The backend is responsible for business rules, data persistence, and processing requests received from the frontend. The frontend consumes data through the API contracts and displays information such as seed records, workflow statuses, and user details.

---

## 05 · Testing

Automated tests help verify the behaviour of authentication components, application views, session utilities, API-related functionality, and portal workflows.

The documented test suite includes:

- Authentication form tests.
- Login and landing-page tests.
- Onboarding component tests.
- Password-screen tests.
- Portal and notification tests.
- Session and API utility tests.
- OpenAPI specification tests.

Tests should be run using the scripts configured in `package.json`. The exact commands depend on the repository's current test configuration.

---

<div style="margin-top:36px;padding:24px;background:#185C32;border-radius:12px;color:#FFFFFF;text-align:center;">
<div style="font-size:12px;letter-spacing:2px;color:#BFE3C7;">ZIBO · MBEGU LINK</div>
<div style="font-size:20px;font-weight:700;margin:10px 0;">Technology Supporting Seed Conservation</div>
<div style="font-size:13px;line-height:1.8;color:#E5F5E8;">A connected frontend experience for community seed banks and institutional conservation partners.</div>
</div>