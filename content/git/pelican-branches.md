Title: Backing Up Site When Using Pelican and github pages
Date: 2015-03-22
Category: git
Tags: git, pelican
Slug: back-up-site-structure
Authors: Tushar Tyagi
Summary: Pelican uses ghp-import to push data in the master branch, in here I'm saving the whole site structure in a backup branch

# Backup the master branch of Pelican

Pelican is an excellent blog generator, write your posts in Markdown/RST and
whoosh, post them online. There are many themes and plugins available.

Combine this with github userpages and you have a fully functional, and really
fast blog available, for free, with all your blog posts in plain text (ok,
markdown), there are no databases losses, and site is overall fast.

Anyhow, as per pelican [documentation](http://docs.getpelican.com/en/3.5.0/tips.html#user-pages)
in order to publish the pages on github, we have to use `ghp-import` which creates
a new branch `gh-pages` locally and we have to push this to the master branch
using these three commands:

	:::bash
	$ pelican content -o output -s pelicanconf.py
	$ ghp-import output
	$ git push git@github.com:tushartyagi/tushartyagi.github.io.git gh-pages:master

This was working fine for some time, but then one day my laptop died, I didn't
save the article, and didn't push it on master. :(

So I created a new branch, called backup, and started pushing the data of
master on this new branch. Since I'm also modifying the theme on a constant
basis these days, this data needs to have a backup somewhere, it's now on
github.

	:::bash
	$ git checkout -b backup
	$ git add .
	$ git commit -m 'A new branch for backups'
	$ git push git@github.com:tushartyagit/tushartyagi.github.io.git master:backup
