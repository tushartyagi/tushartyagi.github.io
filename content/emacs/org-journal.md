Title: Writing a journal in emacs
Date: 2015-04-13
Category: Emacs
Tags: emacs, org-mode
Slug: writing-a-journal-in-emacs
Authors: Tushar Tyagi
Summary: How to start writing a journal in emacs.

# Journal in Emacs #

## My Story ##

I have trying to learn a bit more about emacs as well, it has always drawn me
towards itself -- when I started using it, maybe it was the old school appeal
which this editor had. Sure, Vim had it too but using HJKL for moving the
cursor around and pressing double escape didn't appeal to the kid me.
`C-x C-f` to open a file, `C-x C-c` exited the editor -- how cool was that.
I know this isn't the best of rating and differentiating about the two great
editors, but I chose Emacs and was hooked for some stupid reason.

In the last few days, I've started messing around with the idea that I should
start writing a bit of emacs lisp to extend the power, and to mould the editor
in the way I want it to work. That quickly turned into looking around on
the internet how to go about it, and soon I came across org-mode.

While I still have a lot to learn about the features of that mode, the first
thing I wanted to do was to start taking notes in that. People use it for a
lot of complex tasks: a TODO list for instance, but I'm not *that* hooked to
emacs. I still use Trello for creating my TODO list. Maybe someday when I'll
start living more than half of my working day in emacs, I'll start using it for
so many other purposes as well.

## And we move ahead! ##
The first thing in order to create a journal is to create an org file. These
usually end with an extension of `org`. So place these at some convinient
location, mine is placed in `~/.org/journal.org`.

The journalling is done by using `org-capture` mode which allows us to quickly
capture notes and ideas. These captured elements can be of multiple types;
what types? That you have to decide: Todo list, Journal, Appointment,
Project-ideas, etc. anything which has a form of sometype can be used.

Specifically to journalling, we'll start what is called a *capturing template*
because we want each of our journal entry to have some fixed structure.
Remember that this structure (hence the template) is supposed to be present for
almost everything: Todo lists have predefined structure, Appointments have one,
and so forth.

For the above two things, let's make some changes in our `.emacs` file:

* require org

		(require 'org)

* Associate files ending with `.org` (and in my case `.txt`) with org-mode.

		(add-to-list 'auto-mode-alist '("\\.\\(org\\|txt\\)" . org-mode)) 

* Setup a global keybinding for capturing notes (this isn't set by default):

		(global-set-key (kbd "C-c c") 'org-capture)

* Setup `org-capture-templates` which tells us what to capture for our entry:

		(setq org-capture-templates
			'(("t" "Todo" entry (file+headline "~/.org/inbox.org" "Inbox")
				"* TODO %?\n  %i\n  %a")
			  ("j" "Journal" entry (file+datetree "~/.org/journal.org")
				"* %?\nEntered on %U\n  %i\n  %a")))

Now the template has a fixed structure (in-depth treatment can be found
in the official docs
[here](http://orgmode.org/manual/Template-elements.html#Template-elements):

	(key description type target template)

**Key** is the key which you'll press after `C-c c` (or whatever you bound
`org-capture` to. In the above example, pressing `j` will create a journal
entry.

**Description** is the description of the template which comes up when you
press `C-c C-c` chord.

**Type** is the type of entry you want in the org-file, it can be:

* entry
* item
* checkitem
* table-line
* plain

**Target** Where the item you're created using this template should go, there
are again multiple options you can pick, but `file+datetree` creates a datetree
inside the file that you give as the parameter of this template (journal.org in
this case).

**Template** defines the body of this new item. This follows escaped string
format which allows for a lot of functionality. More information can be
found in the official docs
[here](http://orgmode.org/manual/Template-expansion.html#Template-expansion):

To sum up, you'll need to enter the following lines in your emacs file:

	(require 'org)
	(add-to-list 'auto-mode-alist '("\\.\\(org\\|txt\\)" . org-mode)) 
	(global-set-key (kbd "C-c c") 'org-capture)
	(setq org-capture-templates
		'(("j" "Journal" entry (file+datetree "~/.org/journal.org")
		"* %?\nEntered on %U\n  %i\n")))

Then while working in org mode, press `C-c c`, you'll get a new buffer with
the options (which as of now only contains *[j] Journal*), press j, a new
entry pops up, enter something and save! Congrats, you've created your first
journal entry using org-mode.

