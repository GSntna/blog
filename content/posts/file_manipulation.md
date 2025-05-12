---
date: 2025-05-10T19:14:59+02:00
# description: ""
# image: ""
lastmod: 2025-05-10
showTableOfContents: true
tags: ["python", "pathlib", "os", "notes"]
title: "Reading, writing and organizing files"
type: "post"
---

## Navigating through system

### Path library

The `Path` module from the `pathlib` library allows to manipulate file paths and
navigate through the system. These are some functions in the `Path` module:

```py
>>> from pathlib import Path
>>> Path('folder', 'subfolder', 'file.txt')  # returns path (adapted to os)
PosixPath('folder/subfolder/file.txt')
>>> Path.home()                              # returns home directory
PosixPath('/home/user')
>>> Path.cwd()                               # get current working directory
PosixPath('/home/user/current/working/directory')

# Getting ancestor folder of a path
>>> Path.cwd().parents[0]
PosixPath('/home/user/current/working')
>>> Path.cwd().parents[1]
PosixPath('/home/user/current/')
>>> Path.cwd().parents[5]
PosixPath('/home')

# get the attributes of a path
>>> p = Path('/home/user/current/file.txt')
>>> p.anchor
'/'
>>> p.parent
PosixPath('/home/user/current/file.txt')
>>> p.name
'file.txt'
>>> p.stem
'file'
>>> p.suffix
'.txt'
>>> p.drive  # what's before the anchor ('C:' in Windows)
''
>>> p.exists() # check if file/dir exists
True
>>> p.is_dir() # check if it is a directory
False
>>> p.is_file() # check if it is a file
True
```

### os library

Another library used for navigation and manipulation of files and directories is
`os`. This library is older than `pathlib`, so it can do most of the things that
`pathlib` does. Here are some useful functions not covered by the latter:

```py
>>> import os
>>> os.makedirs('/home/user/new/folder/paths')  # create one or multiple folders
>>> os.path.abspath('.')  # returns the absolute path
>>> os.path.isabs('.')  # boolean if path is absolute
False
>>> os.sep  # gets the separator for the OS
'/'  # '\\' for Windows
>>> os.path.getsize('./path') # returns path's size
23453
>>> os.listdir('./path')
['list', 'with', 'files.xlsx']

>>> list(Path.cwd().glob('*.txt'))  # get list of txt files in cwd
[PosixPath('./file.txt')], PosixPath('./file2.txt')
```

### Navigation functions summary

| Function | Description |
| -------- | ----------- |
| `Path.home()` | returns home directory |
| `Path.cwd()` | returns current working directory |
| `p.parents[n]` | p being a path object, parents[n] return n up parent folders path |
| `p.anchor` | returns path anchor |
| `p.parent` | returns Path object with p path |
| `p.name` | returns file name with extension |
| `p.stem` | returns file name without extension |
| `p.suffix` | returns file extension |
| `p.drive` | returns what's before the anchor "C:" for Windows |
| `list(p.glob('*.txt'))` | returns all the documents that match the pattern (regex-like) in that directory |
| `p.exists()` | boolean to check if p exists |
| `p.is_file()` | boolean to check if p is a file |
| `p.is_dir()` | boolean to check if p is a directory |
| `os.makedirs(path)` | creates one or more folders |
| `os.pah.abspath('.')` | returns absolute path of the argument |
| `os.path.isabs('.')` | boolean, returns if path is absolute |
| `os.sep` | returns the operating system separator |
| `os.path.getsize(path)` | returns path's size in bytes |
| `os.listdir(path)` | returns list with files and folders in path |

## Reading/Writing process

The more common way of writing or reading a file is using the `open(file, mode)`
function and file objects. The steps to follow are:

1. Call the `open()` function to return a `File` object.
2. Call the `read()` or `write()` method on the `File` object.
3. Close the file by calling the `close()` method on the `File` object.

```py
file = open('some_file.txt', 'r') # read, r is default
file.read() # returns string with text
file.readlines() # returns list of strings, where each element is a line

file = open('some_file.txt', 'w') # write
file.write('write this text to a file.\n')

file = open('some_file.txt', 'a') # append
file.write('this text will be appended to the file')

file.close() # closes the file
```

## Organizing files

### Shell utilities (shutil)

The built-in module `shutil`has functions to copy, move, rename and delete
files.

#### Copy, move and rename

```py
import shutil, os
from pathlib import Path

# Copy
>>> shutil.copy('./test.txt', './src/') # copy to src folder
'./src/test.txt'
>>> shutil.copy('./test.txt', './src/test2.txt') # copy with different name
'./src/test2.txt'

# Move and rename
>>> shutil.move('./test.txt', './src/') # move to src folder
'./src/test.txt'
>>> shutil.move('./test.txt', './src/test2.txt') # move and rename
'./src/test2.txt'
```

##### Safely removing files with send2trash

To send a file/folder to trash instead of permanently removing it, the third
party library `send2trash` can be used.

Syntax: `send2trash.send2trash('filename.txt')`

### Walking a directory tree

To iterate over all the folder and files inside a folder the `os.walk(path)`
function can be used. It returns three values on each iteration of the loop:

* a string of the curent folder's name,
* a list of strings of folders in the current folder,
* a list of strings of files in the current folder

```py
import os

for folderName, subfolders, filenames in os.walk('pathto/foldertowalk/'):
    print('The current folder is ' + folderName)

    for subfolder in subfolders:
        print('SUBFOLDER OF ' + folderName + ': ' + subfolder)

    for filename in filenames:
        print('FILE INSIDE ' + folderName + ': '+ filename)

    print('')
```

### Compressing files

#### Reading ZIP files

To read a zip file and explore its contents:

```py
>>> import zipfile, os
>>> from pathlib import Path

>>> zip = zipfile.ZipFile('./src.zip')  # create object for compressed file
>>> zip.namelist()  # get contents
['src/', 'src/SUMMARY.md', 'src/homepage.md', 'src/regex.md', 'src/read_write.md']
>>> file = zip.getinfo('src/regex.md')  # select a file to get info
>>> file.file_size  # file size
9078
>>> file.compress_size  # compressed file size
3173
>>> zip.close()  # close zip file
```

#### Extracting from zip files

```py
>>> zip = zipfile.ZipFile('./src.zip')
>>> zip.extractall()  # extract all files in zip to cwd
>>> zip.extract('src/regex.md', './some/folder')  # extract only given file to given path
>>> zip.close()
```

#### Creating a zip file

Works similar to the `read()` method. The zip file has to be opened using
`zip = zipfile.ZipFile(zipfile)` but specifying a second argument: `w` or `a` for
writting and appending respectively.

```py
>>> newZip = zipfile.ZipFile('new.zip', 'w')  # write mode
>>> newZip.write('spam.txt', compress_type=zipfile.ZIP_DEFLATED)  # compress spam.txt
>>> newZip.close()
```

### Organizing functions summary

| Function | Description |
| -------- | ----------- |
| `shutil.copy(source, destination)` | copy file at *source* to *destination* folder |
| `shutil.copytree(source, destination)` | copy folder along with subfolders/files to *destination* |
| `shutil.move(source, destination)` | move file at *source* to *destination* folder |
| `os.unlink(path)` | delete file at *path* |
| `os.rmdir(path)` | delete folder at *path*, it has to be empty |
| `os.rmtree(path)` | delete folder at *path* along with all its files and subfolders |
| `send2trash.send2trash(file)` | send a file to trash instead of permanent deletion |
| `os.walk(path)` | iterate over all files under *path* |
| `zip = zipfile.ZipFile(zipfile)` | read zip file | 
| `zip.namelist()` | get contents from zipfile |
| `zip.extractall(path)` | extract files from zip to *path*, defaults to cwd |
| `zip.extract(file, path)` | extract only selected file from zip to given path; if path isn't given it defaults to cwd |
| `newzip = zipfile.ZipFile(newzip, w)` | open zip file in write mode | 
| `newzip.write(file, compress_type)` | writes a file into a zip file, the compress_type defines the compression algorithm |