# Metatron — 内网站点导航台 / Intranet Site Navigator

| **English** | **中文** |
|---|---|
| A single-file static portal that catalogs all the internal sites a team touches every day — consoles, dashboards, CI, docs — with search, category tabs, environment tags, and one-click credential copying. | 一个单文件静态门户,把团队每天要碰的内网站点(云控制台、监控大盘、CI、文档)收拢到一页:支持搜索、分类页签、环境标签、一键复制账号密码。 |
| Ships as plain k8s manifests; no build step, no custom image. | 以裸 k8s manifests 发布,无构建步骤、无自建镜像。 |

| **English** | **中文** |
|---|---|
| - **One HTML file is the whole site** (`manifests/site-nav.html`): inline CSS/JS, zero external assets.<br>- **Kustomize ConfigMap rollouts**: `configMapGenerator` turns the HTML / nginx.conf into content-hashed ConfigMaps — edit the file, merge, and Pods roll automatically (ArgoCD friendly).<br>- **Stock `nginx:alpine`**: nothing to build or push.<br>- **Ingress-ready**: internal ALB + TLS example included. | - **一个 HTML 就是整站**(`manifests/site-nav.html`):内联 CSS/JS,零外部资源。<br>- **Kustomize ConfigMap 滚动**:`configMapGenerator` 把 HTML / nginx.conf 变成带内容哈希的 ConfigMap——改文件、合并,Pod 自动滚动更新(对 ArgoCD 友好)。<br>- **原生 `nginx:alpine`**:不用构建不用推镜像。<br>- **Ingress 就绪**:附内网 ALB + TLS 示例配置。 |

## Layout / 目录结构

```
manifests/
  site-nav.html      the whole site (data + UI in one file) / 整站(数据清单 + UI 一个文件)
  nginx.conf         nginx site config / nginx 站点配置(charset、gzip、SPA 回退、/healthz)
  deployment.yaml    2 replicas of nginx:alpine / 双副本 nginx:alpine,挂 ConfigMap 卷
  service.yaml       ClusterIP :80
  ingress.yaml       internal ALB ingress example / 内网 ALB ingress 示例
  kustomization.yaml configMapGenerator wiring / configMapGenerator 接线
argocd/
  app-metatron.yaml  ArgoCD Application (auto sync + prune + self-heal) / ArgoCD 应用
```

## Run it / 部署

| **English** | **中文** |
|---|---|
| Apply directly, or via ArgoCD: edit `repoURL` / `targetRevision` in `argocd/app-metatron.yaml` first, then apply it. | 直接 apply;或走 ArgoCD——先改 `argocd/app-metatron.yaml` 里的 `repoURL` / `targetRevision`,再 apply 该文件。 |

```bash
kubectl create ns metatron
kubectl apply -k manifests/

# ArgoCD / ArgoCD 方式
kubectl apply -f argocd/app-metatron.yaml
```

## Customizing / 定制

| **English** | **中文** |
|---|---|
| All site entries live in the `SITES` array inside `site-nav.html`. Fields: `cat` (category tab), `env` (`aliyun` / `intranet` / `saas`), `auth`, `tags`, `desc`, and optional `account` / `password` for entries that need standalone credentials. | 所有站点条目都在 `site-nav.html` 的 `SITES` 数组里。字段:`cat`(分类页签)、`env`(`aliyun` / `intranet` / `saas`)、`auth`、`tags`、`desc`,需要独立账号的条目可加 `account` / `password`。 |

```js
{ name: "Grafana", url: "https://grafana.example.com/", cat: "observe",
  env: "aliyun", auth: "aliyun-ram", tags: ["dashboards"], desc: "..." }
```

| **English** | **中文** |
|---|---|
| ⚠️ If you keep credentials in the data list, host the portal somewhere access-controlled (intranet-only ingress). Scrub them before making any copy public. | ⚠️ 如果把凭据放在数据清单里,门户务必部署在访问受控的位置(仅内网 ingress);公开副本应先清掉凭据。 |
