---
title: "Fix sudo problems on git folders"
date: 2021-04-27T15:25:30+01:00
categories:
  - tips
tags:
  - git
  - ubuntu
last_modified_at: 2021-04-27T15:25:30+01:00
---

This tip will change the ownership of the repository folder to your current user, so you can run `git` commands without the need of `sudo` or `root` user.

There is a problem when we make a `git pull` or `git commit` using a `root` user, because it will change the ownership of the `.git` folder, making it impossible for our user to run git commands there.

It'll show errors like these:

``` sh
git pull
error: cannot open .git/FETCH_HEAD: Permission denied
```

To restore the ownership we are going to move to the repository folder and run the following line, this will make our `user` the owner of everything inside that folder.

``` sh
sudo chown -R "${USER:-$(id -un)}" .
```
