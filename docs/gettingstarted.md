
**TorQ** -- Getting Started
===========================

kdb+ is very customisable. Customisations are contained in q scripts (.q
files), which define functions and variables which modify the behaviour
of a process. Every q process can load a single q script, or a directory
containing q scripts and/or q data files. Hooks are provided to enable
the programmer to apply a custom function to each entry point of the
process (.z.p\*), to be invoked on the timer (.z.ts) or when a variable
in the top level namespace is amended (.z.vs). By default none of these
hooks are implemented.

We provide a codebase and a single main script, torq.q. torq.q is
essentially a wrapper for bespoke functionality which can load other
scripts/directories, or can be sourced from other scripts. Whenever
possible, torq.q should be invoked directly and used to load other
scripts as required. torq.q will:

-   ensure the environment is set up correctly;

-   define some common utility functions (such as logging);

-   execute process management tasks, such as discovering the name and
    type of the process, and re-directing output to log files;

-   load configuration;

-   load the shared code based;

-   set up the message handlers;

-   load any required bespoke scripts.

The behavior of torq.q is modified by both command line parameters and
configuration. We have tried to keep as much as possible in
configuration files, but if the parameter either has a global effect on
the process or if it is required to be known before the configuration is
read, then it is a command line parameter.

<a name="torq"></a>

Using torq.q
------------

torq.q can be invoked directly from the command line and be set to
source a specified file or directory. torq.q requires the 5 environment
variables to be set (see section envvar). If using a unix
environment, this can be done with the setenv.sh script. To start a
process in the foreground without having to modify any other files (e.g.
process.csv) you need to specify the type and name of the process as
parameters. An example is below.

    $ . setenv.sh
    $ q torq.q -debug -proctype testproc -procname test1 

To load a file, do:

    $ q torq.q -load myfile.q -debug -proctype testproc -procname test1

It can also be sourced from another script. If this is the case, some of
the variables can be overridden, and the usage information can be
modified or extended. Any variable that has a definition like below can
be overridden from the loading script.

    myvar:@[value;`myvar;1 2 3]

The available command line parameters are:

  |Cmd Line Param|            Description|
  |-------------------------| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
  |-procname x -proctype y   |The process name and process type|
  |-procfile x               |The name of the file to get the process information from|
  |-load x \[y..z\]          |The files or database directory to load|
  |-loaddir x \[y..z\]       |Load all .q, .k files in specified directories|
  |-localtime                |Sets processes running in local time rather than GMT for log messages, timer calls etc. The change is backwards compatible; without -localtime flag the process will print logs etc. in GMT but can also have a different .z.P|
  |-trap                     |Any errors encountered during initialization when loading external files will be caught and logged, processing will continue|
  |-stop                     |Stop loading the file if an error is encountered but do not exit|
  |-noredirect               |Do not redirect std out/std err to a file (useful for debugging)|
  |-noredirectalias          |Do not create an alias for the log files (aliases drop any suffix e.g. timestamp suffix)|
  |-noconfig                 |Do not load configuration|
  |-nopi                     |Reset the definition of .z.pi to the initial value (useful for debugging)|
  |-debug                    |Equivalent to \[-nopi -noredirect\]|
  |-usage                    |Print usage info and exit|

  

In addition any process variable in a namespace (.\*.\*) can be
overridden from the command line. Any value supplied on the command line
will take priority over any other predefined value (.e.g. in a
configuration or wrapper). Variable names should be supplied with full
qualification e.g. -.servers.HOPENTIMEOUT 5000.

<a name="env"></a>

Environment Variables 
---------------------

Five environment variables are required:

  |Environment Variable|   Description|
  |----------------------| -----------------------------------------------------|
  |KDBCONFIG              |The base configuration directory|
  |KDBCODE                |The base code directory|
  |KDBLOGS                |Where standard out/error and usage logs are written|
  |KDBHTML                |Contains HTML files|
  |KDBLIB                 |Contains supporting library files|


torq.q will check for these and exit if they are not set. If torq.q is
being sourced from another script, the required environment variables
can be extended by setting .proc.envvars before loading torq.q.

<a name="procid"></a>

Process Identification
----------------------

At the crux of AquaQ TorQ is how processes identify themselves. This is
defined by two variables - .proc.proctype and .proc.procname which are
the type and name of the process respectively. These two values
determine the code base and configuration loaded, and how they are
connected to by other processes. If both of these are not defined, the
TorQ will attempt to use the port number a process was started on to
determine the code base and configuration loaded.

The most important of these is the proctype. It is up to the user to
define at what level to specify a process type. For example, in a
production environment it would be valid to specify processes of type
“hdb” (historic database) and “rdb” (real time database). It would also
be valid to segregate a little more granularly based on approximate
functionality, for example “hdbEMEA” and “hdbAmericas”. The actual
functionality of a process can be defined more specifically, but this
will be discussed later. The procname value is used solely for
identification purposes. A process can determine its type and name in a
number of ways:

1.  From the process file in the default location of
    $KDBCONFIG/process.csv;

2.  From the process file defined using the command line parameter
    -procfile;

3.  From the port number it is started on, by referring to the process
    file for further process details;

4.  Using the command line parameters -proctype and -procname;

5.  By defining .proc.proctype and .proc.procname in a script which
    loads torq.q.

For options 4 and 5, both parameters must be defined using that method
or neither will be used (the values will be read from the process file).

For option 3, TorQ will check the process file for any entries where the
port matches the port number it has been started on, and deduce it’s
proctype and procname based on this port number and the corresponding
hostname entry.

The process file has format as below.

    aquaq$ cat config/process.csv 
    host,port,proctype,procname
    aquaq,9997,rdb,rdb_europe_1
    aquaq,9998,hdb,hdb_europe_1
    aquaq,9999,hdb,hdb_europa_2

The process will read the file and try to identify itself based on the
host and port it is started on. The host can either be the value
returned by .z.h, or the ip address of the server. If the process can
not automatically identify itself it will exit, unless proctype and
procname were both passed in as command line parameters. If both of
these parameters are passed in then default configuration settings will
be used.

<a name="logging"></a>

Logging
-------

By default, each process will redirect output to a standard out log and
a standard error log, and create aliases for them. These will be rolled
at midnight on a daily basis. They are all written to the $KDBLOGS
directory. The log files created are:

  |Log File|                          Description|
  |---------------------------------| ----------------------------|
  |out\_\[procname\]\_\[date\].log   |Timestamped out log|
  |err\_\[procname\]\_\[date\].log   |Timestamped error log|
  |out\_\[procname\].log             |Alias to current log log|
  |err\_\[procname\].log             |Alias to current error log|

 
The date suffix can be overridden by modifying the .proc.logtimestamp
function and sourcing torq.q from another script. This could, for
example, change the suffixing to a full timestamp.

<a name="config"></a>

Configuration Loading
---------------------

### Default Configuration Loading

Default process configuration is contained in q scripts, and stored in
the $KDBCONFIG /settings directory. Each process tries to load all the
configuration it can find and will attempt to load three configuration
files in the below order:-

-   default.q: default configuration loaded by all processes. In a
    standard installation this should contain the superset of
    customisable configuration, including comments;

-   [proctype].q: configuration for a specific process type;

-   [procname].q: configuration for a specific named process.

The only one which should always be present is default.q. Each of the
other scripts can contain a subset of the configuration variables, which
will override anything loaded previously.

### Application Configuration Loading

Application specific configuration can be stored in a user defined
directory and made visible to TorQ by setting the $KDBAPPCONFIG
environment variable. If $KDBAPPCONFIG is set, then TorQ will search
the $KDBAPPCONFIG/settings directory and load all configuration it can
find. Application configuration will be loaded after all default
configuration in the following order:-

-   default.q: Application default configuration loaded by all
    processes.

-   [\[proctype\]]{}.q: Application specific configuration for a
    specific process type.

-   [\[procname\]]{}.q: Appliction specific configuration for a specific
    named process.

All loaded configuration will override anything loaded previously. None
of the above scripts are required to be present and can contain a subset
of the default configuration variables from the default configuration
directory.

All configuration is loaded before code.

<a name="code"></a>

Code Loading
------------

Code is loaded from the $KDBCODE directory. There is also a common
codebase, a codebase for each process type, and a code base for each
process name, contained in the following directories and loaded in this
order:

-   $KDBCODE/common: shared codebase loaded by all processes;

-   $KDBCODE/\[proctype\]: code for a specific process type;

-   $KDBCODE/\[procname\]: code for a specific process name;

For any directory loaded, the load order can be specified by adding
order.txt to the directory. order.txt dictates the order that files in
the directory are loaded. If a file is not in order.txt, it will still
be loaded but after all the files listed in order.txt have been loaded.

In addition to loading code form $KDBCODE, application specific code can be 
saved in a user defined directory with the same structure as above, and made
visible to TorQ by setting the $KDBAPPCODE environment variable.

If this environment variable is set, TorQ will load codebase in the following order.

-   $KDBCODE/common: shared codebase loaded by all processes;

-   $KDBAPPCODE/common: application specific code shared by all processes;

-   $KDBCODE/\[proctype\]: code for a specific process type;

-   $KDBAPPCODE/\[proctype\]: application specific code for a specific process type;

-   $KDBCODE/\[procname\]: code for a specific process name;

-   $KDBAPPCODE/\[procname\]: application specific code for a specific process name;



Additional directories can be loaded using the -loaddir command line
parameter.

<a name="init"></a>

Initialization Errors
---------------------

Initialization errors can be handled in different ways. The default
action is any initialization error causes the process to exit. This is
to enable fail-fast type conditions, where it is better for a process to
fail entirely and immediately than to start up in an indeterminate
state. This can be overridden with the -trap or -stop command line
parameters. With -trap, the process will catch the error, log it, and
continue. This is useful if, for example, the error is encountered
loading a file of stored procedures which may not be invoked and can be
reloaded later. With -stop the process will halt at the point of the
error but will not exit. Both -stop and -trap are useful for debugging.

**TorQ Finance Starter Pack** -- Getting Started
================================================

Requirements
------------

The TorQ Finance Starter Pack will run on Windows, Linux or OSX. It
contains a small initial database of 130MB. As the system runs, data is
fed in and written out to disk. We recommend that it is installed with
at least 2GB of free disk space, on a system with at least 4GB of RAM.
Chrome and Firefox are the supported web browsers.

It is assumed that most users will be running with the free 32-bit
version of kdb+. TorQ and the TorQ demo pack will run in exactly the
same way on both the 32-bit and 64-bit versions of kdb+.

Installation and Configuration
------------------------------

### Installation

1.  Download and install kdb+ from [Kx Systems](http://kx.com)

2.  Download the main TorQ codebase from
    [here](https://github.com/AquaQAnalytics/TorQ/archive/master.zip)

3.  Download the TorQ Finance Starter Pack from
    [here](https://github.com/AquaQAnalytics/TorQ-Finance-Starter-Pack/archive/master.zip)[

4.  Unzip the TorQ package

5.  Unzip the Demo Pack over the top of the main TorQ package

### Configuration

There are additional optional configuration steps depending on whether
you want to run TorQ across multiple machines and whether you wish to
generate emails from it. Note that if you are sending emails from an
email account which requires SSL authentication from Windows (e.g.
Hotmail, Gmail) then there are some additional steps outlined in the
main TorQ document which should be followed. To run TorQ across machines
you will need to:

1.  Modify config/process.csv to specify the host name of the machine
    where the process runs. In the “host” column of the csv file, input
    the hostname or IP address

If you wish to generate emails from the system you will additionally
have to:

1.  Modify DEMOEMAILRECEIVER environment variable at the top of
    start\_torq\_demo.sh, start\_torq\_demo\_osx.sh or
    start\_torq\_demo.bat

2.  Add the email server details in config/settings/default.q. You will
    need to specify the email server URL, username and password. An
    example is:

        // configuration for default mail server
        \d .email
        enabled:1b
        url:`$"smtp://smtp.email.net:80"        // url of email server
        user:`$"testaccount@aquaq.co.uk"        // user account to use to send emails
        password:`$"testkdb"                    // password for user account

Note that on Windows there may be pop up warnings about missing
libraries. These should be resolved by sourcing the correct libraries.

Start Up
--------

### Windows

Windows users should use start\_torq\_demo.bat to start the system, and
stop\_torq\_demo.bat to stop it. start\_torq\_demo.bat will produce a
series of command prompt. Each one of these is a TorQ process.

![Windows Start Up](graphics/windowslaunch.png)

Windows users should note that on some windows installations the
processes sometimes fail to start correctly and become blocked. The
issue appears to be how the processes connect to each other with
connection timeouts not being executed correctly. During testing, we
obsverved this behaviour on two different windows installations though
could not narrow it down to a specific hardware/windows/kdb+ version
issue. Most versions of windows ran correctly every time (as did all
versions of Linux/OSX).

### Linux and OSX

Linux users should use start\_torq\_demo.sh to start the system, and
stop\_torq\_demo.sh to stop it. OSX users should use
start\_torq\_demo\_osx.sh to start the system, and stop\_torq\_demo.sh
to stop it. The only difference between the respective start scripts is
how the library path environment variable is set. The processes will
start in the background but can be seen using a ps command, such as

    aquaq> ps -ef | grep 'torq\|tickerplant' 
    aquaq    4810 16777  0 15:56 pts/34   00:00:00 grep torq\|tickerplant
    aquaq   25465     1  0 13:05 pts/34   00:00:05 q torq.q -load code/processes/discovery.q -stackid 6000 -proctype discovery -procname discovery1 -U config/passwords/accesslist.txt -localtime
    aquaq   25466     1  0 13:05 pts/34   00:00:29 q tickerplant.q database hdb -stackid 6000 -proctype tickerplant -procname tickerplant1 -U config/passwords/accesslist.txt -localtime
    aquaq   25478     1  0 13:05 pts/34   00:00:17 q torq.q -load code/processes/rdb.q -stackid 6000 -proctype rdb -procname rdb1  -U config/passwords/accesslist.txt -localtime -g 1 -T 30
    aquaq   25479     1  0 13:05 pts/34   00:00:04 q torq.q -load hdb/database -stackid 6000 -proctype hdb -procname hdb1 -U config/passwords/accesslist.txt -localtime -g 1 -T 60 -w 4000
    aquaq   25480     1  0 13:05 pts/34   00:00:05 q torq.q -load hdb/database -stackid 6000 -proctype hdb -procname hdb1 -U config/passwords/accesslist.txt -localtime -g 1 -T 60 -w 4000
    aquaq   25481     1  0 13:05 pts/34   00:00:06 q torq.q -load code/processes/gateway.q -stackid 6000 -proctype gateway -procname gateway1 -U config/passwords/accesslist.txt -localtime -g 1 -w 4000
    aquaq   25482     1  0 13:05 pts/34   00:00:06 q torq.q -load code/processes/monitor.q -stackid 6000 -proctype monitor -procname monitor1 -localtime
    aquaq   25483     1  0 13:05 pts/34   00:00:07 q torq.q -load code/processes/reporter.q -stackid 6000 -proctype reporter -procname reporter1 -U config/passwords/accesslist.txt -localtime
    aquaq   25484     1  0 13:05 pts/34   00:00:04 q torq.q -load code/processes/housekeeping.q -stackid 6000 -proctype housekeeping -procname housekeeping1 -U config/passwords/accesslist.txt -localtime
    aquaq   25485     1  0 13:05 pts/34   00:00:05 q torq.q -load code/processes/wdb.q -stackid 6000 -proctype sort -procname sort1 -U config/passwords/accesslist.txt -localtime -g 1
    aquaq   25486     1  0 13:05 pts/34   00:00:13 q torq.q -load code/processes/wdb.q -stackid 6000 -proctype wdb -procname wdb1  -U config/passwords/accesslist.txt -localtime -g 1
    aquaq   25547     1  0 13:05 pts/34   00:00:13 q torq.q -load tick/feed.q -stackid 6000 -proctype feed -procname feed1 -localtime

### Check If the System Is Running

TorQ includes a basic monitoring application with a web interface,
served up directly from the q process. The monitor checks if each
process is heartbeating, and will display error messages which are
published to it by the other processes. New errors are highlighted,
along with processes which have stopped heartbeating.

![Monitor UI](graphics/monitor_ui_new.png)

The monitor UI can be accessed at the address
http://hostname:monitorport/.non?monitorui where hostname is the
hostname or IP address of the server running the monitor process, and
monitor port is the port. The default monitor port is 6009. Note that
the hostname resolution for the websocket connection doesn’t always
happen correctly- sometimes it is the IP address and sometimes the
hostname, so please try both. To see exactly what it is being returned
as, open a new q session on the same machine and run:

    q)ss[html;"KDBCONNECT"] _ html:`::6009:admin:admin "monitorui[]"
    "KDBCONNECT.init(\"server.aquaq.co.uk\",6009);\n</script>\n    </body>\n</html>\n"

### Connecting To A Running Process

Any of the following can be used to easily interrogate a running q
process.

-   another q process, by opening a connection and sending commands

-   qcon

-   an IDE

The remainder of this document will use either qcon or an IDE. Each
process is password protected but the user:password combination of
admin:admin will allow access.

### Testing Emails

If you have set up emailing, you can test is using the .email.test
function (from any process). This takes a single parameter of the email
address to send a test email to. It returns the size of the email sent
in bytes upon success, or -1 for failure.

    aquaq$ qcon :6002:admin:admin
    :6002>.email.test[`$"testemail@gmail.com"]                                                                                                                                                        
    16831i

To extract more information from the email sending process, set
.email.debug to 2i.

    :6002>.email.debug:2i                                                                                                                                                                                 
    :6002>.email.test[`$"testemail@gmail.com"]                                                                                                                                                        
    16831i

Trouble Shooting
----------------

The system starts processes on ports in the range 6000 to 6014. If there
are processes already running on these ports there will be a port clash-
change the port used in both the start script and in the process.csv
file.

All the processes logs to the $KDBLOG directory. In general each
process writes three logs: a standard out log, a standard error log and
a usage log (the queries which have been run against the pro cess
remotely). Check these log files for errors.

### Debugging

The easiest way to debug a process is to run it in the foreground. By
default, TorQ will redirect standard out and standard error to log files
on disk. To debug a process, start it on the command line (either the
command prompt on Windows, or a terminal session on Linux or OSX) using
the start up line from the appropriate launch script. Supply the -debug
command line parameter to stop it redirecting output to log files on
disk.

If the process hits an error on startup it will exit. To avoid this, use
either -stop or -trap command line flag. -stop will cause the process to
stop at the error, -trap will cause it to trap it and continue loading.
An example is below. This query should be run from within the directory
you have extracted TorQ and the TorQ Finance Starter Pack to.

    q torq.q -load code/processes/rdb.q -stackid 6000 -proctype rdb -procname rdb1 -U config/passwords/accesslist.txt -localtime -g 1 -T 30 -debug -stop

File Structure
--------------

The file structure can be seen below.

    |-- AquaQTorQFinanceStarterPack.pdf
    |-- LICENSE
    |-- README.md
    |-- appconfig
    |   `-- settings        <- modified settings for each process
    |       |-- compression.q
    |       |-- feed.q
    |       |-- gateway.q
    |       |-- killtick.q
    |       |-- monitor.q
    |       |-- rdb.q
    |       |-- sort.q
    |       |-- tickerplant.q
    |       `-- wdb.q
    |-- code
    |   |-- common
    |   |   `-- u.q         <- kdb+ tick pubsub script
    |   |-- hdb         <- extra functions loaded by hdb procs
    |   |   `-- examplequeries.q
    |   |-- processes
    |   |   `-- tickerplant.q
    |   |-- rdb         <- extra functions loaded by rdb procs
    |   |   `-- examplequeries.q
    |   `-- tick            <- kdb+ tick
    |       |-- feed.q      <- dummy feed from code.kx
    |       |-- tick        
    |       |   |-- database.q  <- schema definition file
    |       |   |-- r.q
    |       |   `-- u.q
    |       `-- tick.q      <- kdb+ tick
    |-- config
    |   |-- application.txt     <- TorQ demo pack banner
    |   |-- compressionconfig.csv   <- modified compression config
    |   |-- housekeeping.csv    
    |   |-- passwords
    |   |   |-- accesslist.txt  <- list of user:pass who can connect to proccesses
    |   |   `-- feed.txt        <- password file used by feed for connections
    |   |-- process.csv     <- definition of type/name of each process
    |   `-- reporter.csv        <- modified config for reporter
    |-- hdb             <- example hdb data
    |   `-- database
    |       |-- 2015.01.07
    |       |-- 2015.01.08
    |       `-- sym
    |-- setenv.sh           <- set environment variables
    |-- start_torq_demo.bat     <- start and stop scripts
    |-- start_torq_demo.sh
    |-- start_torq_demo_osx.sh
    |-- stop_torq_demo.bat
    `-- stop_torq_demo.sh

The Demo Pack consists of:

-   a slightly modified version of kdb+tick from Kx Systems

-   an example set of historic data

-   configuration changes for base TorQ

-   additional queries to run on the RDB and HDB

-   start and stop scripts

Make It Your Own
----------------

The system is production ready. To customize it for a specific data set,
modify the schema file and replace the feed process with a feed of data
from a live system.

