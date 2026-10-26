# Security and Privacy

## User Onboarding and Authorisation

Mbegu Link never creates a trusted account with a single sign up click. Onboarding is a deliberate, multi step process that ends with each user holding exactly one role, tied to one verified organisation, with access denied by default until that role is granted. This design exists because the platform's data is communal: a wrong or inflated role does not just expose one person's data it exposes a whole Community Seed Bank's collection and its farmers' identities.

**Step 1: Organisation verification comes before any user.** Before a CSB can onboard anyone, the CSB itself is verified by the Mbegu Link team: the organisation profile is confirmed (against the registry of known Kenyan CSBs), and only then is the organisation marked active in the system. An unverified organisation cannot activate any user, no matter who signs up. This makes the CSB not the individual the root of trust in the platform.

**Step 2: Role-specific onboarding paths.** Each of the three roles is onboarded through the path that matches how that role exists in real life:

- **Seed Custodians (mobile app):** a custodian joins with an invitation code issued by their CSB, then verifies their phone number with a one time OTP. The invitation code binds the account to that specific CSB a code for one CSB cannot activate a user in another and the phone OTP proves the person holds the number they registered. Custodian accounts start with the least privilege in the system: they can register seed, record observations, request loans and post knowledge, but they cannot approve transfers, change thresholds or see other CSBs' records.
- **CSB Coordinators (PWA):** coordinators are registered by the Mbegu Link team after the CSB confirms who the coordinator is. They log in with email and a bcrypt-hashed password. The coordinator role carries the highest community-facing authority master registry management, germination test recording, threshold configuration, transfer approval and support requests to GeRRI so its issuance is deliberately kept out of self-service.
- **GeRRI Specialists (PWA):** specialists are provisioned with verified GeRRI credentials and assigned roles by the team. Their authorisation is structurally limited regardless of seniority: a specialist sees only the seed records a CSB has explicitly linked to a support request, and nothing else from that CSB's collection.

**Step 3: Least privilege and default denial.** Every account has exactly one role; there are no overlapping or combined roles in the MVP. Authorisation is enforced server-side on every request: the API resolves the user's role and organisation, then checks permission against the specific record being touched before any business logic runs. Access is denied by default a new record or knowledge entry is private until someone with the authority deliberately shares it. Permission denials are themselves logged in the audit trail, so repeated denied attempts by any account are visible.

**Step 4: Session and credential hygiene after onboarding.** Sessions use short lived tokens with HttpOnly and Secure cookies and expire after ten minutes of inactivity; logout and password changes invalidate all sessions. Accounts lock temporarily after three failed login attempts, password reset uses single-use email codes, and rate limiting protects both the login and reset endpoints. Secrets and API keys exist only in backend environment variables no role, however senior, receives credentials that could bypass the server-side checks.

## Knowledge Hub Media Security and Content Tracking

The Knowledge Hub lets seed custodians post photos and uniquely for the MVP short videos related to agriculture: how a variety is grown, stored, prepared or saved. Video raises a security problem the rest of the platform does not have: a video can capture faces, voices, farms, exact locations and indigenous knowledge that was never meant to leave the community. The architecture therefore treats every uploaded video as sensitive data by default, and tracks it through its whole lifecycle upload, validation, moderation, visibility change and withdrawal so that nothing becomes public or stays public by accident.

**Upload handling.** Custodians attach media to a knowledge entry from the mobile app. Files are never embedded in the request that creates the entry: the upload is queued until a connection is available, and the file goes directly to dedicated media storage, where the database keeps only its link (the `media_file` table with `storage_url`, `file_type`, `media_size` and `upload_status`). This separation means an uploaded file can never be executed as code inside the application, and a database compromise does not expose the media itself.

**Validation at the boundary.** Every upload is checked server-side for file type and size before it is accepted. Only expected media formats are allowed; an executable or disguised file is rejected, and a failed or interrupted upload is recorded with `upload_status = failed` so it can be retried rather than silently published incomplete. Oversized files are rejected to protect the storage and the coordinator's review workflow from denial-of-service-style abuse.

**Moderation and tracking for security.** Every knowledge entry including each attached video is tracked from creation onward:

- Every entry records **who contributed it**, and the contributor chooses to be credited or anonymous. Anonymous does not mean untracked: the author's identity is always known to the system and to the CSB's approver, even when it is not displayed publicly.
- Making an entry **public is a deliberate, audited step**: a warning explains that public copies cannot be recalled, a named approver in the CSB confirms the publication, and a consent record is written. The publication event lands in the append only audit trail who approved, what was approved, and when so any later dispute about a video's release has an authoritative answer.
- **Sensitive content is flagged before publishing:** entries that name farmers, describe community specific or ritual knowledge, or reveal exact collection locations are surfaced for review, because a video showing a farm's surroundings can reveal more than its author intended.
- **Visibility can be revoked at any time.** The CSB can edit, restrict or withdraw its own entries; withdrawal removes the entry from the hub, and the withdrawal action is itself logged. The warning about irrecoverable copies is shown at publication time precisely because the system cannot undo what someone else has already downloaded.

**Storage and access control of media.** Media objects are private in storage: clients receive short-lived, permission-checked links rather than permanent public URLs. Access follows the entry's visibility level (private by default; organisation; public only after approval), and the same role-based rules, audit logging and revocation apply to media as to text entries. Public knowledge sits in a separate data space from seed-record data, so publishing a video guide never exposes the underlying seed holding, custodian identity or test history.

**Why this matters for Mbegu Link specifically.** The research that produced this platform found that community knowledge storage techniques, local recipes, seed-saving practices lives with individual custodians and elders and is lost when they move on. The Hub exists to preserve it, but only on the community's terms. Tracking every video through consent, approval, audit and withdrawal is what makes custodians safe to contribute: the security design is not a barrier to sharing knowledge, it is the reason the community can trust the Hub with it.

## Security Architecture Overview

Security in Mbegu Link is designed around one principle: community control of seed information is an architectural requirement, not an interface setting. Seed records can contain farmer identities, collection locations and indigenous knowledge, so the system is built so that hiding something in the user interface is never the only protection every permission is enforced on the server, before business logic runs.

The security model rests on the system's layered structure. The mobile app and the Progressive Web App (PWA) are the only two entry points, and both communicate exclusively over HTTPS with the FastAPI backend. No client ever connects to the database directly. This single rule is what makes authentication, role-based authorisation, multi-tenant isolation and audit logging enforceable in one place: the backend is the security boundary for the entire system.

## Layered Security Architecture Diagram

<div class="lightbox-wrapper" style="text-align: center; margin: 1.5rem 0;">
  <img src="../assets/brand/ssad.png" alt="Mbegu Link Security SAD" style="max-width: 100%; height: auto; border-radius: 8px; cursor: zoom-in; border: 1px solid #e1e4e6;" />
  <p style="font-style: italic; margin-top: 0.5rem; color: #555;">
    Figure: Security Architecture Diagram (SSAD). Click on the image to view it full screen. Source: Figma, Mbegu Link Cybersecurity SAD board.
  </p>
</div>

<div class="lightbox" id="ssadLightbox" aria-hidden="true" role="dialog" aria-modal="true">
  <button class="lightbox__close" id="ssadLightboxClose" aria-label="Close">×</button>
  <img class="lightbox__image" id="ssadLightboxImage" alt="Mbegu Link Security SAD" />
</div>

<script>
(function(){
  var img = document.querySelector('.lightbox-wrapper img');
  var lightbox = document.getElementById('ssadLightbox');
  var lightboxImage = document.getElementById('ssadLightboxImage');
  var closeBtn = document.getElementById('ssadLightboxClose');
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

The diagram shows how each layer contributes:

| Layer | Security responsibility |
|---|---|
| **Clients** (Mobile app, PWA) | Collect credentials securely, never store secrets, display errors without technical detail |
| **API layer** (FastAPI) | Authentication, role-based authorisation, input validation, rate limiting, audit writing, secure headers |
| **Data layer** (PostgreSQL, media storage) | Encryption at rest, append-only audit and event tables, database role separation |
| **Infrastructure** (cloud hosting) | TLS termination, environment separation, encrypted separated backups, restricted administrator access |

## Authentication and Access Control

Every user belongs to exactly one organisation, and every account has exactly one role: Seed Custodian, CSB Coordinator, or GeRRI Specialist. Roles decide what a user can see and do, and access is denied by default every request is checked for role and record level permission before it is processed. This gives the system its multi-tenant isolation: one CSB cannot query another CSB's holdings, and a GeRRI specialist can open only the seed records a CSB has explicitly linked to a support request.

Account verification follows each role's real onboarding path:

- **Seed custodians** join with a CSB invitation code and verify their phone number with an OTP. This ties every custodian account to a real, verified Community Seed Bank before it can be activated.
- **CSB Coordinators and GeRRI Specialists** are registered by the Mbegu Link team and log in to the PWA with email and password. An organisation must be verified before its users can be activated.

Passwords are stored only as salted bcrypt hashes and are never logged or displayed. Accounts are temporarily locked after three repeated failed login attempts, password reset uses single-use email codes, and sessions use short-lived tokens with HttpOnly and Secure cookies, expiring after ten minutes of inactivity. Logout and password changes invalidate all active sessions.

## Data Protection and Privacy by Design

Data protection is built into the data model itself, following the Kenya Data Protection Act, 2019:

- **Encryption:** All traffic uses TLS 1.2 or higher. Personal data names, phone numbers, email addresses, consent records and backups are encrypted at rest and in transit.
- **Data minimisation:** The system collects only what its functions need. Device coordinates are used only to calculate nearby CSBs and are never stored.
- **Private by default:** New records and knowledge entries are private by default. Sharing seed records with GeRRI requires consent, and making a knowledge entry public requires a deliberate, confirmed step with a named approver.
- **Separation of data spaces:** Public knowledge is stored separately from private seed-record data, so publishing a guide never exposes an underlying holding.
- **Retention and deletion:** Security logs are retained for 12 months, backups follow a 30-day rolling cycle, and approved deletion requests replace identifying names on retained community records with neutral labels.
- **Cross-border transfers:** Handled where possible by hosting in-region; where processing happens outside Kenya, compliance with the Data Protection Act, 2019 is verified and documented.

## Audit Trail, Threats and Controls

The append-only `audit_record` table is the system's memory of who did what. The backend writes an audit row for every security-relevant action: sign-ins, failed sign-ins, invitation-code use, permission denials, sharing changes, exports, role changes and seed transfers. Rows are never updated or deleted, the application database role holds INSERT and SELECT rights only on this table, and logs never contain passwords, tokens, reset codes or message contents.

The main threats identified in our risk analysis, and the controls that answer them:

| Threat | Required control |
|---|---|
| **Unauthorized access** | Server side role and record-level authorisation; deny by default |
| **Data disclosure** | Encryption in transit and at rest, location masking, public/private separation |
| **Account compromise** | bcrypt hashing, account lockout, single-use reset codes, rate limiting, generic login errors |
| **Injection and XSS** | Server-side validation, parameterised queries, output encoding, Content Security Policy |
| **Malicious uploads** | File type and size validation; media stored outside executable code |
| **Record or flag tampering** | Append-only history, record level permissions, per-record change attribution |
| **Denial of Service** | Rate limiting, monitoring and alerts |
| **Data loss** | Daily encrypted backups stored separately, restore tested before the pilot and quarterly, recovery within 24 hours |

Error handling supports the same goal: users receive short, specific messages, and responses never expose database errors, file paths or software versions. Secrets and API keys live only in environment variables on the backend; debug mode and default accounts are disabled in production; and development, testing and production environments are separated.

Security is verified, not assumed. Before deployment the team completes source code review, dependency and static/dynamic security scanning, API security testing, privacy review, SBOM generation, offline and synchronisation testing, and backup-restore testing. Together with the 2021 Protocol for Collaboration's Prior Informed Consent and Community Data Sharing Agreements, these controls ensure that community seed information stays under community control at every layer of the system.
