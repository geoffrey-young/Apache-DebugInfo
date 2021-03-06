# NAME

Apache::DebugInfo - log various bits of per-request data 

# SYNOPSIS

There are two ways to use this module...

    1) using Apache::DebugInfo to control debugging automatically

      httpd.conf:

        PerlInitHandler Apache::DebugInfo
        PerlSetVar      DebugInfo On

        PerlSetVar      DebugPID On
        PerlSetVar      DebugHeadersIn On
        PerlSetVar      DebugDirConfig On
        PerlSetVar      DebugHeadersOut On
        PerlSetVar      DebugNotes On
        PerlSetVar      DebugPNotes On
        PerlSetVar      DebugGetHandlers On
        PerlSetVar      DebugTimestamp On
        PerlSetVar      DebugMarkPhases On

        PerlSetVar      DebugFile     "/path/to/debug_log"
        PerlSetVar      DebugIPList   "1.2.3.4 1.2.4."
        PerlSetVar      DebugTypeList ".html .cgi"

    2) using Apache::DebugInfo on the fly

      in handler or script:

        use Apache::DebugInfo;

        my $r = shift;

        my $debug = Apache::DebugInfo->new($r);

        # set the output file
        $debug->file("/path/to/debug_log");

        # get the ip addresses for which output is enabled
        my $ip_list = $debug->ip;

        # dump $r->headers_in right now
        $debug->headers_in;

        # log $r->headers_out after the response goes to the client
        $debug->headers_in('PerlCleanupHandler');

        # log all the $r->pnotes at Fixup and at Cleanup
        $debug->pnotes('PerlCleanupHandler','PerlFixupHandler');

# DESCRIPTION

Apache::DebugInfo gives the programmer the ability to monitor various
bits of per-request data.

You can enable Apache::DebugInfo as a PerlInitHandler, in which case
it chooses what request phase to display the appropriate data.  The
output of data can be controlled by setting various variables to On:

    DebugInfo          - enable Apache::DebugInfo handler

    DebugPID           - dumps apache child pid during request init
    DebugHeadersIn     - dumps request headers_in during request init
    DebugDirConfig     - dumps PerlSetVar and PerlAddVar during request init
    DebugGetHandlers   - dumps enabled request handlers during init

    DebugHeadersOut    - dumps request headers_out during request cleanup
    DebugNotes         - dumps request notes during request cleanup
    DebugPNotes        - dumps request pnotes during request cleanup

    DebugTimestamp     - prints localtime at the start of each request
    DebugMarkPhases    - prints the name of the request phase when the
                         phase is entered, prior to any other handlers

Alternatively, you can control output activity on the fly by calling
Apache::DebugInfo methods directly (see METHODS below).

Additionally, the following optional variables hold special arguments:

    DebugFile          - absolute path of file that will store the info
                         don't forget to make the file writable by 
                         whichever user Apache runs as (likely nobody)
                         defaults to STDERR (which is likely error_log)

    DebugIPList        - a space delimited list of IP address for which
                         debugging is enabled
                         this can be a partial IP - 1.2.3 will match
                         1.2.3.5 and 1.2.3.6
                         if absent, defaults to all remote ip addresses

    DebugTypeList      - a space delimited list of file extensions for
                         which debugging is enabled (.cgi, .html...)
                         if absent, defaults to all types

# METHODS

Apache::DebugInfo provides an object oriented interface to allow you 
to call the various methods from either a module, handler, or an
Apache::Registry script.

Constructor:
  new($r)        - create a new Apache::DebugInfo object
                   requires a valid Apache request object

Methods:
  The following methods can be called without any arguments, in which
  case the associated data is output immediately.  Optionally, each
  can be called with a list (either explicitly or as an array) of 
  Perl\*Handlers, which will log the data during the appropriate
  phase:

    headers_in()   - display incoming HTTP headers

    headers_out()  - display outgoing HTTP headers

    notes()        - display strings set by $r->notes

    pnotes()       - display variables set by $r->pnotes

    pid()          - display the apache child process PID

    get_handlers() - display variables set by PerlSetVar and PerlAddVar

    dir_config()   - display the enabled handlers for this request

    timestamp()    - display the current system time

    mark_phases()  - display the phase before executing any other
                     handlers. if given the argument 'All', 
                     mark_phases  will display the entry into all
                     phases after the current phase.  calling with
                     no arguments outputs the current phase 
                     immediately.

    There are also the following methods available for manipulating
    the behavior of the above methods:

    file($file)    - get or set the output file
                     accepts an absolute filename as an argument
                     returns the output filehandle
                     defaults to, but overrides DebugFile above

    ip($list)      - get or set the ip list
                     accepts a space delimited list as an argument
                     defaults to, but overrides DebugIPList above

    type($list)    - get or set the file type list
                     accepts a space delimited list as an argument
                     defaults to, but overrides DebugTypeList above

# NOTES

Setting DebugInfo to Off has no effect on the ability to make direct
method calls.  

Verbose debugging is enabled by setting the variable
$Apache::DebugInfo::DEBUG=1 to or greater.  To turn off all messages
set LogLevel above info.

This is alpha software, and as such has not been tested on multiple
platforms or environments.  It requires PERL\_INIT=1, PERL\_CLEANUP=1,
PERL\_LOG\_API=1, PERL\_FILE\_API=1, PERL\_STACKED\_HANDLERS=1, and maybe 
other hooks to function properly.

# FEATURES/BUGS

Once a debug handler is added to a given request phase, it can
no longer be controlled by ip() or type(). file(), however, takes
affect on invocation.  This is because the matching is done when
the Perl\*Handler is added to the stack, while the output file is
used when the Perl\*Handler is actually executed.

Calling Apache::DebugInfo methods with 'PerlHandler' as an argument
has been disabled - doing so gets your headers and script printed
to the browser, so I thought I'd save the unaware from potential 
pitfalls.

Phase misspellings, like 'PelrInitHandler' pass through without
warning, in case you were wondering where your output went...

The get\_handlers and mark\_phases methods are incomplete, mainly due to
oversights in the mod\_perl API.  Currently (as of mod\_perl 1.2401),
they cannot function properly on the following callbacks: 
  PerlInitHandler
As such, they have been disabled until forthcoming corrections to the
API can be implemented.  PerlHeaderParserHandlers and 
PerlPostRequestHandlers function normally.

The output uri is whatever the uri was when new() was called (either
on the fly or in Apache::DebugInfo::handler).  Thus if the uri has
undergone translation since the new() call the original, not the new,
uri will be output.  This feature can be easily remedied, but having a
changing uri in the output may be confusing when debugging.  Future
behavior will be influenced by user feedback.

# SEE ALSO

perl(1), mod\_perl(1), Apache(3)

# AUTHOR

Geoffrey Young <geoff@cpan.org>

# COPYRIGHT

Copyright (c) 2000, Geoffrey Young.  All rights reserved.

This module is free software.  It may be used, redistributed
and/or modified under the same terms as Perl itself.
