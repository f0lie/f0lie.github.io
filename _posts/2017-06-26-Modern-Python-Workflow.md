---
layout: post
title: How to use python 3's built in virtual environment, venv
---
Modern workflow involves isolated virtual environments. Without virtual environments (`venv`), developers would install packages into the system `python`. The issue with this method is changing the system `python` would impact all users of `python`. This includes scripts that the OS uses.

Potentially, packages would upgrade their dependencies. This change would impact other packages using the same dependency. Changing versions of dependencies can lead to confusing bugs in deeply embeded scripts.

To avoid this, you should use python's virtual environment feature, `venv`. Anyone with the most recent version of `python3` should be able to do this.

```zsh
➜  ~ mkdir -p my_project/.env <1>
➜  ~ python3 -m venv my_project/.env <2>
➜  ~ source my_project/.env/bin/activate <3>
(.env) ➜  ~ where python3 <4>
/Users/adiep/my_project/.env/bin/python3
/usr/local/bin/python3
/usr/local/bin/python3
(.env) ➜  ~ where pip3 <4>
/Users/adiep/my_project/.env/bin/pip3
/usr/local/bin/pip3
/usr/local/bin/pip3
(.env) ➜  ~ pip3 install ... <5>
```
1. Make a dir to store the local versions of `pip3` and `python3`.
2. Use the included `venv` module to create the virtual environment.
3. Change the shell’s path to the local version.
4. Prove that the shell is using the local version of `python`.
5. Using `pip3` would install to the `.env` folder.

Note that by running `source ./.env/bin/activate`, the shell points to the local version of `python` and `pip`. If you forget to type `source ./.env/bin/activate`, you would be using your system's `python` and `pip`. When modifying the environment, changes go locally rather than system wide.

You can add `alias act="source .env/bin/activate"` to your `.zshrc` or `.bashrc` to save time. All you would need to type is `act` to point to local versions of python and pip.

## Using venv to develop forks

Normally, when you install packages, `pip` puts them into the `/lib` folder and without a git repo. It isn't easy to make changes to installed packages because it wasn't design with that in mind.

Editable installs are designed to make changes easier. `pip` installs packages into `.env/src` folder with a git repo included. Thus you can fork a package, use `pip install -e repo` to clone the repo into `.env`, and commit changes. 

The reason why this is useful is because you can run your program with your fork easily. The program doesn't notice any changes. When you do `import ...`, `python` appropriately imports your fork instead of the "offical" one. Keep in mind, you would need to do something like `importlib.reload(module)` or restart your `python` instance to get the changes.

For instance, here I locally install `nltk`. Then I install the forked version of `nltk` from my personal github. I can enter the git repo that contains my forked `nltk`, commit changes, and push them to my repo. After I push the changes, I can make a pull request to the main repo.

```zsh
(.env) ➜  ~ pip3 list <1>
pip (9.0.1)
setuptools (28.8.0)
(.env) ➜  ~ pip3 install nltk <2>
Collecting nltk
Collecting six (from nltk)
  Using cached six-1.10.0-py2.py3-none-any.whl
Installing collected packages: six, nltk
Successfully installed nltk-3.2.4 six-1.10.0
(.env) ➜  ~ pip3 show nltk <3>
Name: nltk
Version: 3.2.4
Summary: Natural Language Toolkit
Home-page: http://nltk.org/
Author: Steven Bird
Author-email: stevenbird1@gmail.com
License: Apache License, Version 2.0
Location: /Users/$USER/my_project/.env/lib/python3.6/site-packages
Requires: six
(.env) ➜  ~ pip3 install -e git+http://github.com/f0lie/nltk#egg=nltk <4>
Obtaining nltk from git+http://github.com/f0lie/nltk#egg=nltk
  Cloning http://github.com/f0lie/nltk to ./.env/src/nltk
Requirement already satisfied: six in ./.env/lib/python3.6/site-packages (from nltk)
Installing collected packages: nltk
  Found existing installation: nltk 3.2.4
    Uninstalling nltk-3.2.4: <5>
      Successfully uninstalled nltk-3.2.4
  Running setup.py develop for nltk
Successfully installed nltk
(.env) ➜  ~ cd .env/src/nltk <6>
(.env) ➜  nltk git:(develop)
(.env) ➜  nltk git:(develop) git remote get-url origin <7>
http://github.com/f0lie/nltk
```
1. venv creates environments with minimal packages
2. Install the "offical nltk" package. Here it's cached.
3. Note how the `nltk` package is installed locally and not system wide
4. Here I use `-e` to install a fork of `nltk` in editable mode. Note how the package is installed into the `.env/src` folder.
5. The "offical" `nltk` is removed.
6. I move to the folder containing my fork.
7. Make sure origin is pointed to my fork.

## Caveats

Since you are running the local version of `python3` with `venv`, packages installed in the system would not be used in the local environment. For example, I like to use `bpython` instead of the included interepter shell. If I run `bpython` with the activated paths, it would run the system's `bpython` and not import `.env/` packages.

Here I show that making a new virtual environment and showing how `bpython` is refering to my system's packages. If I install `bpython` again, it would point to my local environment.

```zsh
➜  ~ mkdir -p my_project/.env
➜  ~ python3 -m venv my_project/.env
➜  ~ cd my_project
➜  my_project act
(.env) ➜  my_project where bpython <1>
/usr/local/bin/bpython
/usr/local/bin/bpython
(.env) ➜  my_project pip3 install bpython
Collecting bpython
  Downloading bpython-0.16-py2.py3-none-any.whl (180kB)
    100% |████████████████████████████████| 184kB 1.4MB/s
Collecting six>=1.5 (from bpython)
  Downloading six-1.10.0-py2.py3-none-any.whl
Collecting greenlet (from bpython)
  Downloading greenlet-0.4.12.tar.gz (57kB)
    100% |████████████████████████████████| 61kB 1.8MB/s
Collecting curtsies>=0.1.18 (from bpython)
  Downloading curtsies-0.2.11-py2.py3-none-any.whl
Collecting pygments (from bpython)
  Downloading Pygments-2.2.0-py2.py3-none-any.whl (841kB)
    100% |████████████████████████████████| 849kB 1.1MB/s
Collecting requests (from bpython)
  Downloading requests-2.18.1-py2.py3-none-any.whl (88kB)
    100% |████████████████████████████████| 92kB 2.4MB/s
Collecting wcwidth>=0.1.4 (from curtsies>=0.1.18->bpython)
  Downloading wcwidth-0.1.7-py2.py3-none-any.whl
Collecting blessings>=1.5 (from curtsies>=0.1.18->bpython)
  Downloading blessings-1.6.tar.gz
Collecting idna<2.6,>=2.5 (from requests->bpython)
  Downloading idna-2.5-py2.py3-none-any.whl (55kB)
    100% |████████████████████████████████| 61kB 7.1MB/s
Collecting urllib3<1.22,>=1.21.1 (from requests->bpython)
  Downloading urllib3-1.21.1-py2.py3-none-any.whl (131kB)
    100% |████████████████████████████████| 133kB 1.5MB/s
Collecting chardet<3.1.0,>=3.0.2 (from requests->bpython)
  Downloading chardet-3.0.4-py2.py3-none-any.whl (133kB)
    100% |████████████████████████████████| 143kB 1.5MB/s
Collecting certifi>=2017.4.17 (from requests->bpython)
  Downloading certifi-2017.4.17-py2.py3-none-any.whl (375kB)
    100% |████████████████████████████████| 378kB 1.7MB/s
Installing collected packages: six, greenlet, wcwidth, blessings, curtsies, pygments, idna, urllib3, chardet, certifi, requests, bpython
  Running setup.py install for greenlet ... done
  Running setup.py install for blessings ... done
Successfully installed blessings-1.6 bpython-0.16 certifi-2017.4.17 chardet-3.0.4 curtsies-0.2.11 greenlet-0.4.12 idna-2.5 pygments-2.2.0 requests-2.18.1 six-1.10.0 urllib3-1.21.1 wcwidth-0.1.7
(.env) ➜  my_project where bpython <2>
/Users/$USER/my_project/.env/bin/bpython
/usr/local/bin/bpython
/usr/local/bin/bpython
```
1. Even though I activated `venv`, `bpython` is pointing to my system's because I haven't installed it locally.
2. Now that I installed it, `bpython` is pointing to my local environment.

## virtualenvwrapper, pyvenv, and etc...?

There are a lot of tools that basically do the same thing. Keep in mind that Python is an old language and the best practices change over time.

[This answers the differences far better than I can.](
https://stackoverflow.com/questions/41573587/what-is-the-difference-between-venv-pyvenv-pyenv-virtualenv-virtualenvwrappe)

In my opinion, focus on why you are using the tool as oppose to learning all of the differences. All of these tools exist because virtual environments are important and that is what you should focus on.
