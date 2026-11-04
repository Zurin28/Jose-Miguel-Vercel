
---
title: ZCMC Referral System MVP - Planned Architecture
tags:
  - school
  - reverse-pitching
  - software-architecture
  - offline-first
  - laravel
  - react-native
---

# 🏗️ ZCMC Referral System MVP — Planned Architecture

## 1. Executive Summary
This document outlines the proposed system architecture for the Zamboanga City Medical Center (ZCMC) Referral System MVP. The solution is designed to address the challenges of fragmented healthcare data and inefficient referral coordination across geographically isolated areas (BaSulTa region) that suffer from intermittent or completely absent internet connectivity.

## 2. Technology Stack
*   **Backend API & Sync Engine:** Laravel (PHP)
*   **Primary Database:** PostgreSQL (Recommended for JSON support) or MySQL
*   **Mobile App (Remote Healthcare Workers):** React Native
*   **Local Mobile Database:** WatermelonDB (Optimized for React Native offline-first sync)
*   **Web Portal (ZCMC Admins & Doctors):** React.js

## 3. High-Level Architecture Diagram

```mermaid
graph TD
    subgraph "Remote Facility (Offline / Intermittent)"
        RN[React Native App]
        LocalDB[(WatermelonDB)]
        RN <-->|Read/Write instantly| LocalDB
    end

    subgraph "ZCMC / Admin (Connected)"
        ReactWeb[React.js Web Portal]
    end

    subgraph "Cloud / On-Prem Server"
        Laravel[Laravel REST API]
        Postgres[(PostgreSQL / MySQL)]
        SyncEngine[Sync Controller]
        Laravel <--> Postgres
        Laravel --- SyncEngine
    end

    %% Sync connections
    LocalDB <.->|Delta Sync (Pull/Push)\nBackground Task| SyncEngine
    ReactWeb <-->|Standard REST API| Laravel
```

## 4. The "Offline-First" Sync Strategy
Given the severe connectivity constraints in the target region, standard REST calls triggered by user actions will fail. The architecture treats the mobile device's local database as the primary data source, syncing with the Laravel backend in the background whenever a connection is detected.

### 4.1. Core Principles
1.  **Soft Deletes:** The Laravel database must use soft deletes (`deleted_at` timestamps). Hard-deleted records on the server cannot be synced to the mobile app for local removal.
2.  **Delta Synchronization:** To conserve low bandwidth, only changed records (deltas) are transmitted.

### 4.2. Sync Flow
A dedicated Sync Controller in Laravel will handle the following operations:
*   **PULL (`GET /api/sync?last_pulled_at={timestamp}`):** 
    *   The mobile app requests records that have changed since its last successful sync.
    *   Laravel returns a compressed JSON payload of all records where `updated_at > last_pulled_at` or `deleted_at > last_pulled_at`.
*   **PUSH (`POST /api/sync`):**
    *   The mobile app sends a batch of all local creations, updates, and deletions.
    *   Laravel processes this batch within a single database transaction to ensure data integrity.

### 4.3. Conflict Resolution
For the MVP phase, a "Last-Write-Wins" strategy (evaluating the `updated_at` timestamps between the client and server) or a simpler "Server-Wins" strategy will be implemented to handle data modification conflicts.

## 5. Core Data Entities (MVP Scope)
To ensure the system remains focused while laying the groundwork for eventual HL7 FHIR compliance, the database will revolve around these entities:

*   **Users:** System actors (Doctors, Nurses, Admins) managed via Role-Based Access Control (RBAC).
*   **Facilities:** Origin and destination points for patient care (e.g., ZCMC, specific rural health units).
*   **Patients (PMI):** Core demographic data.
*   **Encounters/Visits:** Clinical records of a patient visiting a facility.
*   **Referrals:** The workflow entity linking a Patient and an Encounter from an Origin Facility to a Destination Facility, tracked via statuses (`pending`, `accepted`, `in_transit`, `arrived`).

## 6. Security & Privacy Considerations (DPA Compliance)
*   **Data at Rest (Mobile):** The local WatermelonDB instance must be encrypted to protect patient data if the device is lost or stolen.
*   **Data in Transit:** All sync operations and API calls must strictly use HTTPS/TLS.
*   **Authentication:** Laravel Sanctum will handle token-based authentication for the mobile app and SPA authentication for the web portal.
*   **Audit Logging:** Critical actions (viewing/editing patient records) should be logged on the server for compliance and tracking.

---

## 🔗 Connections & Related Notes
* **School Hub:** [[20 - School/# 💻 School & Computer Science (Map of Content)|School & Computer Science]]
* **Feature Specifications:** [[ZCMC Referral System MVP - Proposed Features]] — Concrete functional requirements, offline chat queue, and Kanban triage board.
* **Pitch & Competition Strategy:** [[Panel Discussion - Hack to Win]] — SRS documentation, focusing on simplicity, and pitching real-world impact.
* **Rapid Prototyping:** [[Zero to Live URL (Elvin Ray)]] & [[Build client ready website fast with Claude (Jlo Edios)]] — Fast tooling for web portals and APIs.
* **Codebase Structure:** [[Codebase Understanding & Summary Generation Prompt]] — Standard blueprint for documenting and onboarding agents onto this architecture.
