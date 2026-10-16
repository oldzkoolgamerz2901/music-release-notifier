# Feasibility investigation

Research date: 16 October 2026. This report supplements `music-release-notifier.md`; it does not change its requirements or authorize application implementation.

The design is technically plausible, but feasibility is not yet established. Tidal permission is the first blocker. Personal collection access, catalogue completeness, polling capacity, and actual phone behaviour need live evidence. Google login and low-volume delivery have documented implementation paths; the complete hosting bill remains conditional.

## Evidence and investigation boundaries

- **Verified documentation:** official specifications, documentation, pricing, and published policy text inspected during this investigation. Documentation establishes advertised capability, not access for this particular account.
- **Verified code:** static inspection of the earlier repository at commit [`98f3a5ee50bcf6644acf67acce152691bfa8740e`](https://github.com/huibvanstraten/music-release-notifier-core/tree/98f3a5ee50bcf6644acf67acce152691bfa8740e). This establishes what code exists, not that it currently runs successfully.
- **Unverified:** authenticated Tidal requests, measured rate limits, actual follows, production capacity, email delivery, and the owner's phone. No credentials were used, accounts created, messages sent, services purchased, or application prototypes implemented. Existing tests were inspected, not executed.
- **Recommendations:** proposed ways to close evidence gaps; these are not new requirements or approved infrastructure choices.

The earlier repository was downloaded into a temporary directory for inspection. Tidal's documentation pages rendered as JavaScript shells, so their public assets were read directly. The portal explicitly maps its unversioned developer guidelines and terms links to version 3.0, dated 6 May 2024. This resolves the earlier uncertainty about whether only an obsolete version 2 page contained the alert restriction.

## Assessment of the six questions

| Question | Assessment | Remaining gate |
|---|---|---|
| 1. Permission for notifications | Blocked pending provider clarification/approval | Written answer covering alerts and stored user data |
| 2. Read followed artists | Documented, not account-verified | Supported scopes, consent, complete collection, refresh |
| 3. Netherlands availability and complete discovery | Partially documented | Live regional examples, attribution, edition handling, completeness |
| 4. 15-minute polling | Unmeasured | Complete sweep within budgeted time and actual quotas |
| 5. Pixel/Chrome notifications | Platform-supported, not device-verified | Background, offline, permission, installation and Tidal links |
| 6. Identity, storage, hosting and email under €10 | Plausible, conditional | Full quote, available capacity, operational and security checks |

### 1. Does Tidal permit this use?

**Verified:** the portal-selected [developer guidelines v3](https://developer.tidal.com/documentation/guidelines/guidelines-developer-guidelines-3_0), section III, place alert functionality and applications storing end-user data in a category requiring express written approval. This is not merely a quota question. The [terms v3](https://developer.tidal.com/documentation/guidelines/guidelines-developer-terms-3_0), section II, limit catalogue storage and temporary metadata/artwork caching. Section IV separately addresses development quotas and approval for increased access.

**Unresolved:** whether this particular personal release notifier is permitted, what followed-artist and notification-history state may be retained, and for how long. Neither successful API calls nor the earlier project's existence establish permission. No project-specific approval was found in the inspected material. The guidelines' AI-related restriction also warrants care if real provider content would be passed to coding agents; agent-assisted development is not itself a product feature using AI, but that distinction is not provider approval.

**Recommendation:** obtain written clarification covering personal metadata-only release alerts, collection synchronization, minimal persistent identifiers/history for duplicate prevention and recovery, temporary artwork display/cache, and any development tooling that would process provider content. Do not assume metadata-only or noncommercial use exempts the application. Keep application requirements unchanged while this dependency is resolved.

The current assets inspected were [guidelines](https://developer.tidal.com/assets/developer-guidelines-3_0-DN4FTLfL.js) and [terms](https://developer.tidal.com/assets/developer-terms-3_0-BhtucITZ.js). They are evidence of what the portal currently publishes, not a legal interpretation or a guarantee that these asset URLs remain stable.

### 2. Can third-party OAuth read the owner's follows?

**Verified documentation:** the [official OpenAPI specification](https://tidal-music.github.io/tidal-api-reference/tidal-api-oas.json), version 1.10.181, marks `GET /userCollectionArtists/{id}/relationships/items` as third-party, with `me` supported. It documents cursor pagination and user OAuth. The public `collection.read` scope coexists with internal `r_usr` in the operation's security declaration: that inconsistency needs clarification, not an assumption that internal scopes can be requested.

**Verified prior code:** [`AuthorisationService.kt`](https://github.com/huibvanstraten/music-release-notifier-core/blob/98f3a5ee50bcf6644acf67acce152691bfa8740e/src/main/kotlin/com/hvs/kotlinspringplayground/tidal/authorisation/AuthorisationService.kt#L19) implements client credentials only. [`UserService.kt`](https://github.com/huibvanstraten/music-release-notifier-core/blob/98f3a5ee50bcf6644acf67acce152691bfa8740e/src/main/kotlin/com/hvs/kotlinspringplayground/user/service/impl/UserService.kt#L57) manages an application-owned list. Neither implements importing Tidal follows.

The [authorization guide](https://developer.tidal.com/documentation/api-sdk/api-sdk-authorization) documents authorization code with PKCE and refresh tokens. Client credentials cannot substitute for user authorization to read a personal collection.

**Next evidence:** after permission clarification, use the owner's registered client and consent to request only supported public scopes. Read every page, compare the result with the Tidal collection, inspect granted scopes, and verify token renewal and revocation handling. A partial response must not be interpreted as removal of every absent artist. No need to alter follows for this check.

### 3. Can releases and Netherlands availability be identified correctly?

**Verified documentation:** the specification exposes third-party artist-album listings and country-aware usage rules. Album fields include `albumType`, `version`, `releaseDate`, and external links; artwork is represented through relationships. `releaseDate` is date-only; deprecated `availability` is not the preferred contract. The artist-album operation documents neither newest-first sorting nor a release-date filter. No release webhook was found in this specification; that is not proof that Tidal offers none elsewhere.

**Verified prior code:** [`TidalClient.kt`](https://github.com/huibvanstraten/music-release-notifier-core/blob/98f3a5ee50bcf6644acf67acce152691bfa8740e/src/main/kotlin/com/hvs/kotlinspringplayground/tidal/client/TidalClient.kt#L36) requests artist album IDs and albums using `countryCode=NL`. This is a useful integration starting point. However:

- [`AlbumIdListResponseData.kt`](https://github.com/huibvanstraten/music-release-notifier-core/blob/98f3a5ee50bcf6644acf67acce152691bfa8740e/src/main/kotlin/com/hvs/kotlinspringplayground/tidal/client/response/AlbumIdListResponseData.kt#L5) models only IDs, losing pagination links.
- [`getNewAlbumForArtist`](https://github.com/huibvanstraten/music-release-notifier-core/blob/98f3a5ee50bcf6644acf67acce152691bfa8740e/src/main/kotlin/com/hvs/kotlinspringplayground/artist/service/impl/ArtistService.kt#L63) selects one maximum release date from the fetched page. This can miss other releases on the same date and does not establish newly detected availability.
- [`Album.from`](https://github.com/huibvanstraten/music-release-notifier-core/blob/98f3a5ee50bcf6644acf67acce152691bfa8740e/src/main/kotlin/com/hvs/kotlinspringplayground/tidal/domain/Album.kt#L13) retains only ID, title and date. It discards availability, artwork, links and edition information.
- [`AlbumResponseData.kt`](https://github.com/huibvanstraten/music-release-notifier-core/blob/98f3a5ee50bcf6644acf67acce152691bfa8740e/src/main/kotlin/com/hvs/kotlinspringplayground/tidal/client/response/AlbumResponseData.kt#L14) expects older fields including inline image links. It needs contract verification before reuse.
- The main [`ReleaseService`](https://github.com/huibvanstraten/music-release-notifier-core/blob/98f3a5ee50bcf6644acf67acce152691bfa8740e/src/main/kotlin/com/hvs/kotlinspringplayground/release/service/impl/ReleaseService.kt#L42) imports releases from Spotify. Its test mocks Spotify; it does not demonstrate current Tidal behaviour.

**Blockers:** no live evidence establishes regional playability, source freshness, complete listings, dependable primary-versus-guest attribution, or correct edition suppression. In particular, the inspected album-artist linkage does not itself provide an explicit primary-credit flag. Title matching alone cannot reliably distinguish all remasters, reissues and deluxe editions.

**Next evidence:** inspect read-only responses for representative singles, EPs, albums, featured appearances, remasters, explicit/clean pairs, multi-page discographies and a release unavailable in NL. Compare links and availability with Tidal on the owner's account. Validate artwork and classification separately. Observe a real release transition; do not equate a catalogue creation timestamp with the time it became visible or playable. Preserve the agreed conservative suppression policy for uncertainty.

### 4. Can 50–100 artists be checked every 15 minutes?

**Verified arithmetic, not measured quota:** 96 sweeps/day gives 4,800–9,600 requests/day at one request per artist. With 100 artists, that is one request every nine seconds when evenly spread; four pages per artist gives 38,400/day before enrichment, synchronization and retries. Actual page counts must be measured. The previous one-page estimate was a lower bound, not a complete capacity estimate.

**Verified provider statement:** a Tidal maintainer's July 2025 [rate-limit update](https://github.com/orgs/tidal-music/discussions/135) says 429 responses include `Retry-After`. It does not publish a guaranteed numerical quota. Other participants' observed limits are not an allowance for this project.

**Verified prior code:** `TidalClient` obtains a new token for every request. The newest-album operation performs an artist read, one listing read, and one detail read per returned album, each with its own token call. It has no explicit rate-limit scheduling or retry handling. The outbox's scheduled publisher is message publishing, not a release-discovery scheduler.

**Recommendation:** measure complete listing sweeps first; evaluate supported includes/batching and enrich new or changed candidates rather than repeatedly fetching every album. Cache tokens for their valid lifetime. Do not stop after the first page or assume chronological ordering without evidence. Spread calls and honour provider backoff. Persistent IDs and metadata caching remain subject to question 1.

**Pass evidence:** timestamped request counts, page counts, latency, error/backoff records and complete sweep durations across several cycles, including a representative release period. Permit no destructive or quota-exhausting load test. A sweep must leave room inside the 15-minute target for retries and availability checks. If that cannot be demonstrated, report the target as blocked rather than quietly reducing coverage or frequency.

### 5. Will installed web notifications work on the Pixel?

**Verified platform support:** Firebase documents [HTTPS, permission and service-worker requirements](https://firebase.google.com/docs/cloud-messaging/web/get-started), [background/closed-page notifications and click handling](https://firebase.google.com/docs/cloud-messaging/web/receive-messages), and [supported browsers](https://firebase.google.com/docs/web/environments-js-sdk). This supports investigating Chrome on Android; it does not verify the specific phone or guarantee arrival time. [Angular supports service-worker notification handling](https://angular.dev/ecosystem/service-workers/push-notifications), so the agreed frontend has a compatible path.

**Important distinction:** the [FCM lifespan documentation](https://firebase.google.com/docs/cloud-messaging/customize-messages/setting-message-lifespan) says provider acceptance is not device delivery. Queued messages can expire, and default retention can exceed seven days. A later design should bound message lifetime by the release's remaining catch-up eligibility, avoiding stale alerts after a long offline period. Durable app state is still needed; provider queueing alone does not prove outage recovery or duplicate suppression.

**Unverified:** permission settings, battery saver, closed-tab delivery, reboot/offline behaviour, and whether a notification opens the installed Tidal app rather than its website. [Android App Links](https://developer.android.com/training/app-links/about) depend on the target app's verified domain association and device settings; our notifier cannot establish Tidal's association itself.

**Next evidence:** a separately authorized minimal phone test, independent of Tidal data if useful, covering installation, granted/denied/revoked permissions, foreground/background/closed tabs, locked phone, battery saver, offline/reconnect, restart, multiple releases and actual Tidal URLs. Record accepted-versus-displayed timestamps and duplicate behaviour. No native app is justified by documentation alone. Agree service-worker ownership before combining Angular and Firebase handlers to avoid competing registrations or duplicate display paths.

### 6. Can access, secure state, hosting and email fit €10/month?

**Identity:** Google supports server-side token validation and stable account identification through `sub`; see [validation guidance](https://developers.google.com/identity/gsi/web/guides/verify-google-id-token). [Spring Security supports Google OAuth login](https://docs.spring.io/spring-security/reference/servlet/oauth2/login/core.html). Owner restriction must be enforced on the backend after validating issuer, audience, signature and expiry. A hidden registration screen or frontend email check is insufficient.

**Prior code:** [`UserSecurityConfig.kt`](https://github.com/huibvanstraten/music-release-notifier-core/blob/98f3a5ee50bcf6644acf67acce152691bfa8740e/src/main/kotlin/com/hvs/kotlinspringplayground/config/UserSecurityConfig.kt#L54) is a JWT resource-server configuration with stateless sessions and CSRF disabled. It does not implement Google login, owner restriction or safe cookie-session behaviour. Reusing it unchanged would not satisfy the agreed access model.

**Recommended secure-state approach, not yet selected:** backend-held Tidal refresh tokens encrypted with an authenticated cipher; keep the encryption key outside the database and its backups. Use least-privilege secret files/configuration, protect browser sessions, redact tokens from logs, and test renewal, revocation, restart and backup restore. Database credentials, migrations and repositories in the old project are useful experience, not proof of secure token persistence.

**Budget example:** an always-on EU server with the backend, static frontend and local PostgreSQL could avoid separate database and scheduler charges. Suitability of its resources is an assumption until measured.

| Item | Published amount / evidence |
|---|---|
| Hetzner CX23 | €5.49/month excluding VAT in the [June 2026 price table](https://docs.hetzner.com/general/infrastructure-and-availability/price-adjustment/) |
| Primary IPv4 | €0.50/month excluding VAT, [official pricing](https://docs.hetzner.com/cloud/servers/primary-ips/overview/) |
| Server backups | 20% of server price: €1.098/month excluding VAT in this example, [billing FAQ](https://docs.hetzner.com/cloud/billing/faq/) |
| Subtotal | €7.088/month excluding tax; approximately €8.58 if 21% tax applies (illustrative assumption, not a tax determination) |
| Push | FCM listed as [no-cost](https://firebase.google.com/pricing); backend costs remain separate |
| Email candidate | Resend free tier: 100/day and 3,000/month, [official quotas](https://resend.com/docs/knowledge-base/account-quotas-and-limits) |

**Budget caveats:** the [CX23 product page](https://www.hetzner.com/cloud/cost-optimized/) reported unavailable during this check. The example therefore proves arithmetic plausibility, not a purchasable deployment. Roughly €1.42/month would remain under the illustrative tax assumption for domain renewal and other extras. Verify final checkout, stock, domain cost, backup coverage and resource use before selection; do not substitute a more expensive server without approval. Separate managed databases or paid monitoring are not included. Disk snapshots also need a database-consistent restore procedure.

[Resend requires a verified owned domain for general sending](https://resend.com/docs/dashboard/domains/introduction); its shared [test domain is restricted to the account owner's address](https://resend.com/docs/knowledge-base/403-error-resend-dev-domain), not an established production solution. Free quota is plausible for personal alerts but not proof of inbox delivery. Keep paid overages disabled and test release, failure and recovery emails. The app cannot report its own complete host outage; independent monitoring remains an explicit operational choice.

## Reuse recommendation

Use the earlier core as a reference, not a drop-in foundation. Preserve useful concepts: Kotlin/Spring modules, a provider client boundary, PostgreSQL/JPA/Liquibase experience and persistence-focused tests. Revalidate Tidal DTOs, identity, pagination and token handling against current contracts.

The existing transactional outbox records events and publishes them to external messaging. Publication followed by deletion can still repeat after a crash; it is not proof of duplicate-free phone delivery. Do not import Kafka/RabbitMQ or Keycloak merely to reuse those classes. A database-backed notification work record inside the agreed backend is a candidate for later design, not an approved change here.

## Recommended sequence and exit evidence

1. **Provider permission first:** obtain clarification/approval covering notifications, collection state and required retention. No support message was sent in this investigation.
2. **Read-only account verification:** with approved access and owner consent, verify public scopes, complete follows, renewal, representative regional metadata and artwork. Save only permitted, redacted evidence.
3. **Capacity observation:** measure complete sweeps for the actual collection without aggressive load. Demonstrate the 15-minute target with retries and enrichment included.
4. **Separately authorized phone experiment:** verify delivery and Tidal navigation on the owner's device; distinguish platform support from observed behaviour.
5. **Concrete deployment proposal:** select available hosting, domain/email arrangements and backup strategy, present an all-in monthly estimate within €10, and obtain approval before purchase.

The investigation has completed the accessible documentation and source-code work. It has not closed gates that require a provider decision, authenticated account access, application/prototype implementation or physical-device interaction. None of those gaps is grounds to change the agreed product requirements silently.

## Reproducibility

The inspected source revision is pinned above. The downloaded Tidal OpenAPI version was 1.10.181, SHA-256 `8adcb86b11a9d6ff37cc4b39127cb8ae7ee01260d348e4cb1115bc7c5d9ad5ca`.

Portal asset SHA-256 values:

- `index-BU00rsRS.js`: `aab8f71e7c3aeafb8c540ca1effa46f8032f4d432d7c106cfb06f2cf8452584d`
- `developer-guidelines-3_0-DN4FTLfL.js`: `a8f1435ff8e22381ee7eeec74093cd2facc8f49902f429d0752bf1f5c4a59ed7`
- `developer-terms-3_0-BhtucITZ.js`: `3fe0af70268c0d149a22ebf22846db7a0329a123a574db49a1bfa7809d47b0a0`
