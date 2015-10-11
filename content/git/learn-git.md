Title: Git Notes
Date: 2015-01-15
Category: git
Tags: git, deep-dive
Slug: git-notes
Authors: Tushar Tyagi
Summary: A blog post listing the steps of doing various operations in git,
from creating a repo to doing other things.


### Creating a repo

#### On the server ####
We start with sshing the server, and creating a bare git. Why bare? In order to
prevent overwrites from the local repos.

		$ ssh user@hostname
		$ git init --bare my-project.git 

github doesn't support shell access, so we have to use curl in order to create a
repo from command line:

	    # Taken from http://stackoverflow.com/a/10325316
	    $ curl -u 'USER' https://api.github.com/user/repos -d '{"name":"REPO"}'
		# Then clone the repo at the client, as shown below
		
#### On the client ####

	    $ git init my-project.git
		# Add remote, and push after making changes
		$ git remote add origin git@github.com:USERNAME/my-project.git
		$ git push origin master

### Cloning a repo ####
 
	    $ git clone path-to-repo.git

### Checking out a branch ###
Then there are branches of the repo, which are versions of the project. A 
project can have as many branches as required, where generally each branch adds
a new feature to the project. We can select, or create a new branch using the 
`checkout` command:

        $ git checkout branch_name  # This selects an already present branch
        $ git checkout -b branch_name # Creates a new branch and selects it

### Git File Structure ###
Every git repo has a hidden directory called .git which stores the metadata, 
and has the compressed object database. When a repo is cloned, this folder is 
copied from the remote and a local copy of the files and database is created.

When we checkout a branch, or clone a remote repo, we get what is called as
the `working directory` which contains all the files which we can modify. These 
files have been committed prior to them being available here.

Once we have a local copy of the repo, we can start making changes and, 
if required, save the changes in the database. This is usually a two step 
process, where we first add the modified files in the `staging area`.

The staging area is generally contained in your Git directory, and stores 
metadata about the files. It contains all the changes that will be pushed to 
the database during the next commit. In order to check what all changes have 
been done, we can use the `diff` command which lists the differences between 
working area and staging area.

        # lists the changes between staging area and working dir.
        # If all the changes are staged, this gives nothing
        $ git diff  
        # lists changes between staging area and last commit, use anyone:
        $ git diff --staged 
        $ git diff --cached

All the files in a git repo can be either in the tracked state, or untracked 
state. The status of the files can be checked using `status` command:

        $ git status
        On branch master
        Your branch is up-to-date with 'origin/master'.

        Changes not staged for commit:
          (use "git add <file>..." to update what will be committed)
          (use "git checkout -- <file>..." to discard changes in working directory)

                modified:   modified_file.temp

        Untracked files:
          (use "git add <file>..." to include in what will be committed)

                x.temp

        no changes added to commit (use "git add" and/or "git commit -a")

Further, every tracked file in the git repo is in one of the three stages:
* Modified: When the files have been changed but, if you commit right now these 
    changes will not be saved in the local database. 
* Staged: The files are changed, and the changes have been acknowledged. If you 
    commit right now, the changes will go to the local database. This is also 
    called the `index` of the repository. A file or directory can be staged 
    using `git add <file|directory>`
* Committed: The files are committed, i.e. all the changes that have been done 
    are now saved in the local database. 

        $ git commit 
        $ git commit -m 'message for this commit'
        $ git commit -a -m 'message for this commit'  # skips the staging area


### Removing files ###
There are two roads that we can take here:

* Using `rm <filename>` to delete the file from the working directory. The 
    change is noted by git, but has to be staged manually using 
    `git add <filename>`.
* Using `git rm <filename>` to remove the file from the working directory and 
    index, and stop tracking it. `git rm` removes the files and stages the 
    changes. These just need to be committed.
* Using `git rm --cached <filename>` to remove it from the index but keep the 
    local file. Since the file is present in the working directory, it will 
    appear in the list of untracked files. A commit is needed. 


### Renaming a file ###
Git supports git mv which can help in renamaing a file. If we are using system 
`mv`, much like `rm`, we have to manually stage the changes. `git mv` is a 
shortcut for:

        $ mv <filename-old> <filename-new>
        $ git rm <filename-old>
        $ git add <filename-new>

### Seeing the differences ###
`git log` lists out all the commits that have been made. using `-p` option 
lists commit's info as well. using `--stat` gives information about insersions, 
and deletions in the file.

### Undoing ###
`git reset HEAD <filname>` will unstage the file which has been accidently 
staged in the index.

`git checkout -- <filename>` will write over the file with the previous version 
of the file. Since the <filename> file was not apparently committed (since it 
was on the index), DON'T use this to restore the older version if you don't 
want to lose the changes.

### Working with Remotes ###

`git remote add [shortname] [url]` and then use `git fetch shortname` 

`git push [remote] [local]` => `git push origin master` In order to push, we 
have to pull their work first. There cannot be successive pushing between 
different people without any pulling.

### Branching ###

Before we start with detail of branching, let's take a sidestep in order to 
understand a few concepts.

Git doesn't store each and every file that is changed from one commit to 
another, but it only creates a snapshot of the changes, i.e. the content of the 
repository at the moment of the commit.

When a commit is done, roughly we have the following happening:
* Every file is checksum-ed and stored as a blob object
* The directory tree is created, with checksums of the child files and 
    directories. It's checksum is also calculated.
* Commit object stores the commit metadata, as well as the checksum of the 
    current directory tree. This metadata stores the committer data, and the 
    checksum of the parent commit. 

So we can get the current directory snapshot from any commit object using the 
stored checksum, move to that and get the directory structure using the stored 
hashes of the child files and directories.

Coming back to branching, we can create a new branch using the following 
command: 

        # Creates a new branch, but doesn't check it out
        $ git branch <branch-name>  

        # Creates a new branch and checks it out as well
        $ git checkout -b <branch-name>

While the new branch is created, we are not using it if we used the first 
command. In order to start working on the new branch, use `checkout`.

        $ git checkout <branch-name>


Also with every branch, we have what is called the HEAD pointer, which points 
to the current working branch. If we change the branch, we simply move HEAD to 
point to the new branch. 

The current status of HEAD can be checked using the `--decorate` parameter 
(`--all` checks out all the branches, and `--oneline` prints only one line 
commit message of every commit):

        $ git log --decorate --all --oneline

When, on the current branch, we create a new commit, HEAD moves to the new 
commit (storing the previous commit's hash in the metadata, as already 
discussed).

We can list out all the branches using `git branch` command. Passing `-v` lists 
the last message of each commit as well.

        $ git branch -v 

Using `--merged` and `--no-merged` parameters will show the branches which have 
been and have not been merged to the current branch. Those which are already 
merged can be safely deleted; git won't let you delete the branches listed by 
`--no-merged` parameter.

### Merging ###

Once we are done with a branch, we have to add it back to the main codebase. 
This is process called merging. 

Let's take an example where we need to merge the changes of some branch called 
feature12 into the master branch. It is done in the following manner:

* Checkout the branch where the merge is going to be done, in this case 
`master`.

        $ git checkout master

* Use `git merge <branchname>` to merge the changes:

        $ git merge feature12

Now there are two ways of merging branches, depending on the structure of the 
branch tree. 
1. If master is a direct ancestor of the current branch, git uses 
    fast-forward strategy where the pointer of the master branch (in our case) 
    will be moved to the new branch.
2. If master is on some parallel path from the new branch, and they share a 
    common ancestor (they will have to :)), then git merges the changes in the 
    common ancestor and creates a new commit out of this merge. 
 
### Remote Git ###
First when a `git clone` command is used, git creates a local repo for you to 
work on. The master is the repo that you usually work on, unless the name is 
changed. Even on the remote server there is a branch called master (unless a 
different name is used). After clone, the master of remote is available as 
`origin/master`. 

Every change that we do the local repo takes us away from `origin/master`. If, 
in the meantime, data on the server has also changed, we use the `fetch` 
command to update the local repo.

        $ git fetch origin
        $ git fetch <remote-name>

fetching doesn't allow us to make changes since the pointer still points to the 
remote branch. In order to start working on this, we need to merge the new 
branch with our local branch, or create a new branch entirely:

        $ git merge origin/branchname 
        $ git checkout -b origin/branchname
        $ git checkout -b <local-alias> origin/branchname

`git push <remote> <local>` is used to push the changes that you've done in the 
local branch into the remote branch, which is a shortcut to:

        $ git push <remote-name> localbranch:remotebranch

        $ git branch -vv
        iss53     7e424c3 [origin/iss53: ahead 2] forgot the brackets
        master    1ae2a45 [origin/master] deploying index fix
        * serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] this should do it
        testing   5ea463a trying something new

ahead n means we have n changes that we haven't pushed to the server.
behind n means we have n changes that we haven't merged from the server.
