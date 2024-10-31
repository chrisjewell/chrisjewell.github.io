---
layout: post
title: Learning to Upgrade my Website's Codebase
description: Upgrading from jekyll-theme-chirpy v6.2.2 to v7.1.1
image:
category: [code]
tags:
  - command-line
  - git
  - learning
---

# jekyll-theme-chirpy v6.2.2 to v7.1.1
Followed upgrade guide on theme's [GitHub Wiki](https://github.com/cotes2020/jekyll-theme-chirpy/wiki/Upgrade-Guide), but needed to research along the way

## Step 1: Set up Remote Repositories for Theme (only needed first time)
Because I generated the site's repository using the `chirpy-starter` template, I am able to follow this path. This was my first time upgrading the theme, so I needed to add the remote repositories for the theme so that I could fetch the most recent version and merge it into my local branch (`main`).

Navigate to root dir of site repo and run:
```shell
git remote add chirpy git@github.com:cotes2020/chirpy-starter.git
```

Verify the remote repo was added correctly by running `git remote - v`:
```zsh
% git remote -v
chirpy	git@github.com:cotes2020/chirpy-starter.git (fetch)
chirpy	git@github.com:cotes2020/chirpy-starter.git (push)
...
```

## Step 2: Fetch latest version (upstream `chirpy`)
In root dir of repo, run the following:
```shell
git fetch chirpy --tags
```

This returned a long list of tags/versions since I hadn't upgraded the codebase in over a year.

## Step 3: Attempt to merge (prepare for conflict resolution)
Run a merge command for the latest tag available (`v7.1.1` in my case):
```shell
git merge --squash --allow-unrelated-histories v7.1.1
```

> [!NOTE] 
> The `--squash` flag here tells git to combine all of the changes/diffs between my local version and the remote/latest version into one giant commit. This makes the commit history a lot cleaner since it won't include all of the intermediate history from others. The `--allow-unrelated-histories` flag was something I found out I needed from [this GitHub issue discussion](https://github.com/orgs/community/discussions/22075). This simply allows the merge command to ignore the fact that these two branches don't share a common history.

## Step 4: Conflict Resolution (manual style)
This merge attempt resulted in a ton of merge conflicts that needed to me resolved manually. Specifically I ran into "add/add" conflicts where the auto-merge didn't know what to do, and decided to keep both the new and old versions, resulting in duplication.

A complicating factor was that a few of the core config files for the site had changed formats, meaning that the new version contained the default/empty fields, and the old version contained my current settings. My goal was to migrate to the new formats while keeping as many of my current settings/values intact, meaning that I needed to find a clear way to determine *which* changes should be applied, and which should be dropped.

Merge conflicts have always been tricky for me to figure out, especially on the command line. At the start of this process, I knew some of the basics for change/delta detection in git (`git status`, `git diff [filename]`, etc.), but these weren't giving me the level of detail I needed (i.e. "what *should* the file look like, and what does it *currently* look like?" - that "should" is the tricky part). That said, this process took some time and research to find an approach that would a) safely/effectively deduplicate the files, b) not take forever, and c) be something I could remember and use again when I run into this again in the future. With was [this StackOverflow answer](https://stackoverflow.com/a/19475535) that made things click for me and allowed me to start making progress.


> [!INFO] Further Reading
> GitHub's [Command Line Merge Conflict Resolution Guide](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-using-the-command-line)

The process that I adopted for this looked like this:
1. Run `git status`, and identify an unmerged path/file to work on
2. Run `git diff [before-branch] [after-branch] [filename]` where `before-branch` was `main`, `after-branch` was the tag `v7.1.1`, and `filename` was the selected file from above
	- e.g. `git diff main v7.1.1 _config.yml` 
3. Open the selected file in a text editor (I use BBEdit), and work through the file to manually clean up the duplications (being sure to remove the `>>>>>>`, etc.  artifacts from the merge conflict)
4. Once I was satisfied with the state of the file, I would save the file, then run `git add [filename]` to stage the file for commit.
5. Then repeat this process until all merge conflicts had been resolved.


# Step 5: Git Submodule Research & Updates
I got stuck on merging in updates to a submodule included in the repo. After a bit more research, I found that by running the commands below (found [here](https://github.com/cotes2020/chirpy-static-assets#readme:~:text=%24%20git%20submodule%20init%0A%24%20git%20submodule%20update) on the submodule repo's README), I could get this to "work," and I was able to run `git add assets/lib` to stage the changes. I probably need to come back to figure this out more, but I'm satisfied for now.

```shell
git submodule init
git submodule update assets/lib
```


> [!INFO] Git Submodules
> I found [this StackOverflow answer](https://stackoverflow.com/a/46582526) to be helpful in confirming that the commands above could work, and it also helped me to understand more of what was actually happening.

# Step 6: Commit, Bundle, and Push the Changes
Once I had worked through all of the manual changes needed, then was ready to commit the changes and get them deployed.

The process worked like this:
```shell
git add . && git commit -m "chore: upgrade to v7.1.1"
bundle update 
git push
```

With the push complete, the build and deploy GitHub Action I have set up kicked off, and deployed the changes to my site!

# Step 7: Reflect and Document 
I probably should have been writing this down as I went, and been including screenshots, but oh well. Spending this time here has been worth it, and now I can look back here when I need to upgrade the site again!
