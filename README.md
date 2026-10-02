# hellnet-charts

Helm chart repository for hellnet, served at <https://charts.hellnet.com.br/charts>.

```bash
helm repo add hellnet https://charts.hellnet.com.br/charts
helm repo update
```

## Charts

| Chart | Description |
|---|---|
| `hellnet-service` | Generic chart for a hellnet Go service: Deployment with restricted pod security, optional ConfigMap, Service and Gateway API `HTTPRoute` |
| `test-app` | Minimal chart used to test the repository |

### hellnet-service

```bash
helm install my-service hellnet/hellnet-service \
  --set image.repository=ghcr.io/guilhermelinosp/my-service --set image.tag=v1.0.0 \
  --set containerPort=8080 --set route.enabled=true --set 'route.hostnames={my-service.hellnet.com.br}'
```

The release name names every resource. Main values (see [`hellnet-service/values.yaml`](hellnet-service/values.yaml)):

| Value | Description |
|---|---|
| `image.repository`, `image.tag` | image to run (**required**) |
| `command`, `args` | override the image entrypoint |
| `serviceName` | `HELLNET_SERVICE`, the service name reported by telemetry |
| `config` | plain environment (a ConfigMap loaded with `envFrom`) |
| `secretEnv` | variables read from existing Secrets (`name`, `secretName`, `key`) |
| `containerPort` | HTTP port; `0` means no HTTP (no probes, Service or route) |
| `strategy` | `Recreate` for singletons such as an outbox publisher |
| `route.*` | Gateway API `HTTPRoute` (`hostnames`, `parentRef`) |

Pods run as UID/GID `65532` (numeric, so the kubelet can verify `runAsNonRoot`), with a read-only root filesystem,
no privilege escalation, all capabilities dropped and the `RuntimeDefault` seccomp profile, which satisfies the
`restricted` Pod Security Standard.

### Argo CD

```yaml
source:
  repoURL: https://charts.hellnet.com.br/charts
  chart: hellnet-service
  targetRevision: 0.1.0
  helm:
    valuesObject:
      image: {repository: ghcr.io/guilhermelinosp/my-service, tag: v1.0.0}
```

## Publishing

```bash
helm package <chart> -d charts
helm repo index charts --url https://charts.hellnet.com.br/charts --merge charts/index.yaml
```

Merging to `gh-pages` publishes the repository through GitHub Pages.

## License

[Apache 2.0](LICENSE)
