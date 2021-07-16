

```shell

netstat -ant | grep 8001 |awk '/^tcp/ {++S[$NF]} END {for(a in S) print (a,S[a])}'
netstat -ant|awk '/^tcp/ {++S[$NF]} END {for(a in S) print (a,S[a])}'


/opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-server start
imcall.sh /opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-agent start


/opt/cloudera-manager/cm-5.12.1/log/cloudera-scm-server/cloudera-scm-server.log
/opt/cloudera-manager/cm-5.12.1/log/cloudera-scm-agent/cloudera-scm-agent.log


imcall.sh /opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-agent stop
/opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-server stop
```

