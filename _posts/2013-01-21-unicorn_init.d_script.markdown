---
layout: post
title: "Unicorn init.d script"
---

```bash
#!/bin/bash
#  
# description: Stats service init file
#
# NOTE: for properly execution, the script expects to found unicorn
# executable in $PROJ\_DIR/bin directory
#  
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

## configure your variables here
PID_FILE=/path/to/pidfile.pid
PROJ_DIR=/path/to/project
LOG_FILE=/path/to/log
TAG='[ProjName]'
UNICORN_CONFIG=/path/to/unicorn.rb

OLD_PID_FILE="$PID_FILE.oldbin"

# Start the service
start() { 
  if [[ -f $PID_FILE ]]; then
    echo "There is a PID file in $PID_FILE. Service should be running"
    exit 1
  else
    echo "Starting server: "
    echo "$(date) $TAG Trying to start server..." >> $LOG_FILE
    su - stats -c "bash -c 'cd $PROJ_DIR ; bin/unicorn -c $UNICORN_CONFIG -E production -D'"
    if [[ $? == 0 ]]; then
      echo "$(date) $TAG server started" >> $LOG_FILE
      echo "Ok"
    else
      echo "$(date) $TAG FAILED to start server" >> $LOG_FILE
      echo "Failure"
      exit 1
    fi
  fi
}  

# Stop the service
stop() {
  echo "Stopping server: "
  echo "$(date) $TAG Trying to stop server..." >> $LOG_FILE
  kill -QUIT $(cat $PID_FILE)
  if [[ $? == 0 ]]; then
    echo "$(date) $TAG Server started" >> $LOG_FILE
    echo "Ok"
  else
    echo "$(date) $TAG FAILED to start server" >> $LOG_FILE
    echo "Failure"
    exit 1
  fi
}

# Reload service
reload() {
  echo -n "Reloading server: "
  echo "$(date) $TAG Trying to reload server" >> $LOG_FILE
  kill -USR2 $(cat $PID_FILE)
  sleep 1
  if [[ -f $OLD_PID_FILE ]]; then
    echo "$(date) $TAG Server reloaded" >> $LOG_FILE
    echo "Ok"
    echo "$(date) $TAG Killing old instances..." >> $LOG_FILE
    echo -n "Trying to kill old instances: "
    kill -QUIT $( cat $OLD_PID_FILE )
    echo "$(date) $TAG Old instances killed" >> $LOG_FILE
    echo "Ok"
  else
    echo "$(date) $TAG FAILED to reload server" >> $LOG_FILE
    echo "Failure"
    exit 1
  fi
}
### main logic ###
case "$1" in
start)
  start
  ;;
stop)
  stop
  ;;

restart)
  stop
  sleep 1
  start
  ;;
reload)
  reload
  ;;
*)
  echo $"Usage: $0 {start|stop|restart|reload}"
  exit 1
esac

exit 0
```
