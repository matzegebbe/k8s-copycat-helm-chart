<p align="center">
  <img src="./k8s-copycat-helm-chart-logo.png" alt="k8s-copycat logo" width="180" />
</p>

# k8s-copycat Helm Chart

This repository packages the Helm chart for [k8s-copycat](https://github.com/matzegebbe/k8s-copycat), a controller that mirrors Kubernetes workloads' container images to a target registry. The chart is published as an OCI artifact to GitHub Container Registry (GHCR) and includes sensible defaults to run the controller securely.

## Contents

- [Requirements](#requirements)
- [Quickstart](#quickstart)
- [Configuration](#configuration)
- [Examples](#examples)
- [Development](#development)

## Requirements

- Helm 3.8 or later (OCI support is built-in)
- Kubernetes 1.22+ (as defined in the chart's `kubeVersion`)
- Permissions to push/pull from GHCR and to access your destination image registry

## Quickstart

1. **Authenticate to GHCR** (uses your GitHub token):

   ```bash
   echo "$GITHUB_TOKEN" | helm registry login ghcr.io --username "$GITHUB_ACTOR" --password-stdin
   ```

2. **Install or upgrade the chart** from the OCI registry:

   ```bash
   CHART_VERSION="<desired version>"  # e.g., 0.6.2
   NAMESPACE="k8s-copycat"

   helm upgrade --install k8s-copycat \
     oci://ghcr.io/matzegebbe/charts/k8s-copycat \
     --version "${CHART_VERSION}" \
     --namespace "${NAMESPACE}" \
     --create-namespace
   ```

3. **Inspect defaults** without installing:

   ```bash
   helm show values oci://ghcr.io/matzegebbe/charts/k8s-copycat --version "${CHART_VERSION}" > values.example.yaml
   ```

4. **Uninstall**:

   ```bash
   helm uninstall k8s-copycat --namespace "${NAMESPACE}"
   ```

## Configuration

Override any value with `--set`/`--set-file` or by supplying a custom `values.yaml` during installation. The table below highlights the most important chart options; see [`values.yaml`](./values.yaml) for the full set and defaults.

### Deployment

| Parameter | Description | Default |
| --- | --- | --- |
| `replicaCount` | Number of controller replicas. | `1` |
| `image.repository` | Container image repository. | `ghcr.io/matzegebbe/k8s-copycat` |
| `image.tag` | Image tag (empty defaults to chart `appVersion`). | `""` |
| `image.pullPolicy` | Image pull policy. | `IfNotPresent` |
| `imagePullSecrets` | List of image pull secrets. | `[]` |
| `nameOverride` / `fullnameOverride` | Override chart and release names. | `""` |
| `commonLabels` / `commonAnnotations` | Extra labels/annotations applied to all resources. | `{}` |
| `namespace.create` | Create the target namespace. | `false` |
| `namespace.annotations` / `namespace.labels` | Extra metadata for the namespace when created. | `{}` |

### Security and RBAC

| Parameter | Description | Default |
| --- | --- | --- |
| `serviceAccount.create` | Create a ServiceAccount for the controller. | `true` |
| `serviceAccount.name` | Custom ServiceAccount name. | `""` |
| `serviceAccount.annotations` / `serviceAccount.labels` | Additional metadata for the ServiceAccount. | `{}` |
| `rbac.create` | Create ClusterRole and ClusterRoleBinding. | `true` |
| `rbac.clusterRole.annotations` / `rbac.clusterRole.labels` | Metadata for the ClusterRole. | `{}` |
| `rbac.clusterRole.rules` | RBAC rules for watched resources. | See `values.yaml` |
| `rbac.clusterRoleBinding.annotations` / `rbac.clusterRoleBinding.labels` | Metadata for the ClusterRoleBinding. | `{}` |
| `podSecurityContext` | Pod-level security context. | `{}` |
| `securityContext.readOnlyRootFilesystem` | Mount root filesystem as read-only. | `true` |
| `securityContext.allowPrivilegeEscalation` | Disable privilege escalation. | `false` |
| `securityContext.capabilities.drop` | Linux capabilities to drop. | `["ALL"]` |
| `priorityClassName` | Optional PriorityClass for the pod. | `""` |

### Pod settings

| Parameter | Description | Default |
| --- | --- | --- |
| `command` / `args` | Override container entrypoint/arguments. | `[]` |
| `extraEnv` / `extraEnvFrom` | Additional environment variables. | `[]` |
| `extraVolumeMounts` / `extraVolumes` | Extra mounts and volumes. | `[]` |
| `containerPorts.metrics` | Metrics port. | `8080` |
| `containerPorts.health` | Liveness/readiness port. | `8081` |
| `resources.requests.cpu` / `resources.requests.memory` | Minimum resources. | `50m` / `64Mi` |
| `resources.limits.cpu` / `resources.limits.memory` | Resource limits. | `500m` / `256Mi` |
| `livenessProbe.enabled` | Enable liveness probe. | `true` |
| `readinessProbe.enabled` | Enable readiness probe. | `true` |
| `podAnnotations` / `podLabels` | Extra metadata for pods. | `{}` |
| `podTopologySpreadConstraints` | Topology spread constraints. | `[]` |
| `nodeSelector` / `tolerations` / `affinity` | Scheduling rules for pods. | `{}` / `[]` / `{}` |
| `podDisruptionBudget.enabled` | Create a PodDisruptionBudget. | `false` |
| `podDisruptionBudget.minAvailable` | Minimum available pods if enabled. | `1` |
| `podDisruptionBudget.maxUnavailable` | Maximum unavailable pods if enabled. | `null` |

### Networking and metrics

| Parameter | Description | Default |
| --- | --- | --- |
| `service.enabled` | Create a Service for metrics/health. | `true` |
| `service.type` | Kubernetes service type. | `ClusterIP` |
| `service.port` / `service.targetPort` | Service port/target port. | `8080` / `metrics` |
| `service.portName` | Name for the metrics port. | `http-metrics` |
| `service.annotations` / `service.labels` | Additional metadata for the Service. | `{}` |
| `serviceMonitor.enabled` | Create a `ServiceMonitor` for Prometheus Operator. | `false` |
| `serviceMonitor.namespace` | Namespace for the ServiceMonitor. | `""` |
| `serviceMonitor.interval` / `serviceMonitor.scrapeTimeout` | Scrape interval and timeout. | `30s` / `10s` |
| `serviceMonitor.scheme` / `serviceMonitor.path` | Scrape scheme and path. | `http` / `/metrics` |
| `serviceMonitor.namespaceSelector` | Custom namespace selector. | `null` |
| `serviceMonitor.relabelings` / `serviceMonitor.metricRelabelings` | Optional relabeling rules. | `[]` |
| `serviceMonitor.targetLabels` | Labels to copy onto generated metrics. | `[]` |

### Configuration files

| Parameter | Description | Default |
| --- | --- | --- |
| `configMap.create` | Create the bundled ConfigMap. | `true` |
| `configMap.existingConfigMap` | Use an existing ConfigMap instead of creating one. | `""` |
| `configMap.key` | Key name for the configuration file. | `config.yaml` |
| `configMap.filename` | Filename rendered in the mounted directory. | `config.yaml` |
| `configMap.mountPath` | Directory path where the config is mounted. | `/config` |
| `configMap.defaultMode` | File mode applied to projected files. | `420` |
| `configMap.annotations` / `configMap.labels` | Metadata for the ConfigMap. | `{}` |

### Controller behavior

These values render directly into `config.yaml` and are consumed by the controller. Update them to match your target registry and mirroring preferences.

| Parameter | Description | Default |
| --- | --- | --- |
| `config.targetKind` | Target registry kind (e.g., `docker`, `ecr`). | `docker` |
| `config.logLevel` | Controller log level. | `debug` |
| `config.dryRun` | Simulate reconciliation without pushing images. | `true` |
| `config.dryPull` | Skip pulling source images. | `true` |
| `config.includeNamespaces` | Namespaces to include. | `["k8s-copycat"]` |
| `config.skipNamespaces` | Namespaces to skip. | `[]` |
| `config.skipNames.deployments/statefulSets/daemonSets/jobs/cronJobs/pods` | Resource names to ignore. | `[]` |
| `config.digestPull` | Prefer digest-based pulls. | `false` |
| `config.checkNodePlatform` | Require matching node platform. | `false` |
| `config.mirrorPlatforms` | Target platforms to mirror. | `[]` |
| `config.allowDifferentDigestRepush` | Allow repush with differing digests. | `true` |
| `config.excludeRegistries` | Registries to ignore. | `[]` |
| `config.watchResources` | Specific resources to watch. | `[]` |
| `config.maxConcurrentReconciles` | Concurrent reconciliations. | `1` |
| `config.requestTimeout` | Request timeout in seconds. | `120` |
| `config.failureCooldownMinutes` | Cooldown after a failure. | `null` |
| `config.forceReconcileMinutes` | Forced reconcile interval. | `null` |
| `config.ecr.accountID` / `config.ecr.region` / `config.ecr.repoPrefix` | AWS ECR settings. | `""` / `""` / `""` |
| `config.ecr.createRepo` | Create target ECR repos automatically. | `false` |
| `config.ecr.lifecyclePolicy` | Optional ECR lifecycle policy JSON. | `""` |
| `config.docker.registry` | Target Docker registry. | `""` |
| `config.docker.repoPrefix` | Prefix for mirrored images. | `""` |
| `config.docker.insecure` | Allow insecure registry connections. | `false` |
| `config.registryCredentials` | Auth config for source/target registries. | `[]` |
| `config.pathMap` | Explicit source/target image mappings. | `[]` |

## Examples

### Minimal install pointing to a Docker registry

```yaml
config:
  targetKind: docker
  docker:
    registry: registry.example.com
    repoPrefix: mirror/
  includeNamespaces:
    - default
  dryRun: false
  dryPull: false
serviceMonitor:
  enabled: true
```

Install with:

```bash
helm upgrade --install k8s-copycat \
  oci://ghcr.io/matzegebbe/charts/k8s-copycat \
  --version "${CHART_VERSION}" \
  --namespace "${NAMESPACE}" \
  -f ./custom-values.yaml
```

### Using an existing ConfigMap for controller settings

```yaml
configMap:
  create: false
  existingConfigMap: copycat-config
```

## Automation

- Dependabot runs daily to update GitHub Actions and Docker image tags. The Docker rule tracks the chart's default Copycat image (`ghcr.io/matzegebbe/k8s-copycat`) so new upstream releases surface as PRs that refresh the `image.tag` override in `values.yaml`.

## Development

- Run `helm lint` and `helm template` locally before submitting changes.
- Release automation packages the chart from the repository root and publishes it to `ghcr.io/matzegebbe/charts`. Chart versions follow the Git tag (e.g., tag `v0.6.2` produces chart version `0.6.2`).
- Update [`CHANGELOG.md`](./CHANGELOG.md) and [`Chart.yaml`](./Chart.yaml) metadata when making user-facing changes.
