## Install ECK

- [Download Elastic Cloud on Kubernetes](https://www.elastic.co/jp/downloads/elastic-cloud-kubernetes)

```sh
$ kubectl create -f https://download.elastic.co/downloads/eck/3.1.0/crds.yaml
$ kubectl apply -f https://download.elastic.co/downloads/eck/3.1.0/operator.yaml
```

## Deploy Elasticsearch cluster

- [Deploy an Elasticsearch cluster](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/elasticsearch-deployment-quickstart)

```sh
$ kubectl patch storageclass ceph-rbd -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

```sh
$ cat <<EOF | kubectl delete -f -
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
EOF
```

```sh
$ kubectl apply -f manifests/ingress.yml
```

```sh
$ PASSWORD=$(kubectl get secret quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')
$ curl -u "elastic:$PASSWORD" -k "https://eck.aooba.net"
```

## Deploy a Kibana instance

- [Deploy a Kibana instance](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/kibana-instance-quickstart)
