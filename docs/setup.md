# Getting Started

Today you are going to be doing the following:

 - cloning this repository to your own Github username
 - adding your software to the repository
 - writing a recipe to install, run, and test your software
 - pushing to Github
 - connecting to Continuous Integration to test
 - upon successful testing, publishing your container

The first step, setting up the repository on your Github account, is discussed here.


## 1. Learn about Github

Github is a version control system, meaning that you can store local versions of code (we can refer to this as "local") and then also store a version with Github (called a "remote." We do this so that if your computer explodes, your code is backed up. We also do it for reproducibility. By way of saving a record of changes over time, you could easily revert to a previous change, or make earlier versions of your software available for use.  Github is also really great because it connects with several services that are going to help us by way of triggers called webhooks. 

The basic set of actions that you will run locally is to "commit," which is a statement that "I am happy with these changes and want to save them to my local repository" and then to "push," which says "take my local changes and push them to the remote repository on Github." This set of actions is part of the <a href="https://guides.github.com/introduction/flow/" target="_blank">GitHub Flow</a>, which is something that you should look over if you have never heard of it. 

## 2. Fork this repository
Github is also great because you can take someone else's code base (for example, this repository) and do an action called a "Fork." A fork will take some repository and make a copy of it to your branch. You would want to do this given that you are contributing to the software and want to work on it, or if you want to use the repository as a template. For our purposes, we will do a combination of the two. By forking this repository to your branch you can develop your own recipe and associated software, but keep a pointer to this repository (we call this the upstream) in case the underlying template ever changes (and you want to integrate these changes).  The forking operation is a button in
the upper right of the repository page, and cannot be done programatically. Once forked, you will want to clone the fork of the repo to your computer. Let's say my GitHub username is waffles, and I am using ssh:

```bash
git clone git@github.com:waffles/example.scif.git
cd example.scif
```

## Step 3. Set up your config
The GitHub config file, located at `.git/config`, is the best way to keep track of many different forks of a repository. If you didn't care about maintaining a link to the upstream (this repository) you could skip this step. But since we do, we instead want to edit the configuration file and add the upstream.  I usually open it up right after cloning my fork to add the repository that I forked as a <a href="https://help.github.com/articles/adding-a-remote/" target="_blank">remote</a>, so I can easily get updated from it. Let's say my .git/config first looks like this, after I clone my own branch:


```bash
      [core]
              repositoryformatversion = 0
              filemode = true
              bare = false
              logallrefupdates = true
      [remote "origin"]
              url = git@github.com:waffles/example.scif
              fetch = +refs/heads/*:refs/remotes/origin/*
      [branch "master"]
              remote = origin
              merge = refs/heads/master
```


I would want to add the upstream repository, which is where I forked from.


```bash
      [core]
              repositoryformatversion = 0
              filemode = true
              bare = false
              logallrefupdates = true
      [remote "origin"]
              url = git@github.com:waffles/example.scif
              fetch = +refs/heads/*:refs/remotes/origin/*
      [remote "upstream"]
              url = https://github.com/vsoch/example.scif
              fetch = +refs/heads/*:refs/remotes/origin/*
      [branch "master"]
              remote = origin
              merge = refs/heads/master
```

In the GitHub flow, the master branch is the frozen, current version of the software. If you are making changes or adding a feature, you would checkout a new branch. In the case that you want to update your master branch, you can do:

```bash
git checkout master
git pull upstream master
git push origin master
```

More instructions for Github are provided in the [Development](development.md) docs. This should be substantial for the initial setup of the repository. Since the entire thing is essentially under development, we will just be working with the master (default) branch, so you shouldn't need to worry about checking out new branches.
