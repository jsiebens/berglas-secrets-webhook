# Berglas Secrets webhook

This chart will install a mutating admission webhook, that injects an executable to containers in a deployment/statefulset which than can request secrets using Berglas through environment variable definitions.

## Before you start

Before you install this chart you must create a namespace for it, this is due to the order in which the resources in the charts are applied (Helm collects all of the resources in a given Chart and it's dependencies, groups them by resource type, and then installs them in a predefined order (see [here](https://github.com/helm/helm/blob/release-2.10/pkg/tiller/kind_sorter.go#L29) - Helm 2.10).

The `MutatingWebhookConfiguration` gets created before the actual backend Pod which serves as the webhook itself, Kubernetes would like to mutate that pod as well, but it is not ready to mutate yet (infinite recursion in logic).

### Creating the namespace

The namespace must have a label of `name` with the namespace name as it's value.

set the target namespace name or skip for the default name: berglas-secrets-webhook

```bash
export WEBHOOK_NS=`<namepsace>`
```

```bash
WEBHOOK_NS=${WEBHOOK_NS:-berglas-secrets-webhook}
echo kubectl create namespace "${WEBHOOK_NS}"
echo kubectl label ns "${WEBHOOK_NS}" name="${WEBHOOK_NS}"
```

## Installing the Chart

```bash
helm upgrade --namespace "${WEBHOOK_NS}" --install berglas-secrets-webhook helm-chart
```

## Configuration

The following tables lists configurable parameters of the berglas-secrets-webhook chart and their default values.

|               Parameter             |                    Description                    |                  Default                 |
| ----------------------------------- | ------------------------------------------------- | -----------------------------------------|
|affinity                             |affinities to use                                  |{}                                        |
|image.pullPolicy                     |image pull policy                                  |IfNotPresent                              |
|image.repository                     |image repo that contains the admission server      |weareonthespot/berglas-secrets-webhook    |
|image.tag                            |image tag                                          |0.1.0                                     |
|nodeSelector                         |node selector to use                               |{}                                        |
|replicaCount                         |number of replicas                                 |1                                         |
|resources                            |resources to request                               |{}                                        |
|service.externalPort                 |webhook service external port                      |443                                       |
|service.internalPort                 |webhook service external port                      |443                                       |
|service.name                         |webhook service name                               |berglas-secrets-webhook                   |
|service.type                         |webhook service type                               |ClusterIP                                 |
|tolerations                          |tolerations to add                                 |[]                                        |

## Limitations

The mutator requires that containers specify a `command` in their manifest. If a
container requests Berglas secrets and does not specify a `command`, the mutator
will log an error and not mutate the spec.

## Credits

This project is derived from the Berglas Kubernetes examples available [here](https://github.com/GoogleCloudPlatform/berglas/tree/master/examples/kubernetes), and is strongly inspired by the [Vault Mutating Webhook](https://github.com/innovia/kubernetes-mutation-webhook-vault-secrets)