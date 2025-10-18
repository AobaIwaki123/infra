公式の[Docker Compose](https://github.com/langgenius/dify/blob/main/docker/docker-compose.yaml)ファイルを参考にしてマニフェストファイルを作成した
環境変数については[こちら](https://github.com/langgenius/dify/blob/main/docker/.env.example)を参考にした。

## namespaceの作成

```sh
$ kubectl create namespace dify
```

## Secretの適用

```sh
$ kubectl apply -f secret.yml -n dify
```


# 参考

- [Dify Docker](https://github.com/langgenius/dify/tree/main/docker)
