# shy_script
A portable bash script used to delay commands until users are logged off a host

## Purpose
The main intended use for this script in my environment is to schedule a reboot to occur the next time the host is unused, to cause minimal disruption of services. The target environment is a subnet full of lab computers, which see use both from local users, and remote logins. Other uses are certainly possible.

The command vocabulary used was checked for portability across all the enterprise POSIX operating systems I use in production. Full compatibility is the goal, but testing is ongoing. See "Portability" for details.

## Usage documentation
The simplest useful shy command (and one I use often) is probably simply "shy reboot". This will schedule a reboot the next time no users are logged in to the host. After 24 hours, if the command hasn't run yet, it will expire and notify me that it timed out without executing.

Default options are mostly what I find useful in production. Some of the more useful flags and flag combinations are:  
-c : Asks the script to echo back the command it will run and allow you to confirm before it does its thing.  
-vvI : Tells the script to run the most verbosely, and without forking a daemon. This is useful for getting info about what the script is seeing, especially if it's failing to execute when it should.  
-l : (doesn't take a command to run) - Queries the pidfiles on the local host and shows you a list of all currently waiting shy jobs, and what the command they will run is. See "Logging" for more info.  
-g/-i : Ignores logins from specified or idle users. See "Ignoring users" for some useful semantic info about this flag, and what its absence implies.

Note that due to some of the limitations of the getopt feature set on Solaris, flag parsing can be invasive. Use of the "--" (end of flags) flag is highly recommended if the command to be run also takes flags of its own. Confirm with -c if uncertain.

The unabridged usage documentation from the command is as follows:  
shy [-qiInvchdl] [-g \<user[,user...]\>] [-t timeout] [--] command  
-q ~ quiet  
 *decrease verbosity level (default 2, minimum 0)*  
-v ~ verbose  
*increase verbosity level (maximum 4)*  
-i ~ ignore-idle  
*disregard logged-in users who have not been active in the last 24 hours*  
-I ~ interactive  *this is a capital i*  
*don't spawn a detached shy process, instead block calling terminal until execution*  
-c ~ confirm  
*prompt for confirmation of the payload to be executed as interpreted before going into wait*  
-n ~ no-notify  
*don't send event notices to irc*  
-d ~ deadline  
*make the timeout a deadline, not an expiry (execute is forced instead of cancelled)*  
-g \<user[,user[...]]\> ~ ignore-users  
*ignore logins from the specified user (or list of users)*  
-t \<hours\> ~ timeout  
*timeout duration (in hours, default 24)*  
-- ~ end-of-flags  
*stop parsing flags, and begin command payload*  
-l ~ list  *this is a lowercase L*  
*don't execute a command, instead list currently waiting shy commands*  
-h ~ help  
*print this text*

## Ignoring users
The -g and -i flags are used to exclude certain users from consideration when querying for logins.

The -i flag will ignore all logins that have an idle time of 24 hours or more. Note that in this case, "idle" is as determined by the "w" command. Certain kinds of login sessions don't update idle-time correctly (notably, X sessions started from a local text tty). Use this flag with caution, some environments may not be appropriate for this flag.  
Some investigation may be helpful in determining if this is a problem with your users in your environment.

The -g flag will cause shy to ignore logins belonging to the specified username (or comma-separated list of usernames) when querying for logins.  
Mostly this does what it should, but it's notable that in the case that shy is being run by a non-root user, *and* that the -g flag is *not* specified, "-g $USER" is implied. No such assumption is made when shy is run by a root user, as in some environments (such as mine), root is a shared admin account and multiple people may be logged in on the same machine as root.  
It is, of course, possible to specify "-g root" as the root user, if this is desired, or to specify some "-g nobody" if a non-root user does not wish to ignore themselves.

## Logging and notifications
As soon as a shy job runs (after successfully confirming, if the -c flag is specified), it creates a pidfile whose name reflects its own PID and current status, in /tmp/shy.pidfiles (creating the directory if necessary, with mode 1777).  
This pidfile can be catted. It contains the command payload of the corresponding shy process as its first line, and additionally serves as a log file for the process. Note that especially verbose command payloads (or specifying -vv) may cause this pidfile to grow rather large.

Pidfiles for deceased shy jobs are not removed by the process, although they can be manually deleted, and they should be flushed on a reboot. If you are running a lot of verbose shy jobs, it's theoretically possible that you could fill up /tmp with pidfiles. This is usually bad, so plan accordingly.

Notifications are not really supported in the master branch of this project yet. Syntax errors may result from the eval line inside the notify_irc() function. Specifying the -n flag should solve this problem. Here's why:  
In my environment, there is a chat bot in one of our work channels that pulls event notifications from a messaging queue. A localized branch of this repo curls an API to post event notifications about the ultimate fate of scheduled shy jobs. However since we don't want hooligans doing same, the API call has been removed, and the placeholder isn't really functional yet.  
Sorry. I intend to try to put in something a little more generalized and usable here sometime soon.

## Portability
Tested and built on Ubuntu 14.04, intended for portability to Solaris 11.2, and RHEL/CentOS 5/6/7.

Testing has yielded positive results on Ubuntu 14.04, and RHEL 6. A syntax error with tail is currently causing the script to fail on Solaris 11.2. Testing has not been done on RHEL 5 or 7 yet.

More testing to come.
