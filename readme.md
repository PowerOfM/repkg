```
          _
  ___ ___| |_ ___
 |  _| . | '_| . |
 |_| |  _|_,_|_  |
     |_|     |___|
```
> rpkg - replace package utility

## the problem
I needed to be able to swap configuration files and other files on the fly without having to constantly mess with git branches. The files that needed to be changed were static and could not be easily altered using environment variables.

## the solution
rpkg is a simple, zero-dependency utility that collects a group of files (called packages) and can load/unload them on the fly. The goal is to make something simple, and light. rpkg works by copying files to a hidden `.rpkg/` folder under a package name. The directory structure of all copied files is preserved. When a package is loaded, its files are symbolically linked to the working directory. rpkg doesn't track version changes as git is more than adequate for that.

> **NOTE:** make sure you commit the .rpkg directory to version control.

> **WARNING:** not tested on Windows (this uses symlinks, so it'll probably not work).

## how it works
Inspired by git, rpkg has 4 main functions:
- `add`: stages a file in the working directory for adding to/creating a package
- `commit`: adds the staged files to a new or existing package
- `load`: loads a package by symbolically-linking package files to the working directory
- `unload`: unloads a package by unlinking package in the working directory

To use it, first add the files that will make up the package.
> `rpkg add src/config.json src/static.html src/image.png`

Then commit the files to create a package (package names must be a valid folder names)
> `rpkg commit a`

The added files have been copied to the rpkg store (`.rpkg/a/` in the cwd). It is now safe to modify them to the next variant and repeat the above steps to create another package `b`. To apply the package `a`, use the `load` command:
> `rpkg load a`

If you haven't deleted the files in the working directory, you will be prompted to replace them. Once complete, the files will have been replaced with symbolic links to the `.rpkg/a/` directory. 

> **NOTE:** Any changes made to loaded package files will update the files in package.

To remove the symbolically linked package files, use `unload`:
> `rpkg unload`

When you load a package, the previous one is unloaded first to prevent conflicts.

## under the hood
rpkg is designed to be simple and easy to debug if something goes wrong. The .rpkg folder has the following structure:
```
.rpkg/
├── <package>/
│   └── .rpkg_index         (JSON) List of all files in the package
│   └── <files>             Files that were added to the package and committed
└── .rpkg_state             (JSON) Current state
```
The `.rpkg_state` file is an object with `current` (package name), `packages` (array of package names), and `added` (array of added files).

While you are welcome to change files directly in the `.rpkg/` folder, it is preferred to load a package, make changes, and then unload or switch to another package.

## why is this not tested?!?!
well it works on my machine.

No but seriously, it's working for me across multiple environments. If that's not good enough, PRs are always welcome!

## alternatives
- `make`: if you have experience with it, GNU Make is probably going to be the most powerful and versatile solution
- `.env`: if all you need is variable swapping, dot-env or other similar configuration managers are probably best bets.
- `git`: branch and then apply changes. When the upstream branch changes, rebase? Someone smarter can probably show a better way and explain why this utility is useless but oh well I had fun writing it.