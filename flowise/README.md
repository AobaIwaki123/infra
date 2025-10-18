## 参考

- [Docker Compose](https://github.com/FlowiseAI/Flowise/blob/main/docker/docker-compose.yml)
- [env.example](https://github.com/FlowiseAI/Flowise/blob/main/docker/.env.example)

## namespaceの作成

```sh
$ kubectl create namespace dify
```

## Secretの適用

```sh
$ kubectl apply -f secret.yml -n dify
```
