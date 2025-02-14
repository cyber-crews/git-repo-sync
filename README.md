# git-repo-sync

## Synchronization of Branches of Remote Git-repositories

The **git-repo-sync** synchronizes branches between two remote Git-repositories.<br/>
It is like you have two entry points to a single repository and your two remote Git-repositories will be behaving as a single repository.<br/>

**git-repo-sync** implemented as a bash script.

The main idea of this tool is to install, auto-run every couple of minutes and forget.<br/>
But know "The Trade-off" section below.

## Use cases

* Adhesion of Git-remote-repositories of a client and a software/support supplier.
    * Access to your Git remote repository is restricted by your local network.
    * After completing of some work, remote access to your Git repository could be terminated.
* Provides an independence from your base remote Git repository if it is slow and could be out of service time after time.
* You software teams have independent Git remote repositories.

## How to run it

Copy **git-repo-sync** somewhere

    git clone https://github.com/it3xl/git-repo-sync.git

Let **git-repo-sync** know location of your remote Git repositories.<br/>
Modify `url_a` and `url_b` variables in [default_sync_project.sh](https://github.com/it3xl/git-repo-sync/blob/master/repo_settings/default_sync_project.sh).<br/>
You can use URL-s and file paths.

    url_a=https://example.com/git/my_repo.git
    
    url_b='/c/my-folder/my-local-git-repo-folder'

Run periodically the `git-sync.sh` file, which is located in the root of **git-repo-sync**.<br/>
The `git-sync.sh` will tell you if there are any troubles. The main among them is you need to update awk to gAWK on Ubuntu.

    bash  git-sync.sh

FYI. Call _git-sync.sh_ over the bash as it is not tested for zsh.

## The Trade-off

_The Trade-off_ is an automated Git-conflict solving logic of git-repo-sync.

Even if you run **git-repo-sync** periodically and often, you still have a chance to get a Git-conflict. But a small chance.<br/>
So, you must know what to do in case of Git-conflicts solved by git-repo-sync.

### Minimize chances of The Trade-off

Run **git-repo-sync** before Git-pushing. I.e. synchronize your both Git-remote-repos before pushing into any of them.<br/>
In this case, Git will be responsible for conflict resolution, not **git-repo-sync**.

### When git-repo-sync will be solving the conflicts.

You should have the both

- You run **git-repo-sync** rarely. I.e. someone aready pushed commites exactly to your branch after last running of **git-repo-sync**.
- And you and your teammate have pushed changes to the same Git-branch but through different remote repositories and your remote repositories are no synchronized between your Git-pushes.

Basically, you don't know about **git-repo-sync** until you are in this situation.

### Behavior of git-repo-sync in case of Git-conflicst

**git-repo-sync** sees a Git-conflict and uses one of Conflict Solving strategies described below.<br/>
As a result, you should provide the below steps to fix The Trade-off.

### Your steps to fix The Trade-off

The main idea is "Re-push your local Git-commit in case of a conflict".

- Run **git-repo-sync** to synchronize both Git-remote-repositories (if you have no periodical auto-runs).

- Upload changes from your remote Git-repository to your local repository.
- Check if you local commit have lost its remote counterpart. I.e. the commit exist only in your local repository.
  - Performe Git-merge/rebase of your local commit.
  - Performe Git-push of your changes.

- Run **git-repo-sync** to synchronize your changes with changes from another side Git-remote-repository (if you have no periodical auto-runs).

### How do I know if there were Git-conflicts

- Check it manually. This is described in the above steps.
- **git-repo-sync** has notifications over plain text files. Ask your DevOps to distribute it.

## Using On Linux

Run `git-sync.sh` and it will tell you what **git-repo-sync** needs.<br/>
In most cases you have to install gAWK. This applies to Ubuntu.<br/>
Docker Alpine Linux images require *bash* and *gAWK* to be installed.<br/>
You have to update the *bash* if you use an extra old Linux distro.

## Using on Windows

Ha! You're lucky. You have to do nothing and have five options to run **git-repo-sync**.

Open PowerShell or CMD in the **git-repo-sync** folder and run one of three.

    "C:\Program Files\Git\bin\bash.exe" git-sync.sh
    "C:\Program Files\Git\usr\bin\bash.exe" git-sync.sh
    "C:\Program Files\Git\git-bash.exe" git-sync.sh

Or you can reinstall Git and integrate the bash into your Windows during installation. Then run

    bash  git-sync.sh

Or you can try to update the PATH environment variable. Try to add the following (that wasn't tested by me)

    ;C:\Program Files\Git\cmd;C:\Program Files\Git\mingw64\bin;C:\Program Files\Git\usr\bin

## Do not synchronize all branches

Despite that there are [fair cases](https://github.com/it3xl/git-repo-sync/issues/3#issuecomment-771494886) when it is useful to sync all branches, this is not always a good idea.<br/>
Some well know Git-servers block some branches in different ways. Some of them create "trash"-branches which you do not want to see synchronized.<br/>

So, you can synchronize branches that have special prefixes only.<br/>
You could configure these prefixes in [default_sync_project.sh](https://github.com/it3xl/git-repo-sync/blob/master/repo_settings/default_sync_project.sh) configuration file.<br/>
What's important, these prefixes are related to correspondent *conflict solving strategies*.

## Conflict Solving Strategies

### The Victim Strategy

By default all branches are synced under this strategy.<br/>
You can do whatever you want with such branches from both sides (repositories).<br/>
In case of commit conflicts, any newest commit will win.<br/>
You can relocate branches to any position, delete and move them back in history if you run **git-repo-sync** regularly.<br/>

Use the following variable to limit branches synchronized by this strategy.

    victim_branches_prefix=@

The most common value for victim_branches_prefix is "@".<br/>
In this case only branches that start with `@` will be synchronized.
E.g. `@dev`, `@dev-staging`, `@test`, `@test-staging`, `@my-feature`, etc.

### The Conventional Strategy

By using this strategy you limit what your teammates may do from another side repository with branches on your side remote repository.

Branches with the following prefix will be owned by the repo from [url_a](https://github.com/it3xl/git-repo-sync/blob/master/repo_settings/default_sync_project.sh) variable. Let's call it *A side*.

    side_a_conventional_branches_prefix=client-

Branches with the following prefix will be owned by the repo from [url_b](https://github.com/it3xl/git-repo-sync/blob/master/repo_settings/default_sync_project.sh) variable. Let's call it *B side*.

    side_b_conventional_branches_prefix=vendor-

Other examples of prefix pairs: `a-`, `b-`; `microsoft/`, `google/`; `foo-`, `bar-`;

On the owning side repo: You can do whatever you want with such branches.

On a repo of another side:<br/>
You can do fast-forward updates and merges.<br/>
You can move such branches back in Git-history if you run **git-repo-sync** periodically.

All commit conflicts will be solved in favor of the owning side.<br/>

### Other Unimplemented Strategies

Just propouse something interesting.<br/>
BTW, the Victim and Conventional approaches cover 80% of cases you need (I beleive).

## Disaster Protection

People have to make mistakes to become better. This is normal. But let's protect our clients from such the mistakes.<br/>
Define *sync_enabling_branch* variable

    sync_enabling_branch=it3xl_git_repo_sync_enabled

Its value may represent any branch name.<br/>
Examples: `@test`, `client-prod`, `vendor-master`, `it3xl_git_repo_sync_enabled`.<br/>

The **git-repo-sync** will check if such a branch exist in both remote repositories and that it has the same or related commits, i.e. its commits are located in the same Git-tree.<br/>
This will protect you from occasional adhesion of unrelated git-repositories and deletion of branches that have the same names.<br/>
Git may store many independent projects (trees) in the same repository and this is uncommon behavior for many users.

I advise to use `it3xl_git_repo_sync_enabled` branch name to make this explicit for others that their remote Git-repo is synchronized with another remote repo.<br/>
They could search for the word *it3xl_git_repo_sync_enabled* in the Internet and understand the applied sync solution.

Be aware that a branch mentioned in the `sync_enabling_branch` variable will be alwasy synchronized by **git-repo-sync**.<br/>
Probably this is not a good idea to specify here the `master` branch name because a branch mentioned in `sync_enabling_branch` will be synchronized under the Victim strategy. But you can specify there a branch with one of your conventional prefixes for the Conventional syncing of it. For example `client-master`.

## Notes, Drawbacks & Limitations
* Usage with SSH isn't tested but possible.
* **git-repo-sync** is resilient for HTTP fails and interruptions.
* It has protections from an occasional deletion of your entire remote repository.
* Arbitrary Git-history rewriting is supported.
* Within a single installation, **git-repo-sync** can synchronize as many pairs of Git-repositories as you want. Every sync pair is a sync project for **git-repo-sync**.
* **git-repo-sync** doesn't synchronize Git-tags because some popular Git-servers block manipulations with Git-tags. (I do not want to research and cover all cases by tests for it).
* **git-repo-sync** doesn't attempt to do Git-merge or rebase. Just FYI.

## Support Operations

### Remote Repo Replacing Support

This is a real case of my customer. You may want to synchronize your existing Git-repo with a Git-repo of your new software parnter.

Option 1.<br/>
Create a new git-repo-sync project and use it (project description file or environment variables).

Option 2.<br/>
Modify your existing project. Update its description file or environment variables.<br/>
Delete `git-repo-sync/sync-projects/<your-sync-project-name>` directory.<br/>
Start synchronization as usual.

Option 3.<br/>
Your Git-repository is extra huge and you can't recreate it. This is a TL;DR. Ask a Git-professional for a help. 

## Automation support
* **git-repo-sync** works with remote Git repositories asynchronously, by default.
* It works much faster under \*nix OS-es because Git-bash on Windows is slower. But compare to network latency, this is nothing.
* You can separate change detection and synchronization phases of **git-repo-sync** for readability of CI/CD logs.
* Multiple configuration capabilities are supported. Environment, configuration files, combination of them.
* Integration with **bash Git Credential Helper - [git-cred](https://github.com/it3xl/bash-git-credential-helper)** to obtain credentials from a parent shell environment.
* You shouldn't do anything in case of connectivity fails. Continue to run **git-repo-sync** periodically and everything will be restored automatically.
* After every synchronization, analyze notification files to send notifications about branch deletions or commit conflict solving.  
See `git-repo-sync/sync-projects/<your-sync-project-name>/file-signals/`
  * `notify_solving` - for conflict solving
  * `notify_del` - for deletions
* See [instructions](https://github.com/it3xl/git-repo-sync/blob/master/repo_settings/default_sync_project.sh) on how to configure more synchronization pairs of remote Git repositories.
* Number of pairs is unlimited. Every pair is a separate sync project.

## Notes
* Git-rebases are not used in the implementation as they could bring issues. https://github.com/it3xl/git-repo-sync/issues/15

## Required Specification

* Use any Linux or Windows machine.
* Install Git.
* Check that bash version is 4.2 or above. (that's why there may be problems on macOS)
* Use bash to run **git-repo-sync**.
* For users of \*nix OS-es.
  * update *bash* in old Linux distros. (4.2 or above)
  * check that gAWK (GNU AWK) is installed on your machine. Consider [this case](https://askubuntu.com/questions/561621/choosing-awk-version-on-ubuntu-14-04/561626#561626) if you are going to update mAWK to gAWK on Ubuntu.
* Tune any automation to run **git-repo-sync** periodically - crones, schedulers, Jenkins, GitLab-CI, etc. Or run it periodically yourself.

## Contacts

[it3xl.ru](http://it3xl.ru)
