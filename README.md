# 🚀 Bitbucket → GitHub LFS Migration Guide

This document explains how to migrate large files (**>100 MB**) from your Bitbucket repository to GitHub using **Git Large File Storage (LFS)** with the provided script — `git_lfs_migration.sh`.

> **Note:** Run the script using **Git Bash** for proper compatibility during the migration process.

---

## 📋 Prerequisites

Before running the script, make sure you have:

- **Git** installed  
  ```bash
  git --version
Git LFS installed

bash
Copy code
git lfs install
Access to both Bitbucket and GitHub repositories.

🪜 Step 1: Clone the Bitbucket Repository
Clone your existing Bitbucket repository and switch into it:

bash
Copy code
git clone <bitbucket-repo-url>
cd <repo-name>
If you want to migrate all branches, run:

bash
Copy code
git fetch --all
for branch in $(git branch -r | grep -v '\->'); do
    git branch --track "${branch#origin/}" "$branch" 2>/dev/null || true
done
⚙️ Step 2: Prepare the Migration Script
Place the git_lfs_migration.sh script outside your repository folder — to prevent it from being added to Git history accidentally.

Example structure:

arduino
Copy code
/home/user/
├── git_lfs_migration.sh
└── project/
    ├── .git/
    ├── src/
    └── ...
🚀 Step 3: Run the Migration Script
Run the script and provide your repository folder name as an argument:

bash
Copy code
./git_lfs_migration.sh project
The script will automatically:

Navigate into the specified folder (cd project)

Scan all branches for files larger than 100 MB

Track those files with Git LFS

Rewrite Git history to store them efficiently

Clean up temporary files

🪄 Step 4: Push to GitHub
Now that the migration and cleanup are complete, it’s time to push your repository to GitHub.

🔗 Update the Remote
Point your remote to the new GitHub repository:

bash
Copy code
git remote set-url origin <github-repo-url>
🚀 Push All Branches and Tags
Since the Git history was rewritten during LFS migration, you’ll need to force-push all branches and tags:

bash
Copy code
git push origin --force --all
git push origin --force --tags
⚠️ Handling the 2 GB Push Limit
If you encounter an error like:

java
Copy code
remote: fatal: pack exceeds maximum allowed size (2 GB)
You can push your commits in smaller chunks using the following command:

bash
Copy code
git rev-list --reverse HEAD | perl -ne "print unless \$i++ % <chunk-size>;" | xargs -I{} git push origin {}:refs/heads/<branch-name>
Example:
For the main branch with chunks of 1000 commits:

bash
Copy code
git rev-list --reverse HEAD | perl -ne "print unless \$i++ % 1000;" | xargs -I{} git push origin {}:refs/heads/main
💡 Explanation:

git rev-list --reverse HEAD → Lists all commits in chronological order.

perl -ne "print unless \$i++ % 1000;" → Prints every 1000th commit hash (you can change 1000 to adjust the batch size).

xargs -I{} git push origin {}:refs/heads/main → Pushes each batch of commits incrementally to GitHub.

If the push still fails with the 2 GB limit, try reducing the chunk size (e.g., 500 or 200).
If it succeeds easily, you can increase it (e.g., 1500 or 2000) to speed up the push.

✅ Step 5: Post-Migration Checks
Verify large files are replaced by LFS pointers:

bash
Copy code
git lfs ls-files
Ensure all branches and tags are visible on GitHub.
