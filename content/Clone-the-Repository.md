The preferred way to work with Lingua Franca today is to use the [developer setup](Developer-Eclipse-setup-with-Oomph). If you do not wish to use Eclipse, you can instead clone the git repository. This will enable you to get updates by simply pulling the latest version of the repo and to edit the code in whatever code development environment you prefer.

We recommend cloning the repository into a directory called `git` in your home directory. Make that directory if you don't already have it:

```
 cd ~
 mkdir git
 cd ~/git
```

Sadly, there two ways to clone a repo, using https or ssh, and many people are dogmatic about which they use. Using https:

```
 git clone --recursive https://github.com/icyphy/lingua-franca.git
```

Or, using ssh (requires [setting up your github account](https://help.github.com/en/articles/connecting-to-github-with-ssh)):

```
 git clone --recursive git@github.com:icyphy/lingua-franca.git
```

These will create a directory `lingua-franca` containing a copy of the repository. 

The `--recursive` argument ensures that  [git submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) are also cloned. See [[Creating and Managing Submodules]] for more details. If you omit the `--recursive` argument, then some features, including the C++ and TypeScript targets, will not work. If you previously cloned the lingua-franca repository without `--recursive` or you need to update the repository with changes in the submodules, run from inside the lingua-franca repository:

```
git pull
git submodule update --init --recursive
```

For convenience, on the rest of this page, we assume that you have set an environment variable `LF` equal to the location of this `lingua-franca` repository. For example, you can do this in a bash shell as follows:

```
cd lingua-franca
export LF=$(pwd)
```

For me, I can check this using

```
echo $LF
```

which returns

```
/Users/eal/git/lingua-franca
```

You can now use the [[command line tools]], which build themselves.

