# Introduction

`slopen` is a simple and user-friendly alternative to `xdg-open`.

In Linux, the de-facto standard utility to open a file with default
application is `xdg-open`. Although `xdg-open` is easy to use,
configuration is complicated: you need to create `.desktop` files
and debugging its behaviour is a nightmare.

`slopen` is an alternative with a single simple configuration file.
When it doesn't find a suitable command for a file, the user is
prompted, either directly in the terminal or with `dmenu`.


## Usage

If you want to open `foo.png`, just type

    slopen foo.png

and it will choose the right program to open it according to your
`~/.slopenrc`. When no rules in `~/.slopenrc` matches the file,
it will ask the user either in terminal or with `dmenu`, depending
on whether `slopen` is run from a terminal or not.

## Requirements

1. ksh
2. dmenu

Both should be in package repositories in most Linux distribution.


## Installation

1. Drop the slopen into your `$PATH`.
2. Create `~/.slopenrc` (for example, see `slopenrc.example`)

## Configuration

`slopen` scans through the 'rules' in `~/.slopenrc` one-by-one to decide which command to
use to open a file.
Each rule should look like

     <type>: <regex> : <command>

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

## Tell xdg-open, Firefox and everything to use slopen

While it is possible to just replace the `xdg-open` executable with `slopen`, this is usually not enough to change the behaviour of some applications such as Firefox, which uses the C API directly to access the default application database instead of accessing it through the xdg-open binary.

Since too many applications is using the same xdg-mime database through different interfaces, perhaps the easiest way to make everything use slopen is to directly setting the default application of every MIME types to `slopen` in the xdg-mime database. Doing this will also makes `xdg-open` forward everything to `slopen`.

To do this, create the file `~/.local/share/applications/slopen.desktop` with the following content:
```
[Desktop Entry]
Name=Slopen
Exec=slopen %F
Type=Application
```

Then use the following command to set the default application of all known MIME types to `slopen.desktop`:
```
find /usr/share/applications ~/.local/share/applications \
     -type f -name '*.desktop' \
     -exec awk 'match($0, /^MimeType=(.*)/, m) {\
     	   split(m[1],t,";");\
	   for (i in t) \
	       if (t[i]!="") \
	       	  print(t[i])
     }' {} \; | \
     xargs -I {} xdg-mime default slopen.desktop {}
```

You may want to run this again after installing an application which introduces a new MIME type. So you may want to add it as your cron jobs, into `/etc/rc.local` or to some hooks in your distro's package manager, depending on your preference.
