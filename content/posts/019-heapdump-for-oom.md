---
title: "Bir OOM Hikayesi ve İş görebilir bir Script"
date: 2024-10-26T16:09:14+03:00
draft: false
---

Basic Heapdump Script 

``` bash
#!/bin/bash
_JAVA_HOME="/data/jdk/latest"
_SEARCH_TXT="/my-app/"
_DUMP_DEST="/tmp"
_FILE_NAME="my_app_`hostname -i`_`date '+%Y%m%d_%H%M'`"
_HEALTH_LOG_FILE="/var/log/nginx/access-health-`date +"%Y%m%d"`.log"

_BUCKET_NAME="my-bucket-name"
_BUCKET_DEST="/java-dumps/"

_CPU_THRESHOLD=60
_HEAP_THRESHOLD=0.7
_HEALTH_THRESHOLD=1.5 # LB healtcheck threshold is 2 sec
 
# Get the current CPU usage (user+system) as a percentage
cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')

# get avg of last 10 /health api response time
health_res_avg=$(tail -10 $_HEALTH_LOG_FILE | sed 's/"//g' | awk '{sum += $(NF-2); count++} END {if (count > 0) printf "%.3f\n", sum / count}')

# Check if CPU usage is greater _CPU_THRESHOLD than  & LB nginx health _HEALTH_THRESHOLD
if (( $(echo "$cpu_usage > $_CPU_THRESHOLD " | bc -l) )) || (( $(echo "$health_res_avg > $_HEALTH_THRESHOLD" | bc -l) )); then

        echo "Cpu usage: $cpu_usage"
        echo "Last 10 health resp. time avg: $health_res_avg"

        _PID=`ps -ef | grep java | grep $_SEARCH_TXT | awk '{print $2}'`

        # thread dump 
        x=1; while [ $x -le 3 ]; do $_JAVA_HOME/bin/jstack $_PID >>  $_DUMP_DEST/$_FILE_NAME.tdump; sleep 7;(( x++ )); done
        
        in_use_heap=$($_JAVA_HOME/bin/jstat -gc $_PID | tail -1 | awk '{printf "%10.2f",($3+$4+$6+$8)/1024}' | tr -d ' ')
        max_heap=$($_JAVA_HOME/bin/jstat -gc $_PID | tail -1 | awk '{printf "%10.2f",($1+$2+$5+$7)/1024}' | tr -d ' ')

        # check for non-zero division
        if [[ $in_use_heap > 0 ]];then

                heap_usage_rate=`awk "BEGIN {print $in_use_heap/$max_heap}"`
                echo "Heap Usage Rate: $heap_usage_rate"
                
                # if total use over the threshold
                if [[ $heap_usage_rate > $_HEAP_THRESHOLD ]];then

                        echo "Taking the heap dump... "

                        $_JAVA_HOME/bin/jcmd $_PID GC.heap_dump $_DUMP_DEST/$_FILE_NAME.hprof

                        aws s3 cp $_DUMP_DEST/$_FILE_NAME s3://$_BUCKET_NAME$_BUCKET_DEST

                        rm $_DUMP_DEST/$_FILE_NAME
                fi
        fi

fi

```