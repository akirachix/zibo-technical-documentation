# Mbegu Link — Technical Documentation Site

This repository contains the source files for the Mbegu Link technical documentation portal. The site is built using MkDocs and styled with the Material for MkDocs theme.

## About Mbegu Link

Mbegu Link is a community-controlled platform connecting seed custodians, Community Seed Banks (CSBs), and formal conservation institutions such as GeRRI, keeping seed identity, conservation history, testing, storage, movement, and conservation needs connected.

*   **Seed Registry & Conservation Records:** Structured records for seed identity, local and taxonomic names, source, quantity, storage, testing, germination, regeneration, history, and provenance.
*   **Seed Movement:** Track transfers, loans, requests, sharing, and custody (Sent – In Transit – Received & Confirmed), with structured CSB - GeRRI assistance requests.
*   **Knowledge Sharing:** Share seed knowledge through text, images, and video, with controlled visibility: private, institutional, or public.

## Repository Structure

*   docs/ — The documentation content pages written in Markdown.
    *   docs/assets/ — Images, system diagrams, and brand assets.
    *   docs/stylesheets/ — Custom CSS overrides for team branding colors.
*   mkdocs.yml — The main configuration file handling site settings and navigation sidebar order.
*   .github/workflows/deploy-docs.yml — Automated CI/CD pipeline that builds and deploys the site to GitHub Pages on every push to main.

---

## Local Development Setup

To preview and edit this documentation locally on your machine, follow these steps:

### 1. Prerequisites
Ensure you have Python 3 installed on your system.

### 2. Install Dependencies
Install the specific versions of MkDocs and the Material theme required for this project:
```bash
pip install -r requirements.txt
```

### 3. Start the Live Preview Server
Spin up the local development web server:
```bash
# If mkdocs is globally available:
mkdocs serve

# If installed in your local user directory:
~/.local/bin/mkdocs serve
```
Once running, open your web browser and navigate to http://127.0.0.1:8000. The page will automatically refresh in real-time whenever you save changes to your Markdown files.

### 4. Verify Before Pushing
Always verify that your structural changes and internal relative links are fully intact before opening a pull request:
```bash
# If mkdocs is globally available:
mkdocs build --strict

# If using local user binaries:
~/.local/bin/mkdocs build --strict
```
The --strict flag converts warnings into hard errors. Ensure this check passes with zero warnings before pushing your branch.
