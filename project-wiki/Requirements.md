# Requirements — Senior Design Showcase Display

*Linked from [Design Materials](Design.md)*

---

## User Stories

### Epic 1: Public Kiosk Experience

| ID | User Story | Acceptance Criteria | Priority |
|----|------------|---------------------|----------|
| US-1.1 | As a **prospective student**, I want to **browse projects by category** so I can find projects in my interest area. | - Grid shows filter chips for Category, Year, Tech Stack<br>- Selecting a filter updates results instantly<br>- Multiple filters can be combined<br>- "Clear filters" resets all | Must-have |
| US-1.2 | As a **visitor**, I want to **search for projects by keyword** so I can find specific topics. | - Search input with debounce (300ms)<br>- Searches title, description, team names, tags<br>- Results highlight matched terms<br>- Empty state shows helpful message | Must-have |
| US-1.3 | As a **tour participant**, I want to **view project details** including images, video, and links. | - Tap card → full-screen detail view<br>- Shows: title, year, team, description, tech tags<br>- Hero image/video at top<br>- Gallery of additional images (swipeable)<br>- Video plays inline with custom controls<br>- External links (GitHub, Live Demo) open in new tab<br>- Back gesture/tap returns to grid | Must-have |
| US-1.4 | As a **kiosk user**, I want **touch navigation to feel natural** so I can explore without instruction. | - Tap targets ≥ 48×48px<br>- Swipe to scroll grid (momentum scrolling)<br>- Swipe between detail images<br>- Pinch-to-zoom on images<br>- Edge swipe or back button to go back<br>- No browser UI visible (kiosk mode) | Must-have |
| US-1.5 | As a **passerby**, I want the **display to show something engaging when idle** so I'm drawn to interact. | - After 60s no touch: attract mode activates<br>- Cycles featured projects with Ken Burns effect<br>- Shows project title, year, tags<br>- "Touch anywhere to explore" prompt<br>- Any touch immediately returns to grid<br>- Respects `prefers-reduced-motion` | Must-have |
| US-1.6 | As a **visitor with phone**, I want to **save a project to view later** via QR code. | - Detail view shows QR code button<br>- QR encodes deep link to project<br>- Scanning opens project on phone browser<br>- Works without app install | Could-have |

### Epic 2: Admin Content Management

| ID | User Story | Acceptance Criteria | Priority |
|----|------------|---------------------|----------|
| US-2.1 | As a **department admin**, I want to **log in with my campus credentials** so I don't need another password. | - "Login" redirects to CAS<br>- Successful CAS → returns to CMS dashboard<br>- Session persists across browser restarts<br>- Logout clears session | Must-have |
| US-2.2 | As an **admin**, I want to **add a new project** with all metadata and media. | - Form: title, year, semester, description, short description, team, tech stack (multi-select), tags (multi-select), GitHub URL, Live Demo URL, featured toggle<br>- Drag-drop multiple images/videos<br>- Auto-generates thumbnails<br>- Validation: required fields, URL format, file types<br>- Save as draft or publish immediately | Must-have |
| US-2.3 | As an **admin**, I want to **edit/reorder/delete projects** so the showcase stays current. | - List view with drag-to-reorder (updates `display_order`)<br>- Inline edit for quick changes<br>- Full edit form pre-populated<br>- Archive (soft delete) vs. permanent delete<br>- Bulk actions: feature, archive, delete | Must-have |
| US-2.4 | As an **admin**, I want to **manage tags and categories** so filtering stays organized. | - Separate tag management page<br>- Create/edit/delete tags<br>- Assign color per tag for UI<br>- Tag categories: Year, Domain, Tech Stack, Type<br>- Prevent deletion if tag in use (or reassign) | Should-have |
| US-2.5 | As an **admin**, I want to **see usage analytics** to know what projects get attention. | - Dashboard: total views, unique sessions, avg session time<br>- Top 10 projects by views<br>- Views by category/year<br>- Search queries (top terms, zero-result queries)<br>- Date range selector<br>- Export CSV | Should-have |

### Epic 3: System Reliability & Operations

| ID | User Story | Acceptance Criteria | Priority |
|----|------------|---------------------|----------|
| US-3.1 | As **IT staff**, I want the **kiosk to auto-recover from crashes** so it runs unattended. | - Browser crash → restarts to attract mode in <10s<br>- Network loss → shows cached data, retries silently<br>- Memory >80% → proactive restart<br>- System reboot → kiosk launches automatically<br>- Logs rotated, accessible via SSH | Must-have |
| US-3.2 | As **IT staff**, I want **deployment to be simple and repeatable**. | - Single Docker Compose or Kubernetes manifest<br>- Environment variables for config<br>- DB migrations run automatically<br>- Health check endpoint (`/health`)<br>- Rollback via previous image tag | Must-have |
| US-3.3 | As a **developer**, I want **local dev to mirror production** so I catch issues early. | - `docker-compose up` spins full stack<br>- Hot reload for frontend and backend<br>- Mock CAS for local auth<br>- Seed script for sample data | Must-have |

---

## Non-Functional Requirements

| Category | Requirement | Target |
|----------|-------------|--------|
| **Performance** | Initial load (attract mode) | <3s on kiosk hardware |
| | Grid scroll FPS | 60fps sustained |
| | Detail view transition | <200ms |
| | Search results | <300ms |
| **Reliability** | Uptime (unattended) | 99.9% over 30 days |
| | Crash recovery time | <10s |
| | Data persistence | Zero data loss on crash |
| **Accessibility** | WCAG 2.1 AA (admin CMS) | Compliant |
| | Touch target size | ≥48×48px |
| | Contrast ratio (kiosk) | ≥4.5:1 (high contrast mode) |
| | Reduced motion | Respected in attract mode |
| **Security** | Admin auth | Campus CAS only |
| | API rate limiting | 100 req/min per IP |
| | File upload validation | Type, size, malware scan |
| | CSP headers | Strict policy |
| **Browser Support** | Kiosk target | Chrome/Edge latest stable |
| | Admin CMS | Chrome, Firefox, Safari, Edge (last 2 versions) |
| **Data** | Project capacity | 500+ projects |
| | Media per project | 20 images + 1 video |
| | Video max size | 500MB |
| | Analytics retention | 2 years |

---

## Technical Constraints

| Constraint | Detail |
|------------|--------|
| **Display Hardware** | Large touch monitor (exact specs TBD — see [Design Open Question Q1](Design.md#open-questions)) |
| **OS** | Windows 10/11 (department standard) or Ubuntu (if IT provisions) |
| **Kiosk Browser** | Chrome Enterprise or Edge in kiosk mode (Group Policy managed) |
| **Network** | Wired Ethernet only; no Wi-Fi on display machine |
| **Hosting** | Department on-prem VM or Kubernetes (no public cloud without approval) |
| **Auth** | Campus CAS/SSO (SAML/OIDC) for admin only |
| **Budget** | $0 for hosting (department infrastructure); $0 for licenses (open source only) |
| **Timeline** | Semester 1 demo: Week 13; Final: Week 25 |

---

## Data Requirements

### Project Import Format (Minimum Viable)
```csv
title,year,semester,description,short_description,team_members,tech_stack,tags,github_url,live_demo_url,video_url,featured
"Autonomous Drone Swarm",2024,Fall,"Full description...","A coordinated fleet...","Alice Chen;Bob Smith","ROS 2;Python;C++","Robotics;AI/ML;Embedded","https://github.com/...","https://demo.com","https://video.com",true
```

### Media Naming Convention
- `{project-id}-thumb.{ext}` — 400×300, used in grid
- `{project-id}-hero.{ext}` — 1920×1080, used in detail/attract
- `{project-id}-gallery-{n}.{ext}` — additional images
- `{project-id}-demo.{ext}` — video file (MP4, H.264)

---

## Acceptance Criteria for Major Milestones

### Semester 1 Demo (Week 11) — "Walking Skeleton"

- [ ] Kiosk boots to attract mode on target hardware
- [ ] Touch navigation: grid → detail → back works fluidly
- [ ] 10+ sample projects with images load from CMS
- [ ] Search and filter functional
- [ ] Admin can log in via CAS and add/edit a project
- [ ] No console errors; 60fps scrolling
- [ ] Client accepts at sprint review

### Semester 2 Midpoint (Week 18)

- [ ] Video playback in detail view and attract mode
- [ ] Interactive demo iframe embedding (at least 1 working example)
- [ ] Analytics dashboard shows live data
- [ ] Accessibility audit passed (WCAG AA for admin)
- [ ] 48-hour unattended stability test passed

### Final Release (Week 25)

- [ ] Deployed on DSB 268 display hardware
- [ ] All real project data imported and verified
- [ ] Admin training completed (recorded session + docs)
- [ ] Handoff document: architecture, deployment, troubleshooting, contacts
- [ ] Client signs off on final acceptance

---

## Traceability Matrix

| Feature | User Stories | Design Artifacts | Roadmap Phase |
|---------|-------------|------------------|---------------|
| Project Grid | US-1.1, US-1.2 | Wireframe B, Data Model | Phase 2-3 |
| Detail View | US-1.3 | Wireframe C, Data Model | Phase 3 |
| Touch Navigation | US-1.4 | Architecture (useKiosk hook) | Phase 1, 3 |
| Attract Mode | US-1.5 | Wireframe A, Architecture | Phase 3 |
| QR Codes | US-1.6 | Wireframe C | Phase 4 (could) |
| Admin Auth | US-2.1 | Architecture (CAS) | Phase 2 |
| Project CRUD | US-2.2, US-2.3 | Wireframe D, Data Model | Phase 2-3 |
| Tag Management | US-2.4 | Data Model (TAG) | Phase 4 |
| Analytics | US-2.5 | Data Model (ANALYTICS_EVENT) | Phase 4 |
| Auto-Recovery | US-3.1 | Architecture (watchdog) | Phase 5 |
| Deployment | US-3.2 | Architecture (Docker/K8s) | Phase 5-6 |
| Local Dev | US-3.3 | Architecture (docker-compose) | Phase 1 |

---

*Requirements version 1.0 — Created November 2026. Update with each sprint review.*