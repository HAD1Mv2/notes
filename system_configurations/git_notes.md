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