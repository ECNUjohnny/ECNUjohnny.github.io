+++

date = "2026-09-22"
author = "Johnny"
title = "os.path and pathlib"
tags = ["python", "file system"]

+++

**comparsion between os.path and pathlib**

| utilities | os.path | pathlib |
|:----:|:-----:|:----:|
|path merge|```	os.path.join(base,"sub","file.txt")```|```	base / "sub" / "file.txt"```|
|get file's name|```os.path.basename(p)```|```Path(p).name```|
|get the suffix of a file|```os.path.splitext(p)[1]```|```Path(p).suffix```|
|get the parent folder's name|```os.path.dirname(p)```|```Path(p).parent```|
|whether it exists|```os.path.exists(p)```|```Path(p).exists()```|
|whether is a file|```os.path.isfile(p)```|```Path(p).is_file()```|
|whether is a dir|```os.path.isdir(p)```|```Path(p).is_dir()```|
|get the absolute path|```os.path.abspath(p)```|```Path(p).resolve()```|
|get a list of the files and folders in folder p|```os.listdir(p)```|```Path(p).iterdir()```|
|make a dir|```os.makedirs(p, exist_ok=True)```|```Path(p).mkdir(parents=True, exist_ok=True)```|