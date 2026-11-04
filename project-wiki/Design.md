# Design Materials — Senior Design Showcase Display

## Product Overview

**What is it?**  
A full-screen, touch-enabled web application running in kiosk mode on the large display outside DSB 268. It replaces the current timed slideshow with an interactive exploration interface for senior design projects.

**Who uses it?**
- **Prospective students** (primary) — High school students and families on campus tours. They want to see "what do CSE students actually build?" in an engaging, self-guided way.
- **Current students** — Looking for capstone inspiration, checking what past teams did in their interest area.
- **Faculty & staff** — Using the display as a talking point during tours and open houses.
- **Visitors & alumni** — Casual browsing, nostalgia, seeing department evolution.

**What problem does it solve?**  
The current slideshow is passive, static, and shows generic department info. It doesn't showcase the *outcomes* of the program — the actual projects students build. This application turns the display into a living portfolio that:
- Lets visitors discover projects by category, year, or keyword
- Shows rich media (images, videos, live demo links)
- Requires zero training — walk up, touch, explore
- Runs unattended for months with automatic recovery

**Success looks like:** A prospective student spends 3+ minutes exploring projects on the display during a tour, leaves with a concrete sense of what CSE capstones achieve, and the department staff can update the project catalog without IT help.

---

## Key Features & Capabilities

| # | Feature | Priority | Charter Link | Description |
|---|---------|----------|--------------|-------------|
| F1 | **Project Grid Browser** | Must-have | Scope: Interactive showcase | Touch-scrollable grid of project cards (thumbnail, title, year, tags). Infinite scroll or pagination. |
| F2 | **Project Detail View** | Must-have | Scope: Visual + interactive material | Full-screen detail: description, team, tech stack, images, video, live demo link, GitHub link. |
| F3 | **Touch Navigation** | Must-have | Scope: Touch-screen display | Tap to select, swipe to navigate, pinch-to-zoom images, back gesture. Large touch targets (≥48px). |
| F4 | **Search & Filter** | Must-have | Scope: Find relevant projects | Text search (title, description, team), filter by year, category/tags, tech stack. |
| F5 | **Attract Mode / Idle Loop** | Must-have | Scope: Unattended kiosk | After 60s idle: auto-play highlight reel of projects. Touch any screen to exit to browser. |
| F6 | **Admin CMS** | Must-have | Scope: Client-manageable content | Web-based admin: add/edit/archive projects, upload media, manage tags, reorder featured. Auth via campus SSO. |
| F7 | **Video Playback** | Should-have | Scope: Visual material | Inline video player for project demo videos. Auto-play muted in attract mode; full-screen on tap. |
| F8 | **Interactive Demo Embed** | Should-have | Scope: Interactive material | Iframe/WebView embed for playable web projects (games, tools, visualizations). Sandbox permissions. |
| F9 | **Usage Analytics** | Should-have | Scope: Department insights | Anonymous events: project views, detail opens, video plays, search queries. Dashboard in CMS. |
| F10 | **Kiosk Hardening** | Must-have | Scope: Unattended reliability | Auto-restart on crash, memory watchdog, screen burn-in prevention (subtle motion), disable OS UI. |
| F11 | **Accessibility (WCAG AA)** | Should-have | Scope: Inclusive access | High contrast mode toggle, large text option, screen reader labels (for staff), keyboard fallback. |
| F12 | **Multi-language** | Could-have | — | English default; Spanish/Chinese toggle for tour groups. |
| F13 | **QR Code Deep Links** | Could-have | — | QR on detail view → open project on visitor's phone. |
| F14 | **AI Project Recommender** | Could-have | — | "Similar projects" sidebar based on tags/description embeddings. |

---

## High-Level Structure: System Architecture

```mermaid
graph TB
    subgraph Display["DSB 268 Display Hardware"]
        Kiosk[("Kiosk Browser\nChrome or Edge Kiosk Mode")]
    end

    subgraph App["Showcase Application (PWA)"]
        UI["Touch UI Layer\nReact + TypeScript"]
        State["State Management\nZustand or Redux"]
        API["API Client\nAxios + SWR"]
        Assets["Asset Cache\nService Worker"]
    end

    subgraph Backend["Backend Services (Node.js/Express)"]
        Auth["Auth Middleware\nCampus CAS/SSO"]
        ProjectsAPI["Projects API\nCRUD + Search"]
        MediaAPI["Media API\nUpload/Transform"]
        AnalyticsAPI["Analytics API\nEvent Ingestion"]
        CMS["Admin CMS UI\nProtected Routes"]
    end

    subgraph Data["Data Layer"]
        DB[("PostgreSQL\nProjects, Tags, Analytics")]
        Blob[("Object Storage\nS3-compatible or MinIO\nImages, Videos, Files")]
    end

    subgraph External["External"]
        CAS["Campus CAS/SSO"]
        GitHub["GitHub Links\n(External)"]
        Deploy["Deployment Target\nDept VM or On-Prem K8s"]
    end

    Kiosk --> UI
    UI --> State
    State --> API
    API --> ProjectsAPI
    API --> MediaAPI
    API --> AnalyticsAPI
    UI --> Assets
    Assets --> Blob
    ProjectsAPI --> DB
    MediaAPI --> Blob
    AnalyticsAPI --> DB
    CMS --> Auth
    Auth --> CAS
    ProjectsAPI -.-> GitHub
    Deploy --> Kiosk
    Deploy --> Backend
    Deploy --> Data

    style Kiosk fill:#fff3e0,stroke:#e65100
    style UI fill:#e3f2fd,stroke:#1565c0
    style DB fill:#e8f5e9,stroke:#2e7d32
    style Blob fill:#fce4ec,stroke:#c2185b
    style CAS fill:#f3e5f5,stroke:#7b1fa2
```

### Architecture Notes

- **Single-Page Application (PWA)** — Runs offline-capable via Service Worker. Caches all project data and assets for instant navigation and resilience.
- **Kiosk Mode** — Chrome/Edge launched with `--kiosk --noerrdialogs --disable-infobars`. Auto-restart via systemd/service wrapper.
- **Backend** — Lightweight Node.js/Express API. Could be deployed as serverless functions if department prefers.
- **Database** — PostgreSQL for structured data (projects, tags, analytics events). Simple schema, low write volume.
- **Object Storage** — MinIO (on-prem S3-compatible) for images/videos. Avoids DB bloat, enables CDN later.
- **Authentication** — Campus CAS/SSO for admin CMS only. Public kiosk requires no login.
- **Deployment** — Department VM or on-prem Kubernetes. CI/CD via GitLab CI → Docker → target.

---

## Design Artifacts

### 1. Application Architecture (Component Diagram)

```mermaid
graph LR
    subgraph FE["Frontend (React PWA)"]
        App["App.tsx: Routes, Providers"]
        Pages["Pages"]
        Comp["Components"]
        Hooks["Hooks"]
        Utils["Utils"]
    end

    subgraph BE["Backend (Express)"]
        Routes["Routes"]
        Ctrl["Controllers"]
        Svc["Services"]
        Mid["Middleware"]
        Models["Models"]
    end

    subgraph Shared["Shared Types"]
        Types["showcase-types: Project, Tag, AnalyticsEvent"]
    end

    App --> Pages
    Pages --> Comp
    Pages --> Hooks
    Hooks --> Utils
    Comp --> Hooks
    Pages --> API["API Client"]
    API --> Routes
    Routes --> Mid
    Mid --> Ctrl
    Ctrl --> Svc
    Svc --> Models
    Models --> Types
    FE -.-> Types
    BE -.-> Types
```

**Key Components:**
- **`useProjects` hook** — SWR-powered data fetching with cache, deduping, optimistic updates
- **`useKiosk` hook** — Manages idle timer, attract mode, wake lock, fullscreen
- **`ProjectCard` / `ProjectDetail`** — Primary display components, touch-optimized
- **`AttractLoop`** — Ken Burns-style pan/zoom on featured projects during idle
- **`MediaPlayer`** — Wrapper for `<video>` with custom touch controls, PiP support
- **`DemoIframe`** — Sandboxed iframe for interactive projects, CSP headers

---

### 2. Data Model / Entity Relationship Diagram

```mermaid
erDiagram
    PROJECT ||--o{ PROJECT_TAG : has
    PROJECT ||--o{ PROJECT_MEDIA : has
    PROJECT ||--o{ ANALYTICS_EVENT : generates
    TAG ||--o{ PROJECT_TAG : labels
    MEDIA ||--o{ PROJECT_MEDIA : used_in
    ADMIN_USER ||--o{ PROJECT : manages

    PROJECT {
        uuid id PK
        string title
        string description
        string short_description
        int year
        string semester
        string[] team_members
        string[] tech_stack
        string github_url
        string live_demo_url
        string video_url
        boolean featured
        boolean published
        timestamp created_at
        timestamp updated_at
        int display_order
    }

    TAG {
        uuid id PK
        string name UK
        string category
        string color_hex
    }

    PROJECT_TAG {
        uuid project_id FK
        uuid tag_id FK
    }

    MEDIA {
        uuid id PK
        string filename
        string mime_type
        int size_bytes
        int width
        int height
        int duration_sec
        string storage_path
        timestamp uploaded_at
    }

    PROJECT_MEDIA {
        uuid project_id FK
        uuid media_id FK
        string role
        int display_order
    }

    ANALYTICS_EVENT {
        uuid id PK
        uuid project_id FK
        string event_type
        jsonb metadata
        timestamp occurred_at
        string session_id
    }

    ADMIN_USER {
        uuid id PK
        string cas_uid UK
        string name
        string email
        string role
        timestamp last_login
    }
```

**Notes:**
- `PROJECT.tech_stack` stored as JSON array for flexible querying
- `ANALYTICS_EVENT.event_type`: `view`, `detail_open`, `video_play`, `search`, `filter`, `idle_wake`
- `MEDIA.role`: `thumbnail`, `gallery`, `video`, `demo_embed`
- Soft deletes via `published` flag; admin can archive without losing data

---

### 3. Wireframes / Key Screen Mockups

#### A. Attract Mode (Idle Screen)

```
+------------------------------------------------------------------------------+
| SENIOR DESIGN SHOWCASE                                                       |
| CSE Department - Past Capstone Projects                                      |
+------------------------------------------------------------------------------+
|                                                                              |
|  +------------------------------------------------------------------------+  |
|  |                                                                        |  |
|  |                         [HERO IMAGE / VIDEO]                           |  |
|  |                         Ken Burns pan/zoom effect                      |  |
|  |                         1920x1080 full-screen                          |  |
|  |                                                                        |  |
|  +------------------------------------------------------------------------+  |
|                                                                              |
|  +------------------------------------------------------------------------+  |
|  | Project: Autonomous Drone Swarm                                        |  |
|  | Year: 2024  |  Semester: Fall                                          |  |
|  | Tags: [Robotics] [AI/ML] [Embedded] [Computer Vis]                     |  |
|  +------------------------------------------------------------------------+  |
|                                                                              |
|  +------------------------------------------------------------------------+  |
|  |                  Touch anywhere to explore projects >                  |  |
|  +------------------------------------------------------------------------+  |
|                                                                              |
|                            *  o  o  o  o                                     |
|                            (5 projects rotating)                             |
|                                                                              |
+------------------------------------------------------------------------------+
```

#### B. Project Grid (Main Browser)

```
+----------------------------------------------------------------------------+
| SENIOR DESIGN SHOWCASE                                            [Menu]   |
+----------------------------------------------------------------------------+
|                                                                            |
| [Search projects...]                                                       |
|                                                                            |
| [All v]  [Year: All v]  [Category: All v]  [Tech: All v]                   |
|                                                                            |
+----------------------------------------------------------------------------+
|                                                                            |
| +---------------+ +---------------+ +---------------+ +---------------+    |
| |               | |               | |               | |               |    |
| |     [IMG]     | |     [IMG]     | |     [IMG]     | |     [IMG]     |    |
| |               | |               | |               | |               |    |
| +---------------+ +---------------+ +---------------+ +---------------+    |
| | Drone Swarm   | | StudyBuddy    | | CodeViz       | | EcoTrack      |    |
| | 2024 Fall     | | 2024 Fall     | | 2023 Spring   | | 2023 Fall     |    |
| | [Robotics]    | | [Web] [Ed]    | | [DevTools]    | | [IoT] [Env]   |    |
| | [AI/ML]       | |               | |               | |               |    |
| +---------------+ +---------------+ +---------------+ +---------------+    |
|                                                                            |
| +---------------+ +---------------+ +---------------+ +---------------+    |
| |               | |               | |               | |               |    |
| |     [IMG]     | |     [IMG]     | |     [IMG]     | |     [IMG]     |    |
| |               | |               | |               | |               |    |
| +---------------+ +---------------+ +---------------+ +---------------+    |
| | MedScan       | | GameForge     | | SecureChat    | | RoboArm       |    |
| | 2023 Spring   | | 2022 Fall     | | 2022 Spring   | | 2022 Fall     |    |
| | [Health]      | | [Games]       | | [Security]    | | [Robotics]    |    |
| +---------------+ +---------------+ +---------------+ +---------------+    |
|                                                                            |
| +---------------+ +---------------+                                        |
| |               | |               |                                        |
| |     [IMG]     | |     [IMG]     |                                        |
| |               | |               |                                        |
| +---------------+ +---------------+                                        |
| | BlockChain    | | LangLearn     |                                        |
| | 2024 Fall     | | 2023 Spring   |                                        |
| | [Web3]        | | [NLP] [Ed]    |                                        |
| +---------------+ +---------------+                                        |
|                                                                            |
|              Swipe or drag to scroll - Tap card for details                |
|                                                                            |
+----------------------------------------------------------------------------+
| < Prev           Page 1 of 3           Next >          Showing 10 of 27    |
+-----------------------------------------------------------------------------+
```

#### C. Project Detail View

```
+------------------------------------------------------------------------------+
| < Back                                                        [Featured]     |
+------------------------------------------------------------------------------+
|                                                                              |
|  +------------------------------------------------------------------------+  |
|  |                                                                        |  |
|  |                                                                        |  |
|  |                     [HERO IMAGE / VIDEO PLAYER]                        |  |
|  |                                                                        |  |
|  |                     16:9 aspect ratio - Tap for fullscreen             |  |
|  |                                                                        |  |
|  |                                                                        |  |
|  |                       [<<] [||] [>] [>>] [VOL] [FULL] [PiP]            |  |
|  |                                                                        |  |
|  +------------------------------------------------------------------------+  |
|                                                                              |
|  Autonomous Drone Swarm                                      2024 Fall       |
|                                                                              |
|  A coordinated fleet of 10+ drones performing search-and-rescue              |
|  patterns using decentralized consensus algorithms.                          |
|  Built with ROS 2, Python, and custom PCB flight controllers.                |
|                                                                              |
|  ----------------------------------------------------------------------------|
|                                                                              |
|  Team: Alice Chen, Bob Smith, Carol Davis, Dan Lee                           |
|  Tech: [ROS 2] [Python] [C++] [PCB Design] [ML]                              |
|  Tags: [Robotics] [AI/ML] [Embedded] [Research]                              |
|                                                                              |
|  ----------------------------------------------------------------------------|
|                                                                              |
|  GALLERY (swipe to navigate)                                                 |
|                                                                              |
|  < [thumb1] [thumb2] [thumb3] [thumb4] >                   *  o  o  o        |
|                                                                              |
|  ----------------------------------------------------------------------------|
|                                                                              |
|  LINKS                                                                       |
|                                                                              |
|  [GitHub]       https://github.com/team/drone-swarm                          |
|  [Live Demo]    https://drone-swarm.demo.com                                 |
|  [Video]        https://youtube.com/watch/...                                |
|                                                                              |
|  ----------------------------------------------------------------------------|
|                                                                              |
|  +-------------------------------+       +-----------------------------+     |
|  | [QR Code] Save to Phone       |       | [Share]                     |     |
|  +-------------------------------+       +-----------------------------+     |
|                                                                              |
+------------------------------------------------------------------------------+
```

#### D. Admin CMS - Project List

```
+-----------------------------------------------------------------------------+
| [LOGO] Senior Design Showcase - Admin                       Welcome, admin  |
+-----------------------------------------------------------------------------+
|                                                                             |
| +----------------------+  +-----------------------------------------------+ |
| |                      |  |                                               | |
| | MENU                 |  | PROJECTS                    [+ Add Project]   | |
| |                      |  |                                               | |
| | Dashboard            |  +-----------------------------------------------+ |
| |                      |  |                                               | |
| | Projects <- Active   |  | [Search projects...]                          | |
| |                      |  |                                               | |
| | Tags                 |  | [Year] [Tags] [Status]                        | |
| |                      |  | [Published] [Featured] [Sort: Order]          | |
| | Analytics            |  |                                               | |
| |                      |  +----+----------------------+-----+------+----- + |
| | Settings             |  |Sel | Title                |Year | Tags | Fea  | |
| |                      |  +----+----------------------+-----+------+----- + |
| |                      |  |[ ] | Autonomous Drone Swar|2024F|[R][A]|  *   | |
| |                      |  |[ ] | StudyBuddy           |2024F|[Ed]  |      | |
| |                      |  |[ ] | CodeViz              |2023S|[Dev] |  *   | |
| |                      |  |[ ] | EcoTrack             |2023F|[IoT] |      | |
| |                      |  |[ ] | MedScan              |2023S|[HLTH]|  *   | |
| |                      |  |[ ] | GameForge            |2022F|[Game]|      | |
| |                      |  |[ ] | SecureChat           |2022S|[Sec] |      | |
| |                      |  |[ ] | RoboArm              |2022F|[Tech]|      | |
| |                      |  +----+----------------------+-----+------+------+ |
| |                      |  |                                               | |
| |                      |  | [Drag to reorder rows]                        | |
| |                      |  | [Bulk: Feature] [Delete] [Archive]            | |
| |                      |  |                                               | |
| |                      |  | < Prev   1   2   3   4   5   Next >           | |
| |                      |  |                                               | |
| |                      |  | Last sync: 2 min ago                          | |
| |                      |  | 27 projects - 124 media files                 | |
| |                      |  |                                               | |
| +----------------------+  +-----------------------------------------------+ |
|                                                                             |
+-----------------------------------------------------------------------------+
```

---

## Design Decisions & Open Questions

### Significant Design Decisions

| # | Decision | Rationale | Alternatives Considered |
|---|----------|-----------|------------------------|
| D1 | **PWA (React + Vite) over Electron** | Runs in browser kiosk mode; no native build pipeline; easier deployment to department VM; Service Worker gives offline caching for free. | Electron: heavier, requires code signing, auto-update complexity. Native app: platform-specific, harder to maintain. |
| D2 | **Chrome/Edge Kiosk Mode over full-screen app** | Zero-config on managed department machines; IT can push via Group Policy; standard browser engine; security updates automatic. | Custom kiosk OS (e.g., Ubuntu Kiosk): requires imaging, IT involvement. Electron kiosk: same maintenance as PWA but heavier. |
| D3 | **PostgreSQL + MinIO over single DB blob storage** | Separates structured query needs (search, filter, analytics) from large binary assets. MinIO is S3-compatible, supports presigned URLs, easy to swap for cloud later. | MongoDB: less familiar to team, overkill for relational project data. PostgreSQL BYTEA: bloats DB, slow backups, no CDN integration. |
| D4 | **Campus CAS/SSO only for Admin CMS** | No passwords to manage; department staff already have credentials; aligns with university policy. Public kiosk needs zero auth. | Custom auth (email/password): credential management burden. OAuth (Google/GitHub): not university identities. |
| D5 | **Attract Mode as Ken Burns pan/zoom** | Visually engaging without video playback complexity; works with static images; subtle motion prevents screen burn-in; accessible (respects `prefers-reduced-motion`). | Auto-playing video reel: bandwidth, licensing, accessibility issues. Static carousel: looks like current slideshow. Carousel with auto-advance: less dynamic. |

### Open Questions (with Resolution Target)

| # | Question | Target Resolution | Owner |
|---|----------|-------------------|-------|
| Q1 | **What is the exact hardware spec of the DSB 268 display?** (Resolution, touch points, OS, browser version) | **Phase 1 (Week 2)** — Visit site, document specs, test browser kiosk mode | Team Lead |
| Q2 | **Can department IT provide a dedicated VM/container for hosting?** (Specs, network access, SSL certs, domain) | **Phase 1 (Week 3)** — Request via department IT ticket | Team Lead |
| Q3 | **What is the format/volume of existing senior project archive data?** (Spreadsheets, PDFs, images, videos — how many years, how many projects?) | **Phase 1 (Week 2)** — Meet with department admin/archivist | Content Lead |
| Q4 | **Are there any interactive projects that need special embedding?** (Unity WebGL, WebAssembly, custom servers, hardware demos) | **Phase 2 (Week 6)** — Inventory during data import | Dev Lead |
| Q5 | **Does the client want analytics exposed to students/faculty, or only admins?** (Public dashboard vs. admin-only) | **Phase 2 (Week 7)** — Confirm with client at Sprint 2 review | Team Lead |
| Q6 | **What is the department's policy on external links from the kiosk?** (GitHub, live demos — allowed? sandboxed? blocked?) | **Phase 2 (Week 5)** — Security review with IT | Dev Lead |
| Q7 | **Screen burn-in mitigation: is subtle motion sufficient, or does IT require pixel-shifting?** | **Phase 3 (Week 11)** — Test 48-hr run on actual hardware | Dev Lead |

---

## Related Pages

- [📋 Project Roadmap](Roadmap.md) — Phases, timeline, and milestones
- [📝 Requirements](Requirements.md) — Detailed user stories and acceptance criteria
- [🏠 Wiki Home](Home.md) — Project overview and quick links

---

*Design version 1.0 — Created November 2026. Decisions and open questions will be updated as the project progresses.*