---
layout: post
title: "Org mode part 1"
date: 2015-07-03
---

This is the first article in a series about Org mode.

Org mode is a killer feature of Emacs. Some people use Emacs just for that
mode. It can do many things including organizing notes, project planning, web
publishing and literate programming. You can even write your emacs
configuration in Org mode and publish it: here is
[an example](http://pages.sachachua.com/.emacs.d/Sacha.html) (to make it work,
you only need a tiny `init.el` that loads the Org file and runs the embedded
Lisp code).

The only bad thing about Org mode is that it is not universal, because it is
very tied to Emacs. There are plugins for Vim and Sublime Text for instance,
but they only cover a fraction of the features that the real thing
provides. This is the reason why Markdown is more popular than Org while being
objectively inferior. Although more and more sites understand Org files (GitHub
certainly does).

Let's get started.

### Outlines

An Org file is a plain text file with headlines, text, and some additional
information such as tags and timestamps. A headline starts with a series of
asterisks. The more asterisks there are, the deeper the headline is.

For example, you can create a file with extension ".org" and with this content:

{% highlight text %}
* Top-level headline
Some text under that headline.
** Second-level headline (child of the top-level headline)
More text.
*** Third-level headline (child of the second-level headline)
** Another second-level headline
{% endhighlight %}

You can make Emacs render this nicely with the
[org bullet](https://github.com/sabof/org-bullets) extension, which masks the
asterisks and displays Unicode bullets instead:

![Org-mode1](/assets/org-mode1.png)

When typing the text above, use `M-RET` (meta + return) to create a new
headline at the same level as the one above it, or a first-level headline if
the document does not have headlines yet. Use `M-LEFT` and `M-RIGHT` to promote
or demote a headline, e.g. change its level. You can also move a headline and
all the text under it up and down using `M-UP` and `M-DOWN`.

Finally the `TAB` key collapse or expand headlines. When a headline is
collapsed, its content is replaced with an ellipsis like so:

![Org-mode2](/assets/org-mode2.png)

`S-TAB` (shift + tab) collapses or expands everything.

### Lists and checkboxes

If you prefer, you can also create hierarchies using lists. For example:

![Org-mode3](/assets/org-mode3.png)

The same keys work with lists, e.g. use `M-RET` to create a new list item. You
can also change the style of your list using `S-LEFT` and `S-RIGHT`. For
example, change the list to use numbers:

![Org-mode4](/assets/org-mode4.png)

Notice that if you move an item up and down with `M-UP` / `M-DOWN`, the numbers
are automatically updated.

To create an item with a checkbox, use `S-M-RET` (shift meta return). Toggle
checkbox using `C-c C-c`.

![Org-mode5](/assets/org-mode5.png)

### TODOs

An alternative to checkboxes are TODO items:

![Org-mode6](/assets/org-mode6.png)

Type `S-M-RET` (shift meta return) to create a new headline that starts with a
TODO. Change the state of a TODO into DONE or vice versa using `S-LEFT` and
`S-RIGHT`.

You can add more states to TODO and
DONE. [Exordium](https://github.com/philippe-grenet/exordium) uses this code to
add the WORK and WAIT states:

{% highlight lisp %}
(setq org-todo-keywords
      '((sequence "TODO" "WORK" "WAIT" "DONE")))
{% endhighlight %}

You can also specify the states on a per-file basis by adding a line like this
at the beginning of the file (save and reopen to make it work):

{% highlight text %}
#+TODO: TODO WAIT | DONE CANCELED
{% endhighlight %}

The vertical bar separates the TODO keywords (states that need action) from the
DONE states (which need no further action).

### Markup

Org's markup syntax is more intuitive than the one of Markdown (IMO):

{% highlight text %}
- *Bold* word
- /Italic/ word
- _Underlined_ word
- ~code~ word
- URL: http://gnu.org or [[http://www.gnu.org/software/emacs/][GNU Emacs]]
- Images: [[/Users/pgrenet/Pictures/tux.jpg]]
{% endhighlight %}

You can make the images display inline using this code in your configuration
(reopen the Org file to make it work):

{% highlight lisp %}
(setq org-startup-with-inline-images t)
{% endhighlight %}

![Org-mode7](/assets/org-mode7.png)

### Tables

Finally the *pi&egrave;ce de r&eacute;sistance*: type this text:

{% highlight text %}
| Name | Phone | Age |
|--
{% endhighlight %}

Then hit `TAB` and see what happen. Voila! The table will automatically
resize itself as you tab and shift-tab to move between cells.
