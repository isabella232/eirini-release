# De-helmifying eirini release

## Approach

- Separated release into its binary components: core, events, metrics, ...
  - Support the `enabled: false` strategy in helm to disabled unwanted components
- Extracted pure YAML from helm templates
  - Secret names and paths defaulted
    - documented in requirements.txt
    - Rely on user to create appropriate secrets in the correct places
  - Internal implementation details removed, e.g. paths to secret files from config map
  - Namespaces removed so that it can be passed in `kubectl apply`
  - Release version SHAs hard-coded
- ConfigMap is the single point of configuration
  - No need to modify other yaml
- Switch to ClusterRoles to support unknown multiple app namespaces for list/watch permissions
- Ignore obsolete components:
  - secret smuggler
  - fluentd logging
  - rootfs patching
  - network policy to allow only ingress from router or adapter
- Ignore app namespace settings:
 - PSP for apps and staging
 - Document these requirements instead

## Problems

- Binding a ClusterRole or Role to a ServiceAccount requires the namespace of the ServiceAccount
regardless of the `kubectl apply` namespace
  - Cannot be templated in pure YAML
  - Options:
    - Only document service account and role requirements, and let implementer create them
      - Is it strange to not provide YAML for this, as its our requirement?
    - Use a templating tool, such as `kustomize`, to insert the namespaces
      - Requires modifying namespace in each component directory, for each target namespace. Tedious.
    - Provide the ServiceAccount, Roles, PSPs, but not the bindings
      - Document the binding requirements instead
- ServiceName and service port are duplicated in the service definition and eirini's configmap
  - Don't think there's a way to remove this duplication without overcomplicating eirini itself
  - Not sure if it's a major problem really, but something worth noting
