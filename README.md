# Introduction
This is just a very basic guide to Git fundamentals for version control and collaboration.

It is by no means exhaustive, but it can help you get your feet wet.

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
GitHub and GitLab are examples of repository hosts;
both will allow you to create unlimited free repositories.

Users will _clone_ the remote repository to their local machine, make changes on their copy of the code, and then _push_ those changes back to the repository at the host.

To clone a repository, execute the command `git clone <path/to/repository>`.
This can be a path to a repository on the local machine (e.g., `git clone ~/projects/git-tutorial`),
or a network path (`git clone ssh://git@github.com:merenze/git-tutorial.git`).

## Remotes

A _remote_ is an alias for a repository location; it helps us avoid having to type a long URL over and over.
When you clone a repository, the default remote `origin` is added.
Any time you use the word `origin` in a Git command, Git knows you mean the location from which the project was cloned.

A remote can be added with the command `git remote add <name> <path/to/remote>`, and removed with `git remote remove <name>`.

Often you will initialize an existing project as a Git repository on your local machine,
but you actually want the "origin" to be at another location.
Git does not add any remotes when you initialize a repository, but you can simply run the command `git remote add origin <path/to/new/origin>`
to set the origin where you want it.

# Version control
Git uses _commits_ for version control.
A commit is basically a snapshot of the code at any given time.
Commits have unique IDs, and the entire history of commits is stored so the code can be _rolled back_ to any commit at any time.
The history of commits can be viewed with `git log`.
For readability, `git log --oneline` may also be a helpful command, as the vital info for each commit is condensed to a single line.

## Staging

For changes to be committed, they must first be _staged_.
This just means Git knows the files for which it should be _tracking_ changes.
* A _staged_ file is always being tracked, and a _tracked_ file is always staged;
  they are basically interchangeable, but _staging_ is what _we_ do and _tracking_ is what _Git_ does.

To see which files' changes are being _tracked_, use `git status`.
If there are any tracked changes in the project directory when you run `git status`, you should also see two more commands:
* `git add <file>` to stage
* `git remove <file>` to unstage
Both of these commands can be run with multiple files (`git add <file1>, <file2>, ...`).
They are also both recursive when run on directories,
so `git add dir` will add all the files in the directory named `dir`, as well as all the files in `dir`'s subdirectories, and so on.
* `git add --all` is the same as `git add .`; every changed file in the project will be staged.

Git does _not_ track directories; it only cares about files.
Empty directories will not be staged.

You can view unstaged changes with `git diff [path/to/file-or-directory]`.
* Ommitting the file path is the same as `git diff .`, which is to say that it will show all unstaged changes in the project.

It is a good idea to check the diff before staging files in order to avoid committing obvious mistakes.
It may also help you decide if the changes should be broken up across multiple commits.

## Committing

To commit all staged changes, use `git commit`.
You will be prompted to write a commit message using your system's default editor.
To skip this prompt, use `git commit [-m|--message] ["Commit message"]`.

Commit messages should be succinct and descriptive, so that at a glance anyone can tell generally what the commit is meant to do.
"Added <featurexyz>" is a good commit message. "Some changes" is not.

For the first commit of a project, "First commit" or "Initial commit" are common appropriate commit messages.

After committing, `git log` should show your latest commit on top.

## Rolling back

Each commit can be referenced by its unique hash, which can be viewed by running `git log`.
Git also maintains a reference to your working commit, called `HEAD`.
By default, `HEAD` points to the latest commit.

### Checkout

When we `checkout` a previous commit, we enter what is called _detached head state_,
which allows us to safely examine the code at this commit without modifying anything at the latest commit.
We can confirm this by the prompt we are greeted with when entering this mode:
```
You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.
```

`HEAD` now points at the checked out commit.

### Reset
