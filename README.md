# ws-apr-2022

## Preparation

Once you clone the repo, all the yaml files to be applied below are in the "manifests" folder.

### Decrease the time to collect flow logs

By default, flow logs are collected every 5 minutes. We will decrease that time to 30 seconds, which will increase the amount of information we must store, and while that is not recommended for production environments, it will help to speed up the time in which events are seen within Calico observability features.

```
kubectl patch felixconfiguration.p default -p '{"spec":{"flowLogsFlushInterval":"30s"}}'
```

## Create a Tier structure

Tiers are a hierarchical construct used to group policies and enforce higher precedence policies that cannot be circumvented by other teams. 

All Calico and Kubernetes security policies reside in tiers. You can start “thinking in tiers” by grouping your teams and types of policies within each group. The command below will create three tiers (quarantine, platform, and security):

```
kubectl create -f manifests/tiers/tiers.yaml
```
For normal policy processing (without apply-on-forward, pre-DNAT, and do-not-track), if no policies within a tier apply to endpoints, the tier is skipped, and the tier’s implicit deny behavior is not executed.

For example, if policy D in Tier 2 includes a Pass action rule, but no policy matches endpoints in Tier 3, Tier 3 is skipped, including the end of tier deny. The first policy with a matching endpoint is in Tier 4, policy J.

##


