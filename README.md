# KADM App Configs

This repository is the GitOps source for application deployment configuration in KADM.

Responsibilities:

- application deployment manifests
- Kustomize overlays per environment
- release-console application registry data

Source code, tests, and image build workflows stay in the application repositories. Operational YAML stays here.

Current layout:

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

`apps/apps.json` is the current registry consumed by KADM Release Console and by `kadmctl configure-delivery`.
