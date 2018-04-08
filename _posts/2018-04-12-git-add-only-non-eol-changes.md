---
layout: post
title:  "How to `git add` only non-EOL changes"
date:   2018-04-12 20:02:00 +08:00
categories: coding git
redirect_to: https://ferriplus.gitlab.io/posts/2018/04/12/git-add-only-non-eol-changes.html
---
Formatting and whitespace issues are frustrating and subtle problems that many developers encounter when collaborating, especially cross-platform. It's very easy for patches or other collaborated work to introduce subtle whitespace changes because editors "smartly" introduce them. And if your files ever touch a Windows system, their line endings might be replaced. 

Git has a few configuration options to help with these issues:

```console
git config --global core.autocrlf input
# Configure Git on OS X or Linux to properly handle line endings

git config --global core.autocrlf true
# Configure Git on Windows to properly handle line endings
```

But what if the above is not enough? The answer is a one-liner:

```console
git diff -U0 --ignore-space-at-eol --no-color | git apply --cached --ignore-whitespace --unidiff-zero -
```

A closer look at the above command:

1. `git diff` Generate patch.

    * `-U0` or `--unified=0` Generate diffs with 0 lines of context.  
    This option averts context matching issues.

    * `--ignore-space-at-eol` Ignore changes in whitespace at EOL.  
    That is what we want. And there are other `--ignore-*` options like `--ignore-space-change`, etc.

    * `--no-color` Turn off colored diff.  
    Since it is for pipeline, don't bother for visual.

2. `git apply` Apply a patch.

    * `--cached` Only apply to the index without touching the working tree.  
    Leave changes in whitespace at EOL unstaged.

    * `--ignore-whitespace` Ignore changes in whitespace in context lines if necessary.  
    This option averts whitespace fixing.

    * `--unidiff-zero` Bypass unified diff checks.  
    We are sure about no breaks.

    * `-` Read the patch from the standard input.  
    Pipe.

That's it.

And at the end. You may also make a git alias, so you can use `git addneol` to save all the memorizing and typing in the future.

```text
# e.g. What it looks like in the .gitconfig file.
...
[alias]
    ...
    addneol = !sh -c 'git diff -U0 --ignore-space-at-eol --no-color "$@" | git apply --cached --ignore-whitespace --unidiff-zero -'
    ...
...
```

Refs:  
<https://stackoverflow.com/questions/3515597/add-only-non-whitespace-changes>  
<https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration>  
<https://git-scm.com/docs/git-diff>  
<https://git-scm.com/docs/git-apply>
