# Introduction

This is just a very basic crash course in Git fundamentals for version control and collaboration.

# Repositories

In Git, a _repository_ is just a Git project ("project" is often used interchangeably with "repository").
The repository may exist on your local device or in some network location.

## Initializing

You can create a new project on your local machine with the command `git init [path/to/directory]`.
If the path exists and is a directory, the directory will be made into a Git repository;
otherwise, a new empty directory will be initialized at that location.

Excluding the path argument is the same as `git init .`, which initializes a repository in your working directory.

## Cloning

Typically, a repository is hosted at some network location.
GitHub and GitLab are examples of sites that host Git projects;
both will allow you to create unlimited free repositories.

Users will _clone_ the remote repository to their local machine, make changes on their clone,
and then _push_ those changes back to the repository at the host.
Changes made to the host can be _fetched_ or _pulled_ to the clone.

The host repository is _upstream_ of the clone; the clone is _downstream_ of the host.

To clone a repository, execute the command `git clone <path/to/repository>`.
This can be a path to a repository on the local machine (e.g., `git clone ~/projects/git-tutorial`),
or a network path (`git clone ssh://git@github.com:merenze/git-tutorial.git`).

## Remotes

A _remote_ is an alias for a repository location; it helps us avoid having to type a long URL over and over.
When you clone a repository, the default remote `origin` is added.
Any time you use the word `origin` in a Git command, Git knows you mean the location from which the project was cloned.
* Add a remote: `git remote add <remote-name> <path/to/remote-repository>`
* Remove a remote: `git remote remove <remote-name>`

Often you will initialize an existing project as a Git repository on your local machine,
but you actually want the "origin" to be at another location.
Git does not add any remotes when you initialize a repository.
* To add the origin, use `git remote add origin <path/to/new-origin>`.

# Version control
Git uses _commits_ for version control.
A commit is basically a snapshot of the code at any given time.
Commits have unique _hashes_, or IDs, and the entire history of commits is stored so the code can be _rolled back_ to any commit at any time.
The history of commits can be viewed with `git log`.
* For readability, `git log --oneline` may also be a helpful command, as the vital info for each commit is condensed to a single line.

## Staging

For changes to be committed, they must first be _staged_.
This just means Git knows the files for which it should be _tracking_ changes.
* A _staged_ file is always being tracked, and a _tracked_ file is always staged;
  they are basically interchangeable, but _staging_ is what _we_ do and _tracking_ is what _Git_ does.

To see which files' changes are being _tracked_, use `git status`.
If there are any tracked changes in the project directory when you run `git status`, you should also see two more commands:
* `git add <file>` to stage
* `git reset <file>` to unstage
  * Resetting _only_ unstages files; the changes are still there.
    To _discard_ changes since the last commit, use `git reset --hard`.
  * Hard resets do not work with paths.
    If you want to discard changes for _specific files_, use `git checkout HEAD -- <file>`.
    
Both of these commands can be run with multiple files (`git add <file1> [file2] ...`).
They are also both recursive when run on directories,
so `git add dir` will add all the files in the directory named `dir`, as well as all the files in `dir`'s subdirectories, and so on.
* `git add --all` is the same as `git add .`; every changed file in the project will be staged.

Git does _not_ track directories; it only cares about files.
Empty directories will not be staged.

You can view unstaged changes with `git diff [path/to/file-or-directory]`.
* Ommitting the file path is the same as `git diff .`, which is to say that it will show all unstaged changes in the project.

It is a good idea to check the diff before staging files in order to avoid committing obvious mistakes.
It may also help you decide if the changes should be broken up across multiple commits.

### Ignoring changes
Sometimes your project will include files that should _never_ be committed;
for example, a configuration file, which may be different on every machine,
or a file containing sensitive information such as a database password.

To permanently ignore these files, place their relative path names in a file named `.gitignore`.
Git will never interact with any file named in a `.gitignore` file,
will not show these files as either tracked or untracked in `git status`,
and will not add them when using `git add` either implicitly or recursively.

A `.gitignore` file can include paths to files located in subdirectories (e.g., `path/to/file`).
Subdirectories may also contain their _own_ `.gitignore` files (e.g., `path/to/.gitignore`),
which is often preferable in a larger project with a more complex directory structure.
* By including an empty `.gitignore` in directory,
  you can force that directory to be tracked for commits,
  allowing you to maintain directory structure for a project that has otherwise empty directories.

Git can ignore the `.gitignore` file, in which case it will have no effect.
It _cannot_ ignore the `.git` file.

The wildcard operator `*` means all files in the directory should be ignored, including `.gitignore` itself.
The prefix `!` means to _not_ ignore the named file.

The following `.gitignore` will ignore all files in the directory except `hello-world.txt` and `.gitignore`.
```
*
!.gitignore
!helloworld.txt
```

## Committing

To commit all staged changes, use `git commit`.
You will be prompted to write a commit message using your system's default editor.
To skip this prompt, use `git commit [-m|--message] ["Commit message"]`.

Commit messages should be succinct and descriptive, so that at a glance anyone can tell generally what the commit is meant to do.
"Added <featurexyz>" is a good commit message. "Some changes" is not.

For the first commit of a project, "First commit" or "Initial commit" are common appropriate commit messages.

After committing, `git log` should show your latest commit on top.

### Amending

__A note about amending__:
This can be very dangerous.
It is probably okay to amend commits that are not yet pushed, because others are not working off the same commit history.
Commits that are already pushed should only be amended if you are _absolutely sure_ that no one else is working off your branch.

Sometimes you will want to _amend_ a commit, or make changes to your commit history.
This can be done to add or remove changes to a commit, or just to change the commit message.
* First stage any changes you want to add to the previous commit.
* Amend the most recent commit: `git commit --amend`

#### Older commits

__Another note__:
Amending is dangerous. Rebasing is more dangerous.
Do not do this if you don't know what you're doing.

This is more advanced, and may screw everything up,
so before attempting this it is wise to back up your branch.

If for some reason you need to amend an older commit, you can do so using an _interactive rebase_.
* Interactive rebase: `git rebase -i <commit>`
Find the lines with the commits you want to amend, and change `pick` to `edit`.
Save and close the file and you are now rebasing.

Make any amends to the commits you want and move on with `git rebase --continue`.
This likely will create conflicts in commits later in the history;
you will have to resolve those conflicts and amend those commits as well.
  
## Rolling back

Each commit can be referenced by its unique hash, which can be viewed by running `git log`.
Git also maintains a reference to your working commit, called `HEAD`.
By default, `HEAD` points to the latest commit.

### Checkout

When we _checkout_ a previous commit, we enter what is called _detached head state_,
which allows us to safely examine the code at this commit without modifying anything at the latest commit.
* Checkout a commit: `git checkout <commit-hash>`
  * You do not need to include the entire hash (which is very long); the first 6-8 or so characters will do.
  * For all commands, in place of the hash, `HEAD~n` can be used to checkout _n_ commits older than `HEAD`;
    e.g., `git checkout HEAD~2`.

`HEAD` now points at the checked out commit.
 As Git will tell you when entering this state, you can return by performing another checkout.

### Reset

When we _reset_ to a previous commit, we basically undo all commits since that commit.
  * Reset: `git reset <commit-hash> [file]`
    * Ommitting the file name is the same as `git reset <commit-hash> .`.

# Collaboration

  
## Branching

_Branching_ allows us to have several different working versions of a project;
by working on different branches, we minimize interference between multiple people working on the project simultaneously.

The command `git branch` shows the complete list of branches in the repository.
Your active branch should be highlighted.

When you initialize a repository, Git creates a "master" branch.
Generally, you should not develop on the master branch;
common practice is to create a `develop` branch off of `master`,
and to create further branches off `develop` for other working features and updates.
* Create a branch: `git branch <branch-name>`
* Confirm the creation of the branch with `git branch`.
* _Checkout_ (switch to) a branch with `git checkout <branch-name>`; this becomes your working branch, and any commits will be added to this branch.
* You can also create and checkout a new branch with a single command: `git checkout -b <new-branch-name>`

You cannot checkout a branch when you have uncommitted changes.
If you do not want to commit the changes, you can _stash_ them with `git stash`.
* Staged changes will not be added; to include these, first reset those files.

You can now checkout any branch and make any changes.
When you want to reintroduce the changes, checkout the correct branch and use `git stash pop`.
* A common use case for this is when you discover you have been making changes on the wrong branch;
  you can stash the changes, checkout the correct branch, and pop them.
  
Each branch has its own commit history.
When you create a branch, your currently working branch branch becomes its parent branch,
and the new branch is _based_ at the head of its parent's branch.
The base is the commit at which histories diverge, so you can continue making different commits to either the parent or child branch,
but the two branches will have identical histories up to and including the base of the new branch.

### Merging

Branches eventually need to be _merged_.
This brings all of the changes from the target branch into the working branch.
* To merge a child into a parent or _base_ branch, checkout the child branch and use `git merge <child-branch>`.
* Technically, any branch can be merged into any other; be careful with this.
* You can also merge an upstream branch into a local one using `git merge <upstream-repository>/<upstream-branch>`
  * Example: `git merge origin/develop`

Often a base branch will be updated while one of its children is still in development;
for instance, a branch with multiple children may have one of those children merged into it,
and its other children will need to be updated with those changes.

The same merge command is used; checkout the child branch and use `git merge <base-branch>`.
A _merge conflict_ will occur when the working branch and the target branch both include commits that alter the same file.
Git will attempt to _resolve_ conflicts automatically, but in some cases it will not know which changes to keep.

When this happens, you will have to manually resolve the conflicts.
* Directly alter the files to keep only the correct code.
* Stage the files and run `git commit`; Git will create a commit message for you, but you can change this if you wish.

### Deleting

Local branches are deleted with `git branch [-d|--delete] <branch-name>`.

Remote branches are deleted with `git push [-d|--delete] <upstream-repository> <branch-name>`.

## Pushing

Since all users will likely be working in local clones, commits have to be _pushed_ upstream, so they can be integrated into other users' clones.
* To push all commits, use `git push [-upstream-repository] [branch-name]`
  * Example: `git push origin develop`
* __Danger__: If you have revised the history of upstream commits, the push will not be allowed;
  to override this, use `git push [-f|--force]`.
* To set up your local branch to _track_ an upstream branch, use `git push [-u|--set-upstream] [upstream-repository] [branch-name]`.
  * When your local branch is tracking an upstream branch, `git push` can be run with the upstream and branch name ommitted
    to automatically push to the tracked branch.
  
## Fetching

After commits are pushed upstream, they exist on the clone from which they were pushed, and the upstream repository they were pushed too--
but not on any other downstream clones.

You need to _fetch_, or check for, changes from upstream. Then they can be merged into branches on the clone.
* Fetch changes: `git fetch [upstream-repository] [branch-name]`
  * If your branch is tracking one upstream, `git fetch` can be run with no arguments to fetch from that branch.
* Merge changes: `git merge <upstream-repository>/<upstream-branch-name>`

### Pulling

Since fetching is so often immediately followed by merging, we can actually combine these into one step called _pulling_.
* Pull upstream changes: `git pull <upstream-repository> <branch-name>`.
* Again, `git pull` with no arguments will pull changes from the tracked branch.

When working with others on a project, it is important to pull often;
doing so can save you a lot of merge conflict resolution.
