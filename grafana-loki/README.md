## 手順：Helm導入 + ArgoCD管理を並行

### Phase 1: Helmで即座に動作確認

```bash
# 1. Helmでインストール（動作確認用）
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm install loki-stack grafana/loki-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.enabled=true \
  --set prometheus.enabled=false \
  --set promtail.enabled=true
```

```bash
# 2. すぐに動作確認
kubectl get pods -n monitoring

# Grafanaパスワード取得
kubectl get secret --namespace monitoring loki-stack-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

# ポートフォワード
kubectl port-forward --namespace monitoring service/loki-stack-grafana 3000:80
```

ブラウザで `http://localhost:3000` → Explore → Loki でログが見えることを確認

---

### Phase 2: ArgoCD管理への移行準備（並行作業）

#### 1. values.yamlを取得・カスタマイズ

```bash
# デフォルトのvaluesを取得
helm show values grafana/loki-stack > loki-stack-values.yaml
```

最小限のカスタマイズ例（`loki-stack-values.yaml`）:

```yaml
loki:
  enabled: true
  persistence:
    enabled: true
    size: 10Gi
    # storageClassName: your-storage-class  # 必要なら指定

promtail:
  enabled: true

grafana:
  enabled: true
  persistence:
    enabled: true
    size: 1Gi
  adminPassword: "your-secure-password"  # 変更推奨
  
prometheus:
  enabled: false
```

#### 2. Gitリポジトリ構造を作成

```
your-gitops-repo/
├── apps/
│   └── monitoring/
│       ├── loki-stack/
│       │   ├── Chart.yaml
│       │   └── values.yaml
│       └── application.yaml
```

**Chart.yaml**:
```yaml
apiVersion: v2
name: loki-stack
version: 1.0.0
dependencies:
  - name: loki-stack
    version: 2.10.2  # 最新バージョンを確認
    repository: https://grafana.github.io/helm-charts
```

**values.yaml** (上記でカスタマイズしたもの):
```yaml
loki-stack:
  loki:
    enabled: true
    persistence:
      enabled: true
      size: 10Gi
  
  promtail:
    enabled: true
  
  grafana:
    enabled: true
    persistence:
      enabled: true
      size: 1Gi
    adminPassword: "your-secure-password"
    
  prometheus:
    enabled: false
```

**application.yaml** (ArgoCD Application):
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: loki-stack
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-username/your-gitops-repo.git
    targetRevision: main
    path: apps/monitoring/loki-stack
  destination:
    server: https://kubernetes.default.svc
    namespace: monitoring
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

#### 3. Helm依存関係を更新

```bash
cd apps/monitoring/loki-stack
helm dependency update
```

これで `charts/` ディレクトリにloki-stackがダウンロードされます。

#### 4. GitにPush

```bash
git add .
git commit -m "Add loki-stack with ArgoCD"
git push origin main
```

---

### Phase 3: ArgoCD管理への切り替え

動作確認が完了したら：

```bash
# 1. Helmでインストールしたものを削除
helm uninstall loki-stack -n monitoring

# 2. ArgoCD Applicationをデプロイ
kubectl apply -f apps/monitoring/application.yaml

# 3. ArgoCDで同期
argocd app sync loki-stack

# または、ArgoCD UIで手動同期
```

---

## 負荷がない場合の同時並行手順

もし完全に並行したい場合：

### オプションA: 別Namespaceで並行

```bash
# Helm版: monitoring namespace
helm install loki-stack grafana/loki-stack -n monitoring

# ArgoCD版: monitoring-argocd namespace（テスト用）
# application.yamlのnamespaceを変更してデプロイ
```

動作確認後、Helm版を削除してArgoCD版を本番namespaceへ移行。

### オプションB: Helmを直接ArgoCD管理（推奨）

実はArgoCDは直接Helmリポジトリを参照できます：

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: loki-stack
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://grafana.github.io/helm-charts
    chart: loki-stack
    targetRevision: 2.10.2
    helm:
      values: |
        loki:
          enabled: true
          persistence:
            enabled: true
            size: 10Gi
        promtail:
          enabled: true
        grafana:
          enabled: true
          persistence:
            enabled: true
            size: 1Gi
        prometheus:
          enabled: false
  destination:
    server: https://kubernetes.default.svc
    namespace: monitoring
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

この方法なら：
- Gitリポジトリに `Chart.yaml` 不要
- `application.yaml` だけで完結
- 最もシンプル

## おすすめの進め方

1. **今すぐ**: Helmで `helm install` → 動作確認（5分）
2. **並行作業**: ArgoCD Applicationマニフェスト作成 → Gitにpush（10分）
3. **切り替え**: 動作確認OKなら `helm uninstall` → ArgoCD sync（2分）

どの方法で進めますか？オプションBの「直接Helmリポジトリ参照」が一番楽だと思います。
