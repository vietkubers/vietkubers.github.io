---
title: Github workflow for contributing to Kubernetes
tags: [linux, tutorials, kubernetes]
keywords: linux, networking
last_updated: November 16, 2018
sidebar: mydoc_sidebar
permalink: 2018-16-16-github-workflow.html
folder: mydoc
---


### 1. Big picture

![workflow](/static/img/github/github_workflow.png)


### 2. Fork a repository
* Goto https://github.com/kubernetes/kubernetes
* Hit the `Fork` button to fork your own copy of repo **kubernetes** to your github account

![Fork](/static/img/github/fork.png)


### 3. Clone the forked repository to local

Clone the forked repo in [above step](#2-fork-a-repository) to your local working directory:
```sh
$ git clone https://github.com/$user/kubernetes.git

$ cd kubernetes

$ git remote add upstream https://github.com/kubernetes/kubernetes.git

# Never push to upstream master
$ git remote set-url --push upstream no_push

# Confirm that your remotes make sense:
$ git remote -v
```
`$user` = {your github account name}


### 4. Create a branch to add a new feature or fix issues

Update local working directory:

```sh
$ cd kubernetes

$ git fetch upstream

$ git checkout master

$ git rebase upstream/master
```

{% include note.html content="Please don't use `git pull` instead of `git fetch / rebase`. `git pull` does a merge, which leaves merge commits. These make the commit history messy and violate the principle commits." %}

Create a new branch:
```sh
$ git checkout -b mybranch
```


### 5. Commit and Push

##### Commit

Make any change on the branch `mybranch` then build and test your codes.  
Include in what will be committed:
```sh
$ git add <file>
```
Commit your changes:
```sh
$ git commit
```
Enter your commit message to describe the changes. See the tips for a good commit message at [here](https://chris.beams.io/posts/git-commit/).  
Likely you go back and edit/build/test some more then `git commit --amend`

##### Push

Push your branch `mybranch` to your forked repo on github.com.
```sh
$ git push -f $remotename mybranch
```


### 6. Create a Pull Request

- Go to your fork at https://github.com/$user/kubernetes
- Hit the button ![PR](/static/img/github/compare-pullrequest) next to branch `mybranch`
- Flow the following processes to create a new pull request
