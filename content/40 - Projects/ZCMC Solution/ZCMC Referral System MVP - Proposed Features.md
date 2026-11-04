
---
title: ZCMC Referral System MVP - Proposed Features
tags:
  - school
  - reverse-pitching
  - product-features
  - healthcare-it
  - telemedicine
---

# 📱 ZCMC Referral System MVP — Proposed Features

## 1. Secure & Contextual Chat (The "Viber/Messenger" Replacement)
Currently, healthcare workers rely on non-compliant standard messaging apps. Providing a built-in chat is critical for adoption.
*   **Referral-Based Threads:** Chat should not be a general free-for-all. Every chat thread must be tied directly to a specific `Referral` record. This ensures all communication regarding a patient is contextualized and securely logged with their case.
*   **Offline Message Queueing:** If a user sends a message while offline in a remote facility, it displays as "Queued" and automatically syncs to the backend once a connection is re-established.
*   **Smart Media Sharing:** Doctors frequently need to share photos (e.g., wounds, ECGs). To handle low bandwidth, images should be heavily compressed on the mobile device before sending. The receiving end should see a low-resolution thumbnail with a "Tap to Download High-Res" option to conserve data.

## 2. Core Referral Workflow
*   **"Minimum Data Set" Intake Form:** Instead of a complex Electronic Medical Record (EMR) system, the MVP should use a lean, standardized form. It should only capture essential data: Patient Demographics, Chief Complaint, Current Vitals, and Urgency Level.
*   **Offline Referral Creation & Triage:** Remote clinics must be able to draft and save referral requests entirely offline. They can assign a priority (Routine, Urgent, Emergency) which will immediately flag the system upon the next sync.
*   **Kanban-Style Referral Board (Web Portal):** For ZCMC administrators and receiving doctors, a visual board is highly effective. Referrals move through stages: `Pending Review` -> `Accepted/Rejected` -> `In Transit` -> `Arrived`.

## 3. Notifications & Reliability
*   **SMS Fallback for Emergencies:** Standard push notifications rely on continuous internet access, which is unreliable in the BaSulTa region. If a referral is marked as an "Emergency", the Laravel backend can trigger an automated SMS text message to the on-call ZCMC doctor (e.g., *"Urgent referral pending in system for Patient [Initials]."*).

## 4. Admin & Analytics (For Hospital Leadership)
To provide immediate value to policymakers and hospital managers (the "Carrot" mentioned in the challenge), the web portal should include a simple, real-time dashboard tracking:
*   **Volume:** Total referrals sent/received per day/week, broken down by origin facility.
*   **Efficiency:** Average turnaround time for ZCMC to accept or reject a pending referral request.
*   **Outcomes:** Most common reasons for referrals (helps identify where lower-level facilities need more resources or training).

---

## 🔗 Connections & Related Notes
* **School Hub:** [[20 - School/# 💻 School & Computer Science (Map of Content)|School & Computer Science]]
* **System Architecture:** [[ZCMC Referral System MVP - Planned Architecture]] — Offline-first sync engine, WatermelonDB, and Laravel backend implementation.
* **Pitch & Product Strategy:** [[Panel Discussion - Hack to Win]] — Keeping MVP scope tight and delivering immediate user value.
* **Rapid UI Development:** [[Build client ready website fast with Claude (Jlo Edios)]] & [[Zero to Live URL (Elvin Ray)]] — Rapid web portal dashboards and mockups.
