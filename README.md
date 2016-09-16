## Introduction

This script prints out assorted descriptive information about each [Nessus](http://www.tenable.com/products/nessus-vulnerability-scanner) plugin named on the commandline: id, name, family, category, etc.  It works by reading the plugin directly and parsing out the information of interest from the various `script_*` functions in the its description block.  As such, it only works with plugins written in NASL (`*.nasl`), not NASL include files (`*.inc`), plugins written in C (`*.nes`), or compiled plugins (`*.nbin`) or libraries (`*.nlib`).  It does not require access to a Nessus server but does require read access to the plugin.

The decision about what information to report can be controlled either by setting `@funcs` in this script or by using the option `--functions` on the commandline.  In either case, function names should be specified without the leading `script_` string; for example, `cve_id` represents the information supplied as an argument to `script_cve_id`.  The order in which information is reported is controlled by setting `@func_order` in this script; there is no way to change it via the commandline.

*describe-nessus-plugin* is written in Perl.  It should work on any system with Perl 5.005 or better.  It also requires the following Perl modules:

* `Carp`
* `Getopt::Long`
* `Text::Balanced`
* `Text::Wrap`

If your system does not have these modules installed already, visit [CPAN](http://search.cpan.org/). Note that `Text::Balanced` does not work with versions of Perl older than 5.005; further, it is not included with Perl distributions prior to 5.8.0 so you will probably need to install it if you're running an older version of Perl.


## Installation

* Retrieve [the script](describe-nessus-plugin) and save it locally.
* Verify ownership and permissions on the script - there's no reason why the script itself can't be accessed by any user.
* You may wish to edit the script to adjust the location of the perl interpreter in the first line and to set `@funcs`, `%func_labels`, `@func_order`, and `$lang` to suit your tastes.


## Use

There are several commandline arguments you can use to override variables defined in the script itself:

| Option | Meaning |
| ------ | ------- |
| -d, --debug | Display debugging messages while running. |
| -f, --functions <funcs> | Display information for the specified functions, overriding `@funcs`. Function names are equal to the NASL descriptive information functions, with the leading string `script_` removed. `risk` can also be specified as a function name; it refers to the risk factor, which by convention is specified as part of `script_description`. In addition, `_all_` can be used to represent all possible functions and the prefix `!` to skip specific ones. |
| -w, --width <width> | Use the specified screen width (to control line-wrapping).. |

Notes:

* Commandline arguments take precedence over variables defined in the script. For example, you can describe all functions by using the commandline argument `-f _all_` regardless of how `@funcs` is defined in the script.
* Multiple functions can be specified either by a comma-delimited string or by multiple argument pairs.  For example, `-f "cve_id,bugtraq"` is equivalent to `-f cve_id -f bugtraq`.
* This script reportedly runs fine under Windows, although it does not handle wildcard expansion out of the box. You can work around this by creating `Wild.pm` and adding`-MWild` to the commandline as described in the [perlwin32](http://perldoc.perl.org/perlwin32.html) documentation.


Examples:

| Goal | Invocation |
| -----| ---------- |
| Describe assorted NASL plugins related to MS SQL. | `describe-nessus-plugin /usr/local/lib/nessus/plugins/mssql*.nasl` |
| Show how the script parses the specified NASL plugin. | `describe-nessus-plugin -d wip.nasl` |
| Report CVE ID(s) for all Oracle-related plugins. | `describe-nessus-plugin -f cve_id /usr/local/lib/nessus/plugins/oracle*.nasl` |
| Same as above but avoid line-wrap. | `describe-nessus-plugin -f cve_id -w 999 /usr/local/lib/nessus/plugins/oracle*.nasl` |
| Report all information except the description for all Apache-related plugins. | `describe-nessus-plugin -f _all_ -f '!description' /usr/local/lib/nessus/plugins/apache*.nasl` |

The output produced by *describe-nessus-plugin* is fairly simple to manipulate, if you care to work with it elsewhere.  For example, [desc2csv](desc2csv) is a Perl script to filter the output into CSV format, for import into a  spreadsheet or possibly a database.


## Known Bugs and Caveats

Currently, I am not aware of any bugs in this script.

Understand that this script is not a NASL parser - on one hand, it may not handle some constructs the language allows; on the other, it may accept some the language prohibits.  Still, it seems to properly process most of the scripts currently available in the plugin feed.

Long lines in the output are wrapped and indented to agree with the report format, which sometimes messes up formatting that plugin authors have done.

Some older plugins list some cross-references in comments due to limitations on the number of such references that were supported in earlier versions.  They are not reported by this script since they can not be reliably identified.

*describe-nessus-plugin* may report that a plugin does not have a description part.  This occurs when a plugin has been effectively disabled in a feed; eg, `sendmail_wiz.nasl`. 

*describe-nessus-plugin* can not handle the description in `gentoo_GLSA-200411-32.nasl` and several other local checks because `Text::Balanced` fails to parse the code snippet included in the description.

If you encounter a problem with this script, I encourage you to rerun it in debug mode (eg, add `-d` to your commandline) and examine the resulting output before contacting me.  Often, this will enable you to resolve the problem by yourself.


## Copyright and License

Copyright (c) 2003-2016, George A. Theall.
All rights reserved.

This script is free software; you can redistribute it and/or modify it under the same terms as Perl itself.
