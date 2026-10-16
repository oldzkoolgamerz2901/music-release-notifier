# Music Release Notifier

Status: agreed product design, 16 October 2026. No implementation authorized yet.

## Purpose and success

Build a personal application that alerts the owner about new music from deliberately followed artists. The owner listens on Tidal in the Netherlands and currently follows approximately 50–100 artists. Tidal’s existing notifications can arrive days late and require opening the app. The desired experience is a phone notification that opens the release on Tidal.

This is also a learning project in agentic programming: specifying work, giving coding agents useful context, reviewing their changes, and verifying outcomes while retaining an understanding of the code. Learning unfamiliar infrastructure is not an objective in itself.

The first version serves the owner only. Invited users may follow after validation; public registration and commercial operation are outside the initial scope.

Success means a two-week personal trial with real release alerts, examination of missed and duplicate alerts, observed detection delays, and operation within budget. A controlled notification test precedes the trial. Record source lag separately from application detection delay where it can be observed; a source’s exact publication time may be unavailable.

## Product requirements

### Artist collection and releases

- Treat the owner’s Tidal follows as the authoritative artist list. Import it and periodically synchronize additions and removals, subject to verified API access.
- Prefer importing follows; manual artist selection is a fallback requiring a design revisit if importing is unavailable.
- Following artists on Tidal through this app is a later enhancement, conditional on API support.
- Include new singles, EPs, and albums with a followed artist credited as a primary artist.
- Exclude guest-only appearances, compilations, reissues, and remasters.
- Suppress confidently identified deluxe editions, clean/explicit variants, regional duplicates, and other alternate versions of an existing release. Show uncertain cases in the app without a push alert. This deliberately favours fewer unwanted alerts and may suppress a worthwhile release.
- Notify for a new album or EP even if some of its singles were already notified. Do not resend the individual single alerts merely because those tracks appear on the album.
- Establish a baseline when an artist is first followed: display recent releases without sending historical alerts. Notify about releases detected afterward.
- New-artist discovery and future-release announcements are out of scope.

### Availability and timeliness

An alert means Tidal lists the release as available in the Netherlands. A release announced elsewhere is insufficient by itself. Tidal is the listening destination; other metadata sources may be evaluated if they improve detection, provided Tidal availability is still established.

Target detection within 15 minutes of the release becoming visible to the detection source. This is an initial target subject to API feasibility, not a guaranteed delay from the actual release or a guarantee of phone delivery. Source publication lag, polling intervals, retries, and push delivery all affect the user’s experience.

After an outage, recover previously undelivered alerts for releases from the past seven days. Suppress older catch-up alerts and prevent duplicate notifications. Send at any hour; the phone’s Do Not Disturb settings govern interruptions. App-specific quiet hours are deferred.

### Notifications and interface

Use an installable web app as the leading delivery approach. Initial device target: the owner’s Google Pixel 10a, Android 17, using Chrome. Verify actual notification behaviour on that device before committing to the approach.

Phone notifications should open the corresponding release on Tidal. Email is a manually selectable alternative. Do not infer delivery failure from an unread push notification or automatically send duplicate email alerts. WhatsApp is not required; proactive paid messages conflict with the preference to avoid per-notification fees. A native Android app is unnecessary unless feasibility findings justify it.

The first interface includes:

- Google sign-in and separate Tidal connection management.
- Synchronized followed artists.
- Recent releases with Tidal links.
- Tidal-provided artist and release artwork where available, respecting applicable provider requirements.
- Notification permission/setup and a test-notification button.
- Last successful check and visible synchronization/checking failures.

Artist search, in-app follow management, elaborate visual effects, and public registration are deferred.

### Access and failure reporting

Restrict Google sign-in to the owner’s account initially. Application authentication and authorization are separate from the Tidal OAuth connection. Store credentials and tokens securely; never expose provider secrets to the frontend or logs. Exact identity configuration remains a technical design decision.

Show failures in the app. Send one email when release checking has been unsuccessful for more than an hour, followed by one recovery email. These messages are best-effort when the system can execute and reach email delivery. A complete hosting outage cannot reliably report itself; independent outage monitoring is an explicit operational decision, not an assumed capability.

## Architecture

Use a monorepo containing:

- One Kotlin/Spring backend.
- An Angular frontend written in TypeScript, with installability and web notification support.
- Persistent storage for application state.

Keep clear internal ownership for Tidal connection and collection synchronization, release discovery and availability checks, release identity/baselining, notification preferences and delivery, and operational status. Begin with internal modules rather than independently deployed services. Do not introduce RabbitMQ, Kubernetes, Keycloak, a BPMN engine, or an outbox merely because they appeared in the old proposal; add infrastructure only for a demonstrated need.

The backend owns scheduled checks, persistent release/notification state, retries, and delivery requests. The frontend owns presentation, user interaction, and browser notification setup. Persistent state must allow restart recovery and duplicate suppression. Specific storage technology, transaction strategy, retry policy, synchronization cadence, and retention periods require technical design before implementation.

Firebase Cloud Messaging is the leading web push candidate. It has no messaging charge, but backend hosting and storage remain separate. The web setup requires HTTPS, notification permission, and a service worker. Browser support does not establish an end-to-end delivery guarantee.

## Budget and feasibility

Hosting ceiling: €10 per month, with paid choices presented to the owner before purchase. Aim below the ceiling and avoid per-notification charges. This is a budget constraint, not a verified hosting quote.

Potential costs include backend compute/scheduling, persistent storage, backups and logs, email delivery, and an optional domain. No paid AI capability is required in the notifier itself. Provider selection and email quotas/setup remain unresolved.

Naively checking each artist once every 15 minutes produces 96 checks per artist daily: 4,800–9,600 requests for 50–100 artists, before pagination, availability queries, collection synchronization, and retries. This arithmetic is not evidence that Tidal permits that traffic. Spread checks across the interval and respect rate-limit responses. If invited users share artists, discovery can potentially be shared rather than repeated per user.

### Required feasibility checks

The first eventual implementation milestone must establish:

1. Whether Tidal permits the proposed notification use. Published guidelines contain an ambiguous restriction concerning alarm/alert functionality; applicability and current requirements remain unresolved.
2. Whether third-party OAuth access can read the owner’s followed artists using supported public scopes.
3. Whether release listings and Netherlands availability metadata are accessible and sufficiently complete. A release date alone does not establish an exact publication instant or current playability.
4. Whether measured API limits, pagination, and request costs support checking 50–100 artists at the target interval.
5. Whether installation, permissions, background notifications, offline recovery, and tapping through to Tidal work on the owner’s Pixel/Chrome setup.
6. Whether owner-restricted Google sign-in, secure Tidal token storage, hosting, storage, and email delivery fit the budget and operating requirements.

Do not silently weaken requirements if a check fails. Present the evidence and revisit the affected source, delivery mechanism, cadence, scope, or budget with the owner. Authenticated API checks and prototypes belong to later authorized work; none were performed during this design interview.

## Agent-assisted development plan

No application implementation is authorized by this document update. When development is separately authorized, work in small milestones:

1. Validate feasibility and document evidence, limitations, and resulting technical choices.
2. Establish the monorepo, owner access, Tidal connection, and synchronized artist collection.
3. Implement release detection, Netherlands availability checks, baselining, and duplicate handling.
4. Add the agreed interface, artwork, push setup, Tidal links, and selectable email delivery.
5. Complete retry/recovery behaviour, operational visibility, deployment, and budget checks.
6. Conduct the two-week personal trial and review missed alerts, duplicates, delays, and costs before inviting others.

For each milestone, agree a small specification first. Agents then implement a reviewable change, run meaningful checks, and explain consequential design choices. The owner reviews the design and code before the next milestone. Increase agent autonomy only as the verification process proves reliable.

Focus automated checks on observable behaviour: provider response handling, collection synchronization, regional availability, baselining, identity/deduplication, album-after-single behaviour, retries, and outage recovery. Verify phone delivery on the actual device. Avoid tests that merely mirror implementation details.

## Research references and limits

These sources informed the design; their presence does not mean authenticated access or contractual permission has been established:

- [Tidal official API reference](https://tidal-music.github.io/tidal-api-reference/) and [OpenAPI specification](https://tidal-music.github.io/tidal-api-reference/tidal-api-oas.json): published followed-artist and artist-album endpoints; live access and metadata still require verification.
- [Published Tidal developer guidelines](https://developer.tidal.com/documentation/guidelines-developer-guidelines-2_0): alert-related language needs clarification; current applicability was not established.
- [Firebase Cloud Messaging](https://firebase.google.com/products/cloud-messaging): no messaging charge.
- [Firebase web setup](https://firebase.google.com/docs/cloud-messaging/web/get-started), [background receiving](https://firebase.google.com/docs/cloud-messaging/web/receive-messages), and [supported environments](https://firebase.google.com/docs/web/environments-js-sdk): platform requirements and documented behaviour; actual device testing remains necessary.
- [WhatsApp pricing](https://whatsappbusiness.com/products/platform-pricing/) and [messaging policy](https://whatsappbusiness.com/policy/): proactive template messaging has pricing and consent constraints; WhatsApp is not selected.

This design supersedes the previous four-service architecture and broad technology-learning wishlist. It deliberately leaves provider-dependent implementation choices open until feasibility is established.
