---
layout: post
title:  "Dealing with Anaconda Python in Linux"
date:   2016-09-30 15:00:00
type: post
---

[Anaconda][continuum] is a very handy Python distribution that bundles a slew of scientific packages along with the handy [conda package manager][conda], allowing you to easily update your packages to the latest versions.
However, it doesn't seamlessly integrate with Linux: the installer will simply add a line to your shell configuration file (e.g., `~/.bashrc`) prepending the Anaconda `bin` directory to the `PATH` environment variable.
This means that Anaconda's `python` and other packages will override Linux's system `python` and packages (to see this, run `which python`).
For the most part, this shouldn't be an issue, but in some cases it can cause trouble.

For example, if you use the `yaourt` package manager on Arch Linux, you'll run into two problems with Anaconda Python.
First, Anaconda's `curl` will break `yaourt`.
Second, `yaourt` will blindly install Python packages into the Anaconda Python package directory.
This sometimes causes conflicts with Python packages installed through `pacman` (which are properly installed into the system Python package directory).
I ran into this problem when installing [Reddit Terminal Viewer][rtv].

If you just want to use Anaconda but not have it take over like this, there's an easy fix: instead of *prepending* the Anaconda `bin` directory to `PATH` (which gives it priority), simply *append* it so that system Python (`/usr/bin/python`) and packages take precedence.
In your shell configuration (e.g., `~/.bashrc`), change the line added by the Anaconda installer:

```bash
# Default configuration of Anaconda installer
#export PATH="/home/toban/utilities/anaconda3/bin:$PATH"

# Append Anaconda so that it doesn't override system Python
export PATH="$PATH:/home/toban/utilities/anaconda3/bin"
```

Now, `python` (and `curl`, for example) will still be the system version, but Anaconda applications will also be available (`jupyter`, `ipython`, `conda`, etc).
My Anaconda workflow is to work in a [Jupyter notebook][jupyter] along with a text editor.
Since `jupyter` is tied to the Anaconda `python`, it continues to work seamlessly as before.

If you want to temporarily use Anaconda's `python` at the shell, just use `conda`'s environment manager to activate/deactivate it:

```bash
source activate <env>
source deactivate
```

The `root` environment uses the standard Anaconda `python` installation, so to activate the Anaconda's `python`, just do `source activate root`.
(Note: this only affects the current shell session).

#### Bonus: Python auto-completion in Vim

Since `vim` (and `neovim`) are compiled against the system `python`, they won't get completions from Anaconda packages.
However, the (`neovim`-only) python completions package [`deoplete-jedi`][deoplete] allows you to specify which `python` interpreter to use for the completion server.
Add the following line to your config (with the correct path):

```vim
let g:deoplete#sources#jedi#python_path = '/path/to/anaconda3/bin/python'
```

Another simple (although slightly ugly) workaround is to install any packages you want completions for via the system `pip` or with your system's package manager.
Then, while you'll run your code with Anaconda, `vim` can at least pull completions from the corresponding system Python packages.

[continuum]:https://www.continuum.io/
[conda]:http://conda.pydata.org/docs/
[rtv]:https://github.com/michael-lazar/rtv
[jupyter]:http://jupyter.org/
[deoplete]:https://github.com/zchee/deoplete-jedi
