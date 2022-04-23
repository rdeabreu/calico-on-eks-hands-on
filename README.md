# ws-apr-2022

## Decrease the time to collect flow logs

```
kubectl patch felixconfiguration.p default -p '{"spec":{"flowLogsFlushInterval":"30s"}}'
```


