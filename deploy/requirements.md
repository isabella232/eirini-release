Secrets that Eirini requires:
in the same namespace of eirini components:

| Secret Name | Key         | Value                     |
| ----------- | ----------- | ------------------------- |
| capi        | cc.crt      | client certificate        |
|             | cc.key      | client private key        |
| capiCA      | cc.ca       | cloud controller CA       |
| eirini      | eirini.crt  | eirini client certificate |
|             | eirini.key  | eirini private key        |
| eiriniCA    | eirini.ca   |                           |
| doppler     | doppler.crt |                           |
|             | doppler.key |                           |
| dopplerCA   | doppler.ca  |                           |
| nats        | password    |                           |

optional:

- registry-credentials
  docker private registry credentials
  not needed if bits is not used

in all apps namespaces

- ccUploader
  cc_uploader.crt
  cc_uploader.key
  note: this is only needed if native staging is used
- eirini
  eirini.crt
  eirini.key
  note: this is only needed if native staging is used
- cfCA
  ca.crt
  note: this is only needed if native staging is used

serviceAccounts that Eirini requires:
in all apps namespaces:

- application_service_account
  - roles, bindings, psps required for this to work
    e.g. deploy/example-app/serviceaccount.yaml
- staging_service_account
  note: only needed if native staging is used
  - roles, bindings, psps required for this to work
    e.g. deploy/example-app-staging/serviceaccount.yaml

networkPolicy (optional):

- app network policy
  deny app ingress
  should be managed by the networking team / capi
  e.g.
  ```
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: deny-app-ingress
    namespace: eirini-apps
  spec:
    policyTypes:
    - Ingress
    ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            name: cf
        podSelector:
          matchLabels:
            app.kubernetes.io/component: router
      - namespaceSelector:
          matchLabels:
            name: cf
        podSelector:
          matchLabels:
            app.kubernetes.io/component: adapter
  ```

Hardcode in eirini:

- opi.yml
  opi:
  registry_secret_name: registry-credentials
  eirini_address: "https://eirini_service_name.EIRINI_NAMESPACE.svc.cluster.local:8085"
  EIRINI_NAMESPACE can be an env variable in Eirini
  we can get it using the downward api

        cc_uploader_secret_name: ccUploaderSecret
        cc_uploader_cert_path: cc_uploader.crt
        cc_uploader_key_path: cc_uploader.key

        client_certs_secret_name: eiriniClientSecret
        client_cert_path: eirini.crt
        client_key_path: eirini.key

        ca_cert_secret_name: cfCA
        ca_cert_path: ca.crt

        cc_cert_path: "/workspace/jobs/opi/secrets/cc.crt"
        cc_key_path: "/workspace/jobs/opi/secrets/cc.key"
        cc_ca_path: "/workspace/jobs/opi/secrets/cc.ca"

        client_ca_path: "/workspace/jobs/opi/secrets/eirini.ca"
        server_cert_path: "/workspace/jobs/opi/secrets/eirini-server.crt"
        server_key_path: "/workspace/jobs/opi/secrets/eirini-server.key"

- metrics.yml
  loggergator_cert_path: "/etc/eirini/secrets/doppler.crt"
  loggregator_key_path: "/etc/eirini/secrets/doppler.key"
  loggregator_ca_path: "/etc/eirini/secrets/doppler.ca"

- events.yml
  cc_cert_path: "/etc/eirini/secrets/cc.crt"
  cc_key_path: "/etc/eirini/secrets/cc.key"
  cc_ca_path: "/etc/eirini/secrets/cc.ca"

- staging-reporter.yml
  eirini_cert_path: "/etc/eirini/secrets/eirini-client.crt"
  eirini_key_path: "/etc/eirini/secrets/eirini-client.key"
  ca_path: "/etc/eirini/secrets/eirini-client.ca"

- task-reporter.yml
  cc_cert_path: "/etc/eirini/secrets/cc.crt"
  cc_key_path: "/etc/eirini/secrets/cc.key"
  ca_path: "/etc/eirini/secrets/cc.ca"

- lrp-controller.yml
  eirini_cert_path: "/etc/eirini/secrets/eirini-client.crt"
  eirini_key_path: "/etc/eirini/secrets/eirini-client.key"
  ca_path: "/etc/eirini/secrets/eirini-client.ca"

  eirini_address: "https://eirini_service_name.EIRINI_NAMESPACE.svc.cluster.local:8085"
  EIRINI_NAMESPACE can be an env variable in Eirini
  we can get it using the downward api
