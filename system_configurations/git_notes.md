# Git Notes

- [Git Notes](#git-notes)
  - [Add SSH key to github](#add-ssh-key-to-github)
  - [Git user.name and user.email configuration](#git-username-and-useremail-configuration)
    - [Global Configuration (All Repositories)](#global-configuration-all-repositories)
    - [Local Configuration (Single Repository)](#local-configuration-single-repository)
    - [Verification \& Checking Settings](#verification--checking-settings)
  - [Clone repository](#clone-repository)
  - [Create and Switch Branch](#create-and-switch-branch)
    - [Common Commands](#common-commands)
    - [What if the branch is on a remote repository?](#what-if-the-branch-is-on-a-remote-repository)
    - [Classic Alternative](#classic-alternative)
  - [Git Basic Workflow](#git-basic-workflow)
    - [Push change to repo](#push-change-to-repo)
    - [Create repo locally](#create-repo-locally)
  - [Install Python Package from Github Repo](#install-python-package-from-github-repo)
    - [Basic Installation](#basic-installation)
    - [Advanced Installation Options](#advanced-installation-options)
    - [Private Repositories \& Alternative Protocols](#private-repositories--alternative-protocols)
      - [1. Using SSH](#1-using-ssh)
      - [2. Private Repositories (HTTPS + Token)](#2-private-repositories-https--token)
    - [Adding to requirements.txt](#adding-to-requirementstxt)


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
## Create and Switch Branch

To switch to an existing branch in Git, use the command git switch <branch-name>. This is the modern, dedicated command introduced in Git 2.23 to replace the older, multipurpose git checkout command. [1](https://git-scm.com/docs/git-switch) [2](https://stackoverflow.com/questions/68356181/how-do-i-switch-a-branch-in-git) [3](https://gitbybit.com/gitopedia/git-commands/git-switch)

### Common Commands

* Switch to an existing branch:
  
``` bash
git switch main
```

* Create a new branch and switch to it immediately:
  
```bash
git switch -c feature-branch
```

(The -c flag stands for "create").

* Switch back to the previous branch you were on:
  
``` bash
git switch -
```

[4](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell) 

### What if the branch is on a remote repository?
If the branch exists on GitHub/GitLab but you haven't brought it to your local machine yet, run a fetch first, and then switch to it by name. Git will automatically set up local tracking for you: [5](https://refine.dev/blog/git-switch-and-git-checkout/) [6](https://www.git-tower.com/learn/git/faq/git-checkout-switch-branch) 

```
git fetch origin
git switch remote-branch-name
```

### Classic Alternative
If you are working on a very old version of Git (pre-2.23), you will need to use the classic syntax: [6](https://www.git-tower.com/learn/git/faq/git-checkout-switch-branch) 

* Switch branch: `git checkout branch-name`
* Create and switch: `git checkout -b new-branch-name` [6](https://www.git-tower.com/learn/git/faq/git-checkout-switch-branch) 

## Git Basic Workflow

### Push change to repo
Assuming you already create and clone repo from github and already create some new files to push to github repo, Then to do this 

1. Stage your files to prepare them for the save:
    ``` bash
    git add .
    ```
2. Commit your files to create a local save point:
    ``` bash
    git commit -m "Initial commit" # or any message regarding change you make
    ```
3. Push your code to GitHub:
    ``` bash
    git push -u origin main # change main with the branch you want to push to
    ```

### Create repo locally

- Create new repo in local machine
  ``` bash
  # inside your repo folder run
  git init
  ```
- Link your local repository to your remote GitHub repository 
  ``` bash
  git remote add origin <PASTE_YOUR_GITHUB_URL_HERE>
  ```

## Install Python Package from Github Repo

To install a Python package directly from a Git repository, prepend git+ to the repository URL in your pip install command. [1](https://www.youtube.com/watch?v=r-wwMk5faXo&t=2) [2](https://www.youtube.com/watch?v=3weWR1CMgzo&t=16) 

### Basic Installation

To install the latest commit from the default branch (usually main or master): [3](https://fronkan.hashnode.dev/pip-install-a-git-repository) [4](https://www.youtube.com/watch?v=AQrskWh-F5E&t=72) 

``` bash
pip install git+https://github.com/username/repository.git
```

------------------------------
### Advanced Installation Options
You can target specific versions, branches, or subdirectories by appending parameters to the URL:

| Target Type | Syntax Example |
|---|---|
| **Specific Branch** | `pip install git+https://github.com` |
| **Specific Tag** | `pip install git+https://github.com`|
| **Specific Commit Hash**| `pip install git+https://github.com` |
| **Subdirectory** (if `pyproject.toml `or `setup.py` isn't in the root) | `pip install "git+https://github.com"` |
| **Editable Mode** (for local development) | `pip install -e git+https://github.com` |

------------------------------
### Private Repositories & Alternative Protocols
#### 1. Using SSH
If you have SSH keys set up with your Git provider, use the SSH protocol. Note: Replace the usual colon (:) after the hostname with a forward slash (/): [4](https://www.youtube.com/watch?v=AQrskWh-F5E&t=72) [5](https://www.youtube.com/watch?v=gtyEPynfXY4&t=175) 

``` bash
pip install git+ssh://git@github.com/username/repository.git
```

#### 2. Private Repositories (HTTPS + Token)
For private repositories using HTTPS, you can pass a personal access token or app password: [6](https://docs.readthedocs.com/platform/stable/guides/private-python-packages.html) [7](https://stackabuse.com/bytes/installing-python-packages-from-a-git-repo-with-pip/) 

``` bash
pip install git+https://github.com
```

### Adding to requirements.txt
You can include Git URLs directly in your `requirements.txt` file exactly as they are written above: [5](https://www.youtube.com/watch?v=gtyEPynfXY4&t=175) 

``` text
requests==2.31.0
git+https://github.com
```
