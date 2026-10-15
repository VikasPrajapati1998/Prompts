# Reusable Prompt: Full Security Audit and Testing of Any Frontend + Backend

Copy everything below the line into a Claude Code session opened at the project root.
Fill in the **Parameters** block first; leave a value as `auto` to let Claude discover it.
Using this prompt is your explicit request for Claude to run the commands listed in
section 4 (tests, builds, scanners, local servers, read-only git history checks). That
overrides, for this task only, any standing rule against running validation.

---

## Parameters

```
BACKEND_PATH:        auto            # e.g. ./api, ./AIVA-Backend
FRONTEND_PATH:       auto            # e.g. ./web, ./AIVA-Frontend (or "none")
OTHER_COMPONENTS:    none            # workers, mobile apps, IaC, data pipelines
ENVIRONMENT_FOR_LIVE_TESTS: local    # local | staging  (NEVER production)
BASE_URLS:           auto            # e.g. backend http://localhost:8000, frontend http://localhost:3000
TEST_ACCOUNTS:       none            # names/roles of test users you provide, never real credentials in this prompt
REPORT_DIR:          Changes/Change-NN   # where the report and plan go (create it)
FIX_MODE:            fix-safe        # report-only | fix-safe | fix-all-confirmed
RUN_DEPENDENCY_SCANS: yes            # yes | no (e.g. no network)
RUN_LIVE_TESTS:      yes             # yes | no
OUT_OF_SCOPE:        none            # paths, services, or third parties not to touch
```

## 1. Role and rules of engagement

You are a senior application security engineer auditing code you are authorised to test.

- **Never touch production.** Live tests run only against `ENVIRONMENT_FOR_LIVE_TESTS`
  using local or staging instances and test accounts. If the only reachable target looks
  like production (real domain, real customer data, production database strings), stop
  and ask.
- **Non-destructive only.** No data deletion, no denial-of-service, no load testing
  beyond a small burst needed to prove a rate limit, no exploitation of third-party
  systems, no credential stuffing. Attack payloads are limited to proof-of-concept
  strings against your own test instance.
- **Secrets:** never print, log, copy into reports, or send anywhere the value of a secret
  you encounter. Record the **name and location only** and recommend rotation. Do not read
  `.env`, key stores, or settings files for values unless the audit needs a specific key's
  presence; list names only.
- **No weakening of controls** (auth, TLS verification, CORS, validation) to make a test
  pass. Do not disable security middleware to work around a problem.
- **Do not invent results.** Never report a scan as run, a test as passing, or a fix as
  verified unless it actually happened in this session. If a tool is missing or a command
  fails, say so and mark the item "not verified".
- **Respect the project:** follow its existing conventions, keep changes minimal and
  focused, delete code your fix supersedes, add dependencies only when clearly justified.
  Read the project's own instruction files (`CLAUDE.md`, `ARCHITECTURE.md`, `SETUP.md`,
  `README.md`, contributing and security docs) before starting.
- **Large task:** use subagents for independent areas (see section 5) and show a task
  tracker (Pending / Running / Completed) and refresh it when a status changes. Keep
  subagent prompts lean and have them return findings, not file dumps.

## 2. Phase 1: Discover (read-only)

Produce a short architecture map before judging anything:

1. Languages, frameworks, versions, package managers, build and run commands.
2. Entry points and every route or endpoint, with its **authentication dependency and
   required permission**. Build the **route-to-permission table**; this drives most findings.
3. Datastores and how each is queried (ORM, query builder, raw SQL, stored procedures,
   NoSQL operators, search engines, caches, local file stores).
4. Authentication model: identity provider/SSO, session cookies vs bearer tokens, token
   issuer and algorithm, refresh, logout, any self-issued tokens (share links, API keys,
   password-reset or invite tokens), MFA, service-to-service auth.
5. Authorization model: roles, permissions, tenancy, ownership rules, row/column/data scoping.
6. Trust boundaries and sensitive data flows: user input, file uploads, outbound HTTP,
   third-party APIs and LLMs, message queues, webhooks, background jobs, admin tools.
7. Configuration and secrets: how settings load, secret store, environment variables,
   default values, differences between dev and production.
8. Build and deployment: CI workflows, artifacts, hosting, headers, containers, IaC.
9. Existing security controls, tests and docs. Existing controls are to be **tested, not
   assumed**.

State anything you could not determine and what you assumed.

## 3. Phase 2: Static audit (all 20 areas, plus extras)

For every item record: affected `file:line`, evidence, realistic attack scenario, severity
(Critical / High / Medium / Low / Info), classification (**confirmed**, **missing control**,
**theoretical**), and whether the control is **verified OK**, **not applicable** (give the
reason) or **not verifiable from code** (say what is needed). A package or middleware
existing is not evidence that a control works; trace it from input to enforcement.

### A. Injection and input
1. **SQL / NoSQL / command / template / LDAP / XPath injection.** List every place a query
   or command string is built dynamically. Check parameterisation, identifier allowlists
   against real catalog data, identifier quoting and escaping, LIKE wildcard escaping,
   `ORDER BY`/`LIMIT` handling, and clamped numeric inputs. For apps that run **generated or
   user-supplied SQL** (natural-language-to-SQL, report builders, query consoles): test the
   validator against comma-joins, derived tables, table hints, CTEs, `UNION`, aliases,
   stacked statements, comments, case and Unicode tricks, server-side functions
   (`OPENROWSET`, linked servers, `xp_`/`sp_`), system schemas, and unqualified column names;
   confirm table/column/row-level grants cannot be bypassed by any of these; confirm the
   database login itself is least-privilege/read-only, since code-level checks are the second
   line of defence.
2. **XSS** (stored, reflected, DOM). Unsafe sinks (`innerHTML`, `dangerouslySetInnerHTML`,
   `v-html`, `document.write`, `eval`, `new Function`, unsanitised markdown/HTML renderers,
   `href`/`src` built from data, `javascript:` URLs, SVG, iframes). Include data that reaches
   the page from users, other users, APIs, LLM output and uploaded files. Markdown or rich
   text renderers must not allow raw HTML or auto-loading remote images. Evaluate a CSP.
3. **CSRF.** Establish which authentication mechanisms an endpoint accepts. Bearer-header
   APIs do not need CSRF tokens; **cookie-authenticated** state-changing routes do (token,
   `Origin`/`Referer` validation, `SameSite`). Check every method, not just the main flow.
4. **File upload and download.** Size caps (enforced while streaming, not after reading
   everything), extension plus magic-byte check, archive and zip-bomb limits (entry count,
   ratio, uncompressed size), spreadsheet/CSV **formula injection** on import and export,
   filename handling and path traversal, storage location, content-type on download, access
   control, malware scanning where relevant, row/cell limits.
5. **Other input risks:** open redirects, request-size limits, deserialisation, regex DoS,
   mass assignment, HTTP parameter pollution, header injection, prototype pollution.

### B. Authentication and authorization
6. **Broken object-level authorization (IDOR/BOLA).** For every endpoint taking an id,
   token, filename or session reference, prove the data layer filters by the caller, on
   read, update, delete, export, share. **Check child resources separately** (messages of a
   session, items of a chart, comments of a ticket) and any "get by id" repository method
   that is also used by a public path. Identifiers that leak elsewhere (share links, logs,
   URLs) make this more serious.
7. **Rate limiting and abuse controls.** Login/callback, password reset, OTP, registration,
   search, uploads, exports, expensive or paid operations (LLM calls, report generation),
   public/share endpoints, admin mutations. Check keys (IP, user, credential), proxy header
   trust, multi-worker/multi-instance behaviour (a per-process limiter is multiplied), 429
   body and `Retry-After`, CORS headers on the 429, and fail-open behaviour when the store
   is down.
8. **Token security (JWT/session/API keys).** Algorithm pinned (reject `none`, HS/RS
   confusion), `iss`, `aud`, `exp`, `nbf`, tenant restrictions, key rotation (`kid` miss
   handling), signing-key strength and **no insecure default or fallback key**, token
   lifetime, refresh rotation, revocation, `typ`/purpose claims so tokens cannot be
   confused, tokens in URLs (leak via logs, history, Referer), nothing logged. Self-issued
   links (share, invite, reset) need expiry, revocation, scope limits, and re-checking of the
   owner's current permissions.
9. **Secret management.** Search the working tree, build outputs, frontend bundles,
   containers, CI files, tests, docs and prompts for credentials, keys, tokens, connection
   strings and private keys. Confirm secrets come from a secret manager or environment, not
   source. Run the **git-history** secret check (section 4). Anything found: name and
   location only, recommend revoke and rotate.
10. **Password handling.** Hashing algorithm (Argon2id, bcrypt, scrypt) and parameters,
    reset/change/recovery flows, token entropy and expiry, enumeration, leakage in logs and
    errors, policy. If auth is fully delegated (SSO), confirm by search and mark **not
    applicable** with evidence.
11. **MFA and privileged access.** Where it is enforced (app vs identity provider), whether
    admin and sensitive actions can require step-up (`amr`/`acr` claims), recovery and reset
    paths, brute-force/replay protection.
12. **CORS.** Allowed origins as an explicit allowlist (never `*` with credentials, never
    reflected origins, no localhost in production), allowed methods and headers limited to
    what the client uses, development vs production values, and CORS never treated as
    authorisation.
13. **Client-side token storage and session handling.** Where tokens live (memory,
    `sessionStorage`, `localStorage`, cookies and their `HttpOnly`/`Secure`/`SameSite`
    flags), logout invalidation, idle timeout, refresh handling, multi-tab behaviour,
    cleanup on 401. Assess XSS exposure honestly. Do not change the authentication
    architecture; recommend only.
14. **Server-side permission enforcement.** Deny by default; every route, background job,
    export, websocket/SSE stream and admin function enforces permissions on the server
    regardless of what the UI hides. Look for: routes missing a permission dependency,
    privilege escalation through role/permission editing (can a manager grant more than
    they hold?), mass assignment of role fields, tenant isolation, and **fail-open** paths
    (what happens when the permission store, session store or cache is unavailable? when a
    claim is missing?). Check **debug/auth-bypass flags** (`DISABLE_AUTH`-style settings,
    build-time frontend flags) cannot reach production, and that "is this production?"
    detection is fail-secure (an unset or misspelt environment must not mean "dev").
15. **Row-level security and data scoping.** Where data is restricted per user, tenant or
    category: is it enforced in the query layer, in the database, or only in the UI? Assess
    database RLS feasibility (pooling and session-context leakage, privileged roles, table
    owners bypassing policies) and document; do not enable it blindly. Check **caches** and
    precomputed or stored queries keep the scope (cache keys include the effective filter;
    saved reports re-apply current permissions).

### C. External requests and exposure
16. **Webhook verification.** Provider's documented signature scheme over the raw body,
    constant-time compare, timestamp/nonce replay protection, reject before processing.
    Mark **not applicable** if there are none, after an endpoint inventory proves it.
17. **SSRF and redirects.** Every outbound fetch (URL fetchers, previews, import-from-URL,
    callbacks, image proxies, PDF/HTML renderers). Scheme and host allowlists, resolved-IP
    checks (loopback, private, link-local, cloud metadata), redirect handling, DNS
    rebinding, alternate IP encodings. Also open redirects on login/callback.
18. **Source maps and build artefacts.** Production builds emit no source maps unless
    deliberately served privately; no secrets in bundles; debug tooling and verbose error
    pages disabled; build-time environment variables that are really secrets.
19. **Default and hardcoded credentials.** Seed scripts, migrations, sample accounts, test
    fixtures leaking into production, admin bootstrap, default ports and consoles, overly
    privileged database and cloud identities.
20. **Sensitive data in logs, errors and telemetry.** Tokens, cookies, secrets, passwords,
    personal data, full prompts/queries/result rows, SQL with literals, upstream error bodies,
    stack traces and internal paths in client responses, validation errors that echo input,
    share tokens in URL paths captured by access logs. Redaction filter present and applied
    to all handlers. Audit logs keep identifiers and counters, not content.
21. **Vulnerable dependencies and supply chain.** Manifests and lockfiles for every
    component; pinning policy; test or dev packages shipped to production; abandoned or
    risky libraries (especially for JWT, XML, archive, image, markdown and spreadsheet
    handling); duplicate or unexpected packages that could be typosquats.

### D. Extras (apply when relevant)
- **Security headers** on every host that serves responses: CSP, `X-Content-Type-Options`,
  `Referrer-Policy`, HSTS, frame protection, `Cache-Control` on sensitive responses,
  `Permissions-Policy`. Check how the SPA is actually served (static host, Node, CDN) and
  where headers can be set.
- **Transport and cookies:** TLS enforcement, HSTS, cookie flags, secure forwarded-header
  handling behind proxies (`X-Forwarded-*` trust).
- **LLM / AI features:** prompt injection through user data, uploaded content, vocabulary
  or knowledge bases and other users' content; output rendering (markdown images or links as
  exfiltration channels); system-prompt and data leakage; tool/function-call authorisation;
  guardrails failing open or closed; cost and abuse limits; PII sent to model providers;
  persisted model memory that another user or admin can write.
- **Shared and public links:** what a token exposes, expiry, revocation, enumeration,
  whether the data scope is re-checked against the owner's current permissions.
- **Admin and multi-tenant features:** a user can see or assign only what they themselves
  hold; cross-tenant leakage; admin read/write of other users' data is scoped.
- **Deployment hygiene:** CI uploads only needed files (not the whole checkout with local
  env files), least-privilege workflow tokens, actions pinned to commit SHAs, build, test
  and audit steps present, container user and base image, process timeouts and recycling,
  exposed debug ports and documentation endpoints.
- **Availability:** request size and timeout limits, unbounded queries/exports, regex and
  zip bombs, worker exhaustion.
- **Business logic:** race conditions on limits/quotas, replayable actions, state
  transitions that can be skipped.

## 4. Phase 3: Dynamic testing and scans (you are authorised to run these)

Run what is available and relevant; **detect tools first** (`--version`) and install
scanners only into a throwaway or project-local environment, never globally and never into
production dependencies. Record each command and its result. If a tool is unavailable, say
so and fall back to manual review.

**Existing project checks:** run the project's own test suite, linter, type checker and
production build (frontend and backend). Record failures before changing anything so you
can tell pre-existing problems from ones you cause.

**Dependency and static scanners (if `RUN_DEPENDENCY_SCANS` = yes):**
- Python: `pip-audit`, `bandit -r`, `semgrep --config auto` (if installed), `ruff`/`pylint` security rules.
- JavaScript/TypeScript: `npm audit --omit=dev` (or `pnpm`/`yarn` equivalent), `semgrep`, `eslint` security plugins if configured.
- Other ecosystems: `dotnet list package --vulnerable`, `mvn dependency-check`, `go vet`/`govulncheck`, `cargo audit`, `trivy fs .`.
- Secrets: `gitleaks detect` or `trufflehog` if installed; otherwise a careful regex search
  that reports locations only.
- **Git history (read-only commands are authorised for this audit):**
  `git log --all --full-history -- <env files, key files, settings files>`,
  `git log -p --all -S "<secret-like key name>"`, and `git ls-files` for files that should
  be untracked. Do not rewrite history; recommend rotation and cleanup steps instead.

**Live tests (if `RUN_LIVE_TESTS` = yes, local/staging only):**
1. Start the backend and frontend using the project's documented commands, in the
   background, and stop them when finished. Record the ports.
2. Use test accounts for at least: an unauthenticated caller, a normal user, a second normal
   user (for ownership tests), an admin or manager, and the most privileged role.
3. **Authorization matrix:** for every route in the route table, call it as each role and
   as unauthenticated; confirm 401/403/404 where expected and that responses never contain
   data the caller should not see. Swap object ids between users for every id-taking route
   (IDOR). Replay requests with modified role/permission fields.
4. **Authentication:** missing, malformed, expired, wrong-audience, wrong-issuer and
   `alg:none` tokens; tampered share tokens; expired sessions; logout then reuse.
5. **Injection probes** on every parameter family using harmless payloads (quote
   characters, comment markers, boolean/time-neutral probes, template and path traversal
   strings, formula-prefix cells); for generated-SQL features, send the bypass patterns from
   area 1 to the validator or endpoint and confirm rejection.
6. **XSS probes** through every place user or LLM content is rendered (including shared
   pages and exports); use inert marker payloads and inspect the rendered DOM or response.
7. **Uploads:** oversize, wrong type with right extension, zip bomb (small crafted
   archive), huge row counts, formula cells, odd filenames.
8. **Rate limits:** a small burst (tens of requests, not thousands) against auth, share and
   expensive endpoints; verify 429, `Retry-After`, and that normal use is unaffected.
9. **Headers, cookies and CORS:** inspect response headers on API and static responses;
   preflight with an untrusted origin; cookie flags; cookie-authenticated state-changing
   request with a foreign `Origin`.
10. **Error handling and logging:** trigger 400/401/403/404/422/500 paths and check
    responses for stack traces, SQL, paths or echoed input; inspect the log output produced
    during the tests for tokens, cookies, secrets and personal data.
11. **Startup/config guards:** start the backend with deliberately unsafe production-like
    settings (auth-bypass flag on, weak or missing signing secret, wildcard/localhost CORS,
    debug on, unset environment name) and confirm it refuses or logs loudly.
12. **Frontend build:** production build emits no source maps, no secrets, and fails if an
    auth-bypass flag is enabled; open the built app (headless or via browser tooling if
    available) and check the console for CSP violations during sign-in, token renewal, API
    calls and charts.
13. Optionally, if a DAST tool such as OWASP ZAP is installed, run its baseline scan against
    the local instance only.

Clean up: stop servers, delete temporary files and test data you created, leave no test
accounts or tokens in the repo.

## 5. Working method

1. Discover (section 2), then run section 3 and section 4 in parallel where independent.
   Suggested split for subagents: injection/validation and uploads; authentication,
   authorization and IDOR; infrastructure, logging, rate limiting, dependencies and CI;
   frontend; live testing last, once code fixes settle (a single agent owns running the
   servers).
2. Triage findings. Prioritise by realistic exploitability and impact, not by count.
   Distinguish a confirmed vulnerability from a theoretical risk or an unimplemented
   enhancement.
3. Report **before** fixing (section 6), then fix according to `FIX_MODE`:
   - `report-only`: no code changes.
   - `fix-safe` (default): fix confirmed Critical/High, and Medium/Low where the change is
     small, local and backwards compatible. Substantial changes (auth architecture changes,
     MFA, RLS rollout, token-storage migration, schema migrations, major dependency
     upgrades, CI redesign) go in the remediation plan with steps and rollback instead.
   - `fix-all-confirmed`: also implement the larger confirmed items, each with its
     migration and rollback documented first; never run destructive migrations.
4. Every fix gets a test in the project's existing framework: a case that fails without
   the fix (permitted and prohibited access, malformed input, bypass attempts). If the
   project has no test setup for a layer, document manual steps instead of adding a new
   framework. Make risky behaviour changes configurable with a safe default and say so.
5. After fixing, **re-run** the relevant tests, builds and scans and the live probes that
   reproduced each finding. Mark a finding **Fixed (verified)** only when the repro no longer
   works and the project's tests and build pass; otherwise **Fixed (unverified)** with the
   reason, **Partially fixed**, **Open**, **Needs manual action**, or **Not applicable**.
6. Update documentation that actually changed (architecture/setup/test docs, README
   security section, example env files with names only). Do not rewrite unrelated docs.

## 6. Deliverables

Create `REPORT_DIR` if needed and write:

1. **`SECURITY_AUDIT_REPORT.md`** containing: executive summary with counts by severity and
   status; architecture map and route-to-permission table; one row or section per finding
   (ID, area, title, `file:line`, endpoint/component, severity, classification, evidence,
   attack scenario and impact, remediation, status, verification method and result); the
   **Verified OK** controls with evidence; the **Not applicable** items with reasons; the
   **remediation plan** (prioritised, dependencies, config and migration steps, rollback,
   owner of each decision); behaviour changes users will notice; every command run with its
   outcome (and every command that could not be run, with why); manual checks still needed;
   files changed.
2. The code and test changes themselves (for `fix-safe` and `fix-all-confirmed`).
3. Updated project docs only where they changed.

Do not duplicate documentation that already exists; update it.

## 7. Final response

A concise summary: findings by severity and by classification; what was fixed and verified
versus fixed but unverified; what remains open or needs a human decision; commands run and
their results (including tools that were missing); configuration or deployment actions
required before release; breaking changes or migrations; any secret that must be rotated
(name only); and the path to the report and every changed file.

**Start by reading the project's instruction files and mapping the architecture. Complete
the audit, testing and safe fixes; do not stop after producing a checklist.**
