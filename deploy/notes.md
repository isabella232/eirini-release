# De-helmifying eirini release

## Approach

- Separated release into its binary components: core, events, metrics, ...
  - This supports excluding certain components, like using `enabled: false` in the helm configuration
- Defaulted the eirini core namespace to `eirini-system`
  - This can be overridden with kustomize, ytt etc.
- Extracted pure YAML from helm templates
  - Secret names and paths defaulted
    - documented in requirements.txt
    - Rely on user to create appropriate secrets in the correct places
  - Internal implementation details removed, e.g. paths to secret files from config map
  - Release version SHAs hard-coded
- ConfigMap is the single point of configuration
  - No need to modify other yaml
- Switch to ClusterRoles to support unknown multiple app namespaces for list/watch permissions
- Ignore obsolete components, or where responsibility lies elsewhere:
  - secret smuggler
  - fluentd logging
  - rootfs patching
  - network policy to allow only ingress from router or adapter
- Ignore app namespace settings:
 - e.g. PSPs for apps and staging
 - Document these requirements instead

## Problems

- ServiceName and service port are duplicated in the service definition and eirini's configmap
  - Don't think there's a way to remove this duplication without overcomplicating eirini itself
  - Not sure if it's a major problem really, but something worth noting
