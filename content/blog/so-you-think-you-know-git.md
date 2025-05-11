+++
title = 'So You Think You Know Git?'
date = 2024-04-15T14:39:34+05:30
draft = false
cover.image = "/images/git_cover.jpg"
ShowToc = true
tags = ["linux"]
+++

This article is just about my learnings from the talk [So you Think you know Git](https://youtu.be/aolI_Rz0ZqY?si=HcJvZSBtWKw0Ssdc).

One might think that everything there is to know about Git has already been covered, but that would be wrong. Even today, the Git codebase is seeing around 9 commits per day and 10,000 commits in the last 3 years. That means there are still plenty of new things being added to Git that you might not know about. Although most of these changes are very subtle or very targeted, they could be useful to some people.

## Some Helpful Config Stuff

### Aliasing commands in Git

```bash
$ git config --global alias.staash 'stash --all'
```

The `git stash --all` command allows you to stash not only the changes in your working directory, but also any untracked files. This can be useful if you need to quickly switch branches or undo a large amount of work, without losing any of your changes.

### Running custom bash script

By creating the `staash` alias, you can now simply type `git staash` instead of the longer `git stash --all` command. This can save time and make your Git workflow more efficient, especially if you find yourself frequently needing to stash all your changes.

```bash
$ git config --global alias.bb !better-branch.sh
```

This command creates a global Git alias called `bb` that corresponds to running an exteernal shell scripot named `better-branch.sh`.

The purpose of this alias is to provide a shortcut for running a custom script. You'd just need to type `git bb` to run the script you want.

## Oldies but Goodies

### Git Blame

```bash
$ git blame
```

Okay most of you might know what this command does. This command is used to find the history of a file and understand who made specific changes.

This can be useful for:

- Tracking down the author of a problematic line of code
- Understanding the context and reasoning behind a particular change.
- Identifying areas of a codebase that might need refactoring or further attention.

_But_, there is a variation of this command which most people don't use much.

```bash
 $ git blame -L
```

This command blames only a "L"ittle part of the code only for a particular line range, get it?

```bash
$ git blame -w
```

This command ignores the whitespace when blaming the changes.

```bash
$ git blame -w -C
```

This command ignores the whitespace when blaming the changes along with detection of lines moved or copied in the same commit.

```bash
$ git blame -w -C -C
```

This command ignores whitespace and detect lines moved or copied in the same commit or the commit that created the file.

### Git Log

```bash
$ git log -S
```

This command is used to search the commit history for changes that introduce or remove the specified string using regular expressions.

### Git Diff

```bash
$ git diff --word-diff
```

This command is used to view the differences between two Git commits, files, or branches, with the changes highlighted at the word level instead of the line level.

This is particularly helpful when you want to see the changes between two Git objects in a more granular way. For example, if you want to find which Tailwind class was added when making change to your HTML code.

### Reuse Recorded Resolution

```bash
$ git config --global rerere.enabled true
```

![ReReRe](/images/rerere.png)

> The preimage of the README.md file was recorded for later use in conflicts.

The `rerere` stands for REuse Recorded REsolution.
If you've a merge conflict, you can tell git to remember this and remember how you fixed it. So if you get the same merge conflict again, Git will automatically fix it for you. Isn't that neat!

![ReReRe](/images/rerere_again.png)

> As you can see, the README.md file was staged using the previous resolution method when some conflict happened.

## Some New Stuff

### Branch Column

```bash
$ git branch --column
```

When you use the `git branch` command, it just lists the name of the branches in alphabetical order. But when using this command, it displays the branches in neat columns based on your column config.

### Force with Lease

```bash
git push --force-with-lease
```

This is kind of a "safe" version of force push. It'll check for what it thought the reference was and if it doesn't match it doesn't push it. It'll push as long your reference was the last one on the Git Server.

This comes in handy when you're rebasing or amending your commit messages.

### Maintenance

```bash
$ git maintenance
```

It adds a cron job for Git to run in the background every hour or everyday and do maintenance tasks to the repository.

This can be turned on by running:

```bash
$ git maintenance start
```

This will modify your .git/config file to add `maintenance.strategy` value set to `incremental` which is a shorthand for the following:

- `gc`: disabled
- `commit-graph`: hourly
- `prefetch`: hourly
- `loose-objects`: daily
- `incremental-repack`: daily

## New GitHub stuff

### Allowed merge types

Now you can specify which type of merge commits you want to be allowed to get merged in your repository. This gives you more control over the commit history and can help maintain a cleaner, more linear Git history.

### GitHub Actions

![Github Actions](/images/github-actions.png)
GitHub Actions is a powerful CI/CD (Continuous Integration and Continuous Deployment) platform that allows you to automate your software workflows right from your GitHub repository. You can use pre-built actions or create your own custom actions to build, test, and deploy your code.

### GitHub Codespaces

![Github Codespaces](/images/github-codespaces.png)
GitHub Codespaces is a cloud-based development environment that allows you to code directly from your browser. This can be useful for quickly spinning up a development environment or for collaborating with team members, without the need to set up a local development environment.

### GitHub Discussions

![Github Discussions](/images/github-discussions.png)
GitHub Discussions is a feature that allows you to create and participate in conversations about your project, separate from the traditional issue tracker. This can be useful for things like feature requests, Q&A, or general project discussions.

### GitHub Packages

GitHub Packages is a package hosting service that allows you to publish and consume packages (e.g., npm, Docker, Maven, etc.) directly from your GitHub repository. This can simplify your dependency management and streamline your development workflow.

These are just a few of the newer features and capabilities that GitHub has introduced in recent years. As you can see, GitHub has been continuously enhancing its platform to provide developers with more powerful tools and integrations to streamline their workflows.

## Why Submodules suck?

Git submodules are a way to include one Git repository inside another Git repository. This can be useful when one has a project that relies on code or assets from another project, and one wants to manage that dependency within the main project.

Even this blog's source code uses submodule. I've used [Hugo](https://gohugo.io/), which is a static site generator to build this blog site, and for styling the pages, I've used the [PaperMod](https://themes.gohugo.io/themes/hugo-papermod/). But for including the theme in the code, I'd to use Git submodules, which was the recommended way of installing the theme. While it does work, but it could work in a better fashion.

### Complexity

Working with submodules can add significant complexity to a project. Developers need to understand how to properly initialize, update, and manage the submodules in addition to the main project. This can steepen the learning curve and make the overall development workflow more cumbersome.

### Lack of Discoverability

Submodules can make it harder to understand the full dependency graph of a project. When someone is new to the codebase, it may not be immediately obvious that certain functionality or assets are coming from a submodule, rather than being part of the main project.

### Branching and Merging Issues

Managing branches and merges across the main project and its submodules can be tricky. Developers need to carefully coordinate changes between the different repositories to avoid conflicts and maintain a coherent project state.

### Performance Impacts

Cloning a project with submodules can be significantly slower than a project without them, as Git has to initialize and fetch each submodule separately. This can be a problem for large projects or for developers working on slow or unreliable network connections.

### Versioning Challenges

Keeping the versions of submodules in sync with the main project can be challenging. It's easy for the versions to get out of date, which can lead to compatibility issues or unexpected behavior.

## Conclusion

Git is always changing and improving. Even if you think you know everything about Git, there are always new features, commands, and better ways to use it that can make your work easier.

The important thing is to keep learning. Attend talks, read the documentation, and try out different Git commands and techniques. The more you explore, the more hidden gems you'll find that can streamline your development process and help you manage your code better.

Mastering Git is an ongoing process, not something you can just learn once and be done with. By constantly expanding your Git knowledge and staying up-to-date with the latest changes, you'll be able to use the full power of this essential tool and become a real Git expert. Keep coding and keep learning!
