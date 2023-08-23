# kubectl

## Node - List
```sh
kubectl get node
```

When autoscale using karpenter, need to list with provisioner name
```sh
kubectl get node \
-o=custom-columns="Name:.metadata.name,Provisioner:.metadata.labels.karpenter\.sh/provisioner-name"
``` 

```sh
kubectl get node \
-o=custom-columns="Name:.metadata.name,Provisioner:.metadata.labels.karpenter\.sh/provisioner-name,Type:.metadata.labels.karpenter\.sh/capacity-type,AZ:.metadata.labels.topology\.kubernetes\.io/zone,Instance:.spec.providerID"
``` 
```text
Name                                               Provisioner      Type     AZ                Instance
ip-172-19-74-10.ap-southeast-2.compute.internal    apigee-runtime   spot     ap-southeast-2a   aws:///ap-southeast-2a/i-0c9de2907bd71f472
ip-172-19-74-100.ap-southeast-2.compute.internal   apigee-data      spot     ap-southeast-2a   aws:///ap-southeast-2a/i-08fa9a373f8b22a1d
```

Use AWS CLI
```sh
aws ec2 describe-instances \
--filters Name=tag:aws:eks:cluster-name,Values=${cluster_name} \
--query "Reservations[*].Instances[*].[(Tags[?Key=='node_type'].Value)[0],(Tags[?Key=='karpenter.sh/provisioner-name'].Value)[0],PrivateDnsName,PrivateIpAddress,Placement.AvailabilityZone,InstanceId,InstanceLifecycle]" \
--output text
```

## Node - delete

[How to gracefully remove a node.](https://stackoverflow.com/questions/35757620/how-to-gracefully-remove-a-node-from-kubernetes)

```sh
kubectl drain ${node_name} --ignore-daemonsets --delete-local-data --force
```
```sh
kubectl delete node ${node_name}
```

## Pod - shell

```sh
kubectl run nginx --restart=Never --image=nginx -n ${namespace}
```

```sh
kubectl -it -n ${namespace} nginx -- /bin/bash
```

## Pod - list

Non running pods
```sh
kubectl get pod -A | grep -v 'Running\|Completed' 
```

```sh
kubectl get pod -A | grep -Ev 'Running|Completed' 
```

Use label selector
```sh
kubectl get pod -l label_key=label_value
```

Get all pendeing pod and its intended schedule node
```sh
kubectl get pod -A --field-selector=status.phase=Pending \
-o  custom-columns="POD":.metadata.name,"Node":.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution.nodeSelectorTerms[0].matchFields[0].values[0]
```