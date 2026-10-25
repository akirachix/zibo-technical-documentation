# Overview

Mbegu Link is a working system built by Team Zibo (AkiraChix). This page describes what the current MVP is, how the pieces fit together, and the technical decisions behind it. You can download the complete [Research Material and Problem Statement here (PDF)](assets/brand/research.pdf).

## What the system is

Mbegu Link is a digital bridge between community seed banks (CSBs) and the national genebank (GeRRI). It tracks seed batches across their full conservation lifecycle identity, storage, testing, movement and duplication and connects the people who hold and care for that seed:

- **Seed Registry** — structured records with a persistent Mbegu Link Seed Lot ID, dated events and custody tracking.
- **Seed Exchange** — loans and deposits with a visible request lifecycle.
- **Knowledge Hub** — community knowledge sharing with per-entry visibility and media.
- **Support Requests** — structured assistance requests between CSBs and GeRRI.

## How the system works

Three user roles, two clients, one shared API:

| Role | Client | What they do |
|---|---|---|
| **Seed Custodian** | Mobile (Flutter) | Registers seed, requests loans and deposits, reads and posts knowledge |
| **CSB Coordinator** | PWA | Manages the registry, logs germination tests, reviews loan/transfer/deposit requests |
| **GeRRI Specialist** | PWA | Reviews only seed records shared to GeRRI, responds to support requests |

Both clients talk to one FastAPI backend over HTTPS. Every request carries a user and organisation identity; per-organisation isolation keeps each CSB's data separate. All writes are attributed to a user in an append-only audit log.

## Technical decisions

| Decision | Rationale |
|---|---|
| **FastAPI + PostgreSQL** | Async performance, strict validation (Pydantic), relational integrity for custody chains |
| **Layered backend** | Testable business logic independent of HTTP; swappable data layer (SQLite in tests, PostgreSQL in production) |
| **Append-only history** | Events and request responses are new rows, never updates a falling germination result stays visible with provenance |
| **Germination percent computed by the API** | Never hand-entered; snapshots the threshold at the time, so later threshold changes can't corrupt history |
| **Media as references, not bytes** | `media_files` stores a `storage_url` only; bytes live in Media Storage; SHA-256 dedup prevents re-uploads |
| **Private-by-default visibility** | Knowledge entries start private; only the author widens visibility; hidden/deleted entries return 404 |
| **GeRRI data isolation** | Specialists see only seed records linked to requests sent to GeRRI enforced server side |
| **Continuity of identity** | A persistent Seed Lot ID travels with the seed; community identifiers are preserved as links, never overwritten |

## Current status

| Feature | Status |
|---|---|
| **Knowledge Hub backend** | Implemented, 39 tests |
| **Media files** (dedup, compression, download rules) | Implemented, tested |
| **Audit logs** | Implemented, 17 tests |
| **Seed requests and responses** | Implemented, in test |
| **Mobile screens** (Knowledge Hub flow) | In development on `feature/knowledgehub` |
| **PWA** | In development |

## MVP scope

**In scope:** seed registration, seed movement, custody and institutional support, knowledge sharing, conservation attention, in-app notifications, core APIs.

**Deferred:** seed network coordinator interface, WhatsApp/chatbot integration, audio-based knowledge, offline-first PWA (PWA is online-only; offline is mobile-only).

## Where to go next

- [Getting Started](getting-started.md) — set up and run the project locally
- [Architecture](architecture.md) — system layers and data flow
- [Backend](backend.md) — API layering and conventions
- [Database](database.md) — schema and data design decisions
