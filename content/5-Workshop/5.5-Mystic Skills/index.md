+++
title = "Mystic Skills"
weight = 5
chapter = false
pre = "<b>5.5. </b>"
+++


### **1. Hugo Commands**
- Command to create multiple folders at a time
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
    New-Item -ItemType File -Path (Join-Path $fullPath "index.md") -Force | Out-Null
}
```

- Command to create one folder at a time
```bash
$basePath = "D:\IT\AWS-FCJ\AWS-Workshop\content\5-Workshop"
$folderName = "5.1-Workshop_Overview"   # <-- Change name here

$fullPath = Join-Path $basePath $folderName
New-Item -ItemType Directory -Path $fullPath -Force | Out-Null
New-Item -ItemType File -Path (Join-Path $fullPath "index.md") -Force | Out-Null
```

- Command to 


### **2. Git Commands**
**2.1 Check repository status**
```bash
git status
```

**2.2 List branches (local)**
```bash
git branch
```

**2.3 List all branches (local + remote)**
```bash
git branch -a
```

**2.4 Switch to another branch**
```bash
git checkout <branch-name>
```
Example: git checkout developer

**2.5 Create and switch to a new branch**
```bash
git checkout -b <branch-name>
```
Example: git checkout -b feature/login

**2.6 Pull latest updates from remote**
```bash
git pull
```

**2.7 Fetch updates without merging**
```bash
git fetch
```

**2.8 Add files to staging**
```bash
git add <file>
```
Add all changes: git add .

**2.9 Commit changes**
```bash
git commit -m "your message"
```

**2.10 Push changes to remote**
```bash
git push
```
Push a new branch for the first time:
```bash
git push -u origin <branch-name>
```

2.11 View remote**
```bash
git remote -v
```

**2.12 Merge a branch into the current branch**
```bash
git merge <branch-name>
```
Example:
```bash
git checkout main
git merge developer
```

**2.13 Delete a branch**
Delete local branch: 
```bash
git branch -d <branch-name>
```
Delete remote branch: 
```bash
git push origin --delete <branch-name>
```

**2.14 Undo changes**
Discard changes in a file:
```bash
git checkout -- <file>
```
Reset everything to last commit:
```bash
git reset --hard
```