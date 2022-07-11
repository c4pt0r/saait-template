saait
-----

The most boring static page generator.

Meaning of saai (dutch): boring
Pronounciation:          site

Some parts are intentionally hardcoded in C for simplicity.


Build and install
-----------------

$ make
# make install


Dependencies
------------

- C compiler (C99), tested on gcc and clang.
- libc
- POSIX make (optional).
- mandoc for documentation: https://mdocml.bsd.lv/ (optional).


Tested and works on
-------------------

- OpenBSD
- Linux (glibc).
- Windows (mingw and cygwin).


Example of a workflow
---------------------

This script monitors pages for changes and for new files in the directory then
regenerates static files if this happens. If successful then refreshes the
current window/tab in Firefox.

Dependencies: make, xdotool, entr (https://eradman.com/entrproject/).

	#!/bin/sh
	if test x"$1" = x"rebuild"; then
		make && xdotool search --class firefox key F5
	else
		while :; do find pages | entr -d -p "$(readlink -f "$0")" rebuild; done
	fi


Using Markdown
--------------

A possible method could be to just convert the Markdown to HTML and to run
saait after that as usual.

In this example it uses smu for the Markdown processor, available at:
https://github.com/Gottox/smu:

	cd pages
	for f in *.md; do
		smu -n < "$f" > "$(basename "$f" .md).html"
	done


Documentation
-------------

See the man page of saait(1).


Man page
--------

SAAIT(1)		    General Commands Manual		      SAAIT(1)

NAME
     saait  the most boring static page generator

SYNOPSIS
     saait [-c configfile] [-o outputdir] [-t templatesdir] pages...

DESCRIPTION
     saait writes HTML pages to the output directory.

     The arguments pages are page config files, which are processed in the
     given order.

     The options are as follows:

     -c configfile
	     The global configuration file, the default is "config.cfg".  Each
	     page configuration file inherits variables from this file.	 These
	     variables can be overwritten per page.

     -o outputdir
	     The output directory, the default is "output".

     -t templatesdir
	     The templates directory, the default is "templates".

DIRECTORY AND FILE STRUCTURE
     A recommended directory structure for pages, although the names can be
     anything:
     pages/001-page.cfg
     pages/001-page.html
     pages/002-page.cfg
     pages/002-page.html

     The directory and file structure for templates must be:
     templates/<templatename>/header.ext
     templates/<templatename>/item.ext
     templates/<templatename>/footer.ext

     The following filename prefixes are detected for template blocks and
     processed in this order:

     "header."
	     Header block.

     "item."
	     Item block.

     "footer."
	     Footer block.

     The files are saved as output/<templatename>, for example
     templates/atom.xml/* will become: output/atom.xml.	 If a template block
     file does not exist then it is treated as if it was empty.

     Template directories starting with a dot (".") are ignored.

     The "page" templatename is special and will be used per page.

CONFIG FILE
     A config file has a simple key=value configuration syntax, for example:

     # this is a comment line.
     filename = example.html
     title = Example page
     description = This is an example page
     created = 2009-04-12
     updated = 2009-04-14

     The following variable names are special with their respective defaults:

     contentfile
	     Path to the input content filename, by default this is the path
	     of the config file with the last extension replaced to ".html".

     filename
	     The filename or relative file path for the output file for this
	     page.  By default the value is the basename of the contentfile.
	     The path of the written output file is the value of filename
	     appended to the outputdir path.

     A line starting with # is a comment and is ignored.

     TABs and spaces before and after a variable name are ignored.  TABs and
     spaces before a value are ignored.

TEMPLATES
     A template (block) is text.  Variables are replaced with the values set
     in the config files.

     The possible operators for variables are:

     $	     Escapes a XML string, for example: < to the entity &lt;.

     #	     Literal raw string value.

     %	     Insert contents of file of the value of the variable.

     For example in a HTML item template:

     <article>
	     <header>
		     <h1><a href="">${title}</a></h1>
		     <p>
			     <strong>Last modification on </strong>
			     <time datetime="${updated}">${updated}</time>
		     </p>
	     </header>
	     %{contentfile}
     </article>

EXIT STATUS
     The saait utility exits 0 on success, and >0 if an error occurs.

EXAMPLES
     A basic usage example:

     1.   Create a directory for a new site:

	  mkdir newsite

     2.   Copy the example pages, templates, global config file and example
	  stylesheets to a directory:

	  cp -r pages templates config.cfg style.css print.css newsite/

     3.   Change the current directory to the created directory.

	  cd newsite/

     4.   Change the values in the global config.cfg file.

     5.   If you want to modify parts of the header, like the navigation menu
	  items, you can change the following two template files:
	  templates/page/header.html
	  templates/index.html/header.html

     6.   Create any new pages in the pages directory.	For each config file
	  there has to be a corresponding HTML file.  By default this HTML
	  file has the path of the config file, but with the last extension
	  (".cfg" in this case) replaced to ".html".

     7.   Create an output directory:

	  mkdir -p output

     8.   After any modifications the following commands can be used to
	  generate the output and process the pages in descending order:

	  find pages -type f -name '*.cfg' -print0 | sort -zr | xargs -0 saait

     9.   Copy the modified stylesheets to the output directory also:

	  cp style.css print.css output/

     10.  Open output/index.html locally in your webbrowser to review the
	  changes.

     11.  To synchronize files, you can securely transfer them via SSH using
	  rsync:

	  rsync -av output/ user@somehost:/var/www/htdocs/

TRIVIA
     The most boring static page generator.

     Meaning of saai (dutch): boring, pronunciation of saait: site

SEE ALSO
     find(1), sort(1), xargs(1)

AUTHORS
     Hiltjo Posthuma <hiltjo@codemadness.org>
