# QUICK INFO

If dealing with a new installation, download the git_install script by itself and execute it. It will install and configure git on your machine. 

At the end it will also clone the gitscripts repo with the git scripts created to make it easier to work with git.




# Git Automation Scripts

A collection of bash scripts for automating Git and GitHub operations across multiple accounts.

## Overview

These scripts provide a streamlined workflow for managing GitHub repositories across two accounts: `fortegb` and `akamlibehsafe`. They automatically handle authentication, repository creation, cloning, and pushing operations.

**Note:** These scripts are designed to be kept in the repository folder and accessed via symlinks in `~/bin/`. This allows you to keep the scripts version-controlled while using them from anywhere. See the Setup section for installation instructions.

## Scripts

### 1. `git_install`
Installs Git, Git LFS, GitHub CLI, and all necessary dependencies on macOS.

**Usage:**
```bash
./scripts/git_install
```

**What it does:**
- Installs/updates Homebrew
- Installs Git, GitHub CLI (`gh`), and Git LFS
- Installs additional dependencies (curl, jq)
- Configures Git user settings
- Provides post-installation instructions for PAT setup

### 2. `git_create_from_local`
Creates a new GitHub repository from a local folder.

**Usage:**
```bash
./scripts/git_create_from_local <user/repo>
```

**Example:**
```bash
./scripts/git_create_from_local fortegb/my-new-project
./scripts/git_create_from_local akamlibehsafe/test-repo
```

**What it does:**
- Creates a new repository on GitHub
- Initializes Git in the current directory (if needed)
- Stages and commits existing files
- Pushes to the remote repository

### 3. `git_create_from_remote`
Clones an existing GitHub repository to your local machine.

**Usage:**
```bash
./scripts/git_create_from_remote <user/repo>
```

**Example:**
```bash
./scripts/git_create_from_remote fortegb/existing-repo
./scripts/git_create_from_remote akamlibehsafe/my-project
```

**What it does:**
- Clones the repository using PAT authentication
- Changes into the cloned directory
- Handles Git safe.directory configuration for temporary directories

### 4. `git_push`
Commits and pushes changes from a local Git repository.

**Usage:**
```bash
./scripts/git_push [directory] [-m "commit message"]
```

**Examples:**
```bash
./scripts/git_push                              # Push from current directory
./scripts/git_push /path/to/repo                 # Push from specified directory
./scripts/git_push -m "Update documentation"     # Push with commit message
./scripts/git_push /path/to/repo -m "Fix bug"   # Push from directory with message
```

**What it does:**
- Automatically detects the repository owner from remote URL
- Stages all changes (including untracked files)
- Creates a commit (prompts for message if not provided)
- Pushes to remote using appropriate PAT

### 5. `verify_pat.sh`
Helper script to verify PAT tokens are configured correctly.

**Usage:**
```bash
./scripts/verify_pat.sh
```

**What it does:**
- Checks if PAT variables are set
- Validates tokens with GitHub API
- Verifies tokens belong to correct accounts
- Checks token permissions

### 6. `shell_prompt_setup`
Installs and configures a beautiful, Git-aware shell prompt with Powerlevel10k theme, iTerm2, and required fonts.

**Usage:**
```bash
./scripts/shell_prompt_setup
```

**What it does:**
- Installs/verifies iTerm2 terminal emulator (via Homebrew)
- Configures Zsh as default shell (optional)
- Installs Oh My Zsh framework
- Installs Powerlevel10k theme (Git-aware prompt with icons)
- Installs MesloLGS NF font (Nerd Font with icons)
- Installs zsh plugins:
  - `zsh-autosuggestions` - Command autosuggestions
  - `zsh-syntax-highlighting` - Syntax highlighting
- Configures `.zshrc` with all components
- Provides post-installation instructions

**Note:** This script creates a professional development environment with a visually appealing prompt that shows Git branch, status, and other useful information. Perfect for developers who want an enhanced terminal experience.

## Prerequisites

1. **macOS** (required for `git_install` and `shell_prompt_setup` scripts)
2. **Personal Access Tokens (PATs)** for both GitHub accounts:
   - `GH_TOKEN_fortegb` - for the `fortegb` account
   - `GH_TOKEN_akamlibehsafe` - for the `akamlibehsafe` account

## Setup

### 1. Install Dependencies

**On a fresh Mac, first clone or download this repository:**
```bash
cd ~/Documents  # or wherever you want the repository
git clone <repository-url> gitscripts
cd gitscripts
```

**Then run the installation script:**
```bash
./scripts/git_install
```

This will:
- Install Git, Git LFS, GitHub CLI, and dependencies
- Provide instructions for PAT configuration
- **Automatically offer to set up symlinks** (if run from the repository directory)

**Note:** If `git_install` detects it's in the gitscripts repository, it will automatically offer to run `setup_symlinks.sh` at the end. You can accept this to complete the setup in one go.

### 2. Configure Personal Access Tokens

1. **Generate PATs:**
   - Go to GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
   - Generate new tokens for both accounts
   - Required scope: `repo` (Full control of private repositories)

2. **Store PATs in your shell configuration:**
   
   The `git_install` script automatically detects your shell (bash or zsh) and configures the appropriate file:
   - **Bash**: Uses `.bash_profile` (or `.bashrc` if `.bash_profile` doesn't exist)
   - **Zsh**: Uses `.zshrc`
   
   If you have a `PATs.md` file in the repository, the script will automatically add the tokens to your shell config file.
   
   **Manual configuration:**
   
   For bash (`.bashrc` or `.bash_profile`):
   ```bash
   export GH_TOKEN_fortegb="your_token_here"
   export GH_TOKEN_akamlibehsafe="your_token_here"
   ```
   
   For zsh (`.zshrc`):
   ```bash
   export GH_TOKEN_fortegb="your_token_here"
   export GH_TOKEN_akamlibehsafe="your_token_here"
   ```

3. **Activate the variables:**
   ```bash
   source ~/.bashrc    # for bash
   source ~/.zshrc     # for zsh
   # Or simply close and reopen your terminal
   ```

4. **Verify setup:**
   ```bash
   ./scripts/verify_pat.sh
   ```

### 3. Set Up Symlinks to ~/bin/

**Recommended Approach:** Keep scripts in the repository folder and create symlinks in `~/bin/`.

This approach:
- Keeps scripts version-controlled in the repository
- Allows easy updates via `git pull`
- Makes scripts available system-wide via symlinks

**Automatic Setup (Recommended):**

If you ran `git_install` from the repository directory, it will automatically offer to set up symlinks at the end. Simply accept the prompt.

**Manual Setup:**

If you skipped the automatic setup or need to recreate symlinks:

```bash
cd /path/to/gitscripts
./setup_symlinks.sh
```

This script will:
- Create `~/bin/` directory if it doesn't exist
- Create symlinks in `~/bin/` pointing to scripts in the repository
- Add `~/bin/` to your PATH (if not already present)
- Verify all symlinks are working

**Alternative: Manual Setup**

If you prefer to set up symlinks manually:

```bash
# Create ~/bin if it doesn't exist
mkdir -p ~/bin

# Navigate to the repository folder
cd /path/to/gitscripts

# Create symlinks
ln -s "$(pwd)/scripts/git_install" ~/bin/git_install
ln -s "$(pwd)/scripts/git_create_from_local" ~/bin/git_create_from_local
ln -s "$(pwd)/scripts/git_create_from_remote" ~/bin/git_create_from_remote
ln -s "$(pwd)/scripts/git_push" ~/bin/git_push
ln -s "$(pwd)/scripts/verify_pat.sh" ~/bin/verify_pat.sh
ln -s "$(pwd)/scripts/shell_prompt_setup" ~/bin/shell_prompt_setup

# Ensure ~/bin is in PATH (add to ~/.bashrc, ~/.bash_profile, or ~/.zshrc)
export PATH="$HOME/bin:$PATH"
```

**Important:** 
- Scripts remain in the repository folder (version-controlled)
- Symlinks in `~/bin/` point to the repository scripts
- All usage examples assume scripts are accessible via `~/bin/` in PATH
- After updating the repository (`git pull`), symlinks automatically point to the updated scripts

## Usage Examples

**Note:** All examples assume scripts are accessible via `~/bin/` in your PATH (via symlinks).

### Create a new repository from local folder
```bash
cd /path/to/my/project
git_create_from_local fortegb/my-new-repo
```

### Clone an existing repository
```bash
git_create_from_remote akamlibehsafe/existing-repo
cd existing-repo
```

### Make changes and push
```bash
echo "New content" >> README.md
git_push -m "Update README"
```

### Set up a beautiful shell prompt
```bash
shell_prompt_setup
# Follow the instructions to reload your shell
source ~/.zshrc
# Optional: Customize the prompt
p10k configure
```

### Complete workflow
```bash
# 1. Create repository from local
mkdir my-project
cd my-project
echo "# My Project" > README.md
git_create_from_local fortegb/my-project

# 2. Clone it elsewhere
cd /tmp
git_create_from_remote fortegb/my-project
cd my-project

# 3. Make changes and push
echo "Update" >> README.md
git_push -m "Add update"
```

## Features

- **Automatic account detection** - Scripts automatically select the correct PAT based on username
- **Error handling** - Clear error messages with troubleshooting tips
- **Git LFS support** - Full support for large files via Git LFS
- **Safe directory handling** - Automatically configures Git for temporary directories
- **Token validation** - Verifies PAT tokens before use
- **Comprehensive testing** - All scripts have been tested (see `TEST_SCENARIOS.md`)

## Security Notes

- **Never commit PAT tokens** - The `PATs.md` file is excluded from version control (see `.gitignore`)
- **Secure permissions** - Ensure your shell config file has secure permissions: `chmod 600 ~/.bashrc`
- **Token scope** - Only grant the minimum required scope (`repo`) to your PATs
- **Token rotation** - Regularly rotate your PAT tokens for security

## Troubleshooting

### Common Issues

1. **"PAT variable not set" error:**
   - Ensure PATs are exported in your shell config
   - Run `source ~/.bashrc` (or `~/.zshrc`) to reload
   - Verify with: `echo $GH_TOKEN_fortegb`

2. **"403 Forbidden" error:**
   - Your PAT doesn't have the `repo` scope
   - Regenerate the token with `repo` scope enabled

3. **"401 Unauthorized" error:**
   - Your PAT token may be expired
   - Regenerate the token at https://github.com/settings/tokens

4. **"Repository not found" error:**
   - Verify the repository exists: https://github.com/user/repo
   - Check that your PAT has access to the repository

5. **"dubious ownership" error:**
   - This is automatically handled for `/tmp` directories
   - For other directories, add manually: `git config --global --add safe.directory /path/to/dir`

6. **"p10k configure: command not found" error:**
   - You need to reload your shell configuration first: `source ~/.zshrc`
   - Or open a new terminal window/tab
   - The `p10k` command is only available after Powerlevel10k is loaded

7. **"no such file or directory: /Users/.../.oh-my-zsh/oh-my-zsh.sh" error:**
   - Oh My Zsh installation is incomplete
   - Re-run `shell_prompt_setup` to fix the installation
   - The script will detect and fix incomplete installations

### Verification

Use the verification script to check your setup:
```bash
verify_pat.sh
```

## Testing

Comprehensive test scenarios are documented in `TEST_SCENARIOS.md`. All scripts have been tested and verified to work correctly.

## Files

### Scripts (in `scripts/` directory)
- `scripts/git_install` - Installation script for macOS
- `scripts/git_create_from_local` - Create repository from local folder
- `scripts/git_create_from_remote` - Clone existing repository
- `scripts/git_push` - Commit and push changes
- `scripts/verify_pat.sh` - PAT token verification helper
- `scripts/shell_prompt_setup` - Install and configure Powerlevel10k shell prompt

### Support Scripts
- `setup_symlinks.sh` - **Setup script to create symlinks in ~/bin/**

### Documentation
- `TEST_SCENARIOS.md` - Comprehensive test documentation
- `agents.md` - Detailed specification document
- `.gitignore` - Git ignore file (excludes PATs.md)

## Updating Scripts

Since scripts are kept in the repository folder and accessed via symlinks:

1. **Update the repository:**
   ```bash
   cd /path/to/gitscripts
   git pull
   ```

2. **Symlinks automatically point to updated scripts** - no need to re-run setup!

3. **If you need to recreate symlinks** (e.g., after moving the repository):
   ```bash
   cd /path/to/gitscripts
   ./setup_symlinks.sh
   ```

## License

This project is for personal use. Modify as needed for your requirements.

## Support

For issues or questions:
1. Check `TEST_SCENARIOS.md` for test scenarios
2. Run `./scripts/verify_pat.sh` to check PAT configuration
3. Review error messages for specific troubleshooting tips

---

**Note:** These scripts are designed for macOS. For Linux or Windows, you may need to modify the installation script (`git_install`).

