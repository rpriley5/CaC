# AAP-CaC (AAP 2.4/AWX ➜ AAP 2.5) — Migration Starter Pack

> **Important support notice**  
> This repository is a **community starting point** for moving Automation Controller objects that are exportable via **Config-as-Code (CaC)** from **AAP 2.4 or AWX** into **AAP 2.5**.  
> It is **not** an official or supported migration path from Red Hat. Validate everything in non-production first.

> **Credential secrets cannot be migrated**  
> CaC **does not export secret values**. During import, **credential objects are created without their passwords/tokens/private keys**. You must **re-enter secrets manually** in AAP 2.5 (or seed them via your own secret manager/API) after the import completes.

## What this repo contains

- **`export_24.yml`** — exports CaC data from **AAP ≤ 2.4 / AWX**. Use this when your source is 2.4 or older.  
- **`export_25.yml`** — exports CaC data from **AAP 2.5+** (useful for iterative backups or 2.5→2.5 transfers).  
- **`import_25.yml`** — imports the exported CaC bundle into **AAP 2.5** in dependency-safe order.  
- **`vars.yml`** — example variables file you customize for your environments.  
- **`collections/`** — pinned collections for repeatable runs (install with `ansible-galaxy`).  

---

## Prerequisites

- **Ansible** on your runner host.  
- Network/API access to **source controller** (AAP 2.4/AWX) and **destination controller** (AAP 2.5).  
- **Personal Access Tokens** (PATs) with admin-level permissions for both.  
- (Recommended) a non-prod target for dry runs.

Install required collections (from the included lock/requirements under `collections/` if present):

```bash
ansible-galaxy install -r collections/requirements.yml
```

---

## Configure `vars.yml`

Create or edit `vars.yml` in the repo root. The exact variable names in your playbooks may vary slightly; here’s a **working template**:

```yaml
# --- Source (AAP 2.4 or AWX) ---
controller_username: "admin" # MUST BE ADMIN USER
controller_password: "xxxxxxxxxxxxxx"
controller_hostname: "aap.example.com" # DO NOT INCLUDE http:// or https://
controller_api_plugin: awx.awx.controller_api


# --- Destination (AAP 2.5) ---
aap_username: "admin" # MUST BE ADMIN USER
aap_password: "xxxxxxxxxxxxxx"
aap_hostname: "aap.example.com" # DO NOT INCLUDE http:// or https://

# General
output_path: "./exports"
flatten_output: true
aap_validate_certs: "false"
controller_validate_certs: "false"
```

> Keep tokens out of Git. Prefer environment variables + `ansible-vault` or your secret manager.

---

## Run order (copy/paste)

### A) Export from AAP 2.4 / AWX
```bash
ansible-playbook export_24.yml -e @vars.yml
```

### B) Export from AAP 2.5
```bash
ansible-playbook export_25.yml -e @vars.yml
```

### C) Import into AAP 2.5
```bash
ansible-playbook import_25.yml -e @vars.yml
```

> **After the import:** re-enter all credential secrets in AAP 2.5.

---

## Typical object coverage

- Organizations, teams, users
- Credential types, **credentials (metadata only)**  
- Projects, inventories/groups/hosts, inventory sources  
- Execution environments, notification templates  
- Job templates, workflow job templates, schedules, RBAC mappings

---

## Tips & troubleshooting

- **401/403**: validate tokens and `*_validate_certs`.  
- **Project sync fails**: re-enter SCM credentials.  
- **Credential test fails**: re-enter secrets.  
- **Re-runs**: imports are idempotent.

---

## FAQ

**Is this an official Red Hat migration?**  
No.

**Why aren’t passwords exported?**  
Secret values are encrypted at rest and not emitted by CaC.

---

