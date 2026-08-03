---
title: New Horizon — Code Reference
tags: [app, reference, new-horizon, launch-priority]
aliases: [wisense_new_horizon, Alignment Engine, new-horizon-code]
---

# New Horizon — Code Reference

**Repo:** `C:\development\projects\wisense_new_horizon`
**Bundle ID:** TBD (not in README)
**Stack:** Flutter · Supabase · Duffel API · Viator/Expedia affiliate · google_generative_ai · go_router
**Platforms:** iOS · Android · Web
**Most active project (last commit 14 min ago).**

## File inventory

### lib/ (60+ Dart files)

#### app.dart + main.dart
- `app.dart` — root app widget
- `main.dart` — entry point

#### core/ (organized by concern)

##### core/ai/
- `agent_bridge.dart` — **ToolRegistry routing point** (governed by Travel Data Integrity MDT extension). All travel API interactions route through here.

##### core/config/
- `affiliate_config.dart` — Viator/Expedia affiliate config
- `app_config.dart` — build config

##### core/constants/
- `api_config.dart` — API endpoints

##### core/di/
- `app_dependencies.dart` — dependency injection

##### core/governance/ (4)
- `governance_log_store.dart` — governance log interface
- `governance_log_store_io.dart` — native implementation
- `governance_log_store_types.dart` — log types
- `governance_log_store_web.dart` — web implementation
- `orchestrator.dart` — governance orchestration

##### core/models/
- `unified_travel_offer.dart` — unified travel offer domain type

##### core/router/
- `app_router.dart` — go_router config
- `routes.dart` — route definitions

##### core/services/ (18)
- `affiliate_booking_url.dart` — affiliate URL builder
- `affiliate_launch.dart` — affiliate launch handler
- `affiliate_link_factory.dart` — affiliate link factory
- `affiliate_link_inspector.dart` — link validation
- `alignment_engine.dart` — alignment matching engine
- `auth_service.dart` — Supabase auth
- `consensus_match_service.dart` — **Supabase-backed partner match** — persists ratify/veto swipes, realtime channel sync
- `duffel_payment_service.dart` — Duffel payment intents
- `gemini_service.dart` — Gemini AI integration
- `geo_location_provider.dart` — geolocator wrapper
- `hydration_service.dart` — data hydration
- `itinerary_cache_service.dart` — itinerary caching
- `partner_link_coordinator.dart` — partner linking
- `party_link_service.dart` — party/group linking
- `search_context_resolver.dart` — search context
- `search_history_repository.dart` — search history
- `service_initializer.dart` — service init
- `stripe_payment_service.dart` — Stripe payments
- `supabase_duffel_client.dart` — Supabase→Duffel proxy client
- `transit_lifecycle.dart` — transit lifecycle

##### core/ui/
- `swipe_decision_card.dart` — **swipeable decision card** (like/veto travel offers)

##### core/widgets/ (6)
- `affiliate_link_preview_sheet.dart`
- `app_tab_shell.dart` — tab shell
- `compact_delete_button.dart`
- `glass_app_bar.dart` — glassmorphic app bar
- `glass_card.dart` — glassmorphic card
- `nav_glyph_icons.dart` — nav icons

#### features/ (9 feature dirs, MVVM pattern)

##### alignment/ (consensus matching)
- `logic/consensus_engine.dart` — consensus logic
- `logic/consensus_state.dart` — consensus state
- `logic/offer_vibe_scorer.dart` — vibe scoring
- `logic/vibe_match_engine.dart` — vibe matching
- `model/alignment_types.dart` — type defs
- `model/governance_events.dart` — governance events
- `model/human_veto_wrapper.dart` — Human Veto gate wrapper
- `model/user_vault.dart` — user vault
- `ui/consensus_deck.dart` — **Consensus Deck** (the swipe deck UI)
- `ui/offer_card.dart` — offer card

##### auth/ (MVVM)
- `model/auth_state.dart`
- `view/auth_screen.dart`
- `viewmodel/auth_view_model.dart`

##### debug/
- `view/affiliate_smoke_test_screen.dart`
- `view/duffel_smoke_test_screen.dart`

##### governance/
- `view/human_veto_queue_screen.dart`
- `viewmodel/human_veto_view_model.dart`

##### governor/ (richest feature — travel offer management)
- `logic/governor_map_projection.dart` — map projection
- `model/governor_map_pin.dart` — map pin model
- `model/governor_models.dart` — governor models
- `model/offer_amenities.dart` — amenities
- `model/swipeable_governor_card_data.dart` — **swipeable card data model**
- `model/travel_offer.dart` — travel offer
- `services/curator_service.dart` — offer curation
- `view/governor_hub_dashboard.dart` — dashboard
- `view/governor_hub_screen.dart` — hub screen
- `view/governor_search_category_routing.dart` — search routing
- `view/native_detail_view.dart` — detail view
- `view/partner_link_screen.dart` — partner link
- `viewmodel/governor_map_location_controller.dart` — map controller
- `viewmodel/offer_lifecycle_controller.dart` — offer lifecycle
- `widgets/governor_card.dart` — governor card
- `widgets/governor_card/flight_governor_card.dart` — flight card
- `widgets/governor_card/governor_card_factory.dart` — card factory
- `widgets/governor_card/governor_card_frame.dart` — card frame
- `widgets/governor_card/hotel_governor_card.dart` — hotel card
- `widgets/governor_card/tour_governor_card.dart` — tour card
- `widgets/governor_card_image.dart` — card image

##### itinerary/ (MVVM)
- `model/`, `view/`, `viewmodel/`

##### search/ (MVVM)
- `model/`, `view/`, `viewmodel/`

##### transit/
- `data/`, `logic/`, `model/`

##### vault/ (MVVM)
- `data/`, `model/`, `view/`, `viewmodel/`

### supabase/
- `functions/` — edge functions
- `migrations/` — DB migrations

## Dependencies (pubspec.yaml)

| Package | Purpose |
|---------|---------|
| `supabase_flutter` | Auth + database + realtime |
| `go_router` | Navigation |
| `geolocator` | GPS location |
| `json_annotation` | JSON serialization |
| `google_generative_ai` | Gemini AI |
| `cached_network_image` | Image caching |
| `wisense_core` (vendored) | Shared core (divergent fork) |
| `wisense_ui` (vendored) | Shared UI (divergent fork) |

## Architecture

- **Pattern:** MVVM per feature (model/view/viewmodel)
- **State management:** ViewModels + go_router
- **Navigation:** go_router (routes.dart)
- **DI:** app_dependencies.dart
- **Travel API:** All routed through `agent_bridge.dart` (ToolRegistry) → supabase_duffel_client / affiliate services. No direct partner HTTP from views.
- **Consensus Deck:** Swipeable card UI (like/veto travel offers). `consensus_match_service.dart` persists swipes to Supabase `swipes` table + realtime channel sync between partners.
- **Human Veto:** `human_veto_wrapper.dart` + `human_veto_queue_screen.dart` — destructive/quota actions need explicit ratification.
- **Governance log:** `governance_log_store.dart` with platform-specific implementations (io/web).

## Git state

- **Branch:** `main` ✓
- **Last commit:** 14 min ago — "fix: payment dialog overflows, dead-context guards, checkout test seam"
- **Working tree:** Clean (only untracked AGENTS.md/CLAUDE.md/.cursor config)
- **Branches:** 28 cursor/* branches accumulated (need cleanup)

## Launch blockers

- [ ] **README is boilerplate** — needs real README matching Apex/COMMS LINK format
- [ ] [[Fork Reconciliation]] — vendored wisense_core/wisense_ui diverged from canonical
- [ ] 28 cursor branches — delete merged ones
- [ ] Untracked AGENTS.md/CLAUDE.md — commit or .gitignore
- [ ] `flutter analyze` + `flutter test` baseline

Related: [[projects/dormant/New Horizon]], [[Code Reuse Analysis]], [[Fork Reconciliation]], [[Tripartite Protocol]]