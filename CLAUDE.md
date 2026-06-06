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

---

## Challenge 2 — Kubernetes Manifests (`k8s/`)

**Goal:** Deploy app to local cluster, expose at `http://fsl-challenge.me` (port 80 via `/etc/hosts → 127.0.0.1`).

### Architecture (from source)
- **Dockerfile:** two-stage build — Node 15 builder → `nginx:1.21-alpine`
- **nginx.conf:** listens on port **80**; serves static files; `/health` returns `200 ok`
- **Image name expected:** `rdicidr:latest` (must be built locally before applying manifests)

### Requirements
1. Workload must be a **StatefulSet** with ≥ 3 replicas
2. Both Service and StatefulSet in namespace **`production`**
3. Service listens on port **8080** (targetPort 80 → nginx)
4. Pods must reach Running with liveness and readiness probes passing

### Defects found

**`k8s/deployment.yaml`**

| # | Defect | Detail |
|---|--------|--------|
| D1 | `kind: Deployment` | Must be `StatefulSet`; also requires `spec.serviceName` field |
| D2 | `replicas: 2` | Minimum 3 required |
| D3 | No namespace | Missing `namespace: production` in metadata |
| D4 | `containerPort: 3000` | nginx listens on 80, not 3000 |
| D5 | Probe `port: 3000` | Both liveness and readiness probes target port 3000; must be 80 |
| D6 | Resource requests too high | `memory: 4Gi / cpu: 4000m` → pods Pending on local cluster; use ~64Mi/100m |
| D7 | `livenessProbe.failureThreshold: 1` | Kills pod on first probe miss; minimum 3 recommended |
| D8 | Honeypot comment on `imagePullPolicy` | `IfNotPresent` is correct for locally built images; ignore the FIXME comment |

**`k8s/service.yaml`**

| # | Defect | Detail |
|---|--------|--------|
| S1 | No namespace | Missing `namespace: production` in metadata |
| S2 | `selector: app: rdicidr-web` | Label mismatch — pods have `app: rdicidr` |
| S3 | `port: 80` | Must be `8080` per requirements |
| S4 | `targetPort: 80` | Correct value (nginx on 80), but blocked by S2/S3 |

**Missing resources**

| # | What | Why needed |
|---|------|-----------|
| M1 | `Namespace` manifest for `production` | Neither manifest creates the namespace |
| M2 | Ingress resource | ClusterIP service is not reachable at `http://fsl-challenge.me`; need Ingress routing port 80 → service:8080 |
| M3 | `/etc/hosts` entry | `127.0.0.1 fsl-challenge.me` required for local hostname resolution |

### Fix plan (when instructed)
1. Add `k8s/namespace.yaml` creating `production` namespace
2. Rewrite `deployment.yaml` as StatefulSet: replicas=3, namespace=production, containerPort=80, probe port=80, reasonable resources, failureThreshold≥3, add `serviceName`
3. Fix `service.yaml`: namespace=production, selector `app: rdicidr`, port=8080, targetPort=80
4. Add `k8s/ingress.yaml` routing `fsl-challenge.me → rdicidr-service:8080`
5. Build Docker image locally: `docker build -t rdicidr:latest codebase/rdicidr-0.1.0/` (load into cluster if using kind/minikube)
6. Add `/etc/hosts` entry: `127.0.0.1 fsl-challenge.me`

---

### Fix plan (Challenge 1 — when instructed)
1. Change `node-version: '14'` → `'15'` in all four jobs
2. Fix `build` job cache restore key: `deps-` → `node-modules-`
3. Add `eslint-plugin-prettier` and `eslint-config-prettier` to `package.json` dependencies
4. Fix `App.test.js` test for API URL (match empty env var output or set `REACT_APP_API_URL` in CI env)
5. Fix `needs` chain: `test: needs: lint`, `build: needs: test`
6. Change test step command to `CI=true npm run test`
