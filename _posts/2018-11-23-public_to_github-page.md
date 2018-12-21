---
title: How to public your posts to github-page?
tags: [tutorials]
keywords: tutorials, git, github
last_updated: November 23, 2018
sidebar: mydoc_sidebar
permalink: public_to_github-page.html
folder: mydoc
---

1. **Fork** the repo https://github.com/vietkubers/vietkubers.github.io to your Github account
2. **Clone** the above forked repo to your local directory
```
$ git clone https://github.com/$USER/vietkubers.github.io.git
```
3. **Create** a [markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) file in folder [_post](https://github.com/vietkubers/vietkubers.github.io/tree/master/_posts) and name it as `yyyy-mm-dd-name-of-article.md`
```
$ cd vietkubers.github.io.git/_post
$ touch yyyy-mm-dd-name-of-article.md
```
4. **Modify** the created post with declaration section as:
```
---
title: Deploying multiple nodes with Kubeadm
tags: [linux, tutorials, kubernetes]
keywords: linux, networking
last_updated: November 21, 2018
sidebar: mydoc_sidebar
permalink: yyyy-mm-dd-name-of-article.html
folder: mydoc
---
```
Please refer to [this example post](https://raw.githubusercontent.com/vietkubers/vietkubers.github.io/master/_posts/2018-11-21-deploying-multiplenodes-with-kubeadm.md) for more details.  
5. **Commit**, **push** and create your **Pull Request**  
For the step-by-step process, refer to this article [1]

#### Reference

[1] https://vietkubers.github.io/linux/tutorials/kubernetes/2018/11/16/github-workflow.html  



*Author: [truongnh1992](https://github.com/truongnh1992)*
