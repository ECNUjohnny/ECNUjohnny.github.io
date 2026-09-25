+++

authors = "Johnny"
title = "recent studies of python and linux system"
date = "2026-09-25"
tags = [
    "python",
    "os",
    "linux",
]

+++

# Copy file in python

use `shutil` liberary

```
import shutil

shutil.copy(source, target)

shutil.copyfile(source, target)

```

copyfile requires the 'target' to be a complete name of a file, instead of name of a folder. 

copy will copy more datas about a file (.e.g, the Authority) while copyfile would not

# scandir and listdir

```
files = os.listdir(path)

with os.scandir(path) as entries:
    for entry in entries:
        entry.name
        entry.is_file()
        entry.is_dir()
        entry.path

```

scandir is faster than listdir, so use it instead listdir

# linux command line

```bash
ls -l | grep "^d" | wc -l


ls -l | grep "^-" | wc -l

```

count the number of folders / files


```bash

rsync [options] source target

7z <command> [options] <archive_name> [files...]

```
usage of 7-zip and rsync

**rsync**

| Option |     Long Option     |                              Description                              |
| :----: | :-----------------: | :-------------------------------------------------------------------: |
|  `-a`  |     `--archive`     | Archive mode; preserve all file attributes (equivalent to `-rlptgoD`) |
|  `-v`  |     `--verbose`     |                 Display detailed transfer information                 |
|  `-z`  |     `--compress`    |                     Compress data during transfer                     |
|  `-r`  |    `--recursive`    |                      Recursively copy directories                     |
|  `-l`  |      `--links`      |                        Preserve symbolic links                        |
|  `-p`  |      `--perms`      |                       Preserve file permissions                       |
|  `-t`  |      `--times`      |                    Preserve file modification times                   |
|  `-g`  |      `--group`      |                     Preserve file group ownership                     |
|  `-o`  |      `--owner`      |                        Preserve file ownership                        |
|  `-D`  |     `--devices`     |                 Preserve device files (superuser only)                |
|  `-h`  |  `--human-readable` |               Display numbers in a human-readable format              |
|    —   |     `--progress`    |                       Display transfer progress                       |
|    —   |      `--delete`     |    Delete files in the destination that do not exist in the source    |
|    —   | `--exclude=PATTERN` |                    Exclude files matching `PATTERN`                   |
|    —   | `--include=PATTERN` |                    Include files matching `PATTERN`                   |

**7-zip**

| Command | Description |
|:------:|-------------|
| `a` | Add files to the archive |
| `e` | Extract files (ignore directory structure) |
| `x` | Extract files (preserve directory structure) |
| `l` | List the contents of the archive |
| `d` | Delete files from the archive |
| `t` | Test the integrity of the archive |