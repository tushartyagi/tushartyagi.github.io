Title: HTTPS to SSH
Date: 2015-03-03
Category: git
Tags: git, basics
Slug: https-to-ssh
Authors: Tushar Tyagi
Summary: Changing the remote url to ssh since it's easier after it's set up.

One of the biggest pain-points that I've faced after using HTTPS urls for
remote repos is that I have to keep providing the credentials after every push.
It's okay when you push once or twice in a day, but anything more than that
becomes frustrating really fast, especially if the password is like [correct
horse battery staple](https://xkcd.com/936/).

After changing in repo's directory, using `git remote -v` prints the urls
we're using

	:::bash
	$ git remote -v
	origin	https://github.com/tushartyagi/tushartyagi.github.io.git (fetch)
	origin	https://github.com/tushartyagi/tushartyagi.github.io.git (push)


The ssh url is of the form "git@provider:username/reponame.git" which, for the
above repo becomes: `git@github.com:tushartyagi/tushartyagi.github.io.git`

Using `git remote set-url` command, we provide the new url for the branch.
Since this is a
[userpage](https://help.github.com/articles/user-organization-and-project-pages)
and these are hosted in origin branch, we will change the remote url of origin
branch:

	:::bash
	$ git remote set-url origin git@github.com:tushartyagi/tushartyagi.github.io.git

The changes will be done, use the `remote -v` command to verify:

	:::bash
	$ git remote -v
	origin	git@github.com:tushartyagi/tushartyagi.github.io.git (fetch)
	origin	git@github.com:tushartyagi/tushartyagi.github.io.git (push)

Now enjoy the convinience of using ssh -- entering a password only a single
time. 
