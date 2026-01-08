# Test Scenarios for Git Automation Scripts

This document provides detailed test scenarios for all four Git automation scripts. Follow these scenarios systematically to ensure all scripts work correctly.

## Prerequisites

Before running tests, ensure:
- PAT variables are set: `GH_TOKEN_fortegb` and `GH_TOKEN_akamlibehsafe`
- You have access to both GitHub accounts: `fortegb` and `akamlibehsafe`
- You're running on macOS (for `git_install` tests)
- All scripts are executable: `chmod +x git_*`

---

## Script 1: `git_create_from_local`

### Test 1.1: Create repository from empty folder (fortegb)
**Purpose**: Verify script creates repo from an empty directory

**Steps**:
1. Create a temporary test directory:
   ```bash
   mkdir -p /tmp/test_create_empty_fortegb
   cd /tmp/test_create_empty_fortegb
   ```
2. Verify directory is empty: `ls -la` (should show only `.` and `..`)
3. Run script:
   ```bash
   ~/bin/git_create_from_local fortegb/test-empty-repo
   ```
4. **Expected results**:
   - ✅ Script creates repository on GitHub
   - ✅ Git is initialized locally
   - ✅ Remote origin is configured
   - ✅ Empty repository is pushed (or no push error for empty repo)
   - ✅ Repository exists at `https://github.com/fortegb/test-empty-repo`
5. **Verify on GitHub**: Check that repository exists
6. **Cleanup**: Delete repository on GitHub if you want to rerun this test

**Success criteria**: All expected results are met, no errors

---

### Test 1.2: Create repository from folder with files (fortegb)
**Purpose**: Verify script commits and pushes existing files

**Steps**:
1. Create a temporary test directory:
   ```bash
   mkdir -p /tmp/test_create_with_files_fortegb
   cd /tmp/test_create_with_files_fortegb
   ```
2. Create some test files:
   ```bash
   echo "Hello World" > README.md
   echo "Test content" > test.txt
   mkdir docs
   echo "Documentation" > docs/notes.md
   ```
3. Run script:
   ```bash
   ~/bin/git_create_from_local fortegb/test-with-files
   ```
4. **Expected results**:
   - ✅ Repository created on GitHub
   - ✅ Git initialized
   - ✅ All files are staged and committed
   - ✅ Files are pushed to remote
   - ✅ Repository contains all files on GitHub
5. **Verify**: Check GitHub repository contains README.md, test.txt, and docs/notes.md
6. **Verify locally**: 
   ```bash
   git log --oneline
   git ls-files
   ```

**Success criteria**: All files are committed and visible on GitHub

---

### Test 1.3: Create repository from folder that's already a git repo (akamlibehsafe)
**Purpose**: Verify script handles pre-initialized git repositories

**Steps**:
1. Create and initialize git repo manually:
   ```bash
   mkdir -p /tmp/test_create_existing_git
   cd /tmp/test_create_existing_git
   git init
   echo "Pre-existing file" > existing.txt
   git add existing.txt
   git commit -m "Pre-existing commit"
   ```
2. Add another file after initial commit:
   ```bash
   echo "New file" > new.txt
   ```
3. Run script:
   ```bash
   ~/bin/git_create_from_local akamlibehsafe/test-existing-git
   ```
4. **Expected results**:
   - ✅ Script detects existing git repo (doesn't reinitialize)
   - ✅ Remote origin is added/updated
   - ✅ New files are staged and committed
   - ✅ All commits (including pre-existing) are pushed
5. **Verify**: Check git log shows both commits on GitHub

**Success criteria**: Existing git repo is preserved and updated correctly

---

### Test 1.4: Test with akamlibehsafe account
**Purpose**: Verify PAT selection works for second account

**Steps**:
1. Repeat Test 1.2 but use `akamlibehsafe` account:
   ```bash
   mkdir -p /tmp/test_create_akamlibeh
   cd /tmp/test_create_akamlibeh
   echo "Test for akamlibehsafe" > README.md
   ~/bin/git_create_from_local akamlibehsafe/test-akamlibeh
   ```
2. **Expected results**:
   - ✅ Script uses `GH_TOKEN_akamlibehsafe` PAT
   - ✅ Repository is created under `akamlibehsafe` account
   - ✅ Repository URL is correct: `https://github.com/akamlibehsafe/test-akamlibeh`

**Success criteria**: Correct account and PAT are used

---

### Test 1.5: Error handling - Invalid format
**Purpose**: Verify script handles invalid input

**Steps**:
1. Run script with invalid format:
   ```bash
   ~/bin/git_create_from_local invalid-format
   ```
2. **Expected results**:
   - ✅ Script shows error message about invalid format
   - ✅ Script exits with error code
   - ✅ No repository is created

**Success criteria**: Clear error message, no partial creation

---

### Test 1.6: Error handling - Missing PAT
**Purpose**: Verify script handles missing PAT variable

**Steps**:
1. Temporarily unset PAT:
   ```bash
   unset GH_TOKEN_fortegb
   ```
2. Run script:
   ```bash
   ~/bin/git_create_from_local fortegb/test-missing-pat
   ```
3. **Expected results**:
   - ✅ Script shows clear error about missing PAT
   - ✅ Instructions provided on how to set PAT
4. **Cleanup**: Restore PAT variable

**Success criteria**: Clear error message with helpful instructions

---

## Script 2: `git_create_from_remote`

### Test 2.1: Clone existing public repository (fortegb)
**Purpose**: Verify script can clone public repositories

**Steps**:
1. Create a test repository on GitHub first (public):
   - Go to GitHub and create `fortegb/test-clone-public`
   - Add a README.md file
2. Navigate to a clean directory:
   ```bash
   cd /tmp
   ```
3. Run script:
   ```bash
   ~/bin/git_create_from_remote fortegb/test-clone-public
   ```
4. **Expected results**:
   - ✅ Repository is cloned to `test-clone-public` directory
   - ✅ Script changes into cloned directory
   - ✅ Remote origin is correctly configured
   - ✅ Files from GitHub are present locally
5. **Verify**:
   ```bash
   cd test-clone-public
   git remote -v
   ls -la
   ```

**Success criteria**: Repository cloned successfully with all files

---

### Test 2.2: Clone existing private repository (akamlibehsafe)
**Purpose**: Verify PAT authentication works for private repos

**Steps**:
1. Create a private test repository on GitHub:
   - Create `akamlibehsafe/test-clone-private` (set as private)
   - Add some files
2. Navigate to clean directory:
   ```bash
   cd /tmp
   ```
3. Run script:
   ```bash
   ~/bin/git_create_from_remote akamlibehsafe/test-clone-private
   ```
4. **Expected results**:
   - ✅ Private repository is cloned successfully
   - ✅ Correct PAT is used for authentication
   - ✅ All files are present

**Success criteria**: Private repo cloned with PAT authentication

---

### Test 2.3: Error handling - Repository already exists locally
**Purpose**: Verify script handles existing directory

**Steps**:
1. First, clone a repository:
   ```bash
   cd /tmp
   ~/bin/git_create_from_remote fortegb/test-clone-public
   cd ..
   ```
2. Try to clone the same repository again:
   ```bash
   ~/bin/git_create_from_remote fortegb/test-clone-public
   ```
3. **Expected results**:
   - ✅ Script detects existing directory
   - ✅ Shows error message about directory already existing
   - ✅ Does not attempt to overwrite
   - ✅ Exits with error code

**Success criteria**: Clear error, no overwrite attempt

---

### Test 2.4: Error handling - Non-existent repository
**Purpose**: Verify script handles missing repositories

**Steps**:
1. Try to clone a non-existent repository:
   ```bash
   cd /tmp
   ~/bin/git_create_from_remote fortegb/this-repo-does-not-exist-12345
   ```
2. **Expected results**:
   - ✅ Script shows clear error message
   - ✅ Error mentions repository not found
   - ✅ Provides helpful troubleshooting tips
   - ✅ Exits with error code

**Success criteria**: Clear, actionable error message

---

### Test 2.5: Clone repository with subdirectories
**Purpose**: Verify complex repository structure is cloned correctly

**Steps**:
1. Create a repository with nested structure on GitHub
2. Clone it:
   ```bash
   cd /tmp
   ~/bin/git_create_from_remote akamlibehsafe/test-nested-structure
   ```
3. **Expected results**:
   - ✅ All subdirectories are cloned
   - ✅ File structure is preserved
   - ✅ Git history is intact
4. **Verify**:
   ```bash
   cd test-nested-structure
   find . -type f
   git log --oneline
   ```

**Success criteria**: Complete repository structure cloned

---

## Script 3: `git_push`

### Test 3.1: Push with uncommitted changes (fortegb)
**Purpose**: Verify script stages, commits, and pushes new changes

**Steps**:
1. Clone or create a test repository:
   ```bash
   cd /tmp
   ~/bin/git_create_from_remote fortegb/test-push-existing
   # OR create a new one first
   cd test-push-existing
   ```
2. Make changes:
   ```bash
   echo "New content" >> README.md
   echo "Test file" > test.txt
   ```
3. Run script:
   ```bash
   ~/bin/git_push
   ```
   When prompted, enter commit message: "Test commit from git_push"
4. **Expected results**:
   - ✅ Script detects changes
   - ✅ All changes are staged
   - ✅ Commit is created with your message
   - ✅ Changes are pushed to remote
   - ✅ Changes appear on GitHub
5. **Verify on GitHub**: Check repository shows new commit and files

**Success criteria**: All changes committed and pushed

---

### Test 3.2: Push with commit message argument (akamlibehsafe)
**Purpose**: Verify -m flag works for commit messages

**Steps**:
1. Navigate to a repository with changes:
   ```bash
   cd /tmp/test-push-existing  # or another repo
   echo "Another change" >> README.md
   ```
2. Run script with message flag:
   ```bash
   ~/bin/git_push -m "Automated commit message"
   ```
3. **Expected results**:
   - ✅ Script uses provided commit message
   - ✅ No prompt for commit message
   - ✅ Commit and push succeed

**Success criteria**: Commit message from argument is used

---

### Test 3.3: Push from different directory
**Purpose**: Verify directory argument works

**Steps**:
1. Create changes in a repository:
   ```bash
   cd /tmp/test-push-existing
   echo "Change from different dir" > change.txt
   ```
2. Run script from different location:
   ```bash
   cd /tmp
   ~/bin/git_push test-push-existing -m "Change from different directory"
   ```
3. **Expected results**:
   - ✅ Script changes to specified directory
   - ✅ Changes in that directory are committed
   - ✅ Push succeeds

**Success criteria**: Directory argument works correctly

---

### Test 3.4: Push with no changes
**Purpose**: Verify script handles repository with no changes

**Steps**:
1. Navigate to a clean repository (no uncommitted changes):
   ```bash
   cd /tmp/test-push-existing
   git status  # Should show "nothing to commit, working tree clean"
   ```
2. Run script:
   ```bash
   ~/bin/git_push
   ```
3. **Expected results**:
   - ✅ Script detects no changes
   - ✅ Skips commit step
   - ✅ Attempts push (may succeed if remote is up to date, or show message)
   - ✅ No errors

**Success criteria**: Script handles no-changes scenario gracefully

---

### Test 3.5: Error handling - Not a git repository
**Purpose**: Verify error when run outside git repo

**Steps**:
1. Navigate to a non-git directory:
   ```bash
   cd /tmp
   mkdir -p not-a-git-repo
   cd not-a-git-repo
   ```
2. Run script:
   ```bash
   ~/bin/git_push
   ```
3. **Expected results**:
   - ✅ Clear error message about not being a git repository
   - ✅ Suggests running from git repo or providing directory
   - ✅ Exits with error code

**Success criteria**: Clear error message

---

### Test 3.6: Error handling - No remote configured
**Purpose**: Verify error when remote is missing

**Steps**:
1. Create a local git repo without remote:
   ```bash
   cd /tmp
   mkdir test-no-remote
   cd test-no-remote
   git init
   echo "test" > file.txt
   git add file.txt
   git commit -m "Initial"
   ```
2. Remove remote (if exists):
   ```bash
   git remote remove origin 2>/dev/null || true
   ```
3. Run script:
   ```bash
   ~/bin/git_push
   ```
4. **Expected results**:
   - ✅ Error message about missing remote
   - ✅ Instructions on how to add remote
   - ✅ Exits with error code

**Success criteria**: Clear error with helpful instructions

---

### Test 3.7: Push to remote with different account detection
**Purpose**: Verify username detection from remote URL

**Steps**:
1. Create a repository under one account:
   ```bash
   cd /tmp
   ~/bin/git_create_from_remote fortegb/test-account-detection
   cd test-account-detection
   ```
2. Verify remote URL:
   ```bash
   git remote get-url origin
   ```
3. Make a change and push:
   ```bash
   echo "Test" > test.txt
   ~/bin/git_push -m "Test account detection"
   ```
4. **Expected results**:
   - ✅ Script correctly detects `fortegb` from remote URL
   - ✅ Uses `GH_TOKEN_fortegb` PAT
   - ✅ Push succeeds

**Success criteria**: Correct account/PAT detected from remote

---

### Test 3.8: Push with untracked files
**Purpose**: Verify script handles untracked files

**Steps**:
1. In a repository, create untracked files:
   ```bash
   cd /tmp/test-push-existing
   echo "untracked" > untracked.txt
   mkdir -p untracked_dir
   echo "content" > untracked_dir/file.txt
   ```
2. Run script:
   ```bash
   ~/bin/git_push -m "Add untracked files"
   ```
3. **Expected results**:
   - ✅ Script detects untracked files
   - ✅ All untracked files are staged
   - ✅ Files are committed and pushed

**Success criteria**: Untracked files are handled correctly

---

## Script 4: `git_install`

### Test 4.1: Fresh installation (simulated - use VM or test machine)
**Purpose**: Test complete installation on clean macOS system

**Steps**:
1. **Setup**: Use a clean macOS system or VM (or temporarily rename tools)
2. Verify tools are not installed:
   ```bash
   which git
   which gh
   which git-lfs
   which brew
   ```
3. Run script:
   ```bash
   ~/bin/git_install
   ```
4. **Expected results**:
   - ✅ Homebrew is installed
   - ✅ Git is installed via Homebrew
   - ✅ GitHub CLI is installed
   - ✅ Git LFS is installed and initialized
   - ✅ curl and jq are available
   - ✅ Post-installation instructions are displayed
5. **Verify installations**:
   ```bash
   git --version
   gh --version
   git lfs version
   brew --version
   ```

**Success criteria**: All tools installed and working

---

### Test 4.2: Installation with Homebrew already present
**Purpose**: Verify script works when Homebrew exists

**Steps**:
1. Verify Homebrew is installed:
   ```bash
   brew --version
   ```
2. If Git/Git LFS/gh are installed, temporarily note versions
3. Run script:
   ```bash
   ~/bin/git_install
   ```
4. **Expected results**:
   - ✅ Script detects Homebrew
   - ✅ Updates Homebrew
   - ✅ Installs missing tools
   - ✅ Upgrades existing tools if newer versions available
5. **Verify**: All tools are present and working

**Success criteria**: Existing Homebrew handled correctly

---

### Test 4.3: Installation with tools already present
**Purpose**: Verify script handles already-installed tools

**Steps**:
1. Ensure Git, Git LFS, and GitHub CLI are installed
2. Run script:
   ```bash
   ~/bin/git_install
   ```
3. **Expected results**:
   - ✅ Script detects existing installations
   - ✅ Shows current versions
   - ✅ Attempts to upgrade (or skips if latest)
   - ✅ Git LFS is verified as initialized
   - ✅ No duplicate installations
4. **Verify**: Tools still work, versions may be updated

**Success criteria**: Existing tools detected and maintained/upgraded

---

### Test 4.4: Git configuration prompt
**Purpose**: Verify Git user configuration works

**Steps**:
1. Temporarily remove Git config (backup first):
   ```bash
   git config --global --unset user.name
   git config --global --unset user.email
   ```
2. Run script:
   ```bash
   ~/bin/git_install
   ```
3. When prompted:
   - Enter a test name: "Test User"
   - Enter a test email: "test@example.com"
4. **Expected results**:
   - ✅ Script prompts for name and email
   - ✅ Configurations are saved
   - ✅ Git config shows correct values
5. **Verify**:
   ```bash
   git config --global user.name
   git config --global user.email
   ```
6. **Cleanup**: Restore your original Git config

**Success criteria**: Git configuration saved correctly

---

### Test 4.5: Git LFS initialization verification
**Purpose**: Verify Git LFS is properly initialized

**Steps**:
1. Run script (if Git LFS not installed) or after installation
2. **Verify Git LFS initialization**:
   ```bash
   git lfs env
   ```
   Should show that LFS is installed and initialized
3. **Test LFS in a repository**:
   ```bash
   mkdir /tmp/test-lfs
   cd /tmp/test-lfs
   git init
   git lfs install
   git lfs track "*.mp4"
   ```
4. **Expected results**:
   - ✅ Git LFS global installation is active
   - ✅ Can initialize LFS in repositories
   - ✅ Can track file types

**Success criteria**: Git LFS fully functional

---

### Test 4.6: Shell configuration detection (bash)
**Purpose**: Verify correct config file detection for bash

**Steps**:
1. Check current shell:
   ```bash
   echo $SHELL
   ```
2. If bash, run script:
   ```bash
   ~/bin/git_install
   ```
3. **Verify**:
   - ✅ Script detects bash shell
   - ✅ Mentions `.bashrc` or `.bash_profile` in instructions
   - ✅ Instructions are correct for bash

**Success criteria**: Correct shell config file identified

---

### Test 4.7: Shell configuration detection (zsh)
**Purpose**: Verify correct config file detection for zsh

**Steps**:
1. Switch to zsh (if available):
   ```bash
   zsh
   ```
2. Run script:
   ```bash
   ~/bin/git_install
   ```
3. **Verify**:
   - ✅ Script detects zsh shell
   - ✅ Mentions `.zshrc` in instructions
   - ✅ Instructions are correct for zsh

**Success criteria**: Correct shell config file identified

---

### Test 4.8: Post-installation instructions display
**Purpose**: Verify instructions are complete and accurate

**Steps**:
1. Run script and review output
2. **Check instructions include**:
   - ✅ How to obtain PATs from GitHub
   - ✅ Required scopes for PATs
   - ✅ Example export commands with correct variable names
   - ✅ Correct shell config file path
   - ✅ How to activate variables
   - ✅ How to verify setup
   - ✅ Security best practices
   - ✅ Git LFS usage guide
3. **Verify accuracy**:
   - Variable names match: `GH_TOKEN_fortegb`, `GH_TOKEN_akamlibehsafe`
   - File paths are correct
   - Instructions are actionable

**Success criteria**: Complete, accurate instructions provided

---

### Test 4.9: Error handling - Non-macOS system (if possible)
**Purpose**: Verify OS check works

**Steps**:
1. If you have access to Linux/Windows, try running script (or modify `OSTYPE` check)
2. **Expected results**:
   - ✅ Script detects non-macOS system
   - ✅ Shows error message
   - ✅ Exits gracefully

**Success criteria**: OS validation works

---

## Integration Tests

### Integration Test 1: Complete workflow
**Purpose**: Test complete workflow using all scripts together

**Steps**:
1. **Install tools** (if needed):
   ```bash
   ~/bin/git_install
   ```
2. **Create repository from local**:
   ```bash
   mkdir /tmp/integration-test
   cd /tmp/integration-test
   echo "Integration test" > README.md
   ~/bin/git_create_from_local fortegb/integration-test
   ```
3. **Clone repository**:
   ```bash
   cd /tmp
   ~/bin/git_create_from_remote fortegb/integration-test
   cd integration-test
   ```
4. **Make changes and push**:
   ```bash
   echo "Update" >> README.md
   ~/bin/git_push -m "Integration test update"
   ```
5. **Verify on GitHub**: All steps completed successfully

**Success criteria**: Complete workflow works end-to-end

---

### Integration Test 2: Both accounts
**Purpose**: Verify scripts work with both GitHub accounts

**Steps**:
1. Create repo under `fortegb`:
   ```bash
   mkdir /tmp/test-fortegb
   cd /tmp/test-fortegb
   echo "Fortegb repo" > README.md
   ~/bin/git_create_from_local fortegb/test-account1
   ```
2. Create repo under `akamlibehsafe`:
   ```bash
   cd /tmp
   mkdir test-akamlibeh
   cd test-akamlibeh
   echo "Akamlibeh repo" > README.md
   ~/bin/git_create_from_local akamlibehsafe/test-account2
   ```
3. **Verify**: Both repositories exist under correct accounts

**Success criteria**: Both accounts work correctly

---

## Test Checklist

Use this checklist to track completed tests:

### Script 1: git_create_from_local
- [ ] Test 1.1: Empty folder (fortegb)
- [ ] Test 1.2: Folder with files (fortegb)
- [ ] Test 1.3: Existing git repo (akamlibehsafe)
- [ ] Test 1.4: akamlibehsafe account
- [ ] Test 1.5: Invalid format error
- [ ] Test 1.6: Missing PAT error

### Script 2: git_create_from_remote
- [ ] Test 2.1: Clone public repo (fortegb)
- [ ] Test 2.2: Clone private repo (akamlibehsafe)
- [x] Test 2.3: Existing directory error
- [x] Test 2.4: Non-existent repository error
- [x] Test 2.5: Nested structure clone

### Script 3: git_push
- [x] Test 3.1: Push with uncommitted changes
- [x] Test 3.2: Push with -m flag
- [x] Test 3.3: Push from different directory
- [x] Test 3.4: Push with no changes
- [x] Test 3.5: Not a git repo error
- [x] Test 3.6: No remote error
- [x] Test 3.7: Account detection
- [x] Test 3.8: Untracked files

### Script 4: git_install
- [x] Test 4.1: Fresh installation
- [x] Test 4.2: Homebrew present
- [x] Test 4.3: Tools already installed
- [x] Test 4.4: Git configuration
- [x] Test 4.5: Git LFS verification
- [x] Test 4.6: Bash shell detection
- [~] Test 4.7: Zsh shell detection (N/A - not using zsh)
- [x] Test 4.8: Instructions display
- [~] Test 4.9: OS validation (N/A - running on macOS)

### Integration Tests
- [x] Integration Test 1: Complete workflow
- [x] Integration Test 2: Both accounts

---

## Notes

- **Test Data**: Use descriptive repository names like `test-<scenario>-<timestamp>` to avoid conflicts
- **Cleanup**: Consider cleaning up test repositories after testing, or use a dedicated test account
- **PAT Scopes**: Ensure PATs have `repo` scope for all operations
- **Rate Limits**: Be aware of GitHub API rate limits when running multiple tests
- **Network**: Some tests require internet connection and GitHub access

---

## Troubleshooting

If tests fail:
1. **Verify PAT tokens are correctly configured:**
   ```bash
   ./verify_pat.sh
   ```
   This will check both PAT tokens and verify they have the correct permissions.

2. **Check PAT variables are set:**
   ```bash
   echo $GH_TOKEN_fortegb
   echo $GH_TOKEN_akamlibehsafe
   ```
   If empty, set them in your shell config file and reload.

3. **Common 403 Error - "Resource not accessible by personal access token":**
   This error means your PAT doesn't have the required `repo` scope.
   
   **To fix:**
   - Go to https://github.com/settings/tokens
   - Find your token for the account (or create a new one)
   - Make sure the `repo` scope is checked (Full control of private repositories)
   - If creating a new token, copy it and update your environment variable:
     ```bash
     export GH_TOKEN_fortegb="ghp_your_new_token_here"  # Replace with your actual token
     ```
   - Save to your shell config file to make it permanent:
     ```bash
     echo 'export GH_TOKEN_fortegb="ghp_your_new_token_here"' >> ~/.zshrc  # or ~/.bashrc
     source ~/.zshrc  # Reload your shell config
     ```

4. **Common 401 Error - Authentication failed:**
   - Your PAT token may be expired
   - Regenerate the token at https://github.com/settings/tokens
   - Update your environment variable with the new token

5. **Verify PAT belongs to correct account:**
   - The PAT for `GH_TOKEN_fortegb` must be from the `fortegb` account
   - The PAT for `GH_TOKEN_akamlibehsafe` must be from the `akamlibehsafe` account
   - Use `./verify_pat.sh` to check which account each PAT belongs to

6. **Check network connectivity:**
   ```bash
   curl -I https://api.github.com
   ```

7. **Verify scripts are executable:**
   ```bash
   ls -l git_*
   chmod +x git_* verify_pat.sh  # If needed
   ```

8. **Check script paths are correct:**
   - Use absolute paths or ensure scripts are in your PATH
   - Or run with: `./git_create_from_local` (from the script directory)

9. **Git safe.directory warning (especially for /tmp directories):**
   - If you see "fatal: detected dubious ownership in repository" error, this is handled automatically
   - The scripts automatically add temporary directories (`/tmp`, `/private/tmp`, `/var/tmp`) to Git's safe.directory list
   - This is a Git security feature to prevent working with repositories owned by other users
   - If you encounter this in a non-temporary directory, you can manually add it:
     ```bash
     git config --global --add safe.directory ~/bin/directory
     ```

10. **Review error messages for specific issues:**
    - The scripts now provide detailed error messages for common issues
    - 403 = Missing permissions (need `repo` scope)
    - 401 = Invalid/expired token
    - 404 = Repository not found (for clone operations)
    - "dubious ownership" = Directory ownership/permission issue (automatically handled for /tmp)

