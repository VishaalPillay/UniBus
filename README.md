# UniBus - High-Performance Campus Bus Tracking System

**UniBus** is a scalable, real-time transportation network platform designed for SRM University. It provides a native mobile experience for students to track college buses and for drivers to broadcast live location data with minimal latency.

> **System Status:** Architecture upgraded to handle **10,000+ concurrent users** via Redis geospatial caching and WebSocket clustering.

---

## 🏗 High-Scale Architecture

This project has moved beyond a simple web app to a **Distributed Real-Time System** designed like Uber/Rapido.

### The "Stack Shift" (MRVN Stack)
We utilize the **MRVN** (MongoDB, Redis, Varnish/View [React Native], Node.js) architecture to ensure sub-200ms latency.

| Component | Technology | Purpose |
| :--- | :--- | :--- |
| **Mobile Apps** | **React Native** | Cross-platform (iOS/Android) native performance. Required for background GPS services and Mapbox vector rendering. |
| **Backend API** | **Node.js (Express/NestJS)** | High-throughput event loop for handling thousands of concurrent WebSocket connections. |
| **Live Data Store** | **Redis (Geospatial)** | In-memory caching for live bus coordinates. This avoids database I/O bottlenecks. |
| **Persistent DB** | **MongoDB** | Long-term storage for user profiles (Students/Drivers), Route definitions, and Shift logs. |
| **Maps** | **Mapbox GL** | Vector-based maps allowing custom "Night Mode" styles (Uber-like) and high-performance markers. |
| **Real-Time Engine**| **Socket.io + Redis Adapter** | Enables horizontal scaling. A driver on Server A can talk to a student on Server B. |

---

## 📐 System Design & Data Flow

1.  **Driver App (Emitter):**
    * Runs a foreground service (notification) to keep GPS alive even when the phone is locked.
    * Emits `BUS_LOCATION_UPDATE` events via WebSocket every 3 seconds.
2.  **API Gateway (Load Balancer):**
    * Nginx distributes incoming WebSocket connections across multiple Node.js instances.
3.  **Backend Cluster:**
    * Receives coordinates and performs a `GEOADD` command to **Redis**.
    * Publishes the update to a specific Redis channel (e.g., `route_01`).
4.  **Student App (Listener):**
    * Subscribes to the specific route channel.
    * Receives updates immediately via the Redis Pub/Sub mechanism.

---

## 📂 Project Structure (Monorepo)

```text
/unibus-root
├── backend/                  # Node.js Scalable Server
│   ├── src/
│   │   ├── config/           # Redis & Mongo connections
│   │   ├── controllers/      # Auth & API Logic
│   │   ├── gateways/         # WebSocket Gateways (Socket.io)
│   │   ├── models/           # Mongoose Schemas (User, Route)
│   │   └── services/         # RedisService (Geo-hashing logic)
│   ├── redis.conf            # Redis configuration tuning
│   └── ecosystem.config.js   # PM2 Cluster mode config
│
├── mobile-driver/            # React Native App for Drivers
│   ├── src/
│   │   ├── services/         # Background Geolocation Service
│   │   ├── screens/          # Login, RouteSelection, BroadcastView
│   │   └── socket/           # Emitter logic
│   └── android/              # Native Android manifests (Permissions)
│
├── mobile-student/           # React Native App for Students
│   ├── src/
│   │   ├── components/       # Custom Map Markers, BusCard
│   │   ├── screens/          # Dashboard, MapView, RouteList
│   │   └── hooks/            # useBusLocation (Socket hook)
│   └── ios/                  # Native iOS Info.plist (Permissions)
│
└── docs/                     # System Diagrams & API Docs
