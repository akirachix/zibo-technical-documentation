# Deployment Guide: Mbegu Link Platform Architecture

**Document Version:** 1.0.0
**Author/Group:** Group Zibo

## 1. Information Website Deployment (Vercel)

The Mbegu Link public marketing and information portal is structured as a decoupled frontend web layer. It is deployed onto the Vercel platform to guarantee global delivery, integrated SSL generation, and edge caching for static assets.

### Critical Verification Steps

| Step | Action |
| --- | --- |
| 1. Repository Linkage & Project Injection | Log into the Vercel Dashboard, select Add New Project, and authorize access to your version control repository provider (e.g., GitHub). Import the targeting repository sub-directory for the information website. |
| 2. Environment Variable & Framework Configuration | Configure the build command parameters. Select the appropriate runtime framework (e.g., Next.js or Vite/React). Under the Project Environment Variables interface, inject production-level keys (`NEXT_PUBLIC_API_GATEWAY_URL` or `VITE_API_URL`) to link the frontend to the centralized API backend gateway. |
| 3. Edge Compilation & Domain Propagation | Execute the deployment build via the Deploy command. Vercel compiles the web assets at the edge layer, issues an automated Let's Encrypt TLS/SSL certificate, and provisions a public-facing URL domain endpoint. |

## 2. Mobile Application Production Release (Google Play Store)

The real-time online mobile client utilized by Seed Custodians requires distribution through the Google Play Console to reach target low-tier Android hardware configurations deployed across agricultural centers.

### 5-Step Distribution Pipeline

| Step | Action |
| --- | --- |
| 1. Production Build Compilation & Cryptographic Signing | Generate a release-optimized production asset bundle (Android App Bundle: .aab) via your build engine terminal using local compilation routines. |
| — | For Flutter architectures: `flutter build appbundle --release` |
| — | For React Native architectures: `cd android && ./gradlew bundleRelease` |
| — | Ensure the output bundle is cryptographically signed using your private release keystore profile (`upload-key.jks`). |
| 2. Play Console Project Provisioning | Navigate to the Google Play Console, select Create App, configure initial language parameters, and designate the application classification type as a production software product. |
| 3. Internal Testing and Internal Tracks Deployment | Open the Testing -> Internal Testing sub-module interface. Create a new internal release track instance, upload the signed .aab production bundle asset, and populate initial change logging parameters. Assign internal tester proxy emails to evaluate binary execution. |
| 4. Store Listing Asset & Metadata Entry | Complete all mandatory compliance criteria, privacy policies, target age demographic questionnaires, and data safety forms. Upload high-resolution visual layout screenshots displaying user interfaces across distinct display dimensions. |
| 5. Production Release Promotion & Review Request | Navigate to the Production track dashboard, create a new release package, pull forward the verified artifact from the internal testing stage, and initiate the Start Rollout to Production validation script to submit the codebase to Google's app security review team. |

## 3. API Backend Gateway & Relational Migration Blueprint

The centralized API engine is deployed onto an enterprise container architecture or a cloud Linux instance linked directly to a managed PostgreSQL cluster enforcing strict Row-Level Security (RLS) constraints.

### The Execution and Database Migration Flow

| Step | Action |
| --- | --- |
| 1. Asset Pulling & Environment Layer Initialization | Clone the production branch onto the host machine, initialize environment configurations (.env), and verify network routes into the secure PostgreSQL server. |
| 2. Pre-Migration Dry Run Analysis | Run migration assessment routines to evaluate schema delta compatibility against live tables before committing writes to disk. |
| 3. Transactional Database Migration Execution | Execute the schema migrations programmatically within single ACID-compliant transactional wrappers. If any constraint fails, the entire database state rolls back instantly to protect data lineage. |
| 4. Process Supervision & Gateway Booting | Trigger backend server execution loops supervised by a system monitor tool (e.g., PM2 or Docker Compose) to handle active client API traffic over standard TLS channels. |

### Production Database Migration Script Code

Save the following production migration script as `migrations/20261026_001_create_core_ledger_tables.js` within your backend repository directory structure. This script builds out your main seed batch tracking and immutable verification ledgers:

```javascript
/**
 * Mbegu Link Core Database Schema Migration
 * Target Engine: PostgreSQL 15+ / 16+
 * Purpose: Implements Core FTR-02/FTR-03 Tables with Strict Mutability Prevention Checks
 */

exports.up = async function(knex) {
  return knex.schema
    // 1. Core Users Table
    .createTable('users', (table) => {
      table.uuid('user_id').primary().defaultTo(knex.raw('gen_random_uuid()'));
      table.string('email').unique().notNullable();
      table.string('password_hash').notNullable();
      table.enum('role', ['Custodian', 'Coordinator', 'Specialist']).notNullable();
      table.uuid('organization_id').nullable();
      table.timestamps(true, true);
    })

    // 2. Central Relational Seed Lot Registry Table
    .createTable('seed_batches', (table) => {
      table.string('mbegu_id').primary(); // Formatted Unique Mbegu Link Identifier Lot Code
      table.string('local_name').notNullable();
      table.string('taxonomy_name').notNullable();
      table.string('community_identifier').nullable();
      table.decimal('quantity', 12, 2).notNullable();
      table.string('duplication_status').defaultTo('NOT_DUPLICATED');
      table.timestamps(true, true);
    })

    // 3. Append-Only Conservation Records (Germination Tests / Storage Logs)
    .createTable('conservation_records', (table) => {
      table.uuid('record_id').primary().defaultTo(knex.raw('gen_random_uuid()'));
      table.string('mbegu_id').references('mbegu_id').inTable('seed_batches').onDelete('RESTRICT').onUpdate('CASCADE');
      table.decimal('germination_rate', 5, 2).notNullable();
      table.decimal('storage_temp', 4, 1).nullable();
      table.timestamp('observation_date').defaultTo(knex.fn.now());
      table.text('notes').nullable();
    })
    .then(async () => {
      // Inject Native Database Trigger to Block Out-of-Scope Data Deletion or Record Mutation Attempts
      await knex.raw(`
        CREATE OR REPLACE FUNCTION block_ledger_mutations()
        RETURNS TRIGGER AS $$
        BEGIN
          RAISE EXCEPTION 'CRITICAL SECURITY ERROR: MUTATION ATTEMPTED ON APPEND-ONLY LEDGER LOGS DETECTED.';
          RETURN NULL;
        END;
        $$ LANGUAGE plpgsql;
      `);

      await knex.raw(`
        CREATE TRIGGER lock_conservation_records_mutation
        BEFORE UPDATE OR DELETE ON conservation_records
        FOR EACH ROW EXECUTE FUNCTION block_ledger_mutations();
      `);
    });
};

exports.down = async function(knex) {
  return knex.schema
    .raw('DROP TRIGGER IF EXISTS lock_conservation_records_mutation ON conservation_records')
    .raw('DROP FUNCTION IF EXISTS block_ledger_mutations')
    .dropTableIfExists('conservation_records')
    .dropTableIfExists('seed_batches')
    .dropTableIfExists('users');
};
```