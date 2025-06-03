Documentation


Problems faced in the project:

🚫 How to Remove Large Terraform Files from Git History and Prevent Future Commits
1. Identify the Problem File

If you see an error like:

text
remote: error: File .gitignore/.terraform/providers/registry.terraform.io/hashicorp/aws/5.98.0/linux_amd64/terraform-provider-aws_v5.98.0_x5 is 669.88 MB; this exceeds GitHub's file size limit of 100.00 MB

You have a large file in your Git history that must be removed.
2. Add Terraform Patterns to .gitignore

Edit (or create) a .gitignore file in your repo root and add:

text
.terraform/
*.tfstate
*.tfstate.*
*.tfvars
*.tfvars.json

This prevents Terraform state, provider binaries, and sensitive variable files from being tracked in the future.
3. Remove the File from Git History
Option A: Using git filter-branch (works on most systems)

Run this from your repository root, matching the exact path of the problem file or directory:

bash
git filter-branch -f --index-filter 'git rm -rf --cached --ignore-unmatch .gitignore/.terraform' --prune-empty --tag-name-filter cat -- --all

If you want to be thorough, remove both .terraform and .gitignore/.terraform:

bash
git filter-branch -f --index-filter 'git rm -rf --cached --ignore-unmatch .terraform .gitignore/.terraform' --prune-empty --tag-name-filter cat -- --all

Option B: Using git filter-repo (recommended, if available)

If you have git-filter-repo:

bash
git filter-repo --path .terraform/ --path .gitignore/.terraform/ --invert-paths

4. Commit Any .gitignore Changes

If you updated your .gitignore, commit it:

bash
git add .gitignore
git commit -m "Add .terraform/ and related patterns to .gitignore"

5. Force Push the Cleaned History

⚠️ Warning: This will overwrite the remote repository history. All collaborators must re-clone the repository after this.

bash
git push --force --all
git push --force --tags

6. Verify Removal

Check that the large file is gone from history:

bash
git log -- .gitignore/.terraform
git log -- .terraform

These should return nothing.
7. Tell Collaborators to Re-Clone

Anyone else working on the repository must delete their old clone and re-clone it to avoid re-introducing the large files.
Summary Table
Step	Command/Action
Add to .gitignore	Add .terraform/ and related patterns
Remove from history (filter-branch)	See command above
Remove from history (filter-repo)	See command above
Commit .gitignore	git add .gitignore && git commit -m "..."
Force push	git push --force --all && git push --force --tags
Verify	git log -- .terraform
Re-clone for all collaborators	Everyone must delete and re-clone the repository
Best Practices

    Never commit .terraform/, state files, or secrets.

    Always keep .terraform/ and related patterns in your .gitignore.

    If you ever accidentally commit a large or sensitive file, use these steps to clean your history.

