---
layout: post
title: "My fancy Zsh prompt"
date: 2016-06-11
---

I switched from `bash` to `zsh` a few years ago and I am never looking back! It
has awesome tab completion: for example `cd doc/sh/pl` becomes `cd
Documents/shared/planning`. It also expands git commands and branches,
environment variables and other things. It has extended file globbing which
provides a good replacement for `find`. If you spend a lot of time in the
shell, you should really give `zsh` a try.

Whatever shell you use, it's worth spending a few minutes configuring your
prompt. The prompt is something you will see literally thousands of times a
day. So why not make it useful?

Here is mine. First, let's open a new shell in the home directory:

![zsh prompt](/assets/zsh_prompt1.png)

Simple and to the point. `~` sweet `~`! Let's go into some directory:

![zsh prompt](/assets/zsh_prompt2.png)

It displays the path, from the home directory, with the current directory
displayed in bold ("planning"). To keep the prompt short, we only display the
last 3 subdirectories:

![zsh prompt](/assets/zsh_prompt5.png)

And now let's move into a git repo:

![zsh prompt](/assets/zsh_prompt3.png)

The prompt displays the path starting at the root of the repo ("org"), and also
the current branch in green ("master"). Now let's move into a subdirectory in
the same repo:

![zsh prompt](/assets/zsh_prompt4.png)

The root of the repo is highlighted ("org"), and the current directory is
displayed in bold like before. We could also display something indicating if
the repo is clean or if it has uncommitted changes, but I prefer to keep it
simple.

The colors are from the
[Tomorrow Night](https://github.com/chriskempson/tomorrow-theme) theme, which
also happens to be the default theme in
[Exordium](https://github.com/philippe-grenet/exordium).

Here is the code (in `~/.zshrc`):

{% highlight python %}
# Zsh options
setopt prompt_subst
autoload -U colors && colors

# Colors
BLACK=$'\033[0m'
GREEN=$'\033[38;5;148m'
BLUE=$'\033[38;5;117m'
DARK_BLUE='\033[38;5;4m'

current_git_branch() {
    git rev-parse --abbrev-ref HEAD 2> /dev/null | sed -e 's/\(.*\)/(\1)/g'
}

current_directory() {
    PROMPT_PATH=""
    CURRENT=`dirname ${PWD}`
    if [[ $CURRENT = / ]]; then
        PROMPT_PATH=""
    elif [[ $PWD = $HOME ]]; then
        PROMPT_PATH=""
    else
        GIT_REPO_PATH=$(git rev-parse --show-toplevel 2>/dev/null)
        if [[ -d $GIT_REPO_PATH ]]; then
            # We are in a git repo. Display the root in color, then the path
            # starting from the root.
            if [[ $PWD -ef $GIT_REPO_PATH ]]; then
                # We are at the root of the git repo.
                PROMPT_PATH=""
            else
                # We are not at the root of the git repo.
                BASE=$(basename $GIT_REPO_PATH)
                GIT_ROOT="%{$fg_bold[red]%}%{$DARK_BLUE%}${BASE}%{$reset_color%}"
                REAL_PWD=$PWD:A
                PATH_TO_CURRENT="${REAL_PWD#$GIT_REPO_PATH}"
                PATH_TO_CURRENT="%{$BLUE%}${PATH_TO_CURRENT%/*}%{$reset_color%}"
                PROMPT_PATH="${GIT_ROOT}${PATH_TO_CURRENT}/"
            fi
        else
            # We are not in a git repo.
            PATH_TO_CURRENT=$(print -P %3~)
            PATH_TO_CURRENT="%{$BLUE%}${PATH_TO_CURRENT%/*}%{$reset_color%}"
            PROMPT_PATH="${PATH_TO_CURRENT}/"
        fi
    fi
    echo "${PROMPT_PATH}%{$reset_color%}%{$fg_bold[red]%}%{$BLUE%}%1~%{$reset_color%}"
}

export PROMPT=$'$(current_directory) %{$GREEN%}$(current_git_branch)%{$BLACK%}%# '
{% endhighlight %}

Notice the function `current_git_branch`: this is the fastest way I have found
to get the name of the branch. The trick is to make the execution of the prompt
as fast as possible, since it gets executed every time you hit Enter.
