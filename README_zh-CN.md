# Metatron — 内网站点导航台

| 中文说明 | English |
|---|---|
| [🇨🇳 中文文档](./README_zh-CN.md) | [🇬🇧 English](./README.md) |
| 一个单文件静态门户,把团队每天要碰的内网站点(云控制台、监控大盘、CI、文档)收拢到一页:搜索、分类页签、环境标签、一键复制账号密码。 | A single-file static portal that catalogs all the internal sites a team touches every day — consoles, dashboards, CI, docs — with search, category tabs, environment tags, and one-click credential copying. |

## 特性

- **一个 HTML 就是整站**(`manifests/site-nav.html`):内联 CSS/JS,零外部资源。
- **Kustomize ConfigMap 滚动**:用 `configMapGenerator` 把 HTML / nginx.conf 变成带内容哈希的 ConfigMap——改文件、合并,Pod 自动滚动更新(对 ArgoCD 友好)。
- **原生 `nginx:alpine`**:不用构建不用推镜像,哪里都能拉。
- **Ingress 就绪**:附内网 ALB + TLS 的示例配置。

## 目录结构

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

## 部署

```bash
kubectl create ns metatron
kubectl apply -k manifests/
```

或走 ArgoCD:改 `argocd/app-metatron.yaml` 里的 `repoURL`/`targetRevision`,然后
`kubectl apply -f argocd/app-metatron.yaml`。

## 定制

所有站点条目都在 `site-nav.html` 的 `SITES` 数组里:

```js
{ name: "Grafana", url: "https://grafana.example.com/", cat: "observe",
  env: "aliyun", auth: "aliyun-ram", tags: ["大盘"], desc: "..." }
```

字段:`cat`(分类页签)、`env`(`aliyun` / `intranet` / `saas`)、`auth`、`tags`、`desc`,需要独立账号的条目可加 `account`/`password`。按自己的清单调整即可。

> ⚠️ 如果把凭据放在数据清单里,门户务必部署在访问受控的位置(仅内网 ingress);公开副本应先清掉凭据。
