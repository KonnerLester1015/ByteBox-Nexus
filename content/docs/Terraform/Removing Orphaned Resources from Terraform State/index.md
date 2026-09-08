---
title: "Removing Orphaned Resources from Terraform State"
---

## When to use this

You'll hit this when trying to delete an HCP Terraform (Terraform Cloud) workspace and the **"Manually destroy"** step fails, usually because:

- The target infrastructure (VM, instance, etc.) was already deleted outside of Terraform (manually, or the host/cluster it lived on is gone).
- The provider block for that resource has since been removed from the `.tf` config (e.g. the module was refactored out), but the resource is still tracked in state.

The workspace UI will show something like:

> **Delete workspace** — This workspace manages N resources. You must destroy these resources before you can delete this workspace.

And attempting **Queue destroy plan** fails with:

{{< callout type="error" >}}
**Error: Provider configuration not present**

To work with module.<module_name>.<resource_type>.<resource_name> (orphan) its
original provider configuration at module.<module_name>.provider["registry.terraform.io/<provider>"]
is required, but it has been removed. This occurs when a provider configuration
is removed while objects created by that provider still exist in the state.
Re-add the provider configuration to destroy this resource, after which you
can remove the provider configuration again.
{{< /callout >}}

Since the resource no longer exists in reality, there's nothing to actually destroy; you just need to tell Terraform to forget about it in state.

## Prerequisites

- Terraform CLI installed, version matching or exceeding the workspace's required version (check the workspace's **Settings > General > Terraform version**). If your local version is older, either upgrade the CLI or pass `-ignore-remote-version` on `init` (see [Gotchas](#gotchas) below).
- An HCP Terraform account with permissions on the org/workspace, and access to run `terraform login`.
- The exact **organization name** and **workspace name**, copied from the workspace URL/settings in the HCP Terraform UI (case-sensitive).

## Steps

### 1. Create a scratch working directory

This does **not** need to contain the actual DevOps repo or module code; remote state lookups only need a `cloud` block.

```powershell {filename="PowerShell"}
mkdir tf-state-cleanup
cd tf-state-cleanup
```

### 2. Add a minimal `cloud` block

Create `main.tf`:

```hcl {linenos=table,hl_lines=[3,5]linenostart=1}
terraform {
  cloud {
    organization = "your-org-name"
    workspaces {
      name = "your-workspace-name"
    }
  }
}
```

### 3. Authenticate and initialize

```bash
terraform login
terraform init
```

{{< callout type="warning" >}}
  If you already ran `init` before adding the `cloud` block, clear out any local backend artifacts first so they don't conflict:
  
  ```powershell {filename="PowerShell"}
  Remove-Item -Recurse -Force .terraform -ErrorAction SilentlyContinue
  Remove-Item -Force terraform.tfstate, terraform.tfstate.backup -ErrorAction SilentlyContinue
  terraform init
  ```

{{< /callout >}}


### 4. Confirm you're pointed at the right workspace

```bash
terraform state list
```

Compare the output against the resource count/list shown in the HCP Terraform UI for that workspace. Don't proceed until this matches. This is your safety check that you're editing the correct workspace's state.

### 5. Remove only the orphaned resource

Copy the exact resource address from the error message:

```bash
terraform state rm 'module.<module_name>.<resource_type>.<resource_name>[0]'
```

Example:

```bash
terraform state rm 'module.linux-deploy.vsphere_virtual_machine.template[0]'
```

Terraform will echo back what it removed.

> **Note on data sources:** Resources like `data.vsphere_datacenter.dc`, `data.vsphere_network.network`, etc. are read-only lookups, not managed infrastructure. You generally don't need to remove these since they don't block destroy/delete and aren't "destroyed" in any real sense.

### 6. Re-run the destroy plan and delete the workspace

Back in the HCP Terraform UI:

1. Go to **Settings > Destruction and deletion**.
2. Click **Queue destroy plan**. With the orphaned resource gone from state, the provider error should no longer occur.
3. Once the destroy plan completes, click **Delete workspace**.

If the workspace still won't delete, **Force delete from HCP Terraform** is a safer fallback at this point, since the only real managed resource has already been cleared from state manually.

## Gotchas

- **Version mismatch on init:**

  {{< callout type="error" >}}
  **Error: Incompatible Terraform version**
  
  The local Terraform version (x.x.x) does not meet the version requirements
  for remote workspace ORG/WORKSPACE (~> x.x.x).
  {{< /callout >}}

  Prefer upgrading your local CLI to match. `-ignore-remote-version` works but is best avoided for anything that writes state, since it's not guaranteed the write format lines up cleanly across a large version gap.

- **`state rm` is scoped to a single workspace.** It only affects the state of the workspace named in your `cloud` block. It can't touch other workspaces, projects, or orgs, and it never makes any calls to the actual infrastructure provider (vSphere, AWS, etc.). The only real risk is a typo'd resource address removing the *wrong* resource from the *same* workspace's state. Always confirm with `terraform state list` first and read the confirmation output after `state rm`.

- **This doesn't destroy anything for real.** `state rm` just edits Terraform's bookkeeping. If the resource actually still exists somewhere, this will orphan it (Terraform will no longer track or manage it) only do this when you've confirmed the real-world object is already gone.
