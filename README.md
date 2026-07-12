# KADM App Configs

`kadm-app-configs` 是 KADM 的应用 Git 定义和生产 GitOps 清单仓库。它控制哪些项目可被同步到 Release Console，以及业务应用在集群中的期望配置和 image tag。

## 责任边界

本仓库负责：

- `apps/apps.json`：Git 定义的项目注册表。
- `apps/<app>/base/`：应用 Namespace、Rollout、Service 和入口资源。
- `apps/<app>/overlays/prod/`：生产 Kustomize overlay 和 image tag。

本仓库不负责：

- 应用源码和应用测试。
- 应用镜像构建。
- 平台组件版本和离线资源。
- Release Console 源码。

## 目录模型

```text
apps/
  apps.json
  <app>/
    base/
      namespace.yaml
      rollout.yaml
      service.yaml
      service-preview.yaml
      httproute.yaml
      ingress.yaml
      kustomization.yaml
    overlays/
      prod/
        kustomization.yaml
```

当前 Git 定义包含：

- `demo-hello`
- `demo-hello-spring`

## 项目注册状态

| 状态 | 位置 | 作用 |
| --- | --- | --- |
| Git 定义 | `apps/apps.json` | 可评审的 source of truth |
| source cache | 集群 `kadm-source-apps-config` | Git 定义的集群缓存 |
| effective registry | 集群 `kadm-apps-config` | Release Console 当前生效项目 |

修改 `apps.json` 不会自动更新 effective registry。需要在 Release Console 执行“应用 Git 定义”或等价项目 sync。

`.github/workflows/sync-source-registry.yaml` 只在 `KADM_SYNC_*` secrets 配置完整时刷新 source cache；未配置时 workflow 会成功跳过。

## 应用定义字段

每个项目包含：

- GitHub owner、repo、workflow 和 ref。
- GitOps owner、repo、path、image 和 ref。
- Argo CD Application 名称。
- Rollout namespace 和 name。

结构校验实现位于 `kadm-platform-system/console/src/config.js`。当前没有 JSON Schema；新增字段或约束时必须同步实现、测试和文档。

## 发布与生效

业务应用发布：

```text
Release Console
  -> 触发应用 workflow
  -> 等待 GHCR 镜像
  -> 更新 apps/<app>/overlays/prod/kustomization.yaml
  -> Argo CD sync
  -> Rollout preview
  -> 操作员 promote
```

业务应用的 Argo CD Application 应保持手工同步。Git push 只改变期望状态，不应直接改变线上版本。

base 清单或 overlay 非镜像配置的变更，也需要随一次显式发布或 Argo CD sync 生效。详细矩阵见 `kadm-platform-system/docs/git-change-activation.md`。

## 发布策略与入口

两个 demo 当前都使用 Argo Rollouts blue-green：

- active Service。
- preview Service。
- `autoPromotionEnabled: false`。
- 操作员显式切换。

默认交付模式使用 Gateway API `HTTPRoute`。`kadmctl configure-delivery --ingress-mode traefik` 会额外应用 `ingress.yaml`。

## 数据库与 Secret

数据库 Secret 不提交到 Git。`kadmctl configure-demo-apps` 在目标集群生成或保留高熵密码，创建 `hello-db` 和 `hellospring-db` 运行时 Secret，并使用相同凭据配置 MySQL。

当前两个 Rollout 的 `DB_HOST` 都指向节点级或集群外地址。迁移环境时必须同时确认：

- Rollout 的 `DB_HOST`。
- `configure-demo-apps --db-node-ip/--db-ssh-target`。
- MySQL 网络和用户权限。
- 新 Pod 是否已读取更新后的 Secret。

生产环境应使用正式 Secret 管理方案，并避免把真实凭据提交到 Git。直接同步本仓库前必须先供应对应运行时 Secret，否则 Pod 会保持不可用。

## 验证

注册表语法：

```bash
python3 -m json.tool apps/apps.json >/dev/null
```

如本机已安装带 Kustomize 的 `kubectl`：

```bash
kubectl kustomize apps/demo-hello/overlays/prod >/dev/null
kubectl kustomize apps/demo-hello-spring/overlays/prod >/dev/null
```

发布前还应确认：

- `apps.json` 中的仓库、workflow、GitOps path 和 image 存在。
- overlay 的 image name 与 `apps.json` 中 `gitops.image` 一致。
- Rollout name/namespace 与注册表一致。
- 运行时 Secret、DB_HOST、数据库 TLS、hostnames 和入口模式适合目标环境。
- GHCR tag 可拉取。

## 风险与限制

- 当前没有强制 CI 校验 Kustomize、策略、Secret 或远端引用。
- source cache workflow 可能因 secrets 未配置而跳过。
- GitOps 清单依赖安装流程或外部 Secret 管理系统预先供应数据库 Secret。
- image tag 更新和其他配置变更共享同一 GitOps commit 链路。
- 删除项目并同步会删除受管 Argo CD Application，必须先确认资源和数据影响。

## English Summary

This repository is the Git source for KADM project definitions and production application manifests. Git changes do not deploy automatically; project definitions and application releases require explicit synchronization. Database Secrets are supplied at runtime by `kadmctl configure-demo-apps` or an external secret-management system and are not committed to Git.
