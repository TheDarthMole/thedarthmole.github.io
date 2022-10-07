---
title: Collaborative Development with Git MIT
date: 2022-10-07T08:18:16+00:00
description: Collaborating with Git MIT
menu:
    sidebar:
        name: Collaborating with Git Mit
        identifier: git-mit-setup
        weight: 10
author:
    name: Nick Ruffles
    image: /images/author/nick.png
tags: ["Development", "Git", "Collaborate", "GitHub", "GitLab"]
categories: ["Git"]
hero: git-mit-logo.png
---

[Git MIT](https://github.com/PurpleBooth/git-mit) is a collaboration tool that allows you to show on platforms such as GitHub who you were writing code with.
You may have seen this before when you look at the commits for a repository, as more than one person as the author.

{{< img src="/posts/gitmit/github-git-mit.png" align="center" title="Verified commit using GPG key" >}}


## Installing Git mit

```bash
$ brew install PurpleBooth/repo/git-mit
```

## Setting up co-authors

Start off by cloing the repository that contains [the co-authors](https://github.com/EngineerBetter/git-authors) you may be working with.

```bash
mkdir -p $HOME/.config/git-mit
git clone https://github.com/EngineerBetter/git-authors $HOME/.config/git-mit
```

## Adding ourselves to the authors list

Now we need to add our information to this repository, so that others can collaborate with us by editing the `mit.toml` file.

```toml
[md]
name = "Marcus Dantas"
email = "marcus.dantas@engineerbetter.com"

[nr]
name = "Nick Ruffles"
email = "nick.ruffles@engineerbetter.com"

[tg]
name = "Tom Godkin"
email = "tom.godkin@engineerbetter.com"
```

Finally, commit the updated `mit.toml` file to the remote repo. 
Other users will have to pull the repo in order to update their authors list so that they can collaborate with you.

## Test out Git mit

In a repository you would like to collaborate on, enter the following to initialise git mit on a repository. For every new repository you create or clone, you will need to run this command again.

```bash
$ git mit-install
```

After initialising git mit for the repo, we can start collaborating. 
As seen below, if we want to collaborate with others then we simply enter our own initials (mine are `nr`) then the other collaborators initials from the `mit.toml` file we edited earlier.

```bash
$ git mit nr md tg
```

If we want to code on our own then just enter ours own initials

```bash
$ git mit nr
```

You're all set! Enjoy collaborating with other users on Git

## Optional git message

Marcus introduced me to the `.gitmessage` file recently, this file allows you to generate a template for a commit message allowing you to enforce better commit messages for yourself.

I was reading [this post by Eddie Prislac](https://dev.to/vetswhocode/git-gud-create-a-gitmessage-4ibj) that shows off how you can setup a template commit message. 
Instead of typing `git commit -m "<message here>"` you can just type `git commit` to have the template come up in vim (or your default editor) to edit. 
I would recommend checking out the blog post and setting it up for yourself. 

You should also append the following to the end of the `.gitmessage` file, as it allows git to see who's signed off on commits
(obviously replace the content with your own information)

```yaml
Signed-off-by: Nick Ruffles <nick.ruffles@engineerbetter.com>
```