
## Duplication

```
platform-repo/
├── dev/
│   ├── ingress-nginx.yaml
│   ├── cert-manager.yaml
│   └── kubevirt.yamll
├── qa/
│   ├── ingress-nginx.yaml
│   ├── cert-manager.yaml
│   └── kubevirt.yaml
└── prod/
    ├── ingress-nginx.yaml
    ├── cert-manager.yaml
    └── kubevirt.yaml
```

## Common + Override

```
platform-repo/
├── base/
│   ├── ingress-nginx/
│   ├── cert-manager/
│   └── kubevirt/
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    ├── qa/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
```

## Git Tags

```
platform-repo/
├── releases/
│   ├── 2026.03.0/
│   │   ├── state.yaml
│   │   ├── manifests/
│   │   └── release-notes.md
│   └── 2026.03.2/
│       ├── state.yaml
│       ├── manifests/
│       └── release-notes.md
└── stages/
    ├── dev.yaml
    ├── qa.yaml
    └── prod.yaml
```