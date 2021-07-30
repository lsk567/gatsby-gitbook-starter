Lingua-Franca uses [git submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) to incorporate third-party projects that are needed for some targets and features. For example, the TypeScript target requires the  [reactor-ts repository](https://github.com/icyphy/reactor-ts).

## Using Submodules

When initially cloning the Lingua Franca repository, use the `--recursive` flag to get all included submodules:
```
git clone --recursive https://github.com/icyphy/lingua-franca.git
```

## Pulling a New or Updated Submodule

If a new submodule has been added to the Lingua Franca repository or an existing module has been updated, then all users have to explicitly pull the new or updated module. Sadly, `git pull` alone does not work. A submodule is really a pointer to a *commit* in another repository, not a pointer to the repository. Hence, every user has to do this:

```
git pull
git submodule update --init --recursive
```

Oddly, git reports to users that a submodule has been updated by indicating that the user him/herself has modified a file, such as `reactor-ts`, that contains the key of the submodule commit. To clear this, the user must execute the commands above.

## Adding a Submodule to Lingua Franca

To add a submodule, navigate to the directory where you want the submodule to be located and run:

```
git submodule add <submodule_repository_URL>
```

This command will clone the desired submodule into your file system. It will also modify Lingua Franca's .gitmodules and add a new file to git tracking with the name of the submodule. Your next commit to Lingua Franca will record the new submodule.

## Updating Lingua Franca with Changes to a Submodule

If you make a change with the repository that is included in Lingua Franca as a submodule, and you want that change to propagate to all installations of Lingua Franca, then you need to update the pointer to the particular commit of the submodule within the Lingua Franca repository.  For example, if you have committed an update to reactor-ts repository, then within the Lingua Franca repository, you can pull this new version and commit the new version of the submodule as follows:

```
cd ~/git/lingua-franca/xtext/org.icyphy.linguafranca/src/lib/TS/reactor-ts
git pull origin master
cd ..
git commit reactor-ts/
git push
```

The above assumes that your Lingua Franca repository is in `~/git/lingua-franca`.

Notably `git pull` without an explicit remote and branch won't work the first time you try it because the submodule is locked to a particular commit.

Sadly, all users of the Lingua Franca repository need to do the following to get your updates:

```
git pull
git submodule update --init --recursive
```