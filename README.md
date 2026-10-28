# meelo-lzcapp

[Meelo](https://github.com/Arthi-chaud/Meelo) 的懒猫微服 LPK 打包（镜像模式，LPK v2）。

- 包名：`community.lazycat.app.meelo`
- 上游镜像：`arthichaud/meelo-{server,front,scanner,matcher}`（Docker Hub）+ postgres/meilisearch/rabbitmq/kyoo_transcoder/nginx
- 构建方式：GitHub Actions 云端构建（`ca-x/lazycat-github-action@v1`），本地无需 Docker
- 版本源：`meelo-server` 镜像 tag（`vX.Y.Z` → `X.Y.Z`）
- 发布目标：喵喵商店（私有/社区商店），不出官方商店

## 结构

```
.
├── package.yml            # 包元数据（LPK v2）
├── lzc-manifest.yml       # 运行结构（9 服务：server/front/scanner/matcher/db/meili/transcoder/mq/nginx）
├── lzc-build.yml          # 构建配置
├── icon.png               # 512x512 图标
├── content/
│   ├── nginx.conf.template   # 官方 nginx 路由模板（/api /scanner /matcher → 各服务，/ → front）
│   └── lazycat-injects/lzc-file-chooser-inject.js   # 文件选择器拦截（商店强制）
└── .github/
    ├── lazycat-action.yml  # Action 配置（8 个镜像的版本发现 + mirror 交付）
    └── workflows/lazycat.yml
```

## 免密登录

Meelo 没有独立登录页（认证墙组件全局弹出），采用三阶段联动：

1. `capture-credentials`（request）：拦截 `/api/users`（注册）与登录请求，捕获 username/password
2. `persist-credentials`（response）：2xx 后持久化
3. `autofill-login`（browser）：`simple-inject-password` + `$persist` 自动填充

## 数据目录

- 音乐库与配置：`/lzcapp/var/data`、`/lzcapp/var/config`（server/scanner/matcher 共享，scanner 只读）
- Postgres：`/lzcapp/var/db`；Meilisearch：`/lzcapp/var/meili`；RabbitMQ：`/lzcapp/var/rabbitmq`；转码缓存：`/lzcapp/var/transcoder-cache`

所有持久化数据在 `/lzcapp/var` 下，随应用生命周期管理。
