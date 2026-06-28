# KADM App Configs

`kadm-app-configs` 是 KADM 的应用 GitOps 配置仓库。它是生产应用部署 YAML 和发布控制台应用注册表的来源。

## 职责边界

本仓库负责：

- `apps/apps.json` 应用注册表
- `apps/<app>/base/*` 应用基础 Kubernetes 资源
- `apps/<app>/overlays/prod/*` 生产环境 overlay 和 image tag
- 应用 Secret 示例

本仓库不负责：

- 应用源码
- 应用测试
- 应用镜像构建
- 平台组件离线资源

应用源码仓库只构建并推送镜像。KADM Release Console 发布时会更新本仓库中的对应 image tag，然后触发 Argo CD 同步。

## 当前应用

```text
apps/
  apps.json
  demo-hello/
    base/
    overlays/prod/
  demo-hello-spring/
    base/
    overlays/prod/
```

`apps/apps.json` 当前包含：

- `demo-hello`
- `demo-hello-spring`

## 发布流程

```text
KADM Release Console
  -> trigger application GitHub Actions build
  -> update apps/<app>/overlays/prod/kustomization.yaml
  -> sync Argo CD Application
  -> wait for Argo Rollouts
  -> operator promotes
```

应用生产部署以本仓库为准，不读取应用源码仓库中的 `k8s/` 路径。

---

# KADM App Configs

`kadm-app-configs` is the KADM application GitOps configuration repository. It is the source for production application deployment YAML and the Release Console app registry.

## Responsibility Boundary

This repository owns:

- `apps/apps.json` app registry
- `apps/<app>/base/*` base Kubernetes resources
- `apps/<app>/overlays/prod/*` production overlays and image tags
- Application Secret examples

This repository does not own:

- Application source code
- Application tests
- Application image builds
- Platform offline assets

Application source repositories only build and push images. During release, KADM Release Console updates the matching image tag in this repository and then triggers Argo CD sync.

## Current Apps

```text
apps/
  apps.json
  demo-hello/
    base/
    overlays/prod/
  demo-hello-spring/
    base/
    overlays/prod/
```

`apps/apps.json` currently contains:

- `demo-hello`
- `demo-hello-spring`

## Release Flow

```text
KADM Release Console
  -> trigger application GitHub Actions build
  -> update apps/<app>/overlays/prod/kustomization.yaml
  -> sync Argo CD Application
  -> wait for Argo Rollouts
  -> operator promotes
```

Production deployment is sourced from this repository, not from `k8s/` paths in application source repositories.
