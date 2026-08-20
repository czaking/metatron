# Metatron — Intranet Site Navigator

**English** | [中文](#中文)

## Features

- **One HTML file is the whole site** (`manifests/site-nav.html`): inline CSS/JS, zero external assets.
- **Kustomize ConfigMap rollouts**: `configMapGenerator` turns the HTML / nginx.conf into content-hashed ConfigMaps — edit the file, merge, and Pods roll automatically (ArgoCD friendly).
- **Stock `nginx:alpine`**: nothing to build or push; pull from anywhere.
- **Ingress-ready**: internal ALB + TLS example included.

## Layout

```
manifests/
  site-nav.html      the whole site (data list + UI in one file)
  nginx.conf         nginx site config (charset, gzip, SPA fallback, /healthz)
  deployment.yaml    2 replicas of nginx:alpine, ConfigMap volumes
  service.yaml       ClusterIP :80
  ingress.yaml       internal ALB ingress example
  kustomization.yaml configMapGenerator wiring
argocd/
  app-metatron.yaml  ArgoCD Application (auto sync + prune + self-heal)
```

## Run it

```bash
kubectl create ns metatron
kubectl apply -k manifests/
```

Or via ArgoCD: edit `repoURL`/`targetRevision` in `argocd/app-metatron.yaml`, then
`kubectl apply -f argocd/app-metatron.yaml`.

## Customizing

All site entries live in the `SITES` array inside `site-nav.html`:

```js
{ name: "Grafana", url: "https://grafana.example.com/", cat: "observe",
  env: "aliyun", auth: "aliyun-ram", tags: ["dashboards"], desc: "..." }
```

Fields: `cat` (category tab), `env` (`aliyun` / `intranet` / `saas`), `auth`, `tags`, `desc`, and optional `account`/`password` for entries that need standalone credentials. Adjust to fit your own inventory.

> ⚠️ If you keep credentials in the data list, host the portal somewhere access-controlled (intranet-only ingress). A public copy should scrub them.

---

## 中文

[English](#metatron--intranet-site-navigator) | **中文**

### 特性

- **一个 HTML 就是整站**(`manifests/site-nav.html`):内联 CSS/JS,零外部资源。
- **Kustomize ConfigMap 滚动**:用 `configMapGenerator` 把 HTML / nginx.conf 变成带内容哈希的 ConfigMap——改文件、合并,Pod 自动滚动更新(对 ArgoCD 友好)。
- **原生 `nginx:alpine`**:不用构建不用推镜像,哪里都能拉。
- **Ingress 就绪**:附内网 ALB + TLS 的示例配置。

### 目录结构

```
manifests/
  site-nav.html      整站(数据清单 + UI 都在一个文件里)
  nginx.conf         nginx 站点配置(charset、gzip、SPA 回退、/healthz)
  deployment.yaml    双副本 nginx:alpine,挂 ConfigMap 卷
  service.yaml       ClusterIP :80
  ingress.yaml       内网 ALB ingress 示例
  kustomization.yaml configMapGenerator 接线
argocd/
  app-metatron.yaml  ArgoCD Application(自动同步 + prune + self-heal)
```

### 部署

```bash
kubectl create ns metatron
kubectl apply -k manifests/
```

或走 ArgoCD:改 `argocd/app-metatron.yaml` 里的 `repoURL`/`targetRevision`,然后
`kubectl apply -f argocd/app-metatron.yaml`。

### 定制

所有站点条目都在 `site-nav.html` 的 `SITES` 数组里:

```js
{ name: "Grafana", url: "https://grafana.example.com/", cat: "observe",
  env: "aliyun", auth: "aliyun-ram", tags: ["大盘"], desc: "..." }
```

字段:`cat`(分类页签)、`env`(`aliyun` / `intranet` / `saas`)、`auth`、`tags`、`desc`,需要独立账号的条目可加 `account`/`password`。按自己的清单调整即可。

> ⚠️ 如果把凭据放在数据清单里,门户务必部署在访问受控的位置(仅内网 ingress);公开副本应先清掉凭据。
