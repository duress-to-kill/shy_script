# shy_script
A portable bash script used to delay commands until users are logged off a host

Tested and build on Ubuntu 14.04, intended for portability to Solaris 11.2, and RHEL/CentOS 5/6/7.
More testing to come.

## Purpose
shy was written mostly to facilitate user-friendly patches and reboots in a live production environment. Its main use case in my environment will probably be to delay reboots until a workstation is not being used.

The command vocabulary used was checked for portability across all the enterprise POSIX operating systems I use in production, but the final product hasn't been tested yet. Full compatibility is the goal.

## Usage documentation
shy [-qiInvchdl] [-g <user[,user...]>] [-t timeout] [--] command
  -q -- quiet           decrease verbosity level (default 2, minimum 0)
  -v -- verbose         increase verbosity level (maximum 4)
  -i -- ignore-idle     disregard logged-in users who have not been active in the last 24 hours
  -I -- interactive     don't spawn a detached shy process, instead block calling terminal until execution
  -c -- confirm         prompt for confirmation of the payload to be executed as interpreted before going into wait
  -n -- no-notify       don't send event notices to irc
  -d -- deadline        make the timeout a deadline, not an expiry (execute is forced instead of cancelled)
  -g <user[,user[...]]> ignore logins from the specified user (or list of users)
  -t <timeout>          timeout duration (in hours, default 24)
  -l -- list            don't execute a command, instead list currently waiting shy commands
  -h -- help            print this text
