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