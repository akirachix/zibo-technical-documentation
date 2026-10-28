# Frontend Mobile Documentation

## 1. Overview
<div class="lightbox-wrapper" style="text-align: center; margin: 1.5rem 0;">
  <img src="../assets/brand/mobile.png" alt="Mobile App Overview" style="max-width: 100%; height: auto; border-radius: 8px; cursor: zoom-in; border: 1px solid #e1e4e6;" />
  <p style="font-style: italic; margin-top: 0.5rem; color: #555;">
    Figure: Mobile App Overview. Click on the image to view it full screen.
  </p>
</div>

<div class="lightbox" id="mobileLightbox" aria-hidden="true" role="dialog" aria-modal="true">
  <button class="lightbox__close" id="mobileLightboxClose" aria-label="Close">×</button>
  <img class="lightbox__image" id="mobileLightboxImage" alt="Mobile App Overview" />
</div>

<script>
(function(){
  var img = document.querySelector('.lightbox-wrapper img');
  var lightbox = document.getElementById('mobileLightbox');
  var lightboxImage = document.getElementById('mobileLightboxImage');
  var closeBtn = document.getElementById('mobileLightboxClose');
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

<a href="https://www.figma.com/design/HzXMTo0tqeXmOuVRtQwnNr/Zibo?node-id=25-69&p=f&t=MvVSzES3zp8qodj6-0" target="_blank" rel="noopener noreferrer">View in Figma</a>

The ZIBO Mobile application (`zibo-mobile`) is a cross-platform mobile frontend developed using the Flutter framework inside the `mbegu_link` repository layer. It acts as the direct user-facing interface for the **Mbegulink** digital platform, empowering users to register seed assets, submit micro-loan queries, track processing states across an enterprise dashboard timeline, and consume localized agricultural training packages.


### Main Functional Areas

The mobile workspace manages five primary feature clusters:
* **Gateway & Identity Lifecycle:** Manages splash state validations, onboarding tokens, primary authentication matching, secure user signups, and password restoration processes.
* **Seed Deposit & Registry:** Directs physical deposit configurations, transaction logs, inventory registrations, and asset validation checks.
* **Loan Operations Subsystem:** Provides a robust financial dashboard mapping lines of credit, verifying request histories, and processing ledger details.
* **Knowledge Hub & Forums:** Pulls decentralized tutorial articles, topic categorizations, forum creation pipelines, and community interaction logs.
* **Client Configurations:** Orchestrates local terminal setting adjustments, dynamic validation tracking rules, and hardware preferences.


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


# 3. Project Structure

The project tree decouples data structures, interface components, automation unit tests, and persistent system controllers:

```text
mbegu_link/
├── android/                            
├── assets/
│   ├── fonts/
│   │   ├── Inter-VariableFont.ttf      
│   │   ├── Inter_18pt-Bold.ttf                    
│   └── images/
│       └── logo.png                     
├── ios/                                 
├── lib/
│   ├── app.dart                        
│   ├── main.dart                       
│   │
│   ├── core/
│   │   ├── constants/
│   │   │   ├── app_routes.dart          
│   │   │          
│   │   ├── exceptions/
│   │   │   └── auth_exception.dart      
│   │   ├── services/
│   │   │   ├── auth_api.dart            
│   │   │   └── session_service.dart    
│   │   ├── utils/
│   │   │   └── validators.dart         
│   │   └── widgets/
│   │       ├── theme.dart               
│   │       └── widgets.dart             
│   │
│   ├── models/
│   │   ├── activity_item.dart         
│   │   ├── knowledge.dart                               
│   │
│   ├── screens/
│   │   ├── all_activity_screen.dart    
│   │   ├── auth_screens.dart                     
│   │
│   └── view_models/
│       ├── all_activity_view_model.dart       
│
└── test/                                
Use code with caution.
├── all_activity_screen_test.dart

```

## 4. Architecture Layers

The mobile and web applications use a clean, layered structure to separate user interface elements from data storage logic. Data flows downward from screens to database models, with network tasks handled securely in the middle.

```text
┌────────────────────────────────────────────────────────┐
│                   Presentation Layer                   │
│   Screens (homescreen.dart) and style tokens (theme)   │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│                 State Management Layer                 │
│   ViewModels (registry_view_model.dart) listeners      │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│                     Service Layer                      │
│   session_service.dart handling network requests      │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│                    Data / Model Layer                  │
│   Data parsing classes (seed_deposit.dart structures)  │
└────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

*   **Presentation Layer:** Displays information to the user and captures form selections. It contains no business logic.
*   **State Management Layer:** Acts as the middleman. It listens to user actions on screens and coordinates database changes.
*   **Service Layer:** Handles HTTP networking tasks, API token injections, and connection timeouts.
*   **Data/Model Layer:** Maps incoming JSON server responses directly into structured data shapes.

<div class="mobile-lightbox-wrapper" style="text-align: center; margin: 1.5rem 0;">
  <img src="../assets/brand/mobile.png" alt="Mobile App Overview" id="mobileTriggerImg" style="max-width: 100%; height: auto; border-radius: 8px; cursor: zoom-in; border: 1px solid #e1e4e6;" />
  <p style="font-style: italic; margin-top: 0.5rem; color: #555;">
    Figure: Mobile App Overview. Different screen, and different colours.
  </p>
</div>

<div class="lightbox" id="mobileLightbox" aria-hidden="true" role="dialog" aria-modal="true">
  <button class="lightbox__close" id="mobileLightboxClose" aria-label="Close">×</button>
  <img class="lightbox__image" id="mobileLightboxImage" alt="Mobile App Overview" />
</div>

<script>
(function(){
  var img = document.getElementById('mobileTriggerImg');
  var lightbox = document.getElementById('mobileLightbox');
  var lightboxImage = document.getElementById('mobileLightboxImage');
  var closeBtn = document.getElementById('mobileLightboxClose');
  
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
