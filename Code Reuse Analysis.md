---
title: Code Reuse Analysis
tags: [architecture, decision, code-reuse, quota-saving]
status: open
---

# Code Reuse Analysis — WiSense cross-app sharing

## The goal

Nicholas wants code built once in one app to be reusable in another — e.g., a swipe feature developed in one app used in another without re-implementing. This saves quota (AI agent token cost) and development time.

## What's already shared

### wisense_core (canonical: `C:\development\packages\wisense_core\`)
- `Result<T>` — error handling type
- `ApiException`, `TravelException`
- `api_client.dart` — HTTP wrapper
- Travel models: `flight_detail.dart`, `flight_result.dart`, `flight_query.dart`
- Affiliate: `affiliate_provider.dart`, `expedia_provider.dart`, `viator_provider.dart`
- Repositories: `flight_detail_repository`, `search_history_repository`, `vault_repository`
- Workflows: `flight_detail_workflow`, `travel_search_workflow`, `travel_search_controller`
- `recent_search_service`

### wisense_ui (canonical: `C:\development\packages\wisense_ui\`)
- `loading_indicator.dart` — WiSenseLoadingIndicator
- `error_banner.dart` — WiSenseErrorBanner
- `spacing.dart` — WiSenseSpacing
- `text_styles.dart` — WiSenseTextStyles
- `wisense_theme.dart` — theme
- `glass_panel.dart` — glassmorphic panel
- `snack_message.dart` — snackbar helper
- Flight UI: `flight_card`, `flight_detail_screen`, `flight_result_list`, `flight_search_form`, `flight_search_screen`

### Problem: New Horizon has a divergent vendored fork
New Horizon vendors its own copies at `projects/wisense_new_horizon/packages/wisense_core` and `.../wisense_ui` with ~20 extra files (unified travel offers, hydration, more affiliate providers) and drift in shared files. See [[Fork Reconciliation]].

## Patterns duplicated across apps (reuse candidates)

### 1. Swipe gesture detection
**Apex** (`shift_calendar_grid.dart`):
- `GestureDetector` + `onHorizontalDragEnd` with velocity threshold (200)
- Navigates calendar weeks/months

**New Horizon** (`swipe_decision_card.dart` + `consensus_deck.dart`):
- Swipeable card UI (like/veto travel offers)
- Supabase-backed swipe persistence + realtime sync

**COMMS LINK:**
- GestureDetector for taps only, no swipe navigation

**Reuse opportunity:** The velocity-threshold horizontal swipe gesture from Apex could be extracted into a reusable `SwipeNavigator` widget in `wisense_ui`. The card-swipe decision pattern from New Horizon could become a `SwipeDecisionCard` widget. Different use cases, but the gesture layer is shared.

### 2. AppConfig pattern
All three apps have their own `AppConfig`:
- COMMS LINK: `lib/constants/app_config.dart`
- Apex: `lib/core/app_config.dart`
- New Horizon: `lib/core/config/app_config.dart`

**Reuse opportunity:** Base `AppConfig` class in `wisense_core` with common fields (privacy URL, feature flags, environment detection). Each app extends it with app-specific config.

### 3. Notification service
- COMMS LINK: `notification_service.dart` + `notification_service_stub.dart` (web)
- Apex: `core/notification_service.dart` + `push_notification_service.dart` (FCM)
- New Horizon: no dedicated notification service visible

**Reuse opportunity:** Base notification interface in `wisense_core`. Platform-specific implementations (local-only, FCM push) as subclasses or strategies.

### 4. Supabase auth
- Apex: `auth_page.dart` — Supabase auth
- New Horizon: `auth_service.dart` + `auth/` feature (MVVM) — Supabase auth
- COMMS LINK: no auth (on-device only)

**Reuse opportunity:** Supabase auth flow (login/signup/session) extracted to `wisense_core` as a reusable `AuthService` + `AuthState`. Both Apex and New Horizon would consume it.

### 5. App lock (biometric + PIN)
- COMMS LINK only: `lock_service.dart` + `lock_screen.dart`
- Uses local_auth + flutter_secure_storage + crypto

**Reuse opportunity:** `LockService` + `LockScreen` promoted to `wisense_ui` / `wisense_core`. Any future app that needs app lock gets it for free.

## Proposed action plan

### Phase 1: Reconcile the fork (prerequisite)
Must resolve [[Fork Reconciliation]] first — can't share code cleanly when the shared packages are forked.

### Phase 2: Extract reuse candidates
Priority order (by impact + ease):
1. **AppConfig base** — simplest, lowest risk, all three apps benefit
2. **Swipe gesture widget** — the exact pattern Nicholas mentioned. Extract Apex's velocity-threshold swipe + New Horizon's card-swipe into `wisense_ui`
3. **Notification service interface** — base class in `wisense_core`, implementations per app
4. **Supabase auth service** — extract from Apex/New Horizon into `wisense_core`
5. **App lock** — promote from COMMS LINK into `wisense_ui`

### Phase 3: Document reusable patterns
Each promoted component gets a note in this vault so future app builds know what's available.

## Quota impact

Extracting reusable code means future AI agent sessions spend fewer tokens:
- Agent reads the shared widget's API instead of re-deriving swipe logic from scratch
- App build prompts can reference `use WiSenseSwipeNavigator from wisense_ui` instead of implementing
- Over multiple app builds, this compounds — exactly the quota saving Nicholas is after

Related: [[COMMS LINK — Code Reference]], [[Apex Scheduler — Code Reference]], [[New Horizon — Code Reference]], [[Fork Reconciliation]]