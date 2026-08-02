# Git Notes

- [Git Notes](#git-notes)
  - [Add SSH key to github](#add-ssh-key-to-github)
  - [Git user.name and user.email configuration](#git-username-and-useremail-configuration)
    - [Global Configuration (All Repositories)](#global-configuration-all-repositories)
    - [Local Configuration (Single Repository)](#local-configuration-single-repository)
    - [Verification \& Checking Settings](#verification--checking-settings)
  - [Clone repository](#clone-repository)


## Add SSH key to github

Adding ssh key of your machine to github allow your machine to have access to your online repos. 

**Steps**

1. Create ssh key in your machines and copy your ssh key to clipboard

``` bash
# Generate ssh key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy ssh key to clipboard
pbcopy < ~/.ssh/id_ed25519.pub
```
2. Copy ssh key to github
   - Log into your GitHub Account.
   - Click your profile photo in the upper-right corner and select Settings.
   - Find the Access section in the left sidebar and click SSH and GPG keys.
   - Click the green New SSH key or Add SSH key button.
   - In the Title field, type a memorable name for the machine (e.g., "Personal Laptop").
   - Keep the Key Type as Authentication Key.Paste your key directly into the Key field.
   - Click Add SSH key.
 - 
3. Test the Connection in your terminal

``` bash
ssh -T git@github.com
```

## Git user.name and user.email configuration

To configure your Git username and email globally for all repositories on your system, open your terminal or Git Bash and run the `git config --global user.name "Your Name"` and `git config --global user.email "your.email@example.com"` commands. These details are permanently baked into any commits you make from that moment forward.

### Global Configuration (All Repositories)

Run these commands in your command line to set your default identity:

**Bash**
``` bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### Local Configuration (Single Repository)

If you need to use a different identity for a specific project (e.g., separating your work and personal accounts), navigate to that project's directory and run the commands without the `--global` flag:

**Bash**


```bash
# Navigate to your project folder first

git config user.name "Your Work Name"
git config user.email "work.email@company.com"
```

### Verification & Checking Settings

You can check your active configurations at any time using the following options:
- Check active username: `git config user.name`
- Check active email:  `git config user.email`
- List all active configuration values: `git config --list`
- Show where settings are coming from: `git config --list --show-origin`

## Clone repository

Go to your workspace folder
```bash
# Go to you workspace folder
cd path/to/your/directory
```

Then run the following code to clone
```bash
git clone git@github.com:username/repository.git
```
