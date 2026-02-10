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
