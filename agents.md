# Git Automation Scripts Specification

## Overview
This document specifies four Git automation scripts for managing repositories across two GitHub accounts: `fortegb` and `akamlibehsafe`. Both accounts have Personal Access Tokens (PATs) stored as local bash variables: `GH_TOKEN_akamlibehsafe` and `GH_TOKEN_fortegb`.

## Installation and Setup

### Repository Structure
- Scripts are kept in the repository folder (version-controlled)
- Symlinks are created in `~/bin/` pointing to the repository scripts
- This allows scripts to be updated via `git pull` while remaining accessible system-wide

### Setup Process

**On a Fresh Mac:**

1. **Clone the repository:**
   ```bash
   cd ~/Documents  # or your preferred location
   git clone <repository-url> gitscripts
   cd gitscripts
   ```

2. **Run the installation script:**
   ```bash
   ./scripts/git_install
   ```
   
   This will:
   - Install Git, Git LFS, GitHub CLI, and dependencies
   - Provide PAT configuration instructions
   - **Automatically offer to set up symlinks** at the end (if run from repository directory)
   
   **Accept the symlink setup prompt** to complete installation in one step.

3. **If you skipped symlink setup, run manually:**
   ```bash
   ./setup_symlinks.sh
   ```
   
   This script will:
   - Create `~/bin/` directory if it doesn't exist
   - Create symlinks for all scripts in `~/bin/`
   - Add `~/bin/` to PATH (if not already present)
   - Verify all symlinks are working

4. **Ensure `~/bin/` is in your PATH:**
   - The setup script attempts to add this automatically
   - If needed, manually add to `~/.bashrc`, `~/.bash_profile`, or `~/.zshrc`:
     ```bash
     export PATH="$HOME/bin:$PATH"
     ```

5. **Reload your shell configuration:**
   ```bash
   source ~/.bashrc  # or ~/.zshrc
   ```

### Updating Scripts
After pulling updates from the repository:
```bash
cd /path/to/gitscripts
git pull
```
Symlinks automatically point to the updated scripts - no re-setup needed!

### Script Location
- **Repository location:** `/path/to/gitscripts/` (where scripts are stored)
- **Symlink location:** `~/bin/` (where scripts are accessed from)
- **Usage:** Scripts are called by name (e.g., `git_create_from_local`) assuming `~/bin/` is in PATH

## Authentication
- **Account 1**: `fortegb` → PAT variable: `GH_TOKEN_fortegb`
- **Account 2**: `akamlibehsafe` → PAT variable: `GH_TOKEN_akamlibehsafe`
- Scripts must automatically select the appropriate PAT based on the user specified in the `user/repo` format.

---

## Script 1: `git_create_from_local`

### Purpose
Create a new GitHub repository from a local folder, initialize it, commit existing files, and push to remote.

### Usage
```bash
git_create_from_local <user/repo>
```

### Parameters
- **Required**: `user/repo` - Format: `username/repository-name` (e.g., `fortegb/test`, `akamlibehsafe/myproject`)

### Execution Context
- Run from any folder (empty or containing files)
- Current working directory becomes the repository root

### Behavior
1. Parse the `user/repo` argument to extract:
   - Username (to determine which PAT to use)
   - Repository name
2. Retrieve the appropriate PAT from environment:
   - If user is `fortegb` → use `GH_TOKEN_fortegb`
   - If user is `akamlibehsafe` → use `GH_TOKEN_akamlibehsafe`
3. Create a new GitHub repository via GitHub API using the PAT
4. Initialize Git repository in current directory (if not already initialized)
5. Add remote origin pointing to the newly created repository
6. Stage all files in the current directory (if any exist)
7. Create initial commit (if files exist)
8. Push to remote repository (main/master branch)

### Edge Cases
- Folder is empty → create repo, initialize git, push empty repo
- Folder already has a `.git` directory → use existing git repo, add remote if missing
- Files already exist → commit and push them
- PAT variable not found → error with clear message

---

## Script 2: `git_create_from_remote`

### Purpose
Clone an existing GitHub repository to local machine and check it out for editing.

### Usage
```bash
git_create_from_remote <user/repo>
```

### Parameters
- **Required**: `user/repo` - Format: `username/repository-name` (e.g., `fortegb/test`, `akamlibehsafe/myproject`)

### Execution Context
- Can be run from any directory
- Repository will be cloned into a subdirectory matching the repository name

### Behavior
1. Parse the `user/repo` argument to extract:
   - Username (to determine which PAT to use)
   - Repository name
2. Retrieve the appropriate PAT from environment:
   - If user is `fortegb` → use `GH_TOKEN_fortegb`
   - If user is `akamlibehsafe` → use `GH_TOKEN_akamlibehsafe`
3. Clone the repository using the PAT for authentication
4. Change directory into the cloned repository
5. Repository is ready for editing (checked out on default branch)

### Edge Cases
- Repository already exists locally → error or prompt for overwrite
- PAT variable not found → error with clear message
- Repository doesn't exist on GitHub → error with clear message
- Network/authentication failure → error with clear message

---

## Script 3: `git_push`

### Purpose
Commit and push changes from a local Git repository, automatically detecting the repository owner.

### Usage
```bash
git_push [directory]
```

### Parameters
- **Optional**: `directory` - Path to git-enabled folder (defaults to current directory if not specified)

### Execution Context
- Run from a git-enabled folder, or specify folder path as argument
- Must be executed within a valid Git repository

### Behavior
1. Determine target directory:
   - If `directory` argument provided → use that path
   - Otherwise → use current working directory
2. Verify the directory is a Git repository (contains `.git` directory)
3. Identify the repository owner and name:
   - Extract from remote URL (origin remote)
   - Parse format: `https://github.com/user/repo.git` or `git@github.com:user/repo.git`
4. Determine which PAT to use based on detected username:
   - If user is `fortegb` → use `GH_TOKEN_fortegb`
   - If user is `akamlibehsafe` → use `GH_TOKEN_akamlibehsafe`
5. Stage all changes (`git add .`)
6. Create commit with a message (may need user input or default message)
7. Push changes to remote using appropriate authentication

### Edge Cases
- Directory is not a Git repository → error with clear message
- No remote configured → error with clear message
- No changes to commit → skip commit, attempt push if needed
- Remote URL doesn't match expected format → error or fallback
- PAT variable not found → error with clear message
- Uncommitted changes conflict → handle appropriately

### Open Questions
- Should commit message be:
  - Interactive prompt?
  - Command-line argument?
  - Auto-generated timestamp message?
  - Default message like "Update from local"?

---

## Script 4: `git_install`

### Purpose
Install Git, Git LFS (Large File Storage), GitHub CLI (`gh`), and all necessary dependencies on a new Mac computer, then provide instructions for configuring Personal Access Tokens.

### Usage
```bash
git_install
```

### Parameters
- None required

### Execution Context
- Run from any directory on a macOS system
- Requires administrator privileges (may prompt for sudo password)
- Designed for fresh Mac installations or systems without Git/GitHub CLI

### Behavior
1. Check if Homebrew is installed:
   - If not installed → install Homebrew
   - If installed → update Homebrew to latest version
2. Install Git via Homebrew:
   - Run `brew install git`
   - Verify installation with `git --version`
3. Install GitHub CLI (`gh`) via Homebrew:
   - Run `brew install gh`
   - Verify installation with `gh --version`
4. Install Git LFS via Homebrew:
   - Run `brew install git-lfs`
   - Verify installation with `git lfs version`
   - Initialize Git LFS globally with `git lfs install`
5. Install any additional dependencies:
   - Ensure `curl` is available (typically pre-installed on macOS)
   - Ensure `jq` is available for JSON parsing (install via `brew install jq` if needed)
6. Configure Git basic settings (if not already configured):
   - Prompt user for name and email, or skip if already configured
7. Display post-installation instructions:
   - Instructions for obtaining PATs from both GitHub accounts
   - Instructions for storing PATs as bash environment variables
   - Instructions for making variables persistent across shell sessions

### Post-Installation Instructions Output
The script should output clear instructions for the user to:

1. **Obtain Personal Access Tokens:**
   - Generate PAT for `fortegb` account from GitHub Settings → Developer settings → Personal access tokens
   - Generate PAT for `akamlibehsafe` account from GitHub Settings → Developer settings → Personal access tokens
   - Required scopes: `repo` (full control of private repositories)

2. **Store PATs as Environment Variables:**
   - Add to shell configuration file (`.bashrc`, `.bash_profile`, or `.zshrc` depending on shell):
     ```bash
     export GH_TOKEN_fortegb="your_pat_token_here"
     export GH_TOKEN_akamlibehsafe="your_pat_token_here"
     ```

3. **Activate Variables:**
   - Source the configuration file: `source ~/.bashrc` (or appropriate file)
   - Or restart terminal session
   - Verify variables are set: `echo $GH_TOKEN_fortegb` and `echo $GH_TOKEN_akamlibehsafe`

4. **Optional: Secure Storage:**
   - Consider using macOS Keychain or a secrets manager for production use
   - Ensure shell configuration files have appropriate permissions (600)

5. **Using Git LFS (Large File Storage):**
   
   Git LFS is now installed and initialized. To use it in your repositories:
   
   **Initialize LFS in a repository:**
   ```bash
   cd /path/to/your/repository
   git lfs install
   ```
   
   **Track specific file types:**
   ```bash
   # Track specific file extensions
   git lfs track "*.mp4"          # Video files
   git lfs track "*.mov"          # Video files
   git lfs track "*.zip"          # Archives
   git lfs track "*.psd"          # Photoshop files
   git lfs track "*.pdf"          # Large PDFs
   git lfs track "*.dmg"          # Disk images
   
   # Track multiple extensions at once
   git lfs track "*.{mp4,mov,zip}"
   
   # Track files in a specific directory
   git lfs track "videos/*"
   ```
   
   **Commit the .gitattributes file:**
   ```bash
   git add .gitattributes
   git commit -m "Add Git LFS tracking"
   ```
   
   **Add and commit large files normally:**
   ```bash
   git add large-file.mp4
   git commit -m "Add large file via LFS"
   git push
   ```
   
   **Verify LFS is working:**
   ```bash
   # Check what files are tracked by LFS
   git lfs ls-files
   
   # Check LFS status
   git lfs status
   ```
   
   **Example workflow for a repository:**
   ```bash
   # Navigate to your repository (e.g., akamlibehsafe/gabinetecamera)
   cd gabinetecamera
   
   # Initialize LFS in the repository
   git lfs install
   
   # Track the file types you need
   git lfs track "*.mp4" "*.mov" "*.zip"
   
   # Commit the .gitattributes file
   git add .gitattributes
   git commit -m "Configure Git LFS for large files"
   
   # Add your large files
   git add large-video.mp4
   git commit -m "Add large video file"
   git push
   ```
   
   **Important Notes:**
   - GitHub provides 1 GB of LFS storage and 1 GB/month bandwidth for free accounts
   - Files over 100 MB are automatically rejected by GitHub unless using LFS
   - If you already committed large files before setting up LFS, you'll need to migrate them (more complex process)

### Edge Cases
- Git already installed → skip installation, verify version
- Git LFS already installed → skip installation, verify version, ensure it's initialized
- GitHub CLI already installed → skip installation, verify version
- Homebrew installation fails → provide manual installation instructions
- User cancels sudo prompt → exit gracefully with instructions
- Shell type detection (bash vs zsh) → use appropriate configuration file
- Configuration file doesn't exist → create it
- PAT variables already exist → warn user, provide option to update

### Dependencies Installed
- **Git**: Version control system
- **Git LFS**: Large File Storage extension for Git
- **GitHub CLI (`gh`)**: Command-line interface for GitHub
- **Homebrew**: Package manager for macOS (if not already installed)
- **jq**: JSON processor (optional, for API response parsing)
- **curl**: HTTP client (typically pre-installed)

---

## Implementation Notes

### Common Requirements
- All scripts should handle errors gracefully with clear error messages
- PAT variables must be available in the shell environment
- Scripts should validate inputs before proceeding
- GitHub API calls should use appropriate endpoints (REST API v3 or GraphQL)
- Authentication should use PAT in HTTPS URLs or as header in API calls

### Technology Considerations
- Scripts can be implemented in Bash, Python, or another suitable language
- GitHub API interaction may require `curl` or a library like `requests` (Python)
- Git operations can use standard `git` command-line tool

---

## Testing Scenarios

### Script 1 (`git_create_from_local`)
- Create repo from empty folder
- Create repo from folder with files
- Create repo from folder that's already a git repo
- Test with both user accounts

### Script 2 (`git_create_from_remote`)
- Clone existing public repo
- Clone existing private repo (with PAT)
- Handle case where repo already exists locally
- Test with both user accounts

### Script 3 (`git_push`)
- Push from folder with uncommitted changes
- Push from folder with no changes
- Push from folder with remote pointing to different account
- Test with both user accounts

### Script 4 (`git_install`)
- Fresh Mac installation (no Git, no Git LFS, no Homebrew)
- Mac with Homebrew but no Git/Git LFS/gh
- Mac with Git/Git LFS/gh already installed
- Test on both bash and zsh shells
- Verify Git LFS is properly initialized after installation
- Verify PAT setup instructions are clear and complete

