# MealFlow — Product Documentation

> **Status:** MVP live, no paying users · **Stage:** Early · **Geography:** Africa · **Team:** Solo founder

---

## Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [User Pain Points](#2-user-pain-points)
3. [Product Overview](#3-product-overview)
4. [Product Requirements Document (PRD)](#4-product-requirements-document-prd)
5. [Architectural Decisions](#5-architectural-decisions)
6. [Product Roadmap](#6-product-roadmap)

---

## 1. Problem Statement

### Background

Corporate meal ordering across Africa — particularly in Nigeria — is a largely unstructured, manual process. Companies that provide daily meals for employees rely on informal channels: WhatsApp group chats, phone calls, and handwritten lists passed between a company admin and a food vendor. This works at very small scale. It breaks down quickly as headcount grows.

The problem is firsthand. As the operator of Resswell Catering, a Lagos-based catering business, I experienced this directly — taking meal orders from multiple corporate clients over WhatsApp, reconciling individual member choices into prep lists manually, handling last-minute changes, and dealing with the downstream chaos of missed orders, wrong quantities, and no audit trail. Every day was a coordination problem dressed up as a food problem.

### The Core Problem

**There is no structured, purpose-built system for restaurant-to-company meal coordination in African markets.** The tools that exist (generic food delivery apps, spreadsheets, WhatsApp) were not designed for this workflow and create compounding inefficiencies on both sides of the transaction.

On the **restaurant/caterer side:**
- No visibility into how many orders are coming until the last minute
- No structured breakdown of meal preferences (protein, side, swallow choices)
- No reliable cutoff enforcement — members submit after prep has started
- No historical data on ordering patterns to plan inventory

On the **company side:**
- Admins spend significant time aggregating orders manually
- No formal approval flow for new members
- No audit trail of who ordered what and when
- No mechanism to handle absences, substitutions, or auto-assignments

### Why Now

Africa's formal employment sector is growing. More companies are offering employee meal benefits as part of compensation. The infrastructure for structured B2B SaaS — smartphones, mobile payments, cloud services — is now accessible enough to build on. The problem is ready to be solved, and no focused product exists to solve it.

---

## 2. User Pain Points

MealFlow serves two distinct user types on a two-sided platform. Each has a different set of pain points.

### 2.1 Restaurant / Caterer

| Pain Point | Description | Severity |
|---|---|---|
| Late order submissions | Members submit after the kitchen has started prep, causing waste and rework | High |
| Manual order aggregation | Counting individual WhatsApp messages to get total quantities per dish | High |
| No structured preference data | Protein and side choices arrive as free text, often inconsistent | High |
| Zero historical data | No way to spot trends or predict demand week over week | Medium |
| No formal client relationship layer | Managing multiple company clients with no shared system | Medium |

### 2.2 Company Admin

| Pain Point | Description | Severity |
|---|---|---|
| Manual member coordination | Chasing individuals to submit their orders every day | High |
| No approval workflow | New employees join a WhatsApp group with no formal onboarding | High |
| No absence handling | When a member is absent, their meal is wasted with no system to flag it | High |
| No order history | No record of what was ordered, by whom, on which date | Medium |
| No accountability layer | If something goes wrong with an order, there is no paper trail | Medium |

### 2.3 Company Member

| Pain Point | Description | Severity |
|---|---|---|
| Friction to submit | Typing out a full order in WhatsApp every day is repetitive | Medium |
| No confirmation | Members don't know if their order was received or processed | High |
| No self-service control | Can't update preferences, mark absence, or track their own history | Medium |

---

## 3. Product Overview

### What is MealFlow?

MealFlow is a two-sided B2B SaaS platform that connects restaurants and corporate clients for structured, daily meal ordering. It replaces WhatsApp-based meal coordination with a purpose-built system that gives restaurants visibility into upcoming orders and gives companies a clean admin layer to manage their meal programme.

### How it Works

1. A restaurant registers on MealFlow and creates their profile.
2. The restaurant creates a menu for a specific company and date, sets a cutoff time, and sends an invite link to the company.
3. A company admin registers via that invite link and is automatically linked to the restaurant.
4. The admin invites employees via a unique member code.
5. Members register, get approved by the admin, and can place their daily meal order before the cutoff.
6. After the cutoff, the restaurant sees a full, structured breakdown of all orders — total counts per dish, per protein, per side — with individual member names.
7. Members who have not ordered by cutoff are auto-assigned their default meal.

### Value Proposition

**For restaurants:** Structured, predictable order data before prep begins. No more last-minute chaos.

**For companies:** A proper admin layer for their meal programme — member management, order history, and zero manual aggregation.

**For members:** A clean, simple interface to place and manage their daily meal with confirmation.

---

## 4. Product Requirements Document (PRD)

### 4.1 Scope — MVP

The MVP covers the core loop: restaurant creates menu → company members order → restaurant sees aggregated output.

### 4.2 User Roles

| Role | Description |
|---|---|
| Restaurant Admin | Registers restaurant, manages menus, views order aggregations, sends invite links |
| Company Admin (Owner) | Registers via restaurant invite, manages company members, approves/rejects members |
| Company Admin | Elevated member with management access, set by owner |
| Company Member | Registers via member invite code, places daily meal orders |

### 4.3 Core Features — MVP

#### Authentication & Onboarding
- Email/password registration with role assignment at signup
- Restaurant invite link flow — company admin auto-linked on registration
- Member invite code flow — member starts as `pending` until approved by admin
- Custom claims via Firebase Auth for role-based access control

#### Menu Management
- Restaurant creates menus per company, per date, per meal type (breakfast/lunch/dinner)
- Menu items support: fixed protein, protein choice, side choice, swallow choice
- Cutoff time set per menu (`HH:mm` string)
- Menu status flow: `draft → pending → active`
- Edit flow: `active → editRequested → editing → pending → active`
- Auto-reject scheduler: menus still pending after cutoff are auto-rejected every 15 minutes

#### Member Management
- Company admin approves or rejects pending members
- Bulk approve (selected or all pending)
- Admin promotion/demotion (owner only)
- Member suspension and reactivation
- Member removal

#### Order Management
- Member selects meal item + optional protein/side/swallow before cutoff
- Auto-assignment after cutoff for members who have not ordered
- Order cancellation on link deactivation or member removal
- Absence marking — admin can mark a member absent, suppressing auto-assign

#### Order Aggregation (Restaurant View)
- Total count per menu item component (e.g. Chicken: 23)
- Per-combo breakdown with member names (e.g. Jollof + Chicken + Salad: 10 — Ada, Emeka…)

#### Notifications
- Email notifications via Gmail/Nodemailer for key events: menu submitted, menu approved, member approved, menu rejected
- In-app FCM push notifications (Firestore `notifications` collection)

#### Feedback
- Members and admins can submit feedback via in-app form
- `sendFeedback` Cloud Function stores submissions

### 4.4 Non-Functional Requirements

| Requirement | Detail |
|---|---|
| Platform | Web (React + Vite), mobile-ready layout architecture |
| Backend | Firebase Cloud Functions (callable), Firestore |
| Auth | Firebase Auth with custom claims |
| Hosting | Firebase Hosting, CI/CD via GitHub Actions |
| Performance | Page loads under 3s on mid-range Android on 4G |
| Security | Role-based Firestore security rules using custom claims |
| Scalability | Flat Firestore schema designed for future MERN migration |

### 4.5 Out of Scope — MVP

- In-app payments
- Multi-restaurant per company
- Native mobile app (Capacitor planned post-MVP)
- White-label theming per company/restaurant
- Self-serve subscription billing
- Analytics dashboard

---

## 5. Architectural Decisions

This section documents key technical decisions and the product reasoning behind them.

### 5.1 Flat Firestore Schema (No Subcollections for Core Data)

**Decision:** `menuItems`, `orders`, `ratings`, `companyMembers` are stored as top-level Firestore collections rather than nested subcollections under their parent documents.

**Reasoning:** A flat schema allows direct querying across entities without requiring knowledge of the parent document path. More importantly, it mirrors how a relational/document database like MongoDB structures data. When MealFlow migrates from Firebase to a MERN stack at scale, the data model translates directly — no restructuring required. This was a deliberate forward-compatibility decision made at schema design, not an afterthought.

### 5.2 React + Firebase for MVP, MERN at Scale

**Decision:** Build the MVP on React + Firebase (Auth, Firestore, Cloud Functions, Hosting). Plan to migrate the backend to Node.js + Express + MongoDB at scale.

**Reasoning:** Firebase eliminates infrastructure overhead for a solo founder at MVP stage — no server provisioning, no DevOps, built-in auth, real-time data. The tradeoff is cost at scale and limited query flexibility. MongoDB was chosen as the target database (over PostgreSQL) because the document model fits MealFlow's data naturally — a menu item is a document with nested options, an order is a document with nested selections. The MERN migration is planned for when MealFlow reaches a scale where Firebase costs become significant or query requirements outgrow Firestore's capabilities.

### 5.3 Custom Claims for Role-Based Access Control

**Decision:** Use Firebase Auth custom claims (`systemRole`, `companyId`, `restaurantId`, `companyRank`) rather than Firestore document lookups for authorization.

**Reasoning:** Claims are available on the client immediately after token refresh and on the server in every Cloud Function call without an extra Firestore read. This keeps authorization fast and cheap. The tradeoff is that claim updates require a token refresh to propagate — acceptable for role changes which are low-frequency events.

### 5.4 Dual Layout Architecture (Mobile + Web)

**Decision:** Separate `MobileLayout` and `WebLayout` components switched by `AppShell` based on platform detection.

**Reasoning:** MealFlow targets African markets where a significant portion of corporate users will access the product on mobile. Rather than a single responsive layout that compromises on both, the dual-layout architecture allows each surface to be designed specifically for its context. The mobile layout will eventually be packaged as a native app via Capacitor.

### 5.5 White-Label Theming (Planned)

**Decision:** All UI colors and design tokens are defined as CSS custom properties (`--mf-color-*`, `--mf-shadow-*`, `--mf-radius-*`) at the `:root` level.

**Reasoning:** This is groundwork for a planned white-label theming feature where restaurant or company clients on paid tiers can apply their own brand colors. Overriding a set of CSS variables per tenant is significantly simpler than maintaining per-tenant stylesheets. The theming system is designed now so it does not require a UI refactor when the feature ships.

---

## 6. Product Roadmap

### Phase 1 — MVP Hardening (Current)
- [x] Full Cloud Functions backend (auth, menus, orders, members, ratings, notifications)
- [x] React frontend — restaurant and company portals
- [x] GitHub Actions CI/CD
- [x] Email notifications via Gmail/Nodemailer
- [x] Auto-reject scheduler for expired pending menus
- [x] Firestore security rules
- [ ] End-to-end QA across all user flows
- [ ] Soft launch with 1–2 pilot clients (Resswell Catering as first restaurant)

### Phase 2 — Growth Features (3–6 months post-launch)
- [ ] In-app notifications (FCM, notification centre UI)
- [ ] Order history for members and admins
- [ ] Ratings and feedback UI (backend complete)
- [ ] Absence management UI (backend complete)
- [ ] Export / print prep list for restaurant (PDF or CSV)
- [ ] Invite code UI with copy button and live countdown timer
- [ ] Analytics dashboard (order trends, popular items, member activity)

### Phase 3 — Monetisation (6–12 months)
- [ ] Subscription tiers (Free / Growth / Pro)
- [ ] Self-serve billing (Paystack integration for African market)
- [ ] White-label theming — restaurant and company brand colors on paid tiers
- [ ] Multi-restaurant per company (company links to more than one vendor)

### Phase 4 — Scale (12–24 months)
- [ ] Native mobile app via Capacitor (iOS + Google Play)
- [ ] MERN stack migration (Node.js + Express + MongoDB) — Firebase backend retired
- [ ] Multi-country expansion beyond Nigeria (Ghana, Kenya, South Africa)
- [ ] Enterprise tier — SSO, advanced analytics, dedicated support
- [ ] Marketplace model — restaurants discoverable by companies without a direct invite

---

*Document authored by Victor Anyiam — Founder, PM & Solo Developer, MealFlow*
*Last updated: June 2026*
