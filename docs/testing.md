# Quality Assurance and Testing

## Quality Assurance Strategy

Mbegu Link holds records that communities depend on for conservation decisions, so quality assurance focuses on the failures that matter most: a seed record that loses its history, a boundary between CSBs that leaks, a transfer that misassigns custody, or a threshold flag that fires wrongly. Testing therefore concentrates on record integrity, privacy isolation, and request lifecycles.

### Test Levels
- **Unit Tests:** Validate single core methods and business constraints (germination calculations, `seeds_germinated` $\le$ `seeds_tested`, threshold boundaries, and append-only correction lines).
- **Integration Tests:** Cover boundary layers between the FastAPI service and the PostgreSQL database engine (multi-tenant tenant schema isolation, route guards, token lifespan invalidation, and server-side row permissions).
- **End-to-End (E2E) Tests:** Trace user workflow paths across application clients (custodian entry logging, coordinator transfer approval tracks, and GeRRI support ticket workflows).
- **User Acceptance Testing (UAT):** Checked directly with regional custodians and network coordinators from partner Community Seed Banks to confirm software layout accuracy prior to production builds.

### Defect Severity Definitions
- **Critical:** Complete data loss, multi-tenant isolation breach, misassigned seed custody, or false flags leading to crop damage.
- **High:** Major workflow blocking (inability to log a germination result, failure to issue seed loans, or broken image uploads).
- **Medium:** Operational failures with correct fallback behaviors or clumsy recovery steps.
- **Low:** Cosmetic layout imperfections, font sizes, or text wrapping bugs.

## Essential Test Matrix

| Case ID | Module | Priority | Type | Test Case Description | Expected Verification Result |
|:---|:---|:---|:---|:---|:---|
| **TC-QA-01** | Registry | Critical | Security | Custodian from CSB A requests a holding belonging to CSB B by ID | `403 Forbidden`; unauthorized permission denial logged to the append-only trail |
| **TC-QA-02** | Registry | Critical | Negative | Register seed with `seeds_germinated` greater than `seeds_tested` | Rejected with validation error; database write blocked |
| **TC-QA-03** | Registry | High | Functional | Register a variety with an unknown scientific taxonomy name | Record saved successfully with a `NULL` scientific metadata marker |
| **TC-QA-04** | Registry | High | Boundary | Germination result exactly at the configured threshold boundary (e.g., 75.00%) | Attention flag does not fire; boundary is handled inclusively per community rule |
| **TC-QA-05** | Registry | High | Negative | Two custodians register the same local seed name for different biological crops | Two distinct variety records generated; silent deduplication merges blocked |
| **TC-QA-06** | Registry | Critical | Negative | Custodian attempts to edit or overwrite an old germination test log entry | Direct modification blocked; adjustments require a new event entry referencing the original row |
| **TC-QA-07** | Movement | Critical | Functional | Full seed transfer lifecycle execution track: Sender initiates $\rightarrow$ Receiver confirms | Custody updates to receiver only upon final receipt verification |
| **TC-QA-08** | Movement | Critical | Negative | Seed transfer request is never confirmed within the agreed network duration | Status locks as *In Progress*; lot stays visible on coordinator outstanding list |
| **TC-QA-09** | Movement | High | Negative | Seed parcel is explicitly marked as lost or damaged in transit | Custody reverts to sender; mandatory exception trace note appended to history |
| **TC-QA-10** | Movement | Medium | Boundary | Partial transfer confirmation: Receiver logs less than total shipped quantity | Received quantity added to target vault; remainder stays on sender profile balances |
| **TC-QA-11** | Movement | High | Negative | Activated seed loan passes due date without an inbound return transaction | Loan flagged as past due; coordinator dashboard raises overdue warnings |
| **TC-QA-12** | Attention | Critical | Functional | Measured germination rates fall below community-configured percentage thresholds | Attention flags trigger prompts for review; no automated database actions taken |
| **TC-QA-13** | Attention | High | Negative | Coordinator adjusts minimum threshold levels after logging test histories | Historical attention flags stay static; changes apply only to future events |
| **TC-QA-14** | Support | Critical | Security | GeRRI specialist attempts to pull seed metrics not linked to any active support request | `403 Forbidden` error; route guard blocks unauthorized regional database scans |
| **TC-QA-15** | Support | High | Negative | GeRRI specialist declines an active institutional support request | Seed lot ownership remains with the originating CSB; rejection rationale logged |
| **TC-QA-16** | Knowledge | Critical | Security | Hub entry containing farmer names and exact GPS coordinates submitted for public sharing | Publication blocked; item queued for administrative review and consent matching |
| **TC-QA-17** | Knowledge | High | Security | Coordinator issues a withdrawal instruction for an active public knowledge entry | Content hidden from search views; logging logs action with hard removal notices |
| **TC-QA-18** | Knowledge | High | Negative | User uploads an executable file disguised with a fake extension (e.g., `.exe` as `.mp4`) | Blocked by server-side byte scanners; `upload_status` records as `failed` |
| **TC-QA-19** | Auth | Critical | Security | User triggers three consecutive invalid password authentication requests | Account locked; event logged; unlock requires single-use verification code |
| **TC-QA-20** | Concurrency| High | Negative | Two application users attempt to update the same seed holding row simultaneously | Server returns a concurrency conflict error to the last request; prevents overwrite collisions |

## End-to-End Testing with Cypress

End-to-End (E2E) tests verify the three journeys that define MVP success. These tests run against a seeded test database containing known users, CSBs, and varieties, and are never executed against production environments. Cypress is used for the Progressive Web Application (PWA) coordinator and GeRRI specialist journeys. The mobile app journeys are exercised through the same API contracts, allowing the PWA tests to double as integration verification of the endpoints called by the app.

### The Seeded Scenario
- **Verified CSBs:** Two organizations (`CSB-A`, `CSB-B`).
- **Actors:** One custodian per CSB, one coordinator per CSB, and one GeRRI specialist.
- **Reference Data:** One shared sorghum variety record with a populated germination history.

### Core Testing Journeys
1. **Seed Registration (Custodian $\rightarrow$ Registry):** The custodian logs in with a phone OTP and registers a variety with its local name, source, and quantity. The record must instantly populate the CSB master registry with its trust signals (owner, custodian, and last updated timestamp).
2. **Seed Movement (Custodian $\rightarrow$ Coordinator):** The custodian of `CSB-A` requests a seed loan of a `CSB-B` variety. The `CSB-B` coordinator approves the request, placing the transfer in transit. Once the `CSB-A` custodian confirms receipt, custody records update, and append-only audit entries are verified for both actions.
3. **Conservation Support (Coordinator $\rightarrow$ GeRRI):** The coordinator records a germination test that drops below the community-configured threshold, triggering an attention flag. The coordinator submits a support request to GeRRI. The GeRRI specialist can view only the specific linked seed lot history, and tracking progresses smoothly to *Completed* upon approval.

### E2E Test Suite Implementation

#### Journey 2: Seed Movement Test Suite
```javascript
describe('Journey 2: Seed movement between CSBs', () => {
  beforeEach(() => {
    cy.loginAsCoordinator('CSB-B'); 
  });

  it('shows a pending loan request from CSB-A custodian', () => {
    cy.visit('/requests');
    cy.contains('[data-cy=request-row]', 'TC-LOAN-SEED-001')
      .should('contain', 'Pending')
      .and('contain', 'CSB-A');
  });

  it('coordinator approves the loan and custody stays until confirmation', () => {
    cy.visit('/requests');
    cy.get('[data-cy=request-row]').contains('TC-LOAN-SEED-001').click();
    cy.get('[data-cy=approve-btn]').click();
    cy.get('[data-cy=request-status]').should('contain', 'Approved');
    
    cy.request('/api/holdings/SEED-HELD-ID').its('body.custodian')
      .should('eq', 'CSB-B-custodian');
  });

  it('denies a CSB-A user access to CSB-B holdings (multi-tenant isolation)', () => {
    cy.loginAsCustodian('CSB-A');
    cy.request({ url: '/api/holdings/SEED-HELD-ID', failOnStatusCode: false })
      .its('status').should('eq', 403);
  });
});
```

#### Journey 3: Conservation Support Test Suite
```javascript
describe('Journey 3: Conservation support to GeRRI', () => {
  it('flag fires only when germination drops below the CSB threshold', () => {
    cy.loginAsCoordinator('CSB-A');
    cy.visit('/attention');
    cy.get('[data-cy=flag-row]').should('contain', 'Below threshold');
  });

  it('GeRRI specialist sees only records explicitly linked to a support request', () => {
    cy.loginAsSpecialist('GeRRI');
    cy.visit('/shared-records');
    cy.get('[data-cy=record-row]').should('have.length', 1); 
    cy.request({ url: '/api/holdings/OTHER-CSB-A-HOLDING', failOnStatusCode: false })
      .its('status').should('eq', 403);
  });
});
```

#### Reusable Support Commands Configuration
```javascript
Cypress.Commands.add('loginAsCoordinator', (csb) => {
  cy.session(csb, () => {
    cy.request('POST', '/api/auth/login', {
      email: `${csb.toLowerCase()}-coordinator@test.mbegu`,
      password: Cypress.env('TEST_PASSWORD'),
    }).then(({ body }) => window.localStorage.setItem('token', body.token));
  });
});
```

### Practical Engineering Setup Notes
* **Target Address:** Keep `baseUrl` pointing to a local workspace loop or an isolated staging target.
* **Database State Reset:** Execute an explicit database reseed hook (`cy.task('db:seed')`) before launching each test execution loop to clear out stray mutations.
* **Boundary Validation Security:** Keep the explicit `403-isolation` checks active within every single end-to-end journey. These assertions serve as high-value perimeter guards, mapping back to core security requirements `TC-QA-01` and `TC-QA-14`.
* **Case ID Numbering Scheme:** The test cases are systematically numbered `TC-QA-01` through `TC-QA-20` to separate them from separate component module tracking ranges within shared spreadsheets.

## API Contract Testing via Postman

To verify contract compliance across boundary layers and validate content tracking states for file uploads, interactions, and media operations inside the **Knowledge Hub** module, the team uses automated Postman collections.

* **Collection Name:** `Zibo_Knowledge hub`
* **Target Environment Schema:** `v2.1.0`
* **Collection Link:** [Active Postman Workspace Collection](https://go.postman.co/collection/56092739-a2101e3f-0fb3-496f-beee-35b556cb9d5f?source=collection_link)

### Essential Endpoint Verification Matrix

#### 1. Fetch Knowledge Entries (`GET`)
Retrieves all accessible knowledge hub posts filtered against active multi-tenant organization identifiers.
* **Request URL:** `{{base_url}}/knowledge/entries`
* **Mandatory Security Headers:**
  * `X-User-Id`: `{{user_id}}`
  * `X-Org-Id`: `{{org_id}}`
  * `Content-Type`: `application/json`

#### 2. Create Knowledge Base Entry (`POST`)
Submits text payloads and saves the auto-generated identity context using a post-request collection variable capture script.
* **Request URL:** `{{base_url}}/knowledge/entries`
* **Body Mode:** `formdata` (`file` payload parameter pointing to source asset)
* **Post-Request Test Extraction Script:**
  ```javascript
  const responseData = pm.response.json();
  if (responseData && responseData.entry_id) {
      pm.collectionVariables.set("latest_entry_id", responseData.entry_id);
      console.log("Saved entry_id for multi-stage media uploads: " + responseData.entry_id);
  }
  ```

#### 3. Attach Binary Media Assets (`POST`)
Uploads physical photo assets or structural content items targeting specific collection entry items.
* **Request URL:** `http://127.0.0`
* **Body Mode:** `formdata` (`file` type boundary containing image byte data)

#### 4. Interaction Metrics & Likes Log (`POST` / `DELETE`)
Verifies execution logic when submitting or removing user likes.
* **Endpoints:**
  * `POST`: `http://127.0.0`
  * `DELETE`: `{{base_url}}/knowledge/entries/{{entry_id}}/like`
* **Payload Blueprint:**
  ```json
  {
    "entry_id": "d8ac339e-5e5d-4a0d-ac6e-46377cfa12d4",
    "author_id": "79177ff2-02ba-4922-bed5-9a83bda3f6ea",
    "title": "Bean seed germination testing",
    "visibility": "private",
    "like_count": 0
  }
  ```

#### 5. Data Access Control & Share Rules (`POST`)
Enforces organization row limits when extending access boundaries to target nodes.
* **Request URL:** `http://127.0.0`
* **Payload Blueprint:**
  ```json
  {
    "shared_with_user_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "shared_with_organisation_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
  }
  ```

#### 6. Visibility Modification (`PATCH`)
Alters metadata classifications from isolated organization modes to public sharing states.
* **Request URL:** `{{base_url}}/knowledge/entries/{{entry_id}}/visibility`
* **Payload Blueprint:**
  ```json
  {
    "visibility": "public"
  }
  ```

#### 7. Media Metadata Registry Verification (`POST`)
Registers storage system tracking flags separate from underlying binary uploads.
* **Request URL:** `{{base_url}}/knowledge/media`
* **Payload Blueprint:**
  ```json
  {
    "media_id": "UUID-string",
    "entry_id": "UUID-string",
    "file_name": "screenshot.png",
    "media_type": "photo",
    "file_size_bytes": 12345
  }
  ```

#### 8. Validation Error Protection Tracking (`POST`)
Ensures the server rejects structurally malformed parameters at the API perimeter gate.
* **Target Under Testing:** Malformed UUID string parameter validations.
* **Expected Error Response Blueprint (`422 Unprocessable Entity`):**
  ```json
  {
    "msg": "Input should be a valid UUID, Invalid group length in group 0: expected 8, found 9",
    "input": "81f606ac5-e1f7-4111-8111-111111111111"
  }
  ```
