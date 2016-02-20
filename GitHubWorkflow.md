# GitHub Workflow
This document provides typical workflow of GitHub from fork to pull request.

## Overview
1. Fork from the original repository and make your own repository
2. Clone your repository to local developing environment
3. Make new branch for editing
4. Edit and commit to the local repository
5. Push to your repository (remote repository)
6. (Synchronize your remote repository and the original repository)
7. (Pull request to the original repository)

### 1. Fork from the original repository and make your own repository

### 2. Clone your repository to local developing environment

```
$ git clone {URL of your repository}
```

(Example) clone Masa's spark repository

```
$ git clone https://github.com/masatoshihanai/spark.git
```

### 3. Make new branch for editing
```
$ cd {local repository}
$ git branch {new branch name}
$ git checkout {new branch name}
```

(Example) make ***dev*** branch
```
$ cd spark
$ git branch dev
$ git checkout dev
```

### 4. Edit and commit to the local repository

```
$ git add {add file}
$ git commit -m "{comment}"
```

(Example) commit dynamic vertex add/delete
```
$ git add .
$ git commit -m "dynamic vertex add/delete"
```

### 5. Push to your repository

```
$ git push origin {new branch name}
```

(Example) push the **dev** branch to your remote repository in GitHub

```
$ git push origin dev
```
### 6. Synchronize your remote repository and the original repository
First, pull **the original repository** (named **upstream**) to local (master branch)
```
git remote add upstream {URL of the original repository}
git pull upstream master
```
Second, update(merge or rebase) **the new branch in local**
```
git checkout {new branch name}
git merge master
```
And then, push **the new branch in local** to **your remote repository**.
```
git push origin {new branch name}
```

### 7. Pull request to the original repository in GitHub
