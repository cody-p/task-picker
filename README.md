# task-picker
A little utility I made for myself, in python.

Currently assumes that the items and data repositories are relative to where it is running. This may be changed in the future to allow configuring.
It doesn't need any system directories, so it can remain OS-independent without me having to write special cases.

# How-to
Here is a short guide on how to use this program.
I'm new to writing comprehensive readmes so excuse me if the formatting is bad.

## data

the data folder is a convenience folder for storing shared files.
I'm considering redesigning the program to just assume everything is here by default
later on.

## items

This is the folder that is searched by task-picker.py when it runs.
You can store both directories and files in it.

### FILES
When a file is processed, it all of the non-property lines will be collected, and
one of them will be picked at random to be the final displayed value.
The following properties can be assigned in a file:

```
name: <PROPERTY NAME>
```

-What the property will be called when printed.
```
import: <FILE NAME>
```

-Another file to import values from. By default the path is considered to be relative
to the folder that the program is running from (which should be the root folder), though it will also attempt
to parse it as being in the same directory as the file itself if that fails.
If at least one of the files in the import chain has a 'name:' property, it will be used.
Names in a file will overwrite names from the files they import.

All other lines are possible values. For example:

#### file: colors
```
name: Color
import: data/secondary-colors
Red
Green
Blue
```
#### file: data/secondary-colors
```
Cyan
Magenta
Yellow
```
If the file 'colors' is anywhere in the 'items' directory tree, it will display like this
in the program output:
```
Color: <VALUE>
```

... where `<VALUE>` is any of the non property lines that are present in either 'colors' or
'secondary-colors'. So it could be Red, Green, Blue, Cyan, Magenta, or Yellow.

### FOLDERS
Folders can be used to create decision branches. By default they do nothing,
but can be given special properties by including a file called `_meta`, which can contain
the following values:
```
single:
```
If this is true, then the picker will only use one of the files / directories
it finds in the folder, picked at random in a manner similar to the values
in a file.
```
name:
```
Same as in a file. This is only displayed if the folder is in single mode.
```
<FOLDER-NAME>: <CHOSEN-SUBITEM-NAME>
```
A folder can be chosen from a single mode parent folder as well as a file,
and the folder's name will be used the same way a file's would.
If name is not given, it will not be shown when used as a super-folder.
If an item with no name is chosen as the output to a higher option,
the filename will be used instead.
```
show:
```
Can override name display when used as a super-folder.
The only meaningful value is "false". Anything else will
be considered True.
```
somedirectory
             |--------- _meta { single: true
             |                  name: Super }
             |
             |---------file-1 { name: SubFile
             |                  A
             |                  B
             |                  C }
             |
             |------subdir-1 ------- _meta { name: Subdir }
             |               |
             |               |------file-2 { name: Sub-Subfile 
             |               |               1
             |               |               2
             |               |               3
             |               |               4 }
             |               |------file-3 { name: Otherfile 
             |                               1
             |                               2
             |                               3
             |                              4 }
             |
             |------subdir-2 ------- _meta { name: Second Subdir
             |               |               show: false }
             |               |
             |               |------file-2 { name: Bar 
             |               |               1
             |               |               2
             |               |               3
             |               |               4 }
             |               |------file-3 { name: Foo 
             |                               1
             |                               2
             |                               3
             |                              4 }
             |
             |------subdir-2 ------ _meta { name: Top level option }
```
Possible output:
```
Super: Subfile
Subfile: B

Super: Subdir
Subdir: Otherfile
Otherfile: 3

#Subdir and Second Subdir are identical in structure
#But subdir-2/_meta has 'show: false', so the dir name doesn't get printed
#when choosing its own subitem.

Super: Second subdir
Bar: 2

Super: Top Level Option
```

A folder is not required to have any subitems. An empty folder with a name defined in
_meta can be used to give a given parent node a result that has no further output,
as shown in the "Super: Top Level Option" output above.
