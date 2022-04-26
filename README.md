# Hands-on EKS Workshop

## Preparation

Once you clone the repo, all the yaml files to be applied below are in the "manifests" folder.

### Create your EKS cluster

Calico can be used as a CNI, or you can decide to use AWS VPC networking and have Calico only as plugin for the security policies. 

We will use the second approach during the workshop. Below an example on how to create a two nodes cluster with an smaller footprint, but feel free to create your EKS cluster with the parameters you prefer. Do not forget to include the region if different than the default on your account.

```
eksctl create cluster --name <CLUSTER_NAME> --version 1.21 --node-type t3.large
```

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

![endpoint-match](./img/endpoint-match.svg)

## Deploy an application

We included an small test application, but you can use your own:

```
kubectl create -f manifests/deployments/yaobank.yaml
```

If using the test application above, expose the frontend service in your EKS cluster:

```
kubectl expose svc customer -n yaobank --type LoadBalancer --name yaobank --port 80
```

If you check your services in the yaobank namespace, you should have an external FQDN associated with the service you created above pointing to an AWS LB. Check you can resolve that FQDN, and then verify you can reach the yaobank application in your browser.

```
kubectl get svc -n yaobank
```

You will see the yaobank deployment the corresponding label "pci=true", that label will be matched with our policies to isolate the PCI workloads, and show how Calico's tiered security policies work.









