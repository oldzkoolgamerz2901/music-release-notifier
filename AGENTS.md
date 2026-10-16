# Agent Instructions

## Project context

Music Release Notifier is a personal application for discovering and notifying the owner about new releases from followed Tidal artists.

Read `music-release-notifier.md` when working on product requirements, architecture, feasibility, or implementation planning. Treat it as the authoritative product design. Preserve its requirements and constraints unless the owner explicitly approves a change.

## Investigate and design

- Inspect relevant code, configuration, documentation, and existing design decisions before proposing changes. Base conclusions on evidence.
- Evaluate every solution against the overall architecture. Explain how it fits existing responsibilities and boundaries, or why a design change is justified.
- Keep behavior with the component that owns it. Respect dependency boundaries and avoid unnecessary coupling.
- Prefer the simplest maintainable solution that satisfies the requirements. Add infrastructure, dependencies, and abstractions only for a demonstrated need.
- Distinguish verified facts, assumptions, risks, and recommendations. Investigate accessible facts rather than asking the owner to supply them.
- Present meaningful architectural choices and tradeoffs for approval before implementing them.

## Collaborate and execute

- Treat the owner as the decision-maker for product scope and consequential architecture choices. Offer candid technical recommendations and challenge assumptions when evidence warrants it.
- Keep work scoped to the requested task. Planning or investigation does not authorize implementation.
- For implementation tasks, agree on a small, reviewable scope before changing code. Make focused changes and avoid unrelated refactoring.
- Follow the repository's established technology choices, conventions, and documented decisions.
- Validate changes with relevant automated checks and, where appropriate, manual verification. Report what was tested, what remains unverified, and any risks.
- Explain consequential changes clearly enough for the owner to understand and review them.

## Project constraints

- Preserve the agreed Kotlin/Spring Boot backend, Angular frontend, and modular-monolith direction unless explicitly revisited.
- Treat Tidal API permissions, access, rate limits, regional availability, and actual web-push behavior as unverified until supported by evidence.
- Keep the €10/month hosting ceiling visible in infrastructure decisions. Obtain approval before introducing paid services.
- Protect credentials, OAuth tokens, and personal data. Keep secrets out of source control, frontend bundles, and logs.
- Use the existing skills for specialized workflows when appropriate. Keep reusable workflow instructions in skills rather than duplicating them here.
