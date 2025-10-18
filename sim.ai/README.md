 ## 注意

まあまあ動いているがエラーが大量に出る。
環境変数の問題みたいだが対処が面倒なので一旦放置

## namespaceの作成

```sh
$ kubectl create namespace sim-ai
```

## Secretの適用

`secret.example.yml` をコピーして `secret.yml` を作成します。

```sh
$ cp secret.example.yml secret.yml
```

`secret.yml` 内の `data` の値はBase64エンコードされている必要があります。
以下のコマンドでランダムな文字列を生成し、Base64エンコードできます。

```sh
# openssl rand -hex 32 | tr -d '\n' | base64
```

`secret.yml` を編集したら、以下のコマンドでSecretを適用します。

```sh
$ kubectl apply -f secret.yml -n sim-ai
```

### Secretの内容

- `POSTGRES_USER`: PostgreSQLのユーザー名 (デフォルト: `postgres`)
- `POSTGRES_PASSWORD`: PostgreSQLのパスワード (デフォルト: `postgres`)
- `POSTGRES_DB`: PostgreSQLのデータベース名 (デフォルト: `simstudio`)
- `BETTER_AUTH_SECRET`: 認証用のシークレットキー。上記コマンドで生成した値を設定します。
- `ENCRYPTION_KEY`: 暗号化用のキー。上記コマンドで生成した値を設定します。
- `COPILOT_API_KEY`: CopilotのAPIキー。外部サービスのAPIキーであり、自身で取得した値を設定する必要があります。詳細は後述の「Copilot APIキーの取得」を参照してください。
- `SIM_AGENT_API_URL`: Sim Agent APIのURL。内部サービスのエンドポイントです。

### Copilot APIキーの取得

`COPILOT_API_KEY` は、GitHub CopilotのAPIを利用するためのキーです。
現在、GitHub CopilotのAPIキーは一般に公開されておらず、特定のパートナーやプログラムを通じてのみ利用可能です。

もし、あなたがアクセス権を持っている場合は、GitHubのドキュメントや開発者向けページからAPIキーを取得してください。

取得したAPIキーは、Base64エンコードして `secret.yml` に設定する必要があります。

```sh
echo -n "your_copilot_api_key" | base64
```
