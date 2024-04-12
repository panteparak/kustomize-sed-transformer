## Kustomize Sed Transformer

### Installation via ArgoCD Helm Chart
```
repoServer:
  volumes:
  - name: argocd-plugins
    emptyDir: {}
  volumeMounts:
  - name: argocd-plugins
    mountPath: "/home/argocd/.local/share/kustomize/plugins/"
  env:
    - name: "KUSTOMIZE_PLUGIN_HOME"
      value: "/home/argocd/.local/share/kustomize/plugins/"
  initContainers:
    - name: download-sed-transformer
      image: alpine:3
      volumeMounts:
        - name: argocd-plugins
          mountPath: /plugins
      env:
        - name: ARGO_CUSTOM_PLUGIN_HOME
          value: /plugins
        - name: ARGO_CUSTOM_PLUGIN_URL
          value: "https://github.com/panteparak/kustomize-sed-transformer/archive/refs/heads/main.tar.gz"
      command: [ "/bin/sh", "-c" ]
      securityContext:
        allowPrivilegeEscalation: false
        runAsUser: 0
      args:
        - whoami && pwd;
          apk --no-cache add curl tar;
          curl -L "$ARGO_CUSTOM_PLUGIN_URL" | tar -xvzf -;
          cp -r kustomize-sed-transformer-main/plugins/* $ARGO_CUSTOM_PLUGIN_HOME;
          chmod -R 777 $ARGO_CUSTOM_PLUGIN_HOME;
          ls -lart $ARGO_CUSTOM_PLUGIN_HOME;
```

### How to use
```
kind: Kustomization
resources:
  - deployment.yml
  - service.yml

helmCharts:
  - name: yugabyte
    includeCRDs: true
    repo: https://charts.yugabyte.com
    releaseName: yugabyte-db
    version: 2.21.0
    additionalValuesFiles:
      - values-yugabyte.yaml
...

# Plugin Usage
transformers:
  - |-
    apiVersion: argoproj.custom.plugins/v1
    kind: SedTransformer
    metadata:
      name: sed-transformer-argocd-tools
    argsOneLiner: s/argocd-tools/yugabyte-db/g

```
