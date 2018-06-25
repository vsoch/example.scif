---
title: Development
sidebar: main_sidebar
permalink: development
folder: docs
---


Once you have a production repository, meaning that it's passed tests, deployed a versioned container, and others are using
your software, you can't make changes and push to master willy-nilly. Instead, you will want to step through a careful
set of stages to add new features or make changes that might look like this:

 1. (optional) Start from / create an issue to document what you are working on. [Issues](https://guides.github.com/features/issues/) are cool because you can add groups of them to a larger goal (called a Milestone), assign one or more people to work on them, or even just make checklists or notes for yourself. It's really just an organizational strategy, but it also asserts your plan publicly so that others can see and add comments or (even better) contribute too!
 2. Checkout a new branch to work on a feature. Give it a meaningful name, like `add/samtools-tests`
 3. Develop your feature, commit the changes, and push to the feature branch
 4. Open a pull request to review the change (and have them automatically tested)
 5. Once testing passes, merge the pull request to integrate changes into the master branch.


These development steps will be discussed in detail here.

## Step 1. Checkout a new Branch
We are going to assume that you are collaborating on the code base with others, and in this case you would want to checkout branches to add features (and not push directly to master). You may not have done this when you were first setting up the repsitory (and it was just you, and there were no production images) so you were just pushing to master. For all other times we will be using <a href="https://guides.github.com/introduction/flow/" target="_blank">Branches</a> to organize our work. 
Branches are a way of isolating features, which makes review easier, but also lets you work on several things in parallel.
When a feature is ready to review to integrate into the master (production) branch, then you do what is called a <a href="https://help.github.com/articles/about-pull-requests/" target="_blank">pull request</a>) to be reviewed and added to the production branch. The cool thing about pull requests is that given some continuous integration is setup, the changes will be tested before you add them to the master branch (an action called a "merge"). Each repository, including your fork, has a main branch, which is usually called "master". As mentioned earlier, the master branch should always be considered the production version of your software, and the various feature branches updated to it. To checkout a new branch, you would do the following:

```bash
git checkout -b development
```

When on master, would be equivalent to doing:

```bash
git checkout -b development master
```

Which in human terms says "create a new branch from master called development."

As another example, let's say that you had a development branch and you wanted to create a new branch called `add/samtools-testing` from development.

```bash
# Checkout a new branch called add/samtools-testing
git checkout -b add/samtools-testing development
```

The addition of the `-b` argument tells git that we want to make a new branch. If I want to just change branches (for example back to master) I can do the same command without `-b`:

```bash
# Change back to master
git checkout master
```

Note that you should commit changes to the branch you are working on before changing branches, otherwise they would be lost. GitHub will give you a warning and prevent you from changing branches if this is the case, so don't worry too much about it.


## Step 2. Make your changes
On your new branch, go nuts! Make changes, test them, and when you are happy with a bit of progress, commit the changes to the branch:

```bash
git commit -a
```

This will open up a little window in your default text editor that you can write a message in the first line. This commit message is important - it should describe exactly the changes that you have made. Bad commit messages are like:

- changed code
- updated files

Good commit messages are like:

- changed function "get_config" in functions.py to output csv to fix #2
- updated docs about shell to close #10

The tags "close #10" and "fix #2" are referencing issues that are posted on the main repo you are going to do a pull request to. Given that your fix is merged into the master branch, these messages will automatically close the issues, and further, it will link your commits directly to the issues they intended to fix. This is very important down the line if someone wants to understand your contribution, or (hopefully not) revert the code back to a previous version.

If you are a pro, you can do this commit in one command with the `-m` flag, which means "message":

```bash
git commit -a -m "I added tests for samtools in test_samtools.py"
```

If you want to be **super** pro, you can actually **sign** your commits. Read about [how to do that here](https://help.github.com/articles/signing-commits-using-gpg/).

## Step 3. Push your branch to your fork
When you are done with your commits, you should push your branch to your fork (and you can also continuously push commits here as you work):

```bash
git push origin add/samtools-tests
```

Note that you should always check the status of your branches to see what has been pushed (or not):

```bash
git status
```


## Step 4. Submit a Pull Request
Once you have pushed your branch, then you can (in the GUI) <a href="https://help.github.com/articles/creating-a-pull-request/" target="_blank">submit a Pull Request</a> via the web interface. Generally, your PR should be submit to the master branch. This will open up a nice conversation interface / forum for the developers (or even just you to review), and will display the result of testing. Once you submit, the continuous integration that is linked with the code base will also be run. 

>> What if I need to make more changes after I open the PR?

If there are more changes needed, you can continue to push commits to your branch and it will update the Pull Request. You don't need to close it / open a new one.

## Step 5. Merge Away, Merill!

Once your tests have passed, and you are a happy camper... Merge Away Merrill!

![assets/img/merge.gif](assets/img/merge.gif)
