+++
title = "Mystic Skills"
weight = 5
chapter = false
pre = "<b>5.5. </b>"
+++


### **1. Hugo Commands**
1.1
```bash
hugo build
```

1.2
```bash
hugo server -D
```


---

### **2\. Git Commands**

2.1 Check repository status

```bash
git status
```

2.2 List local branches

```bash
git branch
```

2.3 List all branches (local + remote)

```bash
git branch -a
```


2.4 Switch to another branch

```bash
git checkout <branch-name>
```

2.5 Create and switch to a new branch

```bash
git checkout -b <branch-name>
```

2.6 Pull the latest updates from remote

```bash
git pull
```


2.7 Fetch updates without merging

```bash
git fetch
```


2.8 Add files to staging

```bash
git add <file>
```

Add all changes:

```bash
git add .
```


2.9 Commit changes

```bash
git commit -m "your message"
```


2.10 Push changes to remote

```bash
git push
```

Push a new branch for the first time:

```bash
git push -u origin <branch-name>
```


2.11 View remote repositories

```bash
git remote -v
```


2.12 Merge a branch into the current branch

```bash
git merge <branch-name>

# Example:
git checkout main
git merge developer
```

2.13 Delete a branch

Delete local branch:

```bash
git branch -d <branch-name>
```

Delete remote branch:

```bash
git push origin --delete <branch-name>
```


2.14 Undo changes

Discard changes in a file:

```bash
git checkout -- <file>
```

Reset everything to the last commit:

```bash
git reset --hard
```
---

### **3\. PowerShell Commands**

3.1 Automatically update _index.md for all weeks in a specific directory

```bash
# Target directory path

$targetDir = "content/1-Worklog/1.3-DuongBinhMinh"

# Check if directory exists

if (-not (Test-Path $targetDir)) {

    Write-Host "Error: Directory not found $targetDir" -ForegroundColor Red

    break

}

# Get subfolders

$folders = Get-ChildItem -Path $targetDir -Directory

foreach ($folder in $folders) {

    if ($folder.Name -match "Week_(\d+)") {

        $weekNum = $matches[1]

        $filePath = Join-Path $folder.FullName "_index.md"

        if (Test-Path $filePath) {

            # Read file content

            $content = Get-Content -Path $filePath -Raw -Encoding UTF8

            # Generate new pre value

            $newPreLine = "pre = `" <b> 1.3.$weekNum. </b> `""

            # Replace old pre line

            $newContent = $content -replace '(?m)^pre\s*=\s*".*"', $newPreLine

            # Overwrite file

            Set-Content -Path $filePath -Value $newContent -Encoding UTF8

            Write-Host "Updated Week $($weekNum): $newPreLine" -ForegroundColor Green

        } else {

            Write-Host "Skipped $folder.Name (_index.md not found)" -ForegroundColor Yellow

        }

    }

}

Write-Host "Update completed!" -ForegroundColor Cyan
```

3.2 Standardize folder structure, convert Leaf Bundles → Branch Bundles, fix frontmatter and internal links

```bash
# --- CONFIGURATION ---

$worklogPath = "content/1-Worklog"

# --- HELPER FUNCTION: Rename index.md to _index.md ---

Function Fix-IndexFileName ($dirPath) {

    $wrongFile = Join-Path $dirPath "index.md"

    $correctFile = Join-Path $dirPath "_index.md"

    if (Test-Path $wrongFile) {

        if (-not (Test-Path $correctFile)) {

            Rename-Item -Path $wrongFile -NewName "_index.md"

            Write-Host "  [File] Renamed index.md -> _index.md" -ForegroundColor Magenta

        }

    }

}

Write-Host "=== STARTING HUGO DATA STANDARDIZATION ===" -ForegroundColor Cyan

# Get all user directories (e.g., 1.1, 1.2, 1.3 ...)

$userFolders = Get-ChildItem -Path $worklogPath -Directory

foreach ($userFolder in $userFolders) {

    if ($userFolder.Name -match "^(\d+\.\d+)-") {

        $userPrefix = $matches[1]

        Write-Host "`n--- Processing User: $($userFolder.Name) (Prefix: $userPrefix) ---" -ForegroundColor Cyan

        # Fix parent index file

        Fix-IndexFileName $userFolder.FullName

        $parentIndexFile = Join-Path $userFolder.FullName "_index.md"

        if (Test-Path $parentIndexFile) {

            $pContent = Get-Content -Path $parentIndexFile -Raw -Encoding UTF8

            # Fix Hugo shortcode format

            $pContent = $pContent -replace '\{\{<\s*relref', ('{' + '{% relref')

            $pContent = $pContent -replace '>\}\}\)', ('%' + '}})')

            # Fix broken internal links

            $pContent = [Regex]::Replace($pContent, 'relref\s*"[^"]*?Week_(\d+)"', {

                param($m)

                $w = $m.Groups[1].Value

                return 'relref "' + "$userPrefix.$w-Week_$w" + '"'

            })

            Set-Content -Path $parentIndexFile -Value $pContent -Encoding UTF8

            Write-Host "  [Link] Updated links in parent _index.md" -ForegroundColor Green

        }

        # Process week folders

        $weekFolders = Get-ChildItem -Path $userFolder.FullName -Directory | Where-Object { $_.Name -match "Week_" }

        foreach ($weekFolder in $weekFolders) {

            if ($weekFolder.Name -match "Week_(\d+)") {

                $weekNum = $matches[1]

                $correctFolderName = "$userPrefix.$weekNum-Week_$weekNum"

                $currentFolderPath = $weekFolder.FullName

                if ($weekFolder.Name -ne $correctFolderName) {

                    Rename-Item -Path $currentFolderPath -NewName $correctFolderName

                    $currentFolderPath = Join-Path $userFolder.FullName $correctFolderName

                    Write-Host "  [Folder] Renamed to: $correctFolderName" -ForegroundColor Yellow

                }

                Fix-IndexFileName $currentFolderPath

                $childIndexFile = Join-Path $currentFolderPath "_index.md"

                if (Test-Path $childIndexFile) {

                    $cContent = Get-Content -Path $childIndexFile -Raw -Encoding UTF8

                    $newPre = "pre = `" <b> $userPrefix.$weekNum. </b> `""

                    if ($cContent -match '(?m)^pre\s*=') {

                        $cContent = $cContent -replace '(?m)^pre\s*=\s*".*"', $newPre

                    } else {

                        $cContent = $cContent -replace '(?m)^weight\s*=\s*(\d+)', "weight = `$1`n$newPre"

                    }

                    Set-Content -Path $childIndexFile -Value $cContent -Encoding UTF8

                    Write-Host "  [Pre] Updated pre to: $userPrefix.$weekNum." -ForegroundColor Gray

                }

            }

        }

    }

}

Write-Host "`n=== COMPLETED! PLEASE RUN: hugo server -D ===" -ForegroundColor Green
```

3.3 Create multiple folders at once

```bash
$basePath = "D:\IT\AWS-FCJ\AWS-Workshop\content\5-Workshop"

$folders = @(

    "5.1-Workshop_Overview",

    "5.2-Prerequisite",

    "5.3-Deploy_Flow",

    "5.4-Clean_Up"

)

foreach ($f in $folders) {

    $fullPath = Join-Path $basePath $f

    New-Item -ItemType Directory -Path $fullPath -Force | Out-Null

    New-Item -ItemType File -Path (Join-Path $fullPath "_index.md") -Force | Out-Null

}
```

3.4 Create a single folder

```bash
$basePath = "D:\IT\AWS-FCJ\AWS-Workshop\content\5-Workshop\5.3-Deploy_Flow"

$folderName = "5.3.3-CI/CD"

$fullPath = Join-Path $basePath $folderName

New-Item -ItemType Directory -Path $fullPath -Force | Out-Null

New-Item -ItemType File -Path (Join-Path $fullPath "_index.en.md") -Force | Out-Null
New-Item -ItemType File -Path (Join-Path $fullPath "_index.vi.md") -Force | Out-Null

```


3.5 Display the entire project structure

```bash
tree /f /a
```


3.6 Display a specific directory structure

```bash
tree content/1-Worklog/1.1-PhanCanhTuanDat /F
```


3.7 Recreate gh-pages worktree safely

```bash
# 0. Remove current public folder
Remove-Item -Recurse -Force .\public

# 1. Remove old gh-pages worktree (even if folder no longer exists)

git worktree remove "D:/IT/AWS-FCJ/AWS-Workshop/public/public" --force

# 2. Clean orphaned worktree metadata

git worktree prune

# 3. Verify remaining worktrees

git worktree list

# 4. Recreate gh-pages worktree at /public

git worktree add -B gh-pages public origin/gh-pages

# 5. Verify status

cd public

git status
```