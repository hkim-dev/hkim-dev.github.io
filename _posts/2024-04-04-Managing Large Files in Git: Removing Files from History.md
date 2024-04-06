---
title: "Managing Large Files in Git: Removing Files from History"
date: 2024-04-04 15:00:00 -0400
categories: "programming"
header:
  teaser: "/assets/images/Git-Logo-2Color.png"
tags:
  - Git
toc: true
---

# Background
Have you ever encountered an error while pushing changes to GitLab, stating that you're attempting to check in one or more blobs that exceed the 100.0MiB limit? The cause of the error itself is pretty self-explanatory, but how can we resolve it? In this article, we'll explore how to resolve this issue effectively.

```shell
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 100.82 KiB | 211.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
remote: GitLab: You are attempting to check in one or more blobs which exceed the 100.0MiB limit:
remote: 
remote: - 7e11e38cda9301485d0584b9accbe9128d9747c1 (101 MiB)
remote: To resolve this error, you must either reduce the size of the above blobs, or utilize LFS.
remote: You may use "git ls-tree -r HEAD | grep $BLOB_ID" to see the file path.
remote: Please refer to https://gitlab.com/help/user/free_push_limit and
remote: https://gitlab.com/help/administration/settings/account_and_limit_setting
remote: for further information.
To https://gitlab.com/hyemin-dev/large-file-test.git
 ! [remote rejected] main -> main (pre-receive hook declined)
error: failed to push some refs to 'https://gitlab.com/hyemin-dev/large-file-test.git'
```

# Resolving the Error

To resolve the issue of exceeding the blob size limit on Gitlab, follow these steps:

## 1. Identify the large file
As suggested in the error message, use the following command to identify the file path:
`git ls-tree -r HEAD | grep $BLOB_ID`

You'll get an output similar to the following:
```
100644 blob 7e11e38cda9301485d0584b9accbe9128d9747c1	path/to/file/large_file.json
```

## 2. Remove the file from history
Using the `git filter-branch` command, rewrite the revision history of your Git repository, allowing you to modify the files, commits, etc. 

```git filter-branch -f --index-filter 'git rm --cached --ignore-unmatch path/to/file/large_file'```

The `--index-filter` option specifies the filter to apply to the index of each revision. The filter is defined as the command `git rm --cached --ignore-unmatch path/to/file/large_file.json`. This command removes the specified file from the index. The `--cached` option is used to ensure that the file is only removed from the index and not from the working directory.

However, it is clearly stated in the latest documentation that the use of `git filter-branch` is not recommended:
> :warning: git filter-branch has a plethora of pitfalls that can produce non-obvious manglings of the intended history rewrite (and can leave you with little time to investigate such problems since it has such abysmal performance). These safety and performance issues cannot be backward compatibly fixed and as such, its use is not recommended. Read more about it [here](https://git-scm.com/docs/git-filter-branch)


Alternatively, we can use [git filter-repo](https://github.com/newren/git-filter-repo/). This is actually the history filtering tool Git recommends using. Install the tool first using your favorite package manager such as `pip` or `brew`. 
Install it by running
```
brew install git-filter-repo
```
or
```
pip install git-filter-repo
```
For a more detailed installation guide, visit this [link](https://github.com/newren/git-filter-repo/blob/main/INSTALL.md).


Run the `git filter-repo` command like the following:
```
git filter-repo --path path/to/file/large_file --invert-paths --force
```
The `--path` option specifies the path to the file you want to remove.



##### References
- https://stackoverflow.com/questions/33330771/git-lfs-this-exceeds-githubs-file-size-limit-of-100-00-mb
- https://git-scm.com/docs/git-filter-branch
- https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository