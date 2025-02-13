# **Git Branch Archival Process**

## **Overview**
This document provides a step-by-step process on archiving outdated Git branches by creating tags for them before deletion. The process ensures that historical changes remain accessible while keeping the repository clean and manageable. This below archival script can be performed based on a specified cutoff date or a predefined list of branches.


## **Prerequisites**
- Ensure you have the necessary permissions to delete remote branches.
- Confirm that you have Git installed and configured on your system.
- Identify branches that need to be archived.

## **Step 1: Identify Branches for Archival**
You can list all remote branches sorted by their last commit date:
```bash
  git for-each-ref --sort=committerdate --format='%(committerdate:short) %(refname:short)' refs/remotes/origin/
```

To identify branches older than a specific date, use:

**Note:** Here for example i was using some random date `2024-04-16`

```bash
  git for-each-ref --sort=committerdate --format='%(committerdate:short) %(refname:short)' refs/remotes/origin/ | awk '$1 < "2024-04-16" {print $2}'
```
Replace `2024-04-16` with the appropriate cutoff date.


## **Step 2: Check for Active Worktrees**
Before deleting branches, ensure they are not in use by any worktree:
```bash
  git worktree list
```
If a branch is in use, remove it from the worktree (If you want to delete it from local):
```bash
  git worktree remove /path/to/worktree
  git worktree prune
```

## **Step 3: Archive and Delete Branches**
Use the following script to automate the archival process:

```bash
#!/bin/bash

# Set the cutoff date for automatic archival (YYYY-MM-DD format)
CUTOFF_DATE=""

# List of specific branches to archive (leave empty if using date-based archival)
BRANCH_LIST=("")

# Determine branches to archive
if [ ${#BRANCH_LIST[@]} -eq 0 ]; then
  echo "Archiving branches older than $CUTOFF_DATE..."
  BRANCHES=$(git for-each-ref --sort=committerdate --format='%(committerdate:short) %(refname:short)' refs/remotes/origin/ | awk '$1 < "'"$CUTOFF_DATE"'" {print $2}')
else
  echo "Archiving specified branches..."
  BRANCHES=${BRANCH_LIST[@]}
fi

for BRANCH in $BRANCHES
do
  BRANCH_NAME=${BRANCH#origin/}  # Remove 'origin/' prefix if present
  TAG_NAME="archive-$BRANCH_NAME"  # Create a tag with prefix "archive-"

  echo "Archiving branch: $BRANCH_NAME -> Tag: $TAG_NAME"

  git checkout $BRANCH_NAME || continue
  git tag -a $TAG_NAME -m "Archived $BRANCH_NAME"
  git push origin $TAG_NAME
  git branch -d $BRANCH_NAME
  git push origin --delete $BRANCH_NAME

done
```

---

### **Execution Steps**

1. Save the script as `archive-branches.sh`.
2. Grant execution permissions:
   
   **For Linux/macOS:**
   ```bash
   chmod +x archive-branches.sh
   ```
   
   **For Windows (Git Bash or PowerShell):**
   ```powershell
   Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
   ./archive-branches.sh
   ```

3. Run the script:
   ```bash
   ./archive-branches.sh
   ```
   

### 2. Worktree Handling

If the script encounters an error while deleting a branch due to an active worktree, remove the worktree first:

```bash
git worktree list
git worktree remove /path/to/worktree
```

Then, rerun the script.

### 3. Restoring Archived Branches

If a branch needs to be restored, checkout the tagged version:

```bash
git checkout -b <branch_name> archive-<branch_name>
git push origin <branch_name>
```

---

## Conclusion

This process ensures that deprecated branches are archived safely while maintaining repository hygiene. The script supports both automated date-based archival and manual branch selection.



**-Seeram Sandeep**
