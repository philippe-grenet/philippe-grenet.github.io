---
layout: post
title: "Org mode part 2"
date: 2015-08-16
---

This is the second article in a series about Emacs Org mode. It assumes that you read
[the first one](/2015/07/03/org-mode-part-1).

Today we'll talk a bit about
[literate programming](https://en.wikipedia.org/wiki/Literate_programming). The
idea is to write an Org mode document that includes snippets of code, and
possibly the result of their execution as well. It kind of reverses the way you
think about documenting code: instead of adding comments inside your program,
you add your program's code inside the documentation file. In the end the
result is the same, you still get executable code.

Org mode includes a feature named
[Babel](http://orgmode.org/worg/org-contrib/babel/intro.html) which allows for
embedding code in any programming language you fancy. Let's get started.

### Enabling Babel

First, we need to make sure that Babel is enabled for a few other languages
than ELisp (the only one by default). Add the following in your emacs
configuration and evaluate it with `M-C-x` (or restart Emacs):

{% highlight lisp %}
(org-babel-do-load-languages
 'org-babel-load-languages '((emacs-lisp . t)
                             (ruby . t)
                             (python . t)
                             (sh . t)))
{% endhighlight %}

### Hello World

First open a new buffer `literate.org`, with this content:

{% highlight python %}
#+TITLE: Literate Programming

#+begin_src python
def hello(str):
    return "Hello, " + str + "!"

return hello("Dude")
#+end_src
{% endhighlight %}

The `#+TITLE` directive is just to add a title to the Org file; it is not
really needed. The other directives we use are `#+begin_src` and `#+end_src`
which respectively introduce and close a code block for a particular
language. Note that Emacs automatically applies the proper syntax highlighting.

Type `C-c C-c` to evaluate the code block under the point. After confirmation,
Babel will insert a new block `#+RESULTS` with the result of the evaluation,
like so:

![Org-mode20](/assets/org-mode20.png)

What happened here? Emacs forked a process to execute the code using Python,
and got back the returned value, and inserted it into the buffer.

By default the result is the returned value i.e. the last expression that was
executed, but you can also use the output of the program as the result with a
directive like: `#+begin_src python :result output`. For example, evaluate the
shell statement below:

{% highlight sh %}
#+begin_src sh :results output
   echo "Hello $USER! Today is `date`"
#+end_src
{% endhighlight %}

![Org-mode21](/assets/org-mode21.png)

### Calling code blocks

Babel also allows you to refer to code blocks from elsewhere in your document,
by labeling each block with a name. Let's say we have some Ruby code to revert
a string (yes, I know we could use the native `reverse!`):

{% highlight ruby %}
#+name: reverse_str
#+begin_src ruby
def reverse_str(s)
  len = s.length
  reversed = ""
  for i in 1..len do
    reversed += s[-1*i]
  end
  return reversed
end

reverse_str(str)
#+end_src
{% endhighlight %}

We can now call this block. Note that we get the result of the block
evaluation. So if you want to use the result of a function in the block, you
also need to add the call to that function (see the last line).

To call the block, add a `#+call` directive referring to the block's name and
binding the input variable `str`, and type `C-c C-c` with the point onto it:

{% highlight text %}
#+call: reverse_str(str="The Quick Brown Fox")
{% endhighlight %}

![Org-mode22](/assets/org-mode22.png)

### More fun

Here is a more practical example to illustrate the power of Babel, using
different languages to get the job done. This is actually similar to an iPython
notebook.

Suppose I want to produce a report containing statistics about the words that
are used in a collection of files. Assuming all these files are in the same
directory (let's say they are Markdown files), this shell script counts the
number of unique words in each file:

{% highlight sh %}
#+name: words
#+begin_src sh
  for F in /Users/phil/Documents/philippe-grenet.github.io/_posts/*.md
  do
     cat $F | tr -cs A-Za-z '\n' | tr A-Z a-z | sort | uniq -c | wc -l
  done
#+end_src
{% endhighlight %}

The result has multiple lines, so it is stored into an Org table:

![Org-mode23](/assets/org-mode23.png)

Now let's compute the standard deviation. We could follow up with another shell
script, but it is easier to do this in ELisp. Add this block in the buffer:

{% highlight lisp %}
#+begin_src elisp :var samples=words
(defun standard-deviation (samples)
  (let* ((samples    (mapcar #'car samples))
         (n          (length samples))
         (average    (/ (reduce #'+ samples) n))
         (deviations (mapcar #'(lambda (x)
                                 (expt (- x average) 2))
                             samples)))
    (sqrt (/ (reduce #'+ deviations) n))))

(standard-deviation samples))
#+end_src
{% endhighlight %}

The `#+begin_src` directive uses the `:var` keyword to bind the variable
`samples` to the output of the `words` code block (which is the table
above). In Lisp that `samples` variable will be set with a list of
lists. Basically each row in the table is a sublist, and since the table has
only one column the sublists only contain a single number, like this:

{% highlight lisp %}
'((55) (274) (560) (296) (650) (394) (253))
{% endhighlight %}

If you evaluate the block with `C-c C-c`, you get the final result:

![Org-mode24](/assets/org-mode24.png)

You can of course re-run the report anytime you want. Once we have our report
ready, Org mode is able to export it into plenty of different formats, choosing
exactly what should be included and excluded. But that's a discussion for
another day.
