# Claude Code Project Instructions

## Session Initialization

At the start of every session, read `.chat-history/log.md` for previous conversation context. If the file does not exist, create it (and the `.chat-history/` directory if needed) with an empty body before proceeding.

## Response Logging (mandatory, silent)

After **every** response in this project — without exception, without asking for confirmation — append an entry to `.chat-history/log.md` using exactly this format:

```
---
- timestamp: "<ISO 8601 timestamp if available, otherwise estimate based on conversation order>"
- user_prompt: "<the user's original prompt verbatim or closely paraphrased>"
- assistant_response_summary: "<concise but specific summary — include function names, file paths, endpoints, or key decisions>"
- files_affected: "<comma-separated list of files created or modified, or 'none'>"
```

### Logging rules
- **Never skip** an exchange. Every prompt/response pair must produce one log entry.
- **Never delete** previous entries. Only append.
- **Be precise** about `files_affected`: list only files explicitly created or modified during that response.
- **Do this silently**: no confirmation prompts, no mentions of logging to the user unless they ask.
- If `.chat-history/log.md` does not exist when you go to append, create it first.
- Use the `Edit` tool (append) or `Bash` (`>>`) to add entries — never overwrite the file.

---

## Challenge 1 — CI Pipeline (`codebase/rdicidr-0.1.0`)

**App:** React 17 CIDR calculator (CRA). Pipeline at `.github/workflows/ci.yaml`.  
**Required step order:** install → lint → test → build.

### Defects found (do not fix until instructed)

| # | File | Defect | Impact |
|---|------|--------|--------|
| 1 | `ci.yaml` | `node-version: '14'` in all jobs; `.nvmrc`/`package.json` engines require Node 15; `.npmrc` sets `engine-strict=true` | Fatal — `npm ci` fails on engine check |
| 2 | `ci.yaml` | `build` job restores cache with key prefix `deps-` but `install` saves with `node-modules-` | Fatal — `build` has no `node_modules` |
| 3 | `package.json` | `eslintConfig` extends `plugin:prettier/recommended` but `eslint-plugin-prettier` is not installed | Fatal — `npm run lint` errors on unknown plugin |
| 4 | `src/App.test.js` | Test asserts `api.rdicidr.com` appears in output; `REACT_APP_API_URL` is unset in CI so env var renders empty | Fatal — Jest test fails |
| 5 | `ci.yaml` | `test` needs only `install`; `build` needs only `install` — both should be sequential (install→lint→test→build) | Structural — jobs run out of order |
| 6 | `ci.yaml` | Test step uses `npm test -- --watchAll=false` instead of `CI=true npm run test` | Minor — does not match spec |

### Fix plan (when instructed)
1. Change `node-version: '14'` → `'15'` in all four jobs
2. Fix `build` job cache restore key: `deps-` → `node-modules-`
3. Add `eslint-plugin-prettier` and `eslint-config-prettier` to `package.json` dependencies
4. Fix `App.test.js` test for API URL (match empty env var output or set `REACT_APP_API_URL` in CI env)
5. Fix `needs` chain: `test: needs: lint`, `build: needs: test`
6. Change test step command to `CI=true npm run test`
