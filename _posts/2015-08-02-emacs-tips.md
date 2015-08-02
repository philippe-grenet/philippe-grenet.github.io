---
layout: post
title: "Emacs Tips"
date: 2015-08-02
---

In this short post we'll review a few features of Emacs that are not well
known, but really worth knowing.

### Repeating commands

The simplest way to repeat a command is to type Control-a number, then the
command. For example if I type `C-6 C-0 ~` (that's 6 then 0 with the Control
key down, then the tilde character), I get a line with 60 tilde:

```
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

Another way is to use the *universal argument*: we saw how to write an
interactive Lisp function that accepts a numeric argument in the
[previous article](/2015/07/10/lisp-part-2). `forward-line` is such a function;
it goes down N lines (by default one line). For example `C-u 1 0 DOWN`
(Control-u then 1 then 0 then the down arrow key) goes down 10 lines. You can
do the same by calling the function explicitly with `C-u 1 0 M-x forward-line`.

Note that using the universal argument does not work for all commands, but when
it works it is generally faster: you call the command one time with an argument
N instead of calling it N times with no argument.

### Registers

Registers are used for saving temporary data in the current session. They can
store positions or bookmarks, text, numbers etc. Registers have a name: `a` to
`z`, `A` to `Z` (names are case-sensitive) and `1` to `9`, which gives you a
total of 62 registers. That's more than enough.

`C-x r` is the prefix for all register operations. Generally you type this
prefix, then a key to specify what operation you want (e.g. save or read), then
the register name which is one extra key.

#### Buffer positions

For example, open any file you have in one of your projects, go to some
position in the file, and hit `C-x r SPC a` (where `SPC` is the space
bar). This command stores the buffer position in register `a`. Now go to a
different buffer such as the scratch buffer, and type `C-x r j a`: this command
will jump back to the buffer and the position you just stored in register `a`.

Note that if you close the file and then retype the same command, Emacs will
ask you if you want to reopen the file, and it will bring you back to the same
position again. This can be useful if you want to keep bookmarks in a large
project, for example if you keep going to the same files and bits within these
files, but you don't want to keep them open all the time.

You can view the content of register `a` with command `M-x view-register` then
`a` at the prompt. It will show a window saying that this register stores a
position in your buffer (note that the position is a number, which represents
the offset from the beginning of the buffer). Type `C-x 1`, or `q` in the other
window, to dismiss it.

#### Bookmarks

A bookmark is similar to a buffer position but with a twist: you give it a name
and it is persisted between Emacs sessions. Note that bookmarks are not
actually associated to registers, but they use the same `C-x r` prefix.

The command is `C-x r m` (mark) which prompts for a name. By default it
proposes the current buffer name but you can choose whatever you want. You can
view the list of bookmarks with `C-x r l` (list). Just click on a bookmark to
jump to it. You can also jump to a bookmark with `C-x r b` (bookmark) which
prompts for the name using auto-complete. The nice thing about bookmarks is
that they are saved on the filesystem when you exit Emacs, and they are
available when you restart it. You can also force save with `M-x
bookmark-save`.

#### Text

Sometimes you want to save a snippet of text somewhere, so you can paste it
later. One way is to use the *kill ring*: `M-w` to save the selected text
(which is what Emacs calls the *region*), then `C-y` to paste. The kill ring
saves all copy operations you have done so far, so if you want to paste the
second previous thing you copied, type `C-y` followed by `M-y` (repeat `M-y` to
go back in history).

Another way is to save the region in a register, which you do with something
like `C-x r s a` (save in register `a`). Now if you want to insert the content
of a register at the current point, type `C-x r i a` (insert the content of
`a`).

#### Windows

This is less useful than the above, but you can also save a window
configuration in a register. For example, split the screen horizontally with
`C-x 2`, then split the current window vertically with `C-x 3`. You now have 3
windows displayed, which you can resize as you see fit.

Let's save this layout in register `a` with `C-r w a`. Now dismiss all other
windows than the current one using `C-x 1`. If you want to restore the layout,
type `C-r j a` (it is the same key for jumping to a buffer position). Note that
this only saves window configurations and not buffers content: if you close one
of the buffers Emacs will not reopen it for you.

#### Summary

The table below summarizes the keys we just learned.

Key binding          | Description
---------------------|---------------------------------------------------------
`C-x r SPC a`        | Store the current position to register `a`.
`C-x r j a`          | *Jump* to the position stored in register `a`,<br> or restore the window positions stored in register `a`.
`C-x r s a`          | *Save* the selected region in register `a`.
`C-x r i a`          | *Insert* the text saved into register `a`.
`C-x r w a`          | Save *window* positions in register `a`.
`C-x r m`            | Save a book*mark*.
`C-x r b`            | Go to a *bookmark*.
`C-x r l`            | *List* bookmarks.

### Macros

Editing macros are a very powerful feature of Emacs. After all, Emacs stands
for "Editing Macros" [^fn-macros].

There are several keys for macros, but really you only need to remember two of
them: `F3` and `F4`. The first one records a new macro. The second one
terminates the recording, if you were recording a macro; otherwise it executes
the macro. If you make an mistake while recording a macro, hit `C-g` to abort,
and start over.

Let's take an example. Suppose I have this text:

{% highlight text %}
the quick brown fox
jumps over
the lazy dog
{% endhighlight %}

Now suppose I want to make each line start with a capital letter and end with a
period. I could edit the text manually because it is only 3 lines, but just
imagine that it is much longer for argument's sake, in order to make the use of
a macro more compelling.

The way to do this with a macro is simple: fix the first line, while recording
a macro. Then execute the macro N times, one time per remaining line. To record
the macro do the following:

- Move the cursor to the beginning (e.g. `M-<`).
- `F3` to start recording.
- `M-c` to capitalize the first word (`The`).
- `C-e` to go to the end of the line.
- `.` to insert a period. Now the first line is good.
- `C-a` to go to the beginning of the line (where we started), and the down
  arrow to go to the next line.

Now type `F4` to stop recording. Then `F4` again to run the macro on the second
line. Then `F4` again to run the macro on the 3rd line. You're done!

{% highlight text %}
The quick brown fox.
Jumps over.
The lazy dog.
{% endhighlight %}

You could also run the macro N times using Control-a number then `F4`, as we
saw earlier. You can also apply the macro to a whole region by selecting the
region and running `M-x apply-macro-to-region-lines`, which is neat.

If you want to see macros at their best (and incidentally Emacs humiliating Vim
at its own game), check out this
[quick video](http://emacsrocks.com/e02.html). The entire series of Emacs Rocks
is worth watching.

-----
That's it for today. Lots more to come. Stay tuned!

[^fn-macros]: Note that Emacs *editing* macros have nothing to do with Lisp macros: one is a trick to save a sequence of keys and repeat it, the other is a Lisp function that executes twice, at compilation time and at run time.
