# Kafka with Strimzi

You should already have a working Kubernetes cluster with a namespace containing your microservices. In this step, we're going to be adding on namespaces to house Strimzi.

## Downloading Strimzi

Download the most recent [Strimzi version (ex: strimzi-0.26.0.zip)](https://github.com/strimzi/strimzi-kafka-operator/releases) release.

Unzip the file with `unzip strimzi-x.xx.x.zip`.

`cd` into the new directory unzipped from the file. You should see a directory named `install`, which contains files we will be using next. Make sure you have `kubectl` set up and logged in.

## Installing Strimzi

Create a new `kafka` namespace:
```
kubectl create ns kafka
```

Map the installation files to the new `kafka` cluster:

#### Linux
```
sed -i 's/namespace: .*/namespace: kafka/' install/cluster-operator/*RoleBinding*.yaml
```
#### Mac
```
sed -i '' 's/namespace: .*/namespace: kafka/' install/cluster-operator/*RoleBinding*.yaml
```

### Deploying the Cluster
Create a new namespace that will be used to deploy your Kafka cluster:
```
kubectl create ns my-kafka-project
```
In your `install` directory, edit the file with the path `install/cluster-operator/060-Deployment-strimzi-cluster-operator.yaml`. Change the value of `STRIMZI_NAMESPACE` to `my-kafka-project`.

```
env:
- name: STRIMZI_NAMESPACE
  value: my-kafka-project
```

Deploy the resources AND give access to control them.
```
kubectl create -f install/cluster-operator/ -n kafka
```
Give permission to the Cluster Operator to monitor your `my-kafka-project` namespace:
```
kubectl create -f install/cluster-operator/020-RoleBinding-strimzi-cluster-operator.yaml -n my-kafka-project
kubectl create -f install/cluster-operator/031-RoleBinding-strimzi-cluster-operator-entity-operator-delegation.yaml -n my-kafka-project
```

Finally, deploy the cluster with your ZooKeeper and Kafka Broker!
```
cat << EOF | kubectl create -n my-kafka-project -f -
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: my-cluster
spec:
  kafka:
    replicas: 1
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
        authentication:
          type: tls
      - name: external
        port: 9094
        type: nodeport
        tls: false
    storage:
      type: jbod
      volumes:
      - id: 0
        type: persistent-claim
        size: 100Gi
        deleteClaim: false
    config:
      offsets.topic.replication.factor: 1
      transaction.state.log.replication.factor: 1
      transaction.state.log.min.isr: 1
  zookeeper:
    replicas: 1
    storage:
      type: persistent-claim
      size: 100Gi
      deleteClaim: false
  entityOperator:
    topicOperator: {}
    userOperator: {}
EOF
```

## Creating Kafka Topics
Wait until your cluster finishes deploying:
```
kubectl wait kafka/my-cluster --for=condition=Ready --timeout=300s -n my-kafka-project
```
You can then add topics that consumers can later subscribe to. Edit the name of the topic by editing "my-topic":
```
cat << EOF | kubectl create -n my-kafka-project -f -
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: my-topic
  labels:
    strimzi.io/cluster: "my-cluster"
spec:
  partitions: 3
  replicas: 1
EOF
```