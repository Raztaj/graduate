# Project Specification: Resilient Participatory Health Mapping Platform (Offline-First)

## 1. Project Overview
### 1.1 Title
Design and Implementation of a Resilient Participatory Health Mapping Platform for Conflict Zones (Offline-First).

### 1.2 Context
In conflict zones like Sudan, the health sector often faces near-total infrastructure collapse. Information about functional health facilities is scarce and changes rapidly due to displacement and active conflict. Frequent internet blackouts make traditional "Always-Online" systems ineffective.

### 1.3 Objective
To develop a community-driven, offline-first Progressive Web App (PWA) that allows users to map and access health services in real-time, even without a stable internet connection.

---

## 2. User Roles and Use Cases
The system identifies three primary actors:

### 2.1 Explorer (Public User)
*   **Search & Filter:** Find health facilities by type, status (open/closed), and available services.
*   **Offline Mapping:** View cached maps and facility locations without internet.
*   **Routing:** Get directions to the nearest functional facility.

### 2.2 Contributor (Active Community Member)
*   **Add Facility:** Report a new health facility (hospital, clinic, pharmacy).
*   **Update Status:** Update the operational status of an existing facility.
*   **Verify Information:** Confirm or dispute reports made by others.
*   **Report Errors:** Flag inaccurate or outdated information.

### 2.3 System Worker (Automated Background Process)
*   **Background Sync:** Synchronize local updates to the server when connection is restored.
*   **Trust Calculation:** Automatically calculate the reliability of reports using the Trust Score algorithm.
*   **Cache Management:** Manage local storage (IndexedDB) and map tiles.

---

## 3. Requirements

### 3.1 Functional Requirements
1.  **Offline Map Access:** Users must be able to view maps and search for services without an active internet connection.
2.  **Local Data Entry:** Contributors must be able to add or update facility reports while offline.
3.  **Automatic Synchronization:** The system must automatically sync pending reports to the cloud once internet connectivity is detected.
4.  **Distributed Verification:** The system must use community feedback to verify reports instead of relying on central administrators.
5.  **Trust Scoring:** Every report must have a visible "Trust Score" calculated by the system.
6.  **Geomasking:** The system must obscure the exact location of contributors to protect their safety in conflict zones.

### 3.2 Non-Functional Requirements
1.  **Performance:** The application should load in less than 3 seconds on a 3G network.
2.  **Security:** Local data (IndexedDB) should be encrypted. Sensitive user coordinates must never be sent to the server.
3.  **Efficiency:** Use Vector Tiles to minimize data consumption and storage footprint.
4.  **Resilience:** The system must remain fully functional (read/write) during prolonged internet outages.

---

## 4. System Architecture & Tech Stack

### 4.1 Architecture
*   **Offline-First:** The local device is the primary source of truth.
*   **Distributed Validation:** Replaces centralized "Admins" with a community-based reputation system.

### 4.2 Technology Stack
*   **Frontend:** React.js (v18), Vite.
*   **Mapping:** MapLibre GL JS (for Vector Tiles), OpenStreetMap Data (converted to PMTiles).
*   **Offline Capabilities:**
    *   **Service Workers:** Using Workbox for caching strategies and background sync.
    *   **IndexedDB:** Local NoSQL database for GeoJSON data and sync queues.
    *   **TanStack Query:** For state management and cache synchronization.
*   **Backend:**
    *   **Firebase Cloud Functions:** Serverless API for trust calculations and data aggregation.
    *   **Firestore:** Primary cloud database.
*   **Design Tools:** Figma, Lucidchart (UML).

---

## 5. Key Algorithms

### 5.1 Trust Score Algorithm
Used to determine the reliability of a health facility report without central oversight.
*   **Formula:** `TrustScore = Σ (UserReputation * FeedbackWeight) * TimeDecayFactor`
*   **Logic:** Reports from users with higher reputation carry more weight. Older reports lose trust over time to ensure freshness.

### 5.2 User Reputation System
*   If a contribution is confirmed by community consensus: `User.Reputation += LearningRate * ContextFactor`.
*   If a contribution is rejected: `User.Reputation -= PenaltyFactor`.

### 5.3 Geomasking (Donut Method)
To protect the security of users reporting from sensitive areas.
*   The original coordinates are shifted by a random distance between a minimum radius (`r_min`) and a maximum radius (`r_max`).
*   This prevents pinpointing a user's exact home or location while maintaining the data's statistical value for the map.

---

## 6. Data Model (Conceptual)

### 6.1 HealthFacility
*   `id`: UUID
*   `name`: String
*   `location`: Point (GeoJSON)
*   `status`: Enum (Open, Closed, Restricted)
*   `services`: Array of Strings
*   `trust_score`: Float
*   `last_updated`: Timestamp

### 6.2 User
*   `id`: UUID
*   `reputation_score`: Float
*   `role`: Enum (Explorer, Contributor)

### 6.3 Contribution
*   `id`: UUID
*   `facility_id`: UUID
*   `user_id`: UUID
*   `type`: Enum (Create, Update, Verify, Dispute)
*   `timestamp`: Timestamp

---

## 7. Implementation & Testing Strategy

### 7.1 Implementation Phases
1.  **Core UI & Map Integration:** Set up React with MapLibre and load offline-optimized vector tiles.
2.  **Offline Engine:** Implement Service Workers and IndexedDB storage.
3.  **Synchronization Logic:** Build the background sync queue and conflict resolution.
4.  **Trust Engine:** Implement the Trust Score and Reputation algorithms in Cloud Functions.

### 7.2 Testing Plan
*   **Unit Testing:** Test trust calculation functions using Jest.
*   **Offline Simulation:** Use Chrome DevTools to verify functionality in "Offline" mode.
*   **Field Testing:** Test on low-end mobile devices to ensure smooth rendering and performance.

---

## 8. Detailed API Specification (RESTful)

### 8.1 Health Facilities
*   `GET /api/v1/facilities`: Retrieve facilities.
    *   **Query Params:** `bbox` (bounding box), `type`, `min_trust`.
    *   **Caching:** Stale-While-Revalidate via Service Worker.
*   `POST /api/v1/facilities`: Create a new facility report.
    *   **Auth Required:** Yes.
    *   **Payload:** `name`, `type`, `geomasked_location`, `status`, `services[]`.
*   `PATCH /api/v1/facilities/{id}`: Update facility status.

### 8.2 Contributions & Verification
*   `POST /api/v1/contributions`: Submit a verification (upvote/downvote/dispute).
    *   **Payload:** `facility_id`, `contribution_type`, `evidence_notes`.
*   `GET /api/v1/users/me/reputation`: Fetch current user's reputation and history.

---

## 9. Synchronization & Conflict Resolution

### 9.1 Sync Strategy
1.  **Local First:** All writes go to `IndexedDB` immediately.
2.  **Background Sync API:** Register a 'sync' event. The browser triggers this when connectivity is stable.
3.  **Idempotency:** All POST/PATCH requests include a `client_uuid` to prevent duplicate processing if a sync retries.

### 9.2 Conflict Resolution (Last-Write-Wins with Trust Override)
*   If two updates for the same facility occur offline:
    1.  The system compares the `timestamp`.
    2.  If the timestamps are close, the update from the user with the **higher Reputation Score** takes precedence.
    3.  If both have equal reputation, the most recent update wins (Last-Write-Wins).

---

## 10. Security & Authentication

### 10.1 Authentication
*   **Provider:** Firebase Auth (supporting Anonymous sign-in for quick reporting, and Phone/Email for established contributors).
*   **JWT:** Tokens used for all write operations to the API.

### 10.2 Data Encryption
*   **In-Transit:** TLS 1.3 for all communications.
*   **At-Rest (Client):** Use `Web Crypto API` to encrypt sensitive records in `IndexedDB` using a key derived from the user's session.

### 10.3 Privacy (Geomasking Implementation)
*   **Minimum Displacement:** 50 meters.
*   **Maximum Displacement:** 500 meters.
*   **Logic:** Calculated on the client side before the payload is sent to the network. The server never sees the "True" coordinate of the contributor.

---

## 11. Infrastructure & Deployment

### 11.1 CI/CD Pipeline
*   **GitHub Actions:**
    *   Linting & Type Checking (ESLint/TypeScript).
    *   Automated Unit Tests (Jest).
    *   Build PWA assets (Vite).
    *   Deploy to Firebase Hosting.

### 11.2 Map Tile Hosting
*   **Strategy:** PMTiles stored on S3-compatible storage (e.g., Cloudflare R2).
*   **CDN:** Global distribution to ensure fast tile fetching even on poor international links.

---

## 12. Advanced Trust Algorithm Details

### 12.1 Formula Components
`T(f) = [ Σ (R(u) * V(u,f)) / Σ R(u) ] * D(t)`

*   `T(f)`: Trust score of facility *f*.
*   `R(u)`: Reputation of user *u*.
*   `V(u,f)`: Vote value (-1 to +1) given by user *u* to facility *f*.
*   `D(t)`: Exponential time decay function `e^(-λt)` where *t* is age of report.

### 12.2 Sybil Attack Resistance
*   New accounts start with a `ReputationScore` of 0.
*   Reputation only increases when a user's report is verified by *independent* users with established high reputation.
*   Rate-limiting on contributions based on IP and Device ID.

---

## 13. Academic & Engineering Assessment (Professor's Perspective)

### 13.1 Theoretical Foundations
This project sits at the intersection of two critical research domains:
*   **Volunteered Geographic Information (VGI):** Leveraging "citizens as sensors" (Goodchild, 2007) to fill information gaps where official data is non-existent.
*   **Offline-First Paradigm:** Moving away from the "Cloud-Centric" model to a "Device-Centric" model, where the server is an optional synchronization point rather than a dependency.

### 13.2 Engineering Methodology (Agile Scrum)
The project follows the **Agile Scrum** framework to manage complexity in a volatile environment:
*   **Sprints:** 2-week development cycles focusing on MVP (Minimum Viable Product) features first (e.g., Offline Map rendering).
*   **Sprint Backlog:** Prioritizes "Mission Critical" features (Offline access) over "Value Added" features (Social sharing).
*   **Artifacts:** Use of Burndown charts to track velocity and ensure timely graduation delivery.

### 13.3 Formal Design & Modeling
To ensure structural integrity, the system is modeled using:
*   **UML Use Case Diagrams:** Mapping interactions between Explorers, Contributors, and the System Worker.
*   **Sequence Diagrams:** Modeling the complex "Offline Write -> Local Store -> Reconnection -> Sync -> Server Ack" lifecycle.
*   **State Machine Diagrams:** Defining the lifecycle of a `HealthFacility` record (Pending -> Verified -> Disputed -> Archived).

### 13.4 Quality Assurance & Evaluation Metrics
Success is measured through both technical and social KPIs:
*   **Accuracy:** The goal is ≥85% accuracy in facility status when compared to ground-truth validations (simulated).
*   **Resilience:** System availability target of 100% during network blackouts (local functionality).
*   **Latency:** Mean time to synchronize local data to the cloud < 10 seconds post-reconnection.
*   **Storage Efficiency:** Map cache should not exceed 50MB for a standard conflict zone region (using PMTiles).

### 13.5 Social Impact & Ethics
*   **Data Sovereignty:** Users maintain control over their data; geomasking ensures the platform cannot be weaponized to target individuals.
*   **Inclusivity:** The PWA design ensures the app works on low-end "Legacy" devices common in conflict zones, avoiding a "Digital Divide."

---

## 14. Future Research Directions
1.  **Mesh Networking:** Implementing Bluetooth/Wi-Fi Direct P2P synchronization for environments with 100% infrastructure destruction.
2.  **NLP Integration:** Using Natural Language Processing to extract health facility status updates from social media (e.g., WhatsApp groups) to pre-populate the map.
3.  **Predictive Analysis:** Using historical trust data and conflict patterns to predict where medical shortages are likely to occur.
