---
title: Identifying DFS Target Paths
type: docs
sidebar:
  open: false
---
## Overview

DFS namespaces present a single logical path (e.g. `\\example.com\share01\Finance`) that hides the actual `\\server\share` target(s) behind it. This is convenient for users but can make troubleshooting harder when you need to know which physical file server a DFS folder actually points to (for backup jobs, permissions audits, migrations, or troubleshooting slow access), you normally have to open each folder's Properties dialog and check the "DFS" tab by hand.

The `Get-DfsFolderTargets.ps1` script automates this by walking every folder under a DFS namespace root and reporting the real target path(s) for each one, along with which target is currently Online.

{{< callout >}}
  Run this script on a DFS server, or any machine with the DFS Namespaces (DFSN) PowerShell module installed. If the module isn't present, install the RSAT DFS tools with:
  ```PowerShell
  Add-WindowsCapability -Online -Name 'Rsat.FileServices.Tools~~~~0.0.1.0'
  ```
{{< /callout >}}

## Use Case

- **Migrations** - confirm which file server currently backs a DFS folder before moving data or decommissioning a server.
- **Permissions audits** - trace a DFS path back to its real UNC share to check NTFS/share permissions directly.
- **Documentation** - generate an inventory of DFS folder-to-server mappings without clicking through Properties dialogs one at a time.

## How It Works

1. **Checks for the DFSN module.** The script looks for `Get-DfsnFolderTarget` and exits early with instructions if the DFSN PowerShell module isn't available.
2. **Enumerates root folders.** It runs `Get-ChildItem` against the namespace path (default `\\example.com\share01`) to list every folder directly under the namespace root.
3. **Queries DFS targets per folder.** For each folder, it calls `Get-DfsnFolderTarget` to retrieve the underlying `\\server\share` target(s) configured for that DFS folder - the same data shown on the folder's Properties > DFS tab.
4. **Filters to Online targets.** Only targets with a `State` of `Online` are returned, so failed-over or disabled targets don't clutter the output.
5. **Reports results.** Each match is output as an object with `Folder`, `DfsPath`, and `TargetPath`. If a folder has no online target, it's still listed with a placeholder message so nothing is silently skipped. Results are printed with `Format-Table -AutoSize`.

## Script

```PowerShell
<#
.SYNOPSIS
    Shows the DFS referral path(s) for each root folder under a DFS namespace,
    matching what you'd see on the "DFS" tab in a folder's Properties dialog.

.DESCRIPTION
    For every folder directly under the namespace (e.g. \\example.com\share01),
    reports the actual \\server\share path(s) the DFS folder points to and which
    target is currently active (Online). Same info as right-click folder >
    Properties > DFS tab.

.PARAMETER NamespacePath
    UNC path of the DFS namespace root. Default: \\example.com\share01

.EXAMPLE
    .\Get-DfsFolderTargets.ps1

.OUTPUTS
    PSCustomObject with properties:
        Folder       : Name of the DFS folder (leaf)
        DfsPath      : Full UNC path of the DFS folder
        TargetPath   : Actual \\server\share target path
.NOTES
    Run from DFS Server
#>

[CmdletBinding()]
param(
    [string]$NamespacePath = '\\example.com\share01'
)

# Detect availability of the DFSN module once up front
$hasDfsnModule = [bool](Get-Command Get-DfsnFolderTarget -ErrorAction SilentlyContinue)

Write-Host ("DFSN module available : {0}" -f $hasDfsnModule) -ForegroundColor DarkGray

if (-not $hasDfsnModule) {
    Write-Warning "The DFSN PowerShell module is not available."
    Write-Warning "Install RSAT DFS tools:  Add-WindowsCapability -Online -Name 'Rsat.FileServices.Tools~~~~0.0.1.0'"
    return
}

# ---------------------------------------------------------------------------
# Get DFS referral target(s) for one namespace folder path
# ---------------------------------------------------------------------------
function Get-DfsTargets {
    param([string]$Path)

    $folderLeaf = Split-Path $Path -Leaf

    try {
        $targets = Get-DfsnFolderTarget -Path $Path -ErrorAction Stop
        Write-Host "  [$folderLeaf] Get-DfsnFolderTarget OK ($($targets.Count) target(s))" -ForegroundColor DarkGray
        return $targets | Where-Object { ("$($_.State)").Trim().ToLower() -eq 'online' } | ForEach-Object {
            [pscustomobject]@{
                Folder     = $folderLeaf
                DfsPath    = $Path
                TargetPath = $_.TargetPath
            }
        }
    }
    catch {
        Write-Host "  [$folderLeaf] Get-DfsnFolderTarget failed: $($_.Exception.Message)" -ForegroundColor DarkYellow
        return @()
    }
}

# ---------------------------------------------------------------------------
# Main
# ---------------------------------------------------------------------------
Write-Host "Reading DFS folders under: $NamespacePath" -ForegroundColor Cyan

if (-not (Test-Path -LiteralPath $NamespacePath)) {
    Write-Error "Cannot access '$NamespacePath'. Check the path and your permissions."
    return
}

$folders = Get-ChildItem -Path $NamespacePath -Directory -ErrorAction SilentlyContinue
Write-Host ("Found {0} root folder(s). Reading DFS referral paths..." -f $folders.Count) -ForegroundColor Cyan

$all = foreach ($folder in $folders) {
    $targets = @(Get-DfsTargets -Path $folder.FullName)

    if ($targets.Count -gt 0) {
        $targets
    }
    else {
        [pscustomobject]@{
            Folder     = $folder.Name
            DfsPath    = $folder.FullName
            TargetPath = '(no online DFS target found)'
        }
    }
}

$all | Format-Table -AutoSize
```

## Example Output

```
DFSN module available : True
Reading DFS folders under: \\example.com\share01
Found 3 root folder(s). Reading DFS referral paths...
  [Finance] Get-DfsnFolderTarget OK (2 target(s))
  [HR] Get-DfsnFolderTarget OK (1 target(s))
  [IT] Get-DfsnFolderTarget OK (1 target(s))

Folder  DfsPath                        TargetPath
------  -------                        ----------
Finance \\example.com\share01\Finance  \\FS01\Finance$
HR      \\example.com\share01\HR       \\FS02\HR$
IT      \\example.com\share01\IT       \\FS01\IT$
```
