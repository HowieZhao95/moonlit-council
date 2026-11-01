# 月下议会 · Moonlit Council

A prompt-driven AI werewolf arena: write a system prompt, customize an avatar,
create 6–18 agents, watch their decisions and compare match rankings.

## Features

- Wolves, gods (seer / witch / hunter / guard) and villagers; night actions,
  public speech, secret ballots, elimination, last words and faction victory.
- Animated moonlit village, custom avatars, optional synthesized sound effects.
- Local strategy demo without an API key. Live play supports OpenAI Terra,
  DeepSeek and OpenRouter free routes with **your own provider key**.
- Default live selection: `gpt-6.1-terra`, low reasoning. Availability depends on
  your provider account; the repository does not grant model access.
- Private agent views, validated structured decisions, bounded retries, explicit
  legal fallback decisions, durable cloud match state and usage receipts.

## Quick start — local demo

Requirements: Node.js 22.13+ (24 LTS recommended), npm, Python 3.10+.

```sh
npm ci
python3 scripts/vendor-runtime.py --fetch
npm run dev
```

Open the local URL printed by the development server and select the strategy demo.
The fetch command restores the exact hash-pinned Pyodide files from its official
CDN. No model API is called by installation or the demo. Generated runtime files
and `public/engine-source.json` are deliberately excluded from Git.

## Architecture and deployment boundary

- **Browser demo:** the actual modified AutoWerewolf Python engine runs in a
  same-origin Pyodide Web Worker. `engine/upstream/` preserves the original subset.
- **Cloud/live matches:** `lib/server-engine.ts` is a separate TypeScript semantic
  port, checked against the Python reference by differential tests. Server state,
  ownership, leases, idempotency and attempt reservations use Cloudflare D1.
- Frontend: React / Next-compatible Vinext / Vite; worker: Cloudflare; schema and
  migrations: `db/` and `drizzle/`. `hosting.config.json` contains only local binding
  names, not a cloud account or database ID.

**This is not a one-command authenticated public deployment.** Hosted API routes
expect identity injected by a trusted Sites dispatch layer. Never expose them
behind a proxy that forwards client-supplied `oai-authenticated-*` headers.
For independent hosting, implement and test a real authentication adapter that
verifies sessions, strips inbound identity headers and injects verified identity;
provision your own D1 binding and apply the included SQL migrations. Until then,
keep the server on localhost and use the local demo. The development mock identity
is for loopback only and must not be used for a public service.

No production database, private hosting metadata or deployment credentials are
included. Build locally with `npm run build`; `npm run start` serves the generated
worker locally, not a public production endpoint.

## Provider keys, cost and privacy

Enter your own key in the live-play UI. Keys are kept in the active page memory and
sent only with the requested provider operation; they are not committed, included
in game transcripts or intentionally persisted by the application. Never put keys
in system prompts, repository files or screenshots. Use provider-side restricted
keys and spending controls as well.

Free OpenRouter routes have a zero-price provider filter, bounded concurrency,
backoff and no silent paid fallback. Free availability and quotas may change.
Paid providers require consent. Application budget reservations are conservative
controls, **not a guarantee of an account-wide invoice ceiling**. Unknown usage or
expired price data pauses the relevant paid workflow. See
[free-model reliability](docs/FREE_MODEL_RELIABILITY.md).

Model reasoning cannot guarantee a nonempty response. Empty / invalid output is
validated and reported, with a bounded recovery or clearly labeled rule-based
fallback rather than silently treating it as a model decision.

## Tests

```sh
python3 -m venv .venv
. .venv/bin/activate
pip install -r engine/requirements.txt
node scripts/package-engine.mjs
node --experimental-strip-types --test tests/*.test.mjs
python3 -m unittest discover -s engine/tests -v
python3 engine/tests/randomized_qa.py
npx tsc --noEmit
python3 scripts/vendor-runtime.py
node scripts/vendor-runtime.mjs
```

Tests use mocks, local SQLite and deterministic rules; they do not require keys or
prove real provider availability. Runtime smoke tests require the downloaded
Pyodide assets. Python reference tests require the requirements above.

## License and attribution

Original Moonlit Council contributions are MIT licensed; third-party material
retains its own terms. See [LICENSE](LICENSE),
[third-party notices](public/THIRD_PARTY_NOTICES.txt), [engine provenance](engine/NOTICE.md)
and [runtime licenses](public/pyodide/README.md).

- AutoWerewolf: modified Python subset and separately identified TypeScript port.
  The pinned upstream declares MIT but does not contain a standalone license file;
  that provenance limitation is documented rather than inventing upstream terms.
- AI Werewolf Live: visual / interaction inspiration, attribution retained.
- Pyodide / CPython / Python wheels: reproducibly fetched unmodified runtime with
  pinned SHA-256, original source links and copied license notices.

This public repository has a fresh history and contains no private deployment
history, provider keys, user game records or evaluation transcripts.
