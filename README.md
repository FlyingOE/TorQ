**NOTE** The `Finance-Starter-Pack/merger` branch tracks changes to *pre-existing* TorQ files only! Files that exist only in the Finance Starter Pack has to be copied over in order to form a complete system.

![TorQ Logo](../master/html/img/AquaQ-TorQ-symbol-small.png)

[Read our documentation](https://AquaQAnalytics.github.io/TorQ/)

The framework forms the basis of a production kdb+ system by implementing some core functionality and utilities on top of kdb+, allowing developers to concentrate on the application business logic. It incorporates as many best practices as possible, with particular focus on performance, process management, diagnostic information, maintainability and extensibility. Wherever possible, we have tried to avoid re-inventing the wheel and instead have used contributed code from code.kx.com (either directly or modified). This framework will be suitable for those looking to create a new kdb+ system from scratch or those looking to add additional functionality to their existing kdb+ systems.

[Have a skim through our brochure](../master/aquaq-torq-brochure.pdf?raw=true) for a bit more information.  The easiest way to get a production capture started is to download and install one of the [Starter Packs](https://github.com/AquaQAnalytics), or [read through our Github-Pages site](https://AquaQAnalytics.github.io/TorQ/).  We also have a [Google Group for questions/discussions](https://groups.google.com/forum/#!forum/kdbtorq). 

## Quick Start

To launch a process wrapped in the framework, you need to set the environment variables and give the process a type and name.  The type and name can be explicitly passed on the command line.  setenv.sh is an example of how to set the environment variables on a unix type system.  For a windows system, see http://www.computerhope.com/issues/ch000549.htm.  kdb+ expects all paths to be / (forward-slash) separated so all paths on all OSs should be forward-slash separated. 

To avoid standard out/err being redirected, used the -debug flag
``` 
./setenv.sh         /- Assuming unix type OS
q torq.q -proctype test -procname mytest -debug
```

To load a file, use -load
```
q torq.q -load mytest.q -proctype test -procname mytest -debug
```
This will launch the a process running within the framework with all the default values.  For the rest, read the document!

## Updating the Documentation with Mkdocs

To make changes to the documentation website you must simply use this command while in the branch you have made the changes on:

`mkdocs gh-deploy`

You will be prompted to enter a username and password, after this the site should have been updated. You can test the site locally if you want using mkdocs. First use the command:

`mkdocs build`

Then:

`mkdocs serve -a YourIp:Port`

Head to the address it gives you to check if your changes have worked. More information about using mkdocs can be found [here](http://www.mkdocs.org/)

## Release Notes

- **1.0.0, Feb 2014**: 
  * Initial public release of TorQ
- **1.1.0, Apr 2014**: 
  * Added compression utilities, HTML5 utilities, housekeeping process, file alerter process, kdb+tick quick start
- **1.2.0, Sep 2014**:	
  * Tested on kdb+ 3.2
  * Added connections to external (non TorQ) processes using nonprocess.csv
  * Modified file alerter with optional switch to move or not move a file if any function fails to process the file
  * Discovery service(s) host:port(s) can be passed on the command line (.servers.DISCOVERY) to a process (this should enable complete bypassing of process.csv if required)
  * Add custom hook (.servers.connectcustom) which is invoked whenever a new connection is made (allows, for example, subscription to a new process)
  * Add optional application detail file ($KDBCONFIG/application.txt) to allow customisation of the start up banner (application version etc.)
  * If required env. variables (KDBCODE, KDBCONFIG, KDBLOG) are not set they will default to $QHOME/code, $QHOME/config, $QHOME/logs respectively (previously the process failed and exited)
- **2.0.1, May 2015**:  
  * Added RDB process which extends r.q from kdb+ tick.
  * Added WDB to write down data periodically throughout the day.  Extends w.q.
  * RDB and WDB allow seamless end-of-day event (no data outage, no tickerplant back pressure)
  * Added Reporting Process to run reports periodically and process the results
  * Added environment variable resolution to process.csv to allow greater portability.  If a process is started without a port specified it will look it up from process.csv based on the proctype and procname.
  * Added -localtime flag to allow process to run in localtime rather than GMT (log message, timer calls etc.).  The change is backwardly compatible - without -localtime flag the process will print logs etc. in GMT but can also have a different .z.P
  * Added Subscription code to manage multiple subscriptions to different data sources
  * Added email library which uses libcurl.  Used to send emails from TorQ processes
  * Added standard monitoring checks to the database code
  * Added data loader script.  Utility functions to load a directory of data into a database in chunks, sort and part at the end
  * Added tickerplant log recovery utilities to recover as many messages as possible from a log file rather than just stopping at the first bad message
  * Added compression process to run and compress a given database
  * Modified compression code to handle par.txt databases
  * Modified compression code and housekeeping process to run with kdb+ 2.*
  * Modified std out/err logging and usage logging to include process name and process type (the logmsg table had changed along with some of the functions in the .lg namespace so you might need to check in case you have overridden any of them)
  * Removed launchtick scripts and some default configuration: to create a test system, install a starter pack
- **2.1.0, July 2015**:
  * Added a chained tickerplant process
  * Updated housekeeping.csv to take in an extra column agemin which represents whether to use minutes or days in find function
  * Updated email libraries
- **2.2.0, October 2015**:
  * Added application configuration management using $KDBAPPCONFIG environmnet variable
  * Added pid, host and port to heartbeat table
  * Changed gateway to be non intrusive
  * Bug fixes
- **2.2.1, November 2015**:
  * Bug fix - fixed endofdaysort to not throw type error when par.txt is available
  * Bug fix - fixed gateway so results are not dropped when a client loses connection and another is querying multiple servers
- **2.3.0, December 2015**:
  * Added optional write down method to wdb process, which allows data to be written to custom partition schemes during the day. 
	At the end of day before being moved to the hdb, data is merged instead of sorted, which will allow the data to be accessed sooner. 
	The optional method may present a significant saving in time for datasets with a low cardinality (small distinct number of elements), i.e. FX market data
  * Added write access control to message handlers (using reval), which restricts the ability of querying clients to modify data in place
  * Added functionality to return approximate memory size of kdb+ objects
  * Bug fixes
- **2.4.0, February 2016**:
  * Added k4unit to allow tests to be automatically run via the -test parameter
  * Extended application config handling so all default config can be found in $KDBAPPCONFIG
  * Bug Fixes
- **2.5.0, April 2016**:
  * Added u.q - publish/subscribe code from KDB+ Tick
  * Improved memusage.q to do sampling
  * Updated dataloader.q
  * Make sure all config can be read from KDBAPPCONFIG
- **2.6.0, August 2016**:
  * Added broadcast publishing
  * Added domain sockets and tcps as ipc connection mechanisms
  * Added fallback from domain sockets to tcp functionality
  * New gateway (which allows attributes to be sent rather than specifying the processes to hit)
  * Heartbeat subscriptions are easier
  * Supports snappy compression
  * Tested with kdb+ 3.4
  * Bug fixes
- **2.6.2, September 2016**:
  * .z.pd and peach logic added to wdb.q
  * Bug fixes
- **2.7.0, November 2016**:
  * Tickerplant incorporated.  Tickerplant has faster recovery for real time subscribers, and better timezone handling
  * Filealerter uses a splayed table to store the table of already processed data.  If it finds a table in current format (flat) it will change it to a splay
  * Gateway bug fixes
  * Small improvements
- **3.0.0, January 2017**:
  * Added a permissioning system, allowing granular control of access to users & groups
  * Added LDAP support, allowing a user to authenticate against an LDAP server
  * Improved documentation now available at http://aquaqanalytics.github.io/TorQ/
- **3.1.0, May 2017**:
  * added kafka which is a real time messaging system with persistent storage in message logs
  * added datareplay.q with functionality for generating tickerplant function calls from historical data which can be executed by subscriber functions
  * added subscribercutoff.q with functionality that can be used to cut off slow subscribers from processes
  * added new write down method for tickerlogreplay so match the writedown methods in the WDB

===========================================================

# TorQ-Finance-Starter-Pack
An example production ready market data capture system, using randomly generated financial data.  This is installed on top of the base TorQ package, and includes a version of [kdb+tick](http://code.kx.com/wsvn/code/kx/kdb+tick).

## Set Up 

Assuming that the [free 32 bit version of kdb+](http://kx.com/software-download.php) is already set up and available from the command prompt as q, then:

1. Download a zip of the latest version of [TorQ](https://github.com/AquaQAnalytics/TorQ/archive/master.zip)
2. Download a zip of [this starter pack](https://github.com/AquaQAnalytics/TorQ-Finance-Starter-Pack/archive/master.zip)
3. Unzip TorQ
4. Unzip the starter pack over the top (this will replace some files)
5. Run the appropriate starts script: start_torq_demo.bat for Windows, start_torq_demo.sh for Linux and start_torq_demo_osx.sh for Mac OS X. 

For more information on how to configure and get started, go to [this site](https://aquaqanalytics.github.io/TorQ-Finance-Starter-Pack/).  You will need to make some modifications if you wish to send emails from the system. 

## Updating the Documentation with Mkdocs

To make changes to the documentation website you must simply use this command while in the branch you have made the changes on:

`mkdocs gh-deploy`

You will be prompted to enter a username and password, after this the site should have been updated. You can test the site locally if you want using mkdocs. First use the command:

`mkdocs build`

Then:

`mkdocs serve -a YourIp:Port`

Head to the address it gives you to check if your changes have worked. More information about using mkdocs can be found [here](http://www.mkdocs.org/)

## Release Notes

- **1.0.1, July 2015**:
  * Added Chained Tickerplant process

- **1.1.0, October 2015**:
  * REQUIRES TORQ 2.2.0
  * Added compatibility with $KDBAPPCONFIG in TorQ 2.2.0 Release
- **1.2.0, April 2016**:
  * REQUIRES TORQ 2.5.0
  * Removed u.q
  * Moved all config directory into appconfig
- **1.2.1, September 2016**:
  * REQUIRES TORQ 2.6.2
  * added broadcast functionality to u.q
  * added sortslave functionality
- **1.3.0, November 2016**:
  * REQUIRES TORQ 2.7.0
  * Removed kdb+ tick code
  * Moved KDBBASEPORT assignment to setenv.sh
  * Feed process uses timer library
