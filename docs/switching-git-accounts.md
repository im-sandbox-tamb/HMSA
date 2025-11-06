# Switching from EMU Account to Personal Account

If you're logged into your Enterprise Managed User (EMU) account on your work laptop and want to use your personal GitHub account instead to clone a repository from github.com, here are several methods to accomplish this:

## Method 1: Using Git Credential Manager (Recommended for HTTPS)

If you're using HTTPS to clone repositories, Git Credential Manager stores your credentials. To switch accounts:

### Windows
1. Open **Credential Manager** (search for it in the Start menu)
2. Go to **Windows Credentials**
3. Find entries starting with `git:https://github.com`
4. Remove/delete these credentials
5. The next time you clone or interact with GitHub, you'll be prompted to authenticate with your personal account

### macOS
1. Open **Keychain Access** (found in Applications > Utilities)
2. Search for `github.com`
3. Delete the stored credentials
4. The next time you clone or interact with GitHub, you'll be prompted to authenticate with your personal account

### Linux
```bash
# Clear stored credentials
git credential-cache exit

# Or if using credential store
rm ~/.git-credentials
```

## Method 2: Using SSH Keys (Recommended for Advanced Users)

SSH keys provide a more secure and flexible way to authenticate, especially when working with multiple accounts:

### Step 1: Generate a new SSH key for your personal account
```bash
ssh-keygen -t ed25519 -C "your-personal-email@example.com" -f ~/.ssh/id_ed25519_personal
```

### Step 2: Add the SSH key to your personal GitHub account
1. Copy the public key:
   ```bash
   cat ~/.ssh/id_ed25519_personal.pub
   ```
2. Go to GitHub.com → Settings → SSH and GPG keys → New SSH key
3. Paste the key and save

### Step 3: Configure SSH to use the correct key
Create or edit `~/.ssh/config`:
```
# Personal GitHub account
Host github.com-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal
    IdentitiesOnly yes

# Work/EMU GitHub account
Host github.com-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
    IdentitiesOnly yes
```

### Step 4: Clone using the appropriate host
```bash
# For personal repositories
git clone git@github.com-personal:username/repo.git

# For work repositories
git clone git@github.com-work:company/repo.git
```

## Method 3: Per-Repository Configuration

You can configure Git to use different credentials for specific repositories:

### Step 1: Clone the repository
```bash
git clone https://github.com/username/repo.git
```

### Step 2: Configure user for this repository
```bash
cd repo
git config user.name "Your Personal Name"
git config user.email "your-personal-email@example.com"
```

### Step 3: Set credential helper to not store globally
```bash
# Use per-repository credentials
git config credential.helper ""
```

When you push or pull, you'll be prompted for your personal account credentials.

## Method 4: Using GitHub CLI

The GitHub CLI (`gh`) makes it easy to switch between accounts:

### Step 1: Install GitHub CLI
Download from https://cli.github.com/

### Step 2: Authenticate with your personal account
```bash
gh auth login
```
Follow the prompts and select:
- GitHub.com (not GitHub Enterprise Server)
- HTTPS or SSH (your preference)
- Authenticate through the web browser

### Step 3: Clone repositories
```bash
gh repo clone username/repo
```

## Method 5: Using Git Credential Helper with Multiple Accounts

Configure Git to use different credential helpers based on the repository path:

### Step 1: Set up conditional includes in your global Git config
Edit `~/.gitconfig`:
```ini
[user]
    name = Your Work Name
    email = your-work-email@company.com

[credential]
    helper = manager-core

# Personal repositories
[includeIf "gitdir:~/personal/"]
    path = ~/.gitconfig-personal
```

### Step 2: Create `~/.gitconfig-personal`
```ini
[user]
    name = Your Personal Name
    email = your-personal-email@example.com

[credential "https://github.com"]
    username = your-personal-username
```

### Step 3: Clone personal repositories into the ~/personal/ directory
```bash
mkdir -p ~/personal
cd ~/personal
git clone https://github.com/username/repo.git
```

## Quick Reference

| Method | Best For | Complexity |
|--------|----------|------------|
| Credential Manager | Quick one-time switch | Easy |
| SSH Keys | Managing multiple accounts long-term | Medium |
| Per-Repository Config | Occasional personal projects | Easy |
| GitHub CLI | Frequently switching accounts | Easy |
| Conditional Includes | Organized workspace with many repos | Medium |

## Troubleshooting

### Issue: "Permission denied" when cloning
- Make sure you've cleared the old credentials
- Verify you're using the correct account credentials
- Check that your personal account has access to the repository

### Issue: Still using work account after clearing credentials
- Try logging out of GitHub.com in your browser
- Clear browser cookies and cache
- Restart your terminal/command prompt

### Issue: SSH key not being recognized
- Verify the SSH key is added to your GitHub account: `ssh -T git@github.com`
- Check SSH config file syntax
- Ensure correct permissions on SSH files:
  ```bash
  chmod 700 ~/.ssh
  chmod 600 ~/.ssh/id_ed25519_personal
  chmod 644 ~/.ssh/id_ed25519_personal.pub
  ```

## Additional Resources

- [GitHub Authentication Documentation](https://docs.github.com/en/authentication)
- [Managing Multiple GitHub Accounts](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-your-personal-account/merging-multiple-personal-accounts)
- [About SSH Keys](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/about-ssh)
- [Git Credential Storage](https://git-scm.com/book/en/v2/Git-Tools-Credential-Storage)
