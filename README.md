<h1 align="center">Akarsh Singh</h1>

<p align="center">
  <b>Software Engineer.</b> I build production systems end to end:<br/>
  database schema, queries, APIs, and the interface on top.
</p>

<p align="center">
  <a href="https://akarshsingh-portfolio.netlify.app/"><img src="https://img.shields.io/badge/Portfolio-c770f0?style=for-the-badge&logo=firefox&logoColor=white" alt="Portfolio"/></a>
  <a href="https://www.linkedin.com/in/akarsh-singh-24436a243/"><img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn"/></a>
  <a href="https://leetcode.com/u/Akarsh_Singh_2211/"><img src="https://img.shields.io/badge/LeetCode_Knight-FFA116?style=for-the-badge&logo=leetcode&logoColor=black" alt="LeetCode"/></a>
  <a href="https://codeforces.com/profile/Unknown_2211"><img src="https://img.shields.io/badge/Codeforces_Specialist-1F8ACB?style=for-the-badge&logo=codeforces&logoColor=white" alt="Codeforces"/></a>
  <a href="mailto:akarshs145@gmail.com"><img src="https://img.shields.io/badge/Email-EA4335?style=for-the-badge&logo=gmail&logoColor=white" alt="Email"/></a>
</p>

<p align="center">
  <img src="https://akarshsingh-portfolio.netlify.app/og-image.png" width="720" alt="Akarsh Singh, Software Engineer"/>
</p>

---

## The 30 second version

Software Engineer at **Varuna Sentinels B.V.** (Netherlands HQ) since June 2025, working on a
B2B procurement platform with buyer, supplier and administrator portals. I report to the CTO
and own features end to end. I design the tables, write the stored procedures, build the APIs,
ship the Angular interface, and then go back and make it fast.

| | |
|---|---|
| ⚡ **48s to 8s** | Rebuilt the heaviest screen in the product. Three more improved by 62 to 71 percent. |
| 🧮 **O(M×N) to O(M+N)** | Correlated subquery replaced with single pass JSON aggregation. |
| 🔑 **Zero accounts needed** | HMAC-SHA256 short tokens exchanged for scoped, short lived JWTs, so external suppliers transact without ever signing up. |
| 🤖 **3 roles, 1 data layer** | Role scoped LLM tooling where a supplier's assistant *structurally* cannot read a buyer's data. |
| 📦 **40+ modules** | Shipped end to end across all three portals. |
| 🛡️ **Cross tenant leak found and fixed** | Traced to a hardcoded identifier. Onboarding hardened into a single rollback safe transaction. |
| 🗓️ **14 months** | Continuous production delivery since June 2025. |

📖 **Deep write ups** with architecture diagrams, rejected alternatives and trade off tables:
[48s to 8s](https://akarshsingh-portfolio.netlify.app/blog/page-load-optimization) ·
[Passwordless access](https://akarshsingh-portfolio.netlify.app/blog/jwt-hmac-token-auth) ·
[Role scoped AI assistant](https://akarshsingh-portfolio.netlify.app/blog/ai-chatbot-gemini) ·
[Real time messaging](https://akarshsingh-portfolio.netlify.app/blog/websocket-chat-system)

---

## How the scope grew

Ordered by what I was trusted to own, not by ticket. Everything below is generalized, with no
internal identifiers, schema names or business rules.

<details>
<summary><b>Jun to Aug 2025 · A reusable data movement layer</b></summary>

<br/>

Bulk import and export was needed on almost every screen, and each one was being built by
hand. I designed a single reusable pattern and rolled it across the catalogue.

* Excel template generation with **ExcelJS**, structured for non technical business users
* Row level validation producing readable per row errors rather than one opaque failure
* Lookup resolution for foreign keys such as categories, brands, units, regions and ports, so
  users paste names and the system resolves identifiers
* **Upsert semantics**, so re importing a corrected sheet updates rather than duplicates
* Multi table inserts wrapped in **Sequelize transactions**, so a bad row never leaves partial data
* **Batch processing** introduced after large uploads hit gateway timeouts. Chunking removed
  the ceiling without changing the request path
* Reusable stored procedures driven by a dynamic operation type, covering add, archive,
  restore, fetch, count and export from one place
* Applied across **25+ business modules**: users, suppliers, buyers, products, services,
  branches, categories, brands, ports, units of measure, sustainability commitments, ISO
  certificates, vessels and tax profiles
* Fixed international data problems along the way. Postcode validation was rewritten for
  alphanumeric formats, and region codes were reconciled against real ISO mappings

</details>

<details>
<summary><b>Aug to Dec 2025 · Workflow and lifecycle systems</b></summary>

<br/>

Procurement is a state machine. A request becomes a quotation, becomes an order, becomes an
invoice, becomes a payment. I built the tracking layer for that lifecycle.

* Lifecycle status tracking with **transactional state transitions and rollback**, so a failed
  step never leaves a record half moved
* Existence checks before insert or update to keep transitions idempotent
* Automatic detection of lapsed requests based on elapsed time against the request date
* Status trackers in the UI with stage aware colour coding, animated transitions and
  responsive timelines
* Buyer, supplier and administrator **dashboards** with KPI cards, Chart.js analytics, count up
  animations via `requestAnimationFrame`, and charts deferred with `IntersectionObserver` so
  off screen ones never render
* Trend analytics over 7 day, 30 day, 2 month and 4 month windows using recursive CTEs
* **Notification infrastructure** with flag gated delivery integrated across 20+ distinct
  workflow events, so a notification is only written when its flag is active
* Push notifications, relative "time ago" formatting, unread counts and dashboard badges
* Full **support ticket system** across buyer, supplier and admin: creation, attachments via S3
  signed URLs, status workflow with controlled transitions, per status counts, filtering,
  pagination, Excel export and SES email notifications

</details>

<details>
<summary><b>Sep 2025 onward · Performance engineering</b></summary>

<br/>

Four core screens took 35 to 48 seconds to load. I profiled them, found the same three causes
each time, and built a method rather than a patch.

**Results**

| Screen | Before | After | Change |
|---|---|---|---|
| Supplier quotation | 48s | 8s | **83% faster** |
| Supplier request list | 35s | 10s | **71% faster** |
| Buyer request list | 40s | 14s | **65% faster** |
| Buyer quotation | 40s | 15s | **62% faster** |

**The method, applied in order and measured after each step**

1. **Consolidate.** Audit every request the screen fires and merge the overlapping ones.
   Twelve calls became three.
2. **Project.** Select only the columns the view renders, and drop every join that no longer
   feeds one. One list query was joining nine tables to display five columns.
3. **Aggregate in SQL.** Replace correlated subqueries and application side stitching with
   `JSON_ARRAYAGG`, `JSON_OBJECT` and `COALESCE`, turning **O(M×N) into O(M+N)**.
4. **Index the filter paths.** Covering indexes for the columns actually used in search, sort
   and pagination.
5. **Defer.** Lazy load below the fold sections, and render charts only on viewport entry.

**Then I went hunting for the same class of defect**

* Recursive CTEs used for date series generation replaced with non recursive equivalents, then
  every remaining procedure audited for unsafe recursion rather than assuming
* **Around 56 search predicates** were failing at runtime on collation mismatches between
  differently configured columns, resolved with explicit collation at the comparison
* Gateway timeouts on bulk operations removed by batching
* Angular side wins including `trackBy`, pre sanitisation and console cleanup. An inbox that
  had been rendering slowly now loads instantly

**What I would do differently.** None of this was caught by instrumentation, because there was
none. The screens got slow gradually and nothing objected. Given the same system again I would
put timing on the read paths *before* optimising a single one of them.

</details>

<details>
<summary><b>Oct 2025 to Feb 2026 · Access, identity and security</b></summary>

<br/>

External suppliers needed to act on documents without holding portal accounts. I designed the
token system that made that safe.

**The two token design**

* Short, opaque tokens derived by **HMAC-SHA256** over document, recipient and issue time,
  keyed by a server secret, then truncated to stay readable inside an email
* Token, resolved context and expiry are persisted. The token is a lookup key rather than a
  payload, so a leaked URL discloses nothing and access can be revoked server side
* On redemption, expiry and revocation are checked once, then exchanged for a **short lived
  JWT** scoped to exactly that document and supplier. Every request after that verifies
  statelessly with no database round trip
* Two middleware layers guard every guest route. One verifies the session, the other **injects**
  document and supplier identity from the token context, overwriting whatever the client sent.
  Validation can be bypassed by a request you did not anticipate. Injection cannot
* Truncation collisions are handled with a uniqueness constraint plus regeneration, so a
  collision becomes an internal retry rather than a supplier seeing someone else's document

**Where it shipped**

* Guest access to purchase order actions: accept, reject and acknowledge
* A complete **guest quoting flow** covering request details, line items, payment terms,
  preview, draft save, attachment upload and delete, and decline with reason. Every endpoint
  received a token scoped twin behind the same middleware pair
* Public compliance document submission via magic link, with no account required
* Standalone layouts with application chrome removed, so a guest never sees a session that
  does not exist

**Identity and hardening**

* Registration rebuilt as a **single transaction** spanning identity provider and database,
  covering address, company, user, contacts and product mappings, with rollback on partial
  failure including a cleanup path for the identity provider when a later step fails
* Temporary password issuance and onboarding email flow
* Guarded execution with fail fast validation at every step, ending a class of half created
  supplier bugs
* **Found and fixed a cross tenant data exposure** caused by a hardcoded identifier
* Diagnosed a production only JWT signing failure. The secret existed in one environment's
  config and was missing from the others. Nothing was wrong with the code. A security feature
  that fails closed will fail closed in production too, and config parity deserves the same
  review attention as the code depending on it

</details>

<details>
<summary><b>Nov 2025 onward · AI systems</b></summary>

<br/>

First a natural language analytics assistant over the dashboards, then something considerably
more ambitious.

**Why it is hard here.** This platform has three roles, and buyers and suppliers are commercial
counterparties negotiating against each other. A supplier learning what a buyer paid a
competitor is not a bad answer. It is a data breach with a conversational interface in front of
it. Correctness was never the constraint. **Isolation was.**

**Architecture, four layers, so the security boundary lives in one place**

| Layer | Responsibility |
|---|---|
| Orchestration | Routes the request, invokes the model, selects a fallback, assembles the reply |
| Registration | Builds the tool manifest *for this role*. A supplier session is never offered buyer tools, so the model cannot call what it was never shown |
| Execution | Validates arguments, clamps ranges, filters fields to the tool's allowlist |
| Retrieval | Queries the data layer, always parameterised by identity resolved from the authenticated session |

**Three rules that make it safe**

1. **Identity comes from the session, never the model.** The model proposes which tool and
   which filters, but can never supply the identifier deciding whose data is read. A
   hallucinated or injected identifier has nowhere to land.
2. **Tools are read only.** The assistant reports on orders, approvals and compliance state. It
   cannot approve, reject, cancel or pay. The worst case from a fully successful prompt
   injection is an incorrect sentence.
3. **Fields are allowlisted, not denylisted.** Banking details, margins and counterparty pricing
   never enter model context at all, rather than relying on a prompt to avoid them.

**Also built**

* Role aware prompts and terminology, so buyers and suppliers get genuinely different assistants
* **Quick actions** that skip the model entirely and route straight to a filtered screen. They
  are faster, cannot hallucinate, and cost nothing. Not every question deserves inference
* **Deterministic fallbacks** computed from the same data the tools read, so an outage degrades
  to a plainer answer rather than an error banner
* Supplier evaluation, catalogue guidance and sustainability compliance actions with deep
  linked calls to action
* Draggable chat widget with edge snapping, a `contenteditable` auto expanding composer, typing
  indicators, relative timestamps and role gated visibility
* Migrated from the managed model endpoint to the direct client library after repeated
  credential format authentication failures, which introduced an ESM into CommonJS module
  problem handled with a dynamic import at the boundary
* Streaming via SSE designed and documented for future rollout

**What is missing.** Tool selection is verified by hand. There is no regression suite proving a
prompt still routes correctly after a manifest change. An evaluation set is the first thing I
would add.

</details>

<details>
<summary><b>Aug 2025 to Jul 2026 · Real time messaging</b></summary>

<br/>

Negotiation was happening over email, outside the system holding the documents being
negotiated. Nobody could reconstruct why a quote changed.

* Built on **API Gateway WebSockets and Lambda** rather than a persistent server. The platform
  already ran serverless, so this added no infrastructure to operate, at the cost of
  externalising all connection state and losing in process broadcast. For two party negotiation
  threads that trade is cheap. For high fan out it would not be
* **Persist first, deliver second.** A message is written durably before any push is attempted,
  so a failed push, a stale connection or an absent recipient costs latency, never a message
* Connections keyed by **user identity rather than connection id**, because a new id is issued
  on every reconnect and mobile clients reconnect constantly
* Delivery failure treated as a signal to drop a stale connection record, not as an error
* Server clock timestamps, ending an ordering bug that only appeared between participants in
  different time zones
* Debounced typing indicators after per keystroke events flooded the socket
* Client side keyed session map with optimistic rendering, reconciled on server confirmation
* Later merged **three separate conversation sources into one unified inbox**, grouped by
  source, with unread counts aggregated and category filters to narrow back down

**Known limitation.** Delivery is at most once past the persistence boundary, because nothing
retries a failed push. That is acceptable for negotiation threads. The fix for anything time
critical is an outbox the delivery path drains.

</details>

<details>
<summary><b>Documents, communications and compliance</b></summary>

<br/>

* **PDF generation** with PDFMake for purchase orders, quotations and request documents, with
  structured layouts, line item tables, totals, incoterms, freight forwarder and port agent
  sections, watermarks, branding and legal terms
* Generated documents uploaded to **S3** with view links embedded in outbound email
* **Around 20 lifecycle email templates** plus compliance and terms templates, built on
  reusable builders and shared content blocks rather than copy paste
* Email safe HTML using table based layouts and inline CSS, validated across **Gmail, Outlook
  and Apple Mail**, including working around client specific CSS limits and broken Unicode icons
* Multi approver email flow with CC, consolidated into one dispatch helper that removed
  duplicate sends
* **Shipped a full PDF layout redesign behind an environment based fallback switch**, so a risky
  visual change could be rolled back instantly. Fixed overflow, long value wrapping and page
  break behaviour without touching the data flow
* **KYC compliance platform** with parent and child schema, lifecycle states, document slot
  mapping, admin review covering approve, reject and request clarification, and a **public magic
  link submission page** letting external parties upload verification documents with no account
* **Digital signatures** with canvas based capture, responsive scaling, draw and upload modes,
  and PDF embedding
* Replaced hardcoded asset URLs with configuration resolved paths across every module, and moved
  logos to CDN delivery so emails render correctly in all environments

</details>

<details>
<summary><b>Feb to Aug 2026 · Invoicing, marketplace and spare parts</b></summary>

<br/>

* **Invoicing across buyer, supplier and admin** with dynamic line items, real time tax and
  total recalculation, bank detail selection, validation gates before send, PDF generation,
  email dispatch, archive and unarchive, payment proof upload and status tracking
* **Commission invoicing** with supplier wise grouping, a commission calculation engine,
  currency conversion through an exchange rate service with caching, and full payment lifecycle
* **Rolling quarter fleet billing** for buyers, with vessel eligibility checks, invoice reference
  generation and bank detail snapshotting
* **Admin dashboard** with KPI cards, graphs, a recent activity feed and custom date range
  reporting with dynamic chart bucketing
* **Marketplace** covering category browsing, cart, guest to user checkout, and product and
  service request creation
* **Service request and quotation subsystem** with a six table schema and foreign key integrity,
  transactional creation with rollback, JSON driven cart to request mapping, autosave on tab
  change, supplier assignment persisted through a mapping table, and a full quotation flow with
  a cost calculation engine and attachment handling
* **Spare parts vertical, owned end to end.** Supplier submission with engineering details,
  custom specifications and technical drawings. A dedicated admin review queue. Approve and
  reject with duplicate approval protection and mandatory rejection reasons. Attachment and
  attribute promotion on approval. And **immediate search index updates instead of waiting for
  the hourly indexing job**, so approved parts are findable at once
* Organisation name change request workflow with modal, validation, backend APIs and branded email
* Authored a **competitive analysis** against a rival platform covering buyer and supplier
  workflows, feature coverage and compliance capability, and presented it to the team
* Documented every major service layer with docblocks covering workflows, parameters and
  dependencies, to cut onboarding and debugging time
* Diagnosed repeated Angular build crashes to memory exhaustion and stabilised the toolchain

</details>

---

## 🔨 Currently building

**RoomBridge**, a short stay booking platform.

Most booking side projects start with the listing grid because it is the fun part. I started
underneath it: a normalized relational schema, a real token lifecycle, and verification that
actually verifies.

**Working today**

* Prisma and PostgreSQL schema across users, listings, bookings, reviews, refresh tokens and
  one time codes, with cascade rules on ownership
* Access and refresh tokens signed with **separate secrets**, and refresh tokens persisted so a
  session can be revoked server side rather than only expiring
* Email verification with one time codes **hashed before storage** and expired after five
  minutes, so a leaked database row yields no usable code
* Role based authorization middleware, Zod validation and typed error handling at every service
  boundary
* Modular router driven by a route manifest, so adding a domain never means editing a growing
  switch statement

**Next**

* Listing create, update, deactivate and filtered browse
* Date range availability with overlap constraints enforced **in the database** rather than only
  the service layer, so two concurrent requests cannot double book
* Reviews gated on a completed stay, so a rating cannot come from someone who never booked

---

## 🧰 Tech

**Production.** Shipped and maintained in a live system.

![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=flat-square&logo=nodedotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express-000000?style=flat-square&logo=express&logoColor=white)
![Angular](https://img.shields.io/badge/Angular-DD0031?style=flat-square&logo=angular&logoColor=white)
![RxJS](https://img.shields.io/badge/RxJS-B7178C?style=flat-square&logo=reactivex&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat-square&logo=mysql&logoColor=white)
![Sequelize](https://img.shields.io/badge/Sequelize-52B0E7?style=flat-square&logo=sequelize&logoColor=white)
![AWS Lambda](https://img.shields.io/badge/AWS_Lambda-FF9900?style=flat-square&logo=awslambda&logoColor=white)
![API Gateway](https://img.shields.io/badge/API_Gateway-FF4F8B?style=flat-square&logo=amazonapigateway&logoColor=white)
![S3](https://img.shields.io/badge/S3-569A31?style=flat-square&logo=amazons3&logoColor=white)
![SES](https://img.shields.io/badge/SES-DD344C?style=flat-square&logo=amazonsimpleemailservice&logoColor=white)
![Cognito](https://img.shields.io/badge/Cognito-DD344C?style=flat-square&logo=amazoncognito&logoColor=white)
![Serverless](https://img.shields.io/badge/Serverless-FD5750?style=flat-square&logo=serverless&logoColor=white)
![WebSockets](https://img.shields.io/badge/WebSockets-010101?style=flat-square&logo=socketdotio&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-000000?style=flat-square&logo=jsonwebtokens&logoColor=white)
![Gemini](https://img.shields.io/badge/Google_Gemini-8E75B2?style=flat-square&logo=googlegemini&logoColor=white)
![Chart.js](https://img.shields.io/badge/Chart.js-FF6384?style=flat-square&logo=chartdotjs&logoColor=white)
![Git](https://img.shields.io/badge/Git-F05032?style=flat-square&logo=git&logoColor=white)

**Working knowledge.** Built real features with it, though not my daily driver.

![React](https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=61DAFB)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white)
![Redux](https://img.shields.io/badge/Redux-764ABC?style=flat-square&logo=redux&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-2D3748?style=flat-square&logo=prisma&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=flat-square&logo=mongodb&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![Tailwind](https://img.shields.io/badge/Tailwind-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white)
![C++](https://img.shields.io/badge/C%2B%2B-00599C?style=flat-square&logo=cplusplus&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=flat-square&logo=openjdk&logoColor=white)

**Things I reach for.** Stored procedures · query optimisation · schema design · transactional
integrity · role based access control · ExcelJS · PDFMake

**Foundations.** Data Structures and Algorithms · System Design · DBMS · Operating Systems ·
Computer Networks · OOP · Concurrency

---

## 📂 Repositories

| Project | What it is | Stack |
|---|---|---|
| **[RoomBridge](https://github.com/Akarsh2012)** | Short stay booking platform. Identity layer complete, booking domain in progress. | TypeScript · Prisma · PostgreSQL · Next.js |
| **[Subscription Tracker API](https://github.com/Akarsh2012/Subscription-Tracker-API)** | Production shaped backend with JWT auth, RBAC, bot protection, scheduled email reminders and centralised error handling. | Node.js · Express · MongoDB |
| **[Shortify](https://github.com/Akarsh2012/Shortify)** | URL shortener with real time click analytics, JWT sessions and protected routes. | Node.js · Express · MongoDB |
| **[ImaginIQ-AI](https://github.com/Akarsh2012/ImaginIQ)** | AI image generation with Cloudinary backed storage and delivery. | MERN · Cloudinary |
| **[HeadlinesHub](https://github.com/Akarsh2012/HeadlinesHub)** | News reader with category filtering and responsive layout. | React |
| **[N-Queens Visualiser](https://github.com/Akarsh2012/N-Queens-Visualiser)** | Recursion and backtracking stepped through visually. | JavaScript |
| **[Weather App](https://github.com/Akarsh2012/Weather-App--by-Akarsh)** | Live weather lookup with dynamic rendering. | JavaScript |
| **[Password Generator](https://github.com/Akarsh2012/Random-Password-Generator)** | Constraint based generation with a strength indicator. | JavaScript |
| **[Company Web Page](https://github.com/Akarsh2012/Company-Web-Page)** | Responsive multi section marketing site. | HTML · CSS · JS |
| **[To-Do List](https://github.com/Akarsh2012/To-Do-List)** | Task CRUD with local storage persistence. | JavaScript |

---

## 💼 Experience

**Software Engineer**, Varuna Sentinels B.V. · *Jun 2025 to Present* · Hybrid, Netherlands HQ
B2B procurement platform across buyer, supplier and administrator portals. Reporting to the CTO.

**SDE Intern**, Bluestock · *Apr to May 2025* · Remote
Built REST APIs supporting data synchronisation across web modules. Implemented backend
services in Node.js and MongoDB, cutting average response time by **25 percent**. Raised test
coverage to **40 percent** with Jest across unit and integration suites.

---

## 🏆 Achievements and education

* **LeetCode Knight**, global rank around 700 in Biweekly Contest 139
* **Codeforces Specialist**, peak rating 1432
* **1st place**, CircuitBuzz hackathon (Avishkar)
* **B.Tech, Electrical Engineering**, MNNIT Allahabad, 2021 to 2025

I studied electrical engineering and finished fairly sure I wanted to write software instead.
Competitive programming was the route across, and more usefully, it taught me to sit with a
problem I cannot immediately solve.

---

## 📊 GitHub

<p align="center">
  <img src="https://github-profile-summary-cards.vercel.app/api/cards/repos-per-language?username=Akarsh2012&theme=radical" alt="Languages"/>
</p>

> Most of my day to day engineering lives in **private company repositories**, so this profile
> reflects personal projects rather than total output. The production work, 40+ modules across
> three portals over 14 months, is written up on
> [my portfolio](https://akarshsingh-portfolio.netlify.app/).

---

<p align="center">
  <i>Currently exploring distributed systems, event driven architecture,<br/>
  and making LLM tooling safe enough to put in front of real commercial data.</i>
</p>

<p align="center">
  <a href="mailto:akarshs145@gmail.com">akarshs145@gmail.com</a> ·
  <a href="https://akarshsingh-portfolio.netlify.app/">Portfolio</a> ·
  <a href="https://www.linkedin.com/in/akarsh-singh-24436a243/">LinkedIn</a>
</p>
