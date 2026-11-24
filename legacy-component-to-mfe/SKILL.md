---
name: legacy-component-to-mfe
description: Analyze any legacy Helios component (AEM + Helios frontend) against a supplied specification and Shell/MFE ownership rules, produce a traceable LLD and code-mapping plan, and generate or migrate implementation code in the verified PACE MFE and Shell locations after authorization.
argument-hint: "component name (AEM component and/or frontend folder), specification path or text, ownership rules, and target app"
---

# Legacy Helios component to PACE MFE

Use this skill when asked to understand a legacy Helios component, create its low-level design (LLD), or implement its migration in the PACE MFE and, where required, the PACE Shell. It works for any Helios component; nothing in it is specific to one component.

## Goal

Translate the behavior and requirements of a selected legacy component into:

1. An evidence-based LLD that distinguishes confirmed behavior from unknowns and proposals.
2. A code-mapping plan that applies the supplied ownership rules to verified Shell and MFE code locations.
3. A PACE MFE implementation and, only where required by verified ownership, a Shell implementation, after the LLD is approved or the user explicitly requests implementation.

Do not assume that a legacy component's markup, styling, APIs, or behavior can be copied directly into the MFE.

## Component identification

Every run targets one component, referred to below as `<component>`. The user may identify it by AEM component name, by frontend folder name, or by both. These two names can differ, so never assume they are equal.

- `<aem-component>`: folder name under the AEM components path (may be nested, e.g. `parent/child`).
- `<frontend-folder>`: folder name under the Helios frontend components path (e.g. a `c_NNN`-style name or a descriptive name).
- `<mfe-folder>`: the folder name in the MFE destination, discovered from the checkout.

If only one of `<aem-component>` or `<frontend-folder>` is supplied, find the other by evidence (see "Pairing check"). If the component cannot be identified, ask a focused question. Do not guess.

## Required inputs and precedence

Accept the specification and ownership rules in any of these forms: pasted into the request, provided as file paths, or identified by name in repository documentation. Resolve paths and read the complete relevant content before planning. If an input is missing, ask only for that missing input; do not make up requirements or ownership decisions.

- **Specification** describes required or reported behavior. Treat it as a claim to trace, not proof of what legacy code does or what the target platform supports. Record its title/version/status when available.
- **User-supplied ownership rules** are the primary architecture constraints for the requested work and take precedence over this skill's default ownership table. Preserve each rule's meaning and map it to code only after checking the actual repositories.
- **Repository code, tests, and approved contracts** are evidence of existing behavior and interfaces. If a supplied requirement or rule conflicts with them, document the conflict and request a decision where it affects behavior, public APIs, backend contracts, analytics, security, or ownership. Never silently resolve it by inventing an API or changing the rule.
- **Skill examples and filenames** are illustrative until confirmed in the checkout. Do not treat example adapters, endpoints, analytics names, or component files as established contracts.
- When multiple specifications or rule sources exist, compare their status and contents. Identify conflicts and ask which source governs only when the conflict changes scope or implementation; otherwise document the precedence used and its rationale.

## Repository discovery

Source and destination locations follow these patterns. Treat them as patterns to confirm in the checkout, not as proof that a path exists.

- **AEM component (legacy):** Helios AEM repository (`helios-parent`) in Bitbucket, then
  `helios-website/src/main/content/jcr_root/apps/helios/components/content/<aem-component>`
  The component may sit in a nested folder (for example a container under a parent folder); look for nested or child component folders and include them in the analysis.
- **Helios frontend (legacy):** Helios ID repository (`helios-id`) in Bitbucket, then
  `helios/static/components/<frontend-folder>`
- **MFE destination:** `pace-mfe-global/editorial/src/<mfe-folder>`. The folder name and app are not known in advance; discover the actual name and casing. If the folder does not exist, follow the naming and structure of the nearest sibling components in the same package.
- **Shell:** `pace-shell-app`. The directory may be checked out under a different name (for example `pace-shell-apps`); discover it. Inspect the Shell for the responsibilities required by the supplied rules (see Phase 2).

The Application project root in Bitbucket is the project-level browse entry point. Use the Bitbucket browse links when a legacy repository is not checked out or when remote source review is needed. Use the local checkout when available for edits and searches, and cite the local repository-relative path in generated LLDs. When checked out, the workspace-relative paths are typically `application/helios-parent/...` and `application/helios-id/...`. Confirm the repository, branch, and paths before drawing conclusions. Do not invent missing paths or assume all repositories are checked out. These internal Bitbucket URLs may require organizational authentication.

### Before beginning

1. Inspect nearby `SKILL.md` files in `pace-aemaacs/nna/.agents/skills` and follow their applicable conventions.
2. Inspect repository instructions such as `AGENTS.md`, `CLAUDE.md`, READMEs, and package scripts that apply to the source and destination.
3. Resolve the supplied specification and ownership rules; extract requirements and constraints into traceable lists before proposing code.
4. Identify the component's AEM and frontend source folders, the Shell and MFE destination applications, naming conventions, registration/dispatch path, and available tests. Confirm each path in the workspace.
5. Ask a focused question if the component, authoritative specification, required ownership decision, or target application cannot be identified. Do not guess.

### Specification location

Look for the specification near the component's documentation, for example under `components/<frontend-folder>/specifications/` or the location the user provides. The exact folder and file names are unverified; confirm them in the checkout. Another design-only draft may exist beside it. Compare status and requirements rather than assuming either is authoritative.

### Pairing check

Confirm that the frontend folder is the frontend of the AEM component before relying on either. Compare root CSS class, data attributes, component IDs, analytics identifiers, and static asset paths referenced by the AEM clientlibs or HTL/JSP. If they conflict, or the pairing cannot be established, report it and ask; do not pick one side. Also inspect shared includes, taglibs, and any child or sibling components the component uses; do not assume their names or structure.

Record the pairing result (confirmed, conflict, or unresolved) in the LLD header.

### Existing MFE implementation

Check whether an MFE implementation of the component already exists under `pace-mfe-global/editorial/src/`. If it does, inspect and adapt it rather than creating a duplicate.

## Operating rules

- Read source files before drawing conclusions. Follow relevant imports, templates, scripts, styles, dialogs, models, and tests.
- Cite evidence using repository-relative paths and, when practical, line numbers.
- Separate findings into **Confirmed**, **Inferred**, **Proposed**, and **Unknown**. Do not present an illustrative filename, API, or architecture as an existing contract.
- Preserve observable legacy behavior unless the user or approved LLD specifies a change.
- Do not copy legacy implementation wholesale. Adapt behavior to the MFE's existing architecture and conventions.
- Do not add dependencies, public APIs, backend contracts, analytics events, or design-system primitives without evidence or explicit approval.
- Do not assume that features commonly associated with this kind of component exist. Implement only behavior confirmed by legacy evidence, the specification, or the approved LLD.
- Keep secrets and privileged requests out of browser-visible code, component props, and public configuration.
- Do not modify the legacy source repositories unless explicitly requested.
- Treat the MFE and Shell as independent Git repositories. Keep changes isolated to each, and never stage unrelated files or run broad staging commands from a parent workspace.
- For an LLD-only request, do not implement code. For implementation, produce or confirm the LLD first unless the user explicitly asks to proceed without a separate review.
- Never claim tests, builds, or validation were run unless they were actually run.

## Phase 1: Analyze the legacy component

Inspect both the AEM component and the Helios frontend implementation, including relevant files referenced by them.

Look for:

- AEM dialogs, templates, HTL/JSP, Sling Models, Java, client libraries, policies, and component configuration.
- Frontend templates, scripts, styles, breakpoints, dependencies, and initialization/teardown behavior.
- Authored fields, defaults, required/optional behavior, empty states, and malformed content handling.
- Nested or child components and their rendering or authoring rules.
- Accessibility, keyboard/focus behavior, responsive layout, RTL, and reduced-motion behavior.
- Analytics, personalization, and external integrations.
- Existing tests, examples of authored content, and migration or platform contracts.

Also check the following only where the component actually uses them, confirming each in code and never assuming:

- Images: references, renditions, responsive candidates, aspect ratios, lazy loading, alt-text and decorative handling.
- Media/video: providers, playback lifecycle, coordination between players.
- Links/CTAs, captions, overlays, and tag-library output.
- Carousels, sliders, tabs, accordions, modals, or other interactive patterns, including state and event handling.
- Data from Site Admin, Car Admin, or other domain/backend services.
- Forms, validation, and API calls.

Build a behavior inventory that maps each relevant authored field and legacy behavior to its source evidence. When behavior is not visible in the checked-out code, mark it unknown rather than assuming.

## Phase 2: Map ownership to PACE architecture

First translate the user-supplied ownership rules into a **Rule → Responsibility → Verified file/API → Change needed? → Evidence** matrix. Apply the rules as constraints, but distinguish the user's requirement from evidence that an example file or contract already exists. Verify actual files and contracts in the destination repository before naming them as confirmed.

The following table is only a generic fallback for concerns not covered by the baseline ownership rules below or by run-specific user input. It does not override those rules or verified repository architecture.

| Concern | Default ownership | Guidance |
|---|---|---|
| Page rendering | Shell | Shell selects and composes components. `src/components/PageRenderer.tsx` is an example dispatch location; verify the actual shell. |
| Component HTML | MFE | MFE renders each component. The component's server entry is responsible for markup where the app's architecture requires it. |
| Authored content | MFE mapping | Normalize authored values into typed component props. `server/mapAemToProps.ts` and `server/types.ts` are examples; verify conventions. |
| Interactive behavior | MFE client | Keep browser interactions in client-side code and follow existing server/client boundaries. |
| Tag-library behavior | Split by responsibility | Formatting may belong in mapping; reusable resolution may belong in a shared utility; repository-backed lookup belongs in an AEM/domain adapter. Example utility names are proposals unless they exist. |
| Analytics | MFE emits; Shell provides | Emit semantic interactions through the established adapter/bridge. The Shell initializes providers. Verify event names and APIs; do not invent contracts. |
| Design system | MFE composes; design system owns | Use supported primitives and published tokens. Do not recreate shared primitives or tokens inside the component. |
| Site Admin | AEM authors; Shell fetches; MFE consumes | Confirm the backend contract and pass only the configuration the component needs. |
| Car Admin / domain data | Backend/domain layer | Shell orchestrates required reads; MFE renders supplied values. Verify the contract before proposing an adapter. |
| Images | MFE maps/renders; platform delivers | Use approved asset delivery behavior. Verify URL, responsive candidate, and fallback contracts. |
| Video | MFE presents; provider adapter manages playback | Keep provider behavior behind established adapters. Credentials and privileged requests stay server-side. |
| Component configuration | Component or owning service | Follow existing config conventions. Keep secrets out of public JSON and client props. |
| Styling | MFE scopes; design system supplies tokens; Shell supplies theme | Prefer stable public hooks and published tokens over generated CSS-module class names or broad global overrides. |

### Baseline ownership rules supplied for this skill

Apply these rules for the current migration workflow unless the user supplies a replacement or clarification for a run. They are architecture requirements, not proof that the named files, helpers, endpoints, or APIs exist. In the examples, `<Component>` stands for the discovered MFE component folder.

| Concern | Ownership rule | Examples supplied by the user (verify before use) |
|---|---|---|
| Main JSP/HTL renderer | Shell selects and composes components; MFE renders each component's HTML. | Shell `src/components/PageRenderer.tsx`; MFE `src/<Component>/server/<Component>.tsx`. |
| Markup and component logic | MFE normalizes authored content, renders markup, and implements local interactions; Shell provides page content and shared context. | MFE `server/mapAemToProps.ts`, `server/types.ts`, server component renderer, and client interaction component. |
| Tag libraries | Split each tag's outcome among component mapping, shared utilities, and server/backend services according to responsibility. | Formatting in mapping; reusable link resolution in a shared utility; repository lookup in an AEM/domain adapter. |
| Analytics | MFE emits semantic interaction events; Shell initializes providers and dispatches through its bridge. | `trackPaceEvent`, `window.paceAnalytics`, and `<Component>/server/analytics.ts` are examples; feature-metadata placement requires architect validation. |
| Design system | MFE composes supported primitives/variants; design system owns reusable primitives/tokens; Shell delivers the selected theme. | Component server renderer imports supported primitives; scoped CSS consumes published tokens. |
| Site Admin | AEM retains authoring/export; Shell fetches and normalizes configuration; MFE consumes only relevant props. | `tooling/aem-helpers/src/getSiteAdminModel.ts` and `/content/{brand}/{market}/{language}/site-config.siteadmin.global.model.json` are examples; verify backend contract. |
| Car Admin | Backend/domain layer resolves vehicle data; Shell orchestrates required reads; MFE renders supplied values. | `server/getVehicleConfiguration.ts` is illustrative; the complete contract is not established by this rule. |
| Images | MFE maps references and renders responsive markup; a shared adapter constructs approved delivery candidates; AEM/platform processes and publishes assets. | Read `image/fileReference`, render `<picture>`/`<img>`, and use a `resolveImageDelivery.ts`-like adapter only if verified. |
| Video | MFE renders player presentation and manages playback lifecycle; a shared/provider adapter encapsulates provider behavior. | `server/VideoComponent.tsx`, `client/VideoPlayer.tsx`, and `client/providers/NativeVideoAdapter.ts`; credentials and privileged requests remain server-side. |
| Other configuration | Place configuration with the component, Shell, domain service, or backend according to ownership. Keep typed consumer contracts and Shell request context in their owning layers; keep secrets out of public JSON and props. | `_<Component>.json`, `server/types.ts`, and `requestContext.ts` are illustrative paths. |
| Style overrides | MFE owns scoped presentation; design system owns tokens/primitives; Shell delivers theme and page structure. Prefer stable public hooks over generated CSS-module class names. | A `data-component` hook is an example, not a required or existing selector. |

If the user provides revised ownership rules at invocation, use those rules for that run and record what changed. Do not silently combine contradictory versions. Examples such as `resolveLink.ts`, `trackPaceEvent`, `getVehicleConfiguration.ts`, `resolveImageDelivery.ts`, `NativeVideoAdapter.ts`, and the example endpoint are not confirmed APIs or files unless they are found in the repository.

### Shell/MFE boundaries

Verify the actual component registry, page dispatch, data-loading boundary, and MFE export/package flow in the Shell. Determine:

- Whether registration and dispatch are generic, config-driven, or generated per component.
- How the component's AEM identifier maps to the MFE component.
- The props, data, and event contracts between Shell and MFE.
- Which responsibilities the supplied rules assign to the Shell (for example shared context, Site Admin fetch, analytics bridge, theme).

Prefer an MFE-only change when the Shell already generically selects and composes registered components. Change Shell code only when the supplied rule and verified integration require it. When no Shell change is needed, record "No Shell change required" with the evidence. Report any mismatch between the requested ownership mapping and current code before implementation.

## Phase 3: Write the LLD

Inspect existing specification locations and formats first. Follow them; do not create a duplicate specification if one already exists.

The LLD should include, as applicable:

- Component identity (`<aem-component>`, `<frontend-folder>`, `<mfe-folder>`), source paths, target path, pairing-check result, status, and validation status.
- Purpose, scope, non-goals, and terminology.
- Evidence-based legacy behavior and authored-field inventory.
- Proposed MFE and Shell ownership and a field/behavior-to-props mapping.
- Rendering and data flow, including Shell/MFE/backend responsibilities.
- Markup, responsive styling, theming, and design-system usage.
- Client interactions, lifecycle, accessibility, RTL, and media behavior where relevant.
- Analytics and integration contracts, clearly labeled confirmed or proposed.
- Empty, invalid, missing, and failure states.
- Migration considerations and compatibility risks.
- Open questions and decisions requiring product, platform, or architect validation.
- A test and validation plan based on available repository tooling.
- A traceability matrix linking each supplied specification requirement and ownership rule to legacy evidence, target responsibility, proposed or verified file location, and planned test.

Label design decisions that lack evidence as **Proposed** or **Open decision**. Do not silently turn them into requirements.

Before implementation, summarize the LLD's important assumptions and unresolved questions for the user. If a decision affects the public API, backend contract, analytics, or design system, request confirmation unless the user has already authorized it.

Do not treat a document labeled draft, design-only, or unapproved as implementation approval. If there are conflicting specifications for the component, surface the concrete conflicts (for example fields, behavior, empty states, analytics, or responsive behavior) and ask for the governing version if they change the implementation.

## Phase 4: Implement only after authorization

Implement in the selected MFE destination, following nearby components and repository conventions. Verify the actual destination path under `pace-mfe-global/editorial/src/` and its naming conventions before editing.

Implementation guidance:

- Keep component props typed and derived from confirmed authoring/data contracts.
- Normalize authored values at the appropriate server or mapping boundary.
- Separate server rendering from browser-only interactions according to the app's framework.
- Use existing interaction, analytics, media, link, image, and design-system adapters when present.
- Use scoped styles and stable public hooks where the project supports them.
- Implement only behavior supported by the legacy evidence or approved LLD.
- Avoid speculative adapters, placeholder APIs, new dependencies, and unrelated refactors.
- Add or update tests using the repository's existing test framework and patterns.
- Update documentation or examples only when needed and consistent with repository conventions.
- Map each accepted rule to the owning Shell or MFE layer before editing. Keep presentation and local interaction in the MFE; keep shared page composition, request context, and orchestration in the Shell only when the verified architecture assigns them there.
- For a component already present in the MFE, inspect and adapt its current implementation rather than creating a duplicate. Check whether Shell registration/dispatch is generated or package-driven before editing Shell files.
- Keep generated files, manifests, and package artifacts consistent with the established build process; do not hand-edit generated output unless the repository explicitly requires it.
- Keep MFE and Shell changes in their own repositories so each can be reviewed and merged as a separate PR.

## Phase 5: Validate and report

Run the most relevant available lint, type-check, unit-test, and build commands in each repository that changed. Discover the correct package scripts first, and prefer targeted runs over full suites. If a command cannot run, report why. Report:

- Files created or changed, per repository.
- Behavior implemented and any intentionally deferred behavior.
- Tests and commands actually run, with their results.
- Remaining unknowns, risks, and approvals needed.

Do not claim full parity with the legacy component unless every relevant behavior has been verified and tested.
