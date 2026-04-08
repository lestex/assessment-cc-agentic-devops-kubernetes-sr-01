# FSL DevOps Challenge — Kubernetes & CI/CD Troubleshooting

**Format:** Live coding session with verbal narration  
**Total time:** ~90 minutes (Part 1: ~40 min / Part 2: ~50 min)  
**AI usage:** Fully permitted. You are expected to narrate your reasoning as you work.

---

## Context

You have inherited a broken deployment for **rdicidr**, a React-based CIDR calculator. The repository includes a GitHub Actions CI pipeline and Kubernetes manifests. Both have been introduced with defects that prevent the application from building, deploying, and serving traffic correctly.

Your job is to find and fix every defect. The codebase is in `codebase/rdicidr-0.1.0/` and the Kubernetes manifests are in `k8s/`.

---

## Part 1 — CI/CD Pipeline (~40 minutes)

The GitHub Actions pipeline at `.github/workflows/ci.yaml` is broken. Examine the pipeline, the application's `package.json`, `.npmrc`, `.nvmrc`, and any other relevant files to identify and fix all defects.

### Deliverables

- Corrected `ci.yaml` (and any other files you modify)
- For each defect found:
  - **What** is broken
  - **Why** it fails (root cause, not just symptom)
  - **What** you changed and why that fix is correct

### Notes

- The pipeline has multiple independent jobs. Failures in one job may mask failures in another — treat each job independently.
- Environment variables available locally (via `.env`) are not automatically available in CI.
- Node version constraints are declared in more than one place in the project.

---

## Part 2 — Kubernetes Manifests (~50 minutes)

The manifests at `k8s/deployment.yaml` and `k8s/service.yaml` have defects across resource scheduling, health checking, networking, and manifest structure.

### Deliverables

- Corrected `deployment.yaml` and `service.yaml`
- For each defect found:
  - **What** is broken
  - **Why** it fails (root cause, not just symptom)
  - **What** you changed and why that fix is correct

### Cluster Verification Requirement

**After each fix, you must run `kubectl apply -f k8s/` and capture the output.** For any fix that changes pod behavior (scheduling, probes, networking), also run and show the output of:

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl get endpoints rdicidr-service
```

Static analysis alone is not sufficient. Fixes that cannot be verified in a running cluster (minikube or kind) must be accompanied by a clear explanation of what the expected cluster behavior change would be and why.

### Notes

- Some defects interact with each other — fixing one may reveal another.
- Not everything that looks like a defect is one. Evaluate each potential issue against the actual runtime behavior of the container before changing it.
- The container image is built and loaded locally. Consider what that means for image pull behavior.
- The nginx configuration in the Dockerfile is relevant to understanding the correct port values.

---

## Narration Expectation

Throughout both parts, **think out loud**. Before making a change, explain:

1. What you observed (the symptom)
2. What you believe the root cause is
3. What you expect the fix to accomplish

If you use an AI tool, narrate what context you gave it, whether its suggestion was correct, and — if you modified or rejected the suggestion — why.

---

## Setup

```bash
# Verify local environment
node --version   # should be 15.x
npm --version    # should be 7.x

# Install dependencies (from codebase/rdicidr-0.1.0/)
npm ci

# Build and run the Docker image locally
docker build -t rdicidr:latest .
docker run -d -p 8080:80 --name rdicidr-test rdicidr:latest
curl http://localhost:8080/health   # should return: ok

# Start a local cluster (choose one)
minikube start
# or
kind create cluster
```
