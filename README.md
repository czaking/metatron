# Metatron — Intranet Site Navigator

| 中文说明 | English |
|---|---|
| [🇨🇳 中文文档](./README_zh-CN.md) | [🇬🇧 English](./README.md) |
| 一个单文件静态门户,把团队每天要碰的内网站点(云控制台、监控大盘、CI、文档)收拢到一页:搜索、分类页签、环境标签、一键复制账号密码。 | A single-file static portal that catalogs all the internal sites a team touches every day — consoles, dashboards, CI, docs — with search, category tabs, environment tags, and one-click credential copying. |

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
