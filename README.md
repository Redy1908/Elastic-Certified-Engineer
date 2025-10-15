## What will we do?

In this guide, we will install an Elasticsearch and Kibana stack on a local Kubernetes cluster (Minikube) using the ECK (Elastic Cloud on Kubernetes) operator.

## 0. Prerequisites

- Helm installed
- Minikube installed and running with at least 2/4 CPUs and 8GB of RAM:

```console
$ minikube start --cpus=4 --memory=8G
```

- Or you can edit the file [helm/eck/kielasticsearchbaelana.yml]() reducing the number of nodes to `1` and execute minikube with the following command:

```console
$ minikube start
```

## 1. Preparation

```console
$ helm repo add elastic https://helm.elastic.co
"elastic" has been added to your repositories

$ helm repo update
Hang tight while we grab the latest from your chart repositories.
...Successfully got an update from the "elastic" chart repository
Update Complete. ⎈Happy Helming!⎈
```

## 2. Install ECK Operator

```console
$ helm upgrade --install elastic-operator elastic/eck-operator \
    --namespace elastic-system \
    --create-namespace
```

With ECK, you define your stack using Kubernetes manifests.

A pre filled Helm values file for elasticsearch is provided at [helm/eck/kielasticsearchbaelana.yml]().

**`helm/eck/elasticsearch.yml`**
```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 9.1.5
  nodeSets:
  - name: default
    count: 3
    config:
      node.store.allow_mmap: false
```

Now, apply the elasticsearch manifest to your cluster:

```console
$ kubectl apply -f helm/eck/elasticsearch.yml -n elastic-system
```

Wait until all pods are in the `Ready` state:

```console
$ watch -n 2 kubectl get pods -n elastic-system
NAME                            READY   STATUS    RESTARTS   AGE
elastic-operator-0              1/1     Running   0          5m4s
quickstart-es-default-0         1/1     Running   0          4m21s
quickstart-es-default-1         1/1     Running   0          4m21s
quickstart-es-default-2         1/1     Running   0          4m21s
```

Retrieve the password for the `elastic` user:

```console
$ kubectl get secret quickstart-es-elastic-user \
    -n elastic-system \
    -o go-template='{{.data.elastic | base64decode}}'
63QB9ewPU7p31eU45KQ00FCn
```

Retrieve the Elasticsearch url (leave the terminal open):

```console
minikube service quickstart-es-http -n elastic-system --url
http://127.0.0.1:42837
```

You can now edit the labs files in the `labs/` folder replacing the `host` and `password` variables with the retrieved values.

A pre filled Helm values file for kibana is provided at [helm/eck/elasticsearch.yml]().

**`helm/eck/kibana.yml`**
```yaml
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: quickstart
spec:
  version: 9.1.5
  count: 1
  elasticsearchRef:
    name: quickstart
  http:
    service:
      spec:
        type: NodePort
```

Now, apply the kibana manifest to your cluster:

```console
$ kubectl apply -f helm/eck/kibana.yml -n elastic-system
```

Wait until all pods are in the `Ready` state:

```console
$ watch -n 2 kubectl get pods -n elastic-system
NAME                            READY   STATUS    RESTARTS   AGE
elastic-operator-0              1/1     Running   0          5m4s
quickstart-es-default-0         1/1     Running   0          4m21s
quickstart-es-default-1         1/1     Running   0          4m21s
quickstart-es-default-2         1/1     Running   0          4m21s
quickstart-kb-86568f9d8-brzpb   1/1     Running   0          4m16s
```

## 3. Access Credentials and UI

Retrieve the password for the `elastic` user:

```console
$ kubectl get secret quickstart-es-elastic-user \
    -n elastic-system \
    -o go-template='{{.data.elastic | base64decode}}'
63QB9ewPU7p31eU45KQ00FCn
```

Retrieve the Kibana url (leave the terminal open):

```console
$ minikube service quickstart-kb-http -n elastic-system --url
http://192.168.49.2:31439
```

Use the retrieved password with the username `elastic` to login.