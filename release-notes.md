# Release Notes — CloudNotes Release Lab

**Forked repo:** https://github.com/AaryanPanda/cloudnotes-release-lab

## Task 1 — Terraform/HCL validation fault

**Original error (paste `terraform validate` output):**

```
Error: Reference to undeclared input variable
  on main.tf line 33, in resource "local_file" "release_manifest":
  33:     bucket      = var.bucket_name_old
```

**Root cause:**

`main.tf` line 33 referenced `var.bucket_name_old`, which is not declared in `variables.tf`.

**Fix applied:**

Changed it to `var.bucket_name`.

**Evidence (clean validate output):**

```
Success! The configuration is valid.
Plan: 2 to add, 0 to change, 0 to destroy.
Saved the plan to: tfplan
```

---

## Task 2 — Reusable module + state isolation fault

**Original problem:**

`main.tf` hand-defined a duplicate `local_file "cloudnotes_bucket"` instead of using `modules/storage`, and `envs/staging/backend.hcl` pointed at dev's state file.

**Fix applied:**

- Replaced the duplicate resource with `module "storage"` (`source = "./modules/storage"`).
- `outputs.tf` now uses `module.storage.bucket_path`.
- Staging path changed to `envs/staging/terraform.tfstate`.

**Evidence:**

```
Initializing modules...
Terraform has been successfully initialized!
Success! The configuration is valid.
```

---

## Task 3 — Container build + secret fault

**Original error (paste `docker build` output):**

```
Dockerfile:15
>>> COPY --from=build /app/node_modules ./node_modules
ERROR: failed to resolve source metadata for docker.io/library/build:latest: pull access denied
```

**Root cause:**

`COPY --from=build` referenced a stage that doesn't exist (the stage is named `deps`), and `ENV API_KEY=sk_live_examplekey123` was hardcoded in the image.

**Fix applied:**

- Line 15: changed to `COPY --from=deps`.
- Line 20: removed the `API_KEY` line. The key is now supplied at runtime through `.env`.

**Evidence (`curl http://localhost:8080/health`):**

```
{"status":"ok","service":"cloudnotes-api","version":"0.1.0","apiKeyConfigured":true}
Security check PASSED.
```
