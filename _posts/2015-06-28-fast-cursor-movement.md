---
layout: post
title: "Fast cursor movement"
date: 2015-06-28
---

I've used Eclipse for many years; what always bothered me about it is that it
forces you to use the mouse for things like switching between
buffers[^fn-eclipse_shortcut]. Which is a very common operation: add an
argument to the definition of a function, switch to the file where it is
called, and change the function call. In fact my Emacs configuration sets up a
very easy [key](https://github.com/philippe-grenet/exordium#keymap) for
switching between the 2 most recently used buffers.

One of the many great things about Emacs (and Vim as well) is that you can do
everything you need without ever using the mouse, and in fact without even
requiring a GUI. This is a killer feature compared to most IDEs because menus
and mice are slow. If your hands don't have to leave the home row, you can
change text almost as fast as you think.

You can get more productive if you know how to move the cursor quickly within a
buffer. There are several clever [extensions](http://emacsrocks.com/e10.html)
for that, but here we'll review a few built-in keys. Note that I'm using the
arrow keys because they are easy to remember[^fn-arrow_keys].

### Beginning and end

These four keys are a must:

Keybinding           | Description
---------------------|---------------------------------------------------------
`C-a`                | Go to the beginning of the line.
`C-e`                | Go to the end of the line.
`M-<`                | Go to the beginning of the buffer.
`M->`                | Go to the end of the buffer.

### Move by words

Keybinding           | Description
---------------------|---------------------------------------------------------
`C-right`            | Move forward one word (`right-word`).
`C-left`             | Move backward one word (`left-word`).

Any major mode may have its own definition of what a word is (it is defined in
the the mode's *syntax table*).

Unfortunately these keys are not symetrical: moving right then left does not
necessarily bring you back where you started. For programming, I found it
useful to define these extra keys for moving by semantic units rather than
words:

{% highlight lisp %}
(define-key global-map [(meta right)]
  #'(lambda (arg)
    (interactive "p")
    (forward-same-syntax arg)))

(define-key global-map [(meta left)]
  #'(lambda (arg)
      (interactive "p")
      (forward-same-syntax (- arg))))
{% endhighlight %}


You can pass a numeric argument to these commands to move by more than a single
word. The *numeric argument* prefix lets you pass a number to a command, and
the prefix is `C-u` followed by the number followed by the command. For example
`C-u 3 C-right` moves forward 3 words.

### Move by paragraphs

Keybinding           | Description
---------------------|---------------------------------------------------------
`C-up`               | Move up one paragraph (`backward-paragraph`).
`C-down`             | Move down one paragraph (`forward-paragraph`).

### Move by s-expression

This is very handy for Lisp. In Lisp, an s-expression (symbolic expression or
sexp) is an atom or a list. For other programming languages, Emacs also
considers strings and blocs between curly braces or square brackets. Moving by
sexp is similar to moving by word, only the prefix is `M-C-`:

Keybinding           | Description
---------------------|---------------------------------------------------------
`M-C-left`           | Move forward one sexp (`forward-sexp`).
`M-C-right`          | Move backward one sexp (`backward-sexp`).

Give it a try, it is more useful than you think.

[^fn-eclipse_shortcut]: Eclipse has a shortcut key that displays a menu of the open files, but it is slow and combersome.

[^fn-arrow_keys]: Touch-type purists prefer to use other keys like `C-f` and `C-b`.
