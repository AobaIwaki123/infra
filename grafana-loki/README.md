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
helm show values grafana/loki-stack > manifests/loki-stack-values.yaml
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
infra/
├── grafana-loki/
│   ├── helm/
│   │   ├── Chart.yaml
│   │   └── values.yaml
│   └── app.yaml
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

**app.yaml** (ArgoCD Application):
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
    path: grafana-loki/helm
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

# 2. ArgoCDで同期
argocd app create loki-stack --file app.yml

argocd app delete loki-stack
```

---
