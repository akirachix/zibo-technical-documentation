# Architecture

This page shows the system structure: the components, how data flows between them, and the rules that keep the design clean. For how to run the system, see [Getting Started](getting-started.md); for code conventions, see [Backend](backend.md).

## System architecture diagram

<div class="lightbox-wrapper" style="text-align: center; margin: 1.5rem 0;">
  <img src="../assets/brand/sad.png" alt="Mbegu Link SAD" style="max-width: 100%; height: auto; border-radius: 8px; cursor: zoom-in; border: 1px solid #e1e4e6;" />
  <p style="font-style: italic; margin-top: 0.5rem; color: #555;">
    Figure: System Architecture Diagram (SAD). <a href="https://lucid.app/lucidspark/be73d626-2a49-4f89-9715-48bca0a7fc51/edit?viewport_loc=-2948%2C-1715%2C6845%2C3892%2C0_0&invitationId=inv_18293e0f-b177-405a-8bb2-49204be1bce8" target="_blank" rel="noopener noreferrer">View SAD in Lucidchart</a>
  </p>
</div>

<div class="lightbox" id="sadLightbox" aria-hidden="true" role="dialog" aria-modal="true">
  <button class="lightbox__close" id="sadLightboxClose" aria-label="Close">×</button>
  <img class="lightbox__image" id="sadLightboxImage" alt="Mbegu Link SAD" />
</div>

<script>
(function(){
  var img = document.querySelector('.lightbox-wrapper img');
  var lightbox = document.getElementById('sadLightbox');
  var lightboxImage = document.getElementById('sadLightboxImage');
  var closeBtn = document.getElementById('sadLightboxClose');
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

## Data model

| Entity | Description | Relationships |
|---|---|---|
| **variety** | Each seed variety once, with local and scientific names (unknown accepted) | 1→many holdings and knowledge entries |
| **holding** | The core seed record: a quantity of one variety held by one custodian or CSB, carrying the Mbegu Link ID, source provenance and parent-holding link for lot lineage | Links to all events, transfers and requests |
| **event** | Append only dated history: germination tests, storage observations, regeneration, safety duplication; corrections are new rows referencing the original | Belongs to one holding |
| **transfer** | Seed movement between custodians or organisations, with custody rules and exception notes | Belongs to one holding |
| **request** | Structured loan, storage and conservation-support requests with lifecycle status | Links requester, holding and target organisation |
| **knowledge_entry** | Practical knowledge with private (default) or public visibility, tags and archive flag | Belongs to one organisation; may link one variety |
| **media_file** | References to photos and videos held in media storage; upload queued until a connection is available | Belongs to one knowledge entry |
| **audit_record** | Append-only trail of system transactions and structural access data | Managed globally at server execution layer 
Link to Data models — <a href="https://docs.google.com/document/d/1NR96Tagi8l5oYZitYwxOw4wsyQlLBHSkolxf_zupkC4/edit?usp=sharing" target="_blank" rel="noopener noreferrer">View Data Models in Google Doc</a>

## Data flow

1. **Custodian → Mobile → API** — signs up with phone number and CSB invitation code, confirmed by OTP sent through Africa's Talking; registers seed, submits loan and deposit requests, posts and reads Knowledge Hub content.
2. **Coordinator → PWA → API** — registers seed, records germination tests, reviews loan/transfer/deposit requests, sends support requests.
3. **Specialist → PWA → API** — receives support requests, submits responses, coordinates safety duplication; sees only seed records shared to GeRRI.
4. **Admin → onboarding script → API** — creates an organisation with its coordinator; the organisation is stored as verified.
5. **API → storage** — seed, event, request and knowledge data goes to PostgreSQL; files go to Media Storage with only a reference stored in the database; OTP messages go out through Africa's Talking.

## Layered view

```text
┌─────────────────────────────────────────────┐
│ Clients     Mobile (Flutter)   PWA (web)    │
├─────────────────────────────────────────────┤
│ Application FastAPI — routers, services,    │
│             repositories, models, schemas   │
├─────────────────────────────────────────────┤
│ Data        PostgreSQL      Media Storage   │
├─────────────────────────────────────────────┤
│ External    Africa's Talking    Email      │
└─────────────────────────────────────────────┘
```

### Responsibilities by layer:

| Layer | Responsibility | What it never does |
|---|---|---|
| **Clients** | Present data, capture field input | Contain business rules |
| **API** | Validate, apply rules, persist, audit | Serve files or render UI |
| **Data** | Store and retrieve | Decide anything |

---

## Deployment Architecture

- **Backend service:** a single FastAPI application, containerised, serving both PWA clients and the mobile app through one HTTPS endpoint. Statelessness at the API layer means it can scale horizontally behind a load balancer as CSB adoption grows.
- **Database:** one PostgreSQL instance with role separation — the application role has INSERT and SELECT rights only on audit_record, so no application code path can alter the audit trail. Backups run on a schedule with tested restore procedures.
- **Media storage:** a separate bucket with private objects; clients receive short-lived, permission-checked links, never permanent public URLs.
- **Clients:** the PWA is served as static assets; the mobile app is distributed as an installable build. Neither embeds secrets — all privileged operations happen server-side.

## Mobile Connectivity and Failure Behaviour

The MVP is designed for online use, but the network in CSB communities is not reliable, so the architecture fails safely rather than assuming connectivity:

- **Queued media uploads:** when a custodian attaches a photo or video without a connection, the mobile app queues the upload and retries until it succeeds; the media_file row exists with upload_status = pending until the file lands in storage.
- **Concurrent edits:** two users editing the same record, or submitting the same request twice, produce a conflict that is shown to a person — the server never silently overwrites one edit with another.
- **Structured errors:** every failure returns a defined error the client can display, so a failed transfer or rejected request is visible to the user, not lost.
- **Interrupted uploads:** a failed or interrupted photo/video upload is recorded as failed and can be retried, so a knowledge entry is never published with a missing attachment unnoticed.
---

## Architectural boundaries

The rules the design depends on:
*   The PWA is strictly online-only; the mobile app is online-first with an internal local storage cache.
*   The database engine is strictly relational — no vector indexing databases, RAG execution models, or semantic retrieval pipeline dependencies.
*   Raw binary file objects never live inside rows; database columns preserve tracking location references only.
*   Every application component strictly patterns after standardized system layering. The schema supporting these boundaries can be reviewed inside [Database](database.md).
