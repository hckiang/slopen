# Introduction

`slopen` is a simple yet user-friendly alternative to `xdg-open`.

In Linux, the de-facto standard utility to open a file with default
application is `xdg-open`. Although `xdg-open` is easy to use,
configuring it can be pretty inconvenient: you need to create
`.desktop` files and debugging its wrong behaviour can be a
nightmare.

`slopen` is a simple tool that open a file with a default application.

## Usage

If you want to open `foo.png`, just type

    slopen foo.png

and it will choose the right program to open it according to your
`~/.slopenrc`. When no rules in `~/.slopenrc` matches the file,
it will ask the user either in terminal or using `dmenu`, depending
on whether `slopen` is run from a terminal or not.

## Requirements

1. ksh
2. dmenu

Both should be in package repositories in most Linux distribution.


## Installation

1. Drop the slopen into your `$PATH`.
2. Create `~/.slopenrc` (for example, see `slopenrc.example`)

## Configuration

`slopen` scans through 'rules' in `~/.slopenrc` one-by-one to decide which command to
use to open a file.
Each rule should look like

     <type>: <regrex> : <command>

Type can be either `M` and `S`.
If type is 'M' then regular expression is matched on the MIME type. If it is 'S'
then it matches file name instead. Whitespaces around the delimiter ':' is ignored.

If multiple rules matches a file, the first rule is always used. Lines started with
'#' are ignored.

Example:

     S: .*\.ps$                  :  zathura
     S: .*\.pdf$                 :  zathura
     S: .*\.png$                 :  qiv
     M: ^inode/directory$        :  rox
     M: ^text/.*                 :  emacs

With this config file, say, when you call `slopen ~`, it will invoke `rox ~`.

## Finding the right MIME type

Sometimes you want to find out which MIME type your file has when writing rules.
You find the MIME type with this command:

    $ file -b --mime-type foo.png
    image/png

