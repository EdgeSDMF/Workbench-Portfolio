# Terac-Assessment
Terac Assessment
Routing Workbench
A local operator console for simulated contributor routing.
Requirements and startup
Use Node.js 22.13+ (or Node.js 24 LTS) and npm. The backend uses built-in node:sqlite, which may emit an experimental warning on Node 22.
npm install
npm test
npm run dev

Open http://127.0.0.1:5173. The API runs at http://127.0.0.1:3001. Alternatively:
npm run build
npm start

Then open http://127.0.0.1:3001. Run one backend process only. Stop with Ctrl+C. State is in data/workbench.sqlite. To reset, stop the backend and delete data/. Do not delete the database while running.
Product model
Create saves a draft; Start routing submits it. A successful engine result creates an offer, not an accepted assignment. Contributor responses are simulated from the detail panel. Accept ends routing. Decline or failure requires a reason and leaves a task needing attention, with explicit retry/manual recovery. Manual assignment creates an offer and replaces any unaccepted offer. Accepted tasks are terminal: interpreted as the prompt's exception to unrestricted operator intervention. Work execution after acceptance is out of scope.
High priority sorts first but does not change matching. Contributors can hold multiple tasks: no capacity rule was specified. Manual offers must match expertise. Retrying can select the same contributor; no fairness or exclusion policy is implied.
Architecture and correctness
React/Vite UI -> Express JSON API -> SQLite. A 250 ms in-process worker handles persisted simulated requests. The UI polls every second. Elapsed time uses server snapshot time. The worker intentionally does not use a database transaction across a network wait.
Each request persists its identity, eligible-at-submission candidate, due timestamp, and mode. Random requests have a 20% chance of no due timestamp; otherwise delay is 5–20 seconds. A null due timestamp is simulation-internal knowledge and is not exposed in the operating API. The demo mode chooser explicitly configures scenarios; it is not an engine diagnosis. Pending requests past 20 seconds show an uncertainty warning, not a failure declaration. There is no automatic timeout or retry. Abandoned requests remain historical records.
A task points to its current request. Manual intervention or retry supersedes the old request. Results only apply if the attempt is still pending, current, and the task still routing. Late results are audited and ignored. These checks and assignment writes occur in one synchronous BEGIN IMMEDIATE transaction. A partial unique index independently enforces one offered/accepted assignment per task. Contributor responses carry an assignment ID, preventing an old response from affecting a replacement offer. Two serialized explicit operator commands use last-command-wins; this demo does not reject all stale operator intentions.
On restart, overdue normal requests resolve on the next worker tick; unresolved requests remain unresolved. This is a persisted simulator, not a real external engine integration. A real provider would require reconciliation/idempotency and durable delivery semantics. Crash consistency is supplied by SQLite transactions. The worker may be delayed while the process is off; routing durations are simulated timestamps, not uptime guarantees.
Demonstrate the important cases
1.	Create a finance task. Choose Return in 5 seconds, Start routing, then Accept.
2.	Create a high-priority task. Choose Never return; start routing. Wait past 20 seconds. Observe uncertainty; manually offer to an eligible contributor.
3.	Start a different task with Return in 20 seconds. Immediately offer manually. Wait for the original deadline and inspect routing ignored history; the manual offer remains.
4.	Decline an offer with a reason. Inspect history, then retry or manually offer. Repeat using Fail before work if desired.
5.	Start routing on several tasks. Refresh the browser. Restart the backend during pending work, then observe persisted/resumed state.
Verification status
The delivery environment could create files but did not provide a Node runtime or npm execution tool. The Node tests, npm install, Vite production build, HTTP behavior, and browser UI have NOT been executed here. Do not describe them as passing before running the commands yourself. Automated tests are supplied for normal acceptance, overrides/late results, indefinite pending and retry, decline/failure recovery, no match, newer-request precedence, independent tasks, restart recovery, and validation. Also manually check mobile layout, keyboard navigation, disconnected API, and two browser tabs.
Dependencies use version ranges; the first install should generate and commit package-lock.json after verification. No lockfile is fabricated.
Intentional omissions
•	No authentication, authorization, CSRF hardening, rate limits, or public hosting. Localhost-only binding; do not expose this demo as a service.
•	No external engine, contributor messaging, production queue, or multi-worker deployment.
•	No contributor editing, workload balancing, or sophisticated recommendations.
•	No completion workflow after acceptance.
•	No full event sourcing: transactional current state plus an append-only audit table. Snapshot returns the latest 500 events globally; pagination and full history export are future work.
•	No end-to-end browser tests or production observability. These are next steps, alongside HTTP integration tests and stale operator command protection.
•	POST task creation is not idempotent: an interrupted response can make a manual retry create another draft. Refresh before retrying an uncertain create.
Suggested five-hour allocation
Planning/data model 25 min; backend/worker 85 min; UI 80 min; tests and edge-case verification 65 min; documentation/walkthrough/session exports 45 min. Respect your actual assessment time budget.
Submission hygiene
Use a neutral repository name. Do not publish a public repository naming the company or exercise. Remove credentials, personal data, unrelated conversations, node_modules, build output, and runtime database files. Follow the assessment's attachment/private-repository instructions. Record an actual walkthrough and export authentic project-specific agent sessions; do not present supplied scripts or notes as recorded evidence.
