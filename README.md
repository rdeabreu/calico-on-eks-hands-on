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

### Connect your cluster to Calico Cloud

Go to the "Managed clusters" section in Calico Cloud once you login, and click on the "Connect Cluster" button, then leave "Amazon EKS" selected, and give a name to your cluster, and click "Next". Read the cluster requirements in teh next section, and click "Next". Finally, copy the kubectl command you must run in order to connect your cluster to the management cluster for your Calico Cloud instance.

![managed-clusters](./img/managed-clusters.png)

## Reduce the Attack surface of your environment, and follow a Zero Trust approach

### Create a Tier structure

Tiers are a hierarchical construct used to group policies and enforce higher precedence policies that cannot be circumvented by other teams. 

All Calico and Kubernetes security policies reside in tiers. You can start “thinking in tiers” by grouping your teams and types of policies within each group. The command below will create three tiers (quarantine, platform, and security):

```
kubectl create -f manifests/tiers/tiers.yaml
```
For normal policy processing (without apply-on-forward, pre-DNAT, and do-not-track), if no policies within a tier apply to endpoints, the tier is skipped, and the tier’s implicit deny behavior is not executed.

For example, if policy D in Tier 2 includes a Pass action rule, but no policy matches endpoints in Tier 3, Tier 3 is skipped, including the end of tier deny. The first policy with a matching endpoint is in Tier 4, policy J.

![endpoint-match](./img/endpoint-match.svg)

### Deploy an application

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

### Apply the Security Policies

Apply all the policies in the directory below:

```
kubectl create -f manifests/netpol
```

That will create a quarantine policy we will use later, another policy to protect our coredns service, and a third policy in case we want to enforce traffic on the k8s nodes themselves.

Now let's create the polcies in the folder below:

```
kubectl create -f manifests/netpol/ws/
```

That will cerate a deny all rule which will catch up all the traffic that has not been matched explicitly, so it will allow us to implement a Zero-Trust approach in our environment. For a further explanation on how a Zero-Trust model can help you to mitigate threats in your environment, and other security policies recommendations and best practices, please check: 

https://docs.tigera.io/security/policy-best-practices

Finally, let's implement some microsegmentation rules for our yaobank application:

```
kubectl create -f manifests/netpol/additional/yaobank 
```

Now you will not be able to reach the application anymore, as we have effectively isolated all endpoints in our environment labeled as pci=true with a single policy, in combination with the default deny rule we implemented at the end of our policy chain.

### Policy recommendation

Now let's see how Calico can help us to build a microsegmentation policy in order to allow the traffic to our frontend service.

Click in the Policy Recommendation button in the Policy Board:

![policy-recommendation](./img/policy-recommendation.png)

Now select the time range you will look back in the flow logs to recommend a policy based on them. You must have tried to access the yaobank service, select the time range to include those attempts, and at least allow for the "flowLogsFlushInterval" you configured in the preparation section, otheriwse, you will not retrieve the data needed.

Select the namespace of the application we want the recommended policy for (yaobank), and the right service (customer-<hash>). Unselect the "Unprotected only" box.

When you click on the "Recommend" button in the top right corner, you will see that Calico recommends to open the traffic to port 80 on Ingress, so we would be able to reach the frontend application again. Click on "Enforce", and then the "Back" button.
  
The policy will be created at the end of your policy chain (at the bottom of the default Tier). You must move the policy to the right order, so it can have effect. In our case, as we would like to hit this policy before the pci isolation policy is done (so we are able to reach the customer service before it is isolated), drag and drop the policy in the board to the right place as indicated by the figure below:

![move-policy](./img/move-policy.png)

Now you should be able to access the yaobank application in your browser.
  
## Compliance reports
  
https://docs.tigera.io/compliance/overview
  
## Housekeeping
  
Remove the previous deployed Security policies:
  
```
kubectl delete -f manifests/netpol/ws/
```
```
kubectl delete -f manifests/netpol
```
```
kubectl delete -f manifests/netpol/additional/yaobank 
```

If you used policy recommendation to create a policy to access the yaobank application, remove it from the Policy Board.

Remove the cluster if not needed:
  
```
eksctl delete cluster <CLUSTER_NAME> 
```


