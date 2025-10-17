## What will we do?

We will install an Elasticsearch and Kibana stack on a local Kubernetes cluster (Minikube) using the ECK (Elastic Cloud on Kubernetes) operator to practice for the [Elastic Certified Engineer](https://www.elastic.co/training/elastic-certified-engineer-exam) exam.

## 0. Prerequisites

- Helm installed
- Minikube installed and running:

```console
$ minikube start
```

- If you want to use more elasticsearch nodes, edit the file `helm/eck/elasticsearch.yml` and change the `count` parameter. Make sure your minikube instance has enough resources to handle the number of nodes you want to create. You can start minikube with more resources like this:

```console
$ minikube start --cpus=4 --memory=8G
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

Create a secret with a fixed password for the `elastic` user:

```console
$ kubectl create secret generic quickstart-es-elastic-user \
  --from-literal=elastic=ElasticCertifiedEngineer \
  -n elastic-system
secret/elastic-credentials created
```

With ECK, you define your stack using Kubernetes manifests.

A pre filled Helm values file for elasticsearch is provided at [helm/eck/elasticsearch.yml]().

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
    count: 1
    config:
      node.store.allow_mmap: false
  http:
    service:
      spec:
        type: LoadBalancer
        ports:
          - port: 9200
            targetPort: 9200
            protocol: TCP
            name: https
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
```

A pre filled Helm values file for kibana is provided at [helm/eck/kibana.yml]().

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
        type: LoadBalancer
        ports:
          - port: 5601
            targetPort: 5601
            protocol: TCP
            name: https
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
quickstart-kb-86568f9d8-brzpb   1/1     Running   0          4m16s
```

## Access the stack

Open a terminal and execute `minikube tunnel`. Leave the terminal open. You can now access the services using the following URLs:

- **Elasticsearch:** [https://127.0.0.1:9200](https://127.0.0.1:9200)
- **Kibana:** [https://127.0.0.1:5601](https://127.0.0.1:5601)

Login with user `elastic` and the password `ElasticCertifiedEngineer`.

## Labs

All the labs for the Elastic Certified Engineer exam can be found in the `labs` folder. Each lab contains 2 files:
- a `.http` file containing the requests you need to perform to complete the lab. You can use it to interact with your Elasticsearch cluster using the [REST Client extension](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) for VSCode, or you can use the [Dev Tools](https://127.0.0.1:5601/app/dev_tools) console in Kibana.

- a `.md` file containing the documentation for the lab.