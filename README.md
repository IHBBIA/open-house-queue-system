# Open House Digital Queue & Engraving Management System

A full-stack, mobile-friendly event operations web application built with **React JS**, **Node.js**, **MySQL**, and **AWS Serverless (API Gateway, Lambda, and WebSockets)**. Designed for campus Open House booths to streamline visitor QR code scanning, custom luggage tag engraving orders, and real-time multi-stage queue synchronization across admin devices.

---

## Real-Time System Architecture

```mermaid
sequenceDiagram
    actor Admin as Booth Helper (React UI)
    participant Auth as Auth & Input Guard
    participant API as AWS API Gateway & Lambda
    participant DB as MySQL Database
    participant WS as AWS WebSocket Lambda

    Admin->>Auth: Scan Visitor QR Code (Qrcodescanner.jsx)
    Auth->>Auth: Verify adminAccessToken & Lock Scanner State
    Auth->>API: POST /stamp (Bearer Token)
    API->>DB: Record Visitor Check-In
    API-->>Admin: Open EngravingSelection.jsx

    Admin->>Auth: Submit Tag Color & Engraving Text
    Auth->>Auth: Validate Length & Profanity Filter
    Auth->>API: POST /queue (Bearer Token)
    API->>DB: Insert Ticket (Status: In Queue)
    API->>WS: Trigger Broadcast (UPDATE_QUEUES)
    WS-->>Admin: Push Live Queue Refresh to BOOTHADMIN Group
```

---

## Core Engineering & Security Features

* **Authentication & Route Protection (`Qrcodescanner.jsx`, `EngravingSelection.jsx`):** Guards admin routes by verifying `adminAccessToken` presence and attaching `Bearer` authorization headers to outbound AWS API requests, redirecting unauthenticated sessions to `/adminlogin`.
* **Input Validation & Content Filtering (`EngravingSelection.jsx`, `AdminQueue.jsx`):** Enforces regex-based profanity filtering, character limits, and strict queue-status allowlist checks (`validateQueueStatus`) to prevent invalid state transitions.
* **Race-Condition Prevention (`Qrcodescanner.jsx`):** Uses scan-locking state flags (`scanningInProgress`) and clean camera lifecycle resets (`qrScannerRef.current.stop()`) to prevent duplicate API submissions during rapid booth scanning.
* **Real-Time Multi-Stage Queueing (`WebSocket Lambda`):** Listens for `UPDATE_QUEUES` WebSocket events under the `BOOTHADMIN` group to automatically synchronize ticket movements (*In Queue -> Engraving -> Pending Collection -> Collected*) across all active tablets without manual page refreshes.
* **Operational Reporting:** Includes live ticket search filtering, "Today / All" view toggles for collected items, and one-click CSV data export.

---

## Visual Proof & Wireframes

### 1. Figma Wireframes vs. Production Queue Interface
![Figma Wireframes](screenshots/01-figma-wireframes.png)
![Admin Queue Modal](screenshots/02-update-ticket-modal.png)

### 2. Multi-Stage Queue Controls & Validation Prompts
![Queue Status Buttons](screenshots/03-queue-status-buttons.png)
![Transition Error Prompt](screenshots/04-queue-error-prompt.png)
