# Kubernetes Redis Cluster

### Create Disks

``` sh
gcloud compute disks create --size=10GB \
  'redis-1' 'redis-2' 'redis-3' \
  'redis-4' 'redis-5' 'redis-6'
```

### Create Redis Cluster Configuration

``` sh
kubectl create configmap redis-conf --from-file=redis.conf
```

### Create Redis Nodes

``` sh
kubectl create -f deployment
```

### Create Redis Services

``` sh
kubectl create -f services
```

### Connect Nodes

``` sh
kubectl run -i --tty ubuntu --image=ubuntu \
  --restart=Never /bin/bash
```

``` sh
apt-get update
apt-get install ruby vim wget redis-tools dnsutils
wget http://download.redis.io/redis-stable/src/redis-trib.rb
```

*Note:* `redis-trib` doesn't support hostnames (see [this issue](https://github.com/antirez/redis/issues/2565)), so we use `dig` to resolve our cluster IPs.


``` sh
./redis-trib.rb create `dig +short redis-1.default.svc.cluster.local`:6379 `dig +short redis-2.default.svc.cluster.local`:6379 `dig +short redis-3.default.svc.cluster.local`:6379
./redis-trib.rb replicate --master-addr `dig +short redis-1.default.svc.cluster.local`:6379 --slave-addr `dig +short redis-4.default.svc.cluster.local`:6379
./redis-trib.rb replicate --master-addr `dig +short redis-2.default.svc.cluster.local`:6379 --slave-addr `dig +short redis-5.default.svc.cluster.local`:6379
./redis-trib.py replicate --master-addr `dig +short redis-3.default.svc.cluster.local`:6379 --slave-addr `dig +short redis-6.default.svc.cluster.local`:6379
```

``` sh
# ips from cluster ip of pods
./redis-trib.rb create --replicas 1 \
  10.131.242.1:6379 \
  10.131.242.2:6379 \
  10.131.242.3:6379 \
  10.131.242.4:6379 \
  10.131.242.5:6379 \
  10.131.242.6:6379
```

### Add a new node

``` sh
gcloud compute disks create --size=10GB 'redis-7'
```

``` sh
kubectl create -f replicaset/redis-7.yaml
```

``` sh
kubectl create -f services/redis-7.yaml
```

``` sh
./redis-trib.rb add-node 10.131.242.7:6379 10.131.242.1:6379
./redis-trib.rb reshard --from <node-id> --to <node-id> --slots <number of slots> --yes <host>:<port>
```
