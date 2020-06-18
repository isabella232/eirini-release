#Eirini Components
Eirini consists of the following components:

| Component        | Description                                                                                                                                                             | Sample Deployment                |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------- |
| core             | The Eirini server                                                                                                                                                       | core/deployment.yaml             |
| events           | Monitors the state of application pods and sends an event upon an application pod crash to the cloud controller                                                         | events/deployment.yaml           |
| lrp_controller   | Controller for the `LRP` (i.e. application) custom resource                                                                                                             | lrp_controller/deployment.yaml   |
| metrics          | Collects various metrics (CPU/disk/memory usage) from application pods and sends metrics events to Loggregator over Doppler                                             | metrics/deployment.yaml          |
| routes           | Sends route configuration of application pods to the configured NATS.io server                                                                                          | routes/deployment.yaml           |
| staging-reporter | Notifies the Eirini server (`core`) whenever an application fails to stage. Consequently the Eirini server notifies the cloud controller (CC) about the staging failure | staging-reporter/deployment.yaml |
| task-reporter    | Notifies the cloud controller that a task has completed of failed                                                                                                       | task-reporter/deployment.yaml    |

All the components above require that the namespace they are deployed to meet certain requirements (see below).

# Secrets

## Required secrets in the Eirini components namespace(s)

The table below describes all the secrets that need to be provided in the namespace where an Eirini component is deployed

| Component        | Secret Name   | Key               | Value                      |
| ---------------- | ------------- | ----------------- | -------------------------- |
| core             | capi          | cc.crt            | CC client certificate      |
|                  |               | cc.key            | CC client private key      |
|                  |               | cc.ca             | CC CA                      |
|                  | eirini-server | eirini-server.crt | eirini server certificate  |
|                  |               | eirini-server.key | eirini server private key  |
|                  |               | eirini.ca         | eirini CA                  |
| ---------------- | -----------   | -----------       | -------------------------- |
| events           | capi          | cc.crt            | CC client certificate      |
|                  |               | cc.key            | CC client private key      |
|                  |               | cc.ca             | CC CA                      |
| ---------------- | -----------   | -----------       | -------------------------- |
| lrp_controller   | eirini-client | eirini-client.crt | eirini client certificate  |
|                  |               | eirini-client.key | eirini client private key  |
|                  |               | eirini.ca         | eirini CA                  |
| ---------------- | -----------   | -----------       | -------------------------- |
| metrics          | doppler       | doppler.crt       | Doppler client certificate |
|                  |               | doppler.key       | Doppler client private key |
|                  |               | doppler.ca        | Doppler CA                 |
| ---------------- | -----------   | -----------       | -------------------------- |
| staging-reporter | eirini-client | eirini-client.crt | eirini client certificate  |
|                  |               | eirini-client.key | eirini client private key  |
|                  |               | eirini.ca         | eirini CA                  |
| ---------------- | -----------   | -----------       | -------------------------- |
| task-reporter    | capi          | cc.crt            | CC client certificate      |
|                  |               | cc.key            | CC client private key      |
|                  |               | cc.ca             | CC CA                      |

## Required secrets in the applications namespace(s) (Optional)

The following secrets need to be provided in the applications namespace(s) if native staging is used:

| Secret Name          | Key               | Value                                    |
| -------------------- | ----------------- | ---------------------------------------- |
| registry-credentials |                   | bits docker private registry credentials |
| cc-uploader          | cc_uploader.crt   | CC client certificate                    |
|                      | cc_uploader.key   | CC client private key                    |
|                      | cc_uploader.ca    | CC CA                                    |
| eirini               | eirini-client.crt | eirini client certificate                |
|                      | eirini-client.key | eirini client private key                |
|                      | eirini.ca         | eirini CA                                |

# Sevice Accounts

The following service accounts, roles, roles bindings and pod security policies are required in the application namespace(s):

| Type                        | Sample Service Account                  |
| --------------------------- | --------------------------------------- |
| Application service account | example-app/serviceaccount.yaml         |
| Staging service account     | example-app-staging/serviceaccount.yaml |

# Applications Network Policies (Optional)

Those should be managed by CAPI/networking. however, here is a sample denying applications ingress

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
