## Build vm with multipass

```
# build k8s master, where control-plane, etcd and all kube-system namespace will be Deployed
multipass launch --name master --cpus 1 --mem 2G --disk 20G --cloud-init cloud-init.yml

# workerX like worker1 worker2 ...
multipass launch --name workerX --cpus 2 --memery 4G --disk 20G --cloud-init cloud-init-worker.yml

# worker kibana
multipass launch --name kibana --cpus 1 --memery 4G --disk 20G --cloud-init cloud-init-worker.yml

# worker ingest
multipass launch --name ingest --cpus 1 --memery 2G --disk 20G --cloud-init cloud-init-worker.yml
```

## Build k8s cluster

- All worker node must join the master node

```
multipass exec master -- sudo microk8s add-node

# follow printed instruction to join master
multipass exec workerX -- sudo microk8s join ...
multipass exec kibana -- sudo microk8s join ...
multipass exec ingest -- sudo microk8s join ...

```

- Check if everything is ok

```
kubectl get nodes

```

## Build ELK by operator ECK

### Attribuate role to each k8s node

- Creates label on each nodes

```
kubectl label nodes worker1 worker2 worker3 elastic-node=yes
kubectl label nodes worker4 kibana-node=yes
kubectl label nodes worker5 kibana-node=yes
```

### Deploye pods with operator eck

- elstic link
- kibana link

### Connect to kibana-node

sometimes the kibana instance need to wait 3-5mn before the front is ready

- recupere le mdp
  ` kubectl get secret -n elastic-system elastic-poc-es-elastic-user -o jsonpath='{.data.elastic}' | base64 -d`

- forward le port 5601
  `kubectl port-forward -n elastic-system service/kibana-poc 5601:5601`

## Know issues

- diskpressure
- pending infinitly
- kibana

## Best practices

### Identifier le Service Elasticsearch

Lister les services dans le namespace elastic-system

```kubectl get svc -n elastic-system

```

- Before apply some change in your manifest, run it as client side

```
kubectl diff -f elasticsearch.yml
kubectl apply --dry-run=client -f elasticsearch.yml # an example
# if everything is ok
kubectl apply -f elasticsearch.yml

```

- Production workload [[https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/virtual-memory]], virtual memory is strongly recommended to increase the kernel setting `vm.max_map_count` to 262144
