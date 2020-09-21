# Pritunl Helm Chart

This project will install a Pritunl server-side deployment in a Kubernetes cluster. It was originally written for use in Amazon EKS.

This creates a `Deployment` of Pritunl clustered with a default replica count of `3`. The Pritunl instances will use the associated `mongodb` database instance to function in a cluster without further configuration.

Don't skip over the "Important Considerations" section at the end before installing!

**NOTE:** Install a `mongodb` chart first! Don't skip this step. Once that's done, you can install the `pritunl` chart using the typical commands. The chart, by default, expects `mongodb` to be reachable via `pritunl-mongodb.{{ .Release.Namespace }}.svc.cluster.local`.

**HoweverIf** - if you want to run with built-in `mongodb` without an external database, set `mongo.useLocalDB` to `true`. This will use `mongodb` directly on the Pritunl `Pods`, but this will not persist your data. This is handy for a proof-of-concept or for testing.

## Prerequisites

* [Helm](https://github.com/helm/helm) must be installed to manage or install the charts.

* It is also assumed that an **EKS cluster** is available to deploy within.

* You should deploy `mongodb` using the official chart before deploying `pritunl`. See https://docs.pritunl.com/docs/securing-mongodb regarding creating a `pritunl` user, etc.

* In order for the annotation `external-dns.alpha.kubernetes.io/hostname: "pritunl.mydomain.com"` to function properly, you must have [external-dns](https://github.com/kubernetes-incubator/external-dns) running in your EKS cluster.

## Usage

The `pritunl` service created as part of the chart will create an ELB if the service type is `LoadBalancer` and uses the following **annotations** to function properly:

* `external-dns.alpha.kubernetes.io/hostname: "pritunl.mydomain.com"`
* `service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "arn:aws:acm:(region):(account-number):certificate/(certificate-id)"`
* `service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp`
* `service.beta.kubernetes.io/aws-load-balancer-ssl-ports: webui`
* `service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy: "ELBSecurityPolicy-TLS-1-2-2017-01"` (default)

## Configurable Values:

| Parameter             | Description                                                                                                                                                | Default           |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- |
| `image.registry`      | Image repository where the `pritunl` image is hosted.                                                                                                      | `docker.io`       |
| `image.repository`    | Image name path.                                                                                                                                           | `eb129/pritunl`   |
| `image.tag`           | Image tag.                                                                                                                                                 | `latest`          |
| `image.pullPolicy`    | Image pull policy.                                                                                                                                         | `Always`          |
| `mongo.serviceName`   | The DNS name of the `mongodb` `Service` running in the cluster.                                                                                            | `pritunl-mongodb` |
| `mongo.useLocalDB`    | If set to `true`, use `mongodb` directly on the Pritunl `Pods`.                                                                                            | `false`           |
| `ports.http`          | The port that the pod will listen for HTTP requests on.                                                                                                    | `80`              |
| `ports.vpn`           | The port that the pod will listen for inbound VPN connection requests on.                                                                                  | `1194`            |
| `ports.webui`         | The port that the pod will listen for the Pritunl user interface on.                                                                                       | `443`             |
| `privileged.enabled`  | Whether or not the containers should be privileged. Should be enabled unless there's a good reason not to be.                                              | `true`            |
| `replicaCount`        | The number of pods that will run in the `ReplicaSet` as a part of the `Deployment`. This is effectively how many HA nodes you want.                        | `3`               |
| `service.annotations` | The annotations required on the `Service` when using this behind an Amazon Elastic Load Balancer. The values are configurable, see the `values.yaml` file. | See `values.yaml` |
| `service.type`        | The type declaration for the `Service`. Can be `NodePort` or `LoadBalancer`.                                                                               | `LoadBalancer`    |
| `tty.enabled`         | Allocate a TTY for the Pritunl containers. This needs to be on so you can gain access to the Pritunl pods for troubleshooting, etc.                        | `true`            |

## Installation

`helm install pritunl --name pritunl`

To test the associated manifests, the following commands can be run independently from within the Helm directory where the chart is stored:
`helm install --debug --dry-run --name pritunl ./pritunl`

## Pritunl Configuration

The entrypoint for the Docker image is a script called `start_pritunl.sh` that bootstraps the `Pods` based on whether or not `mongodb` is external or colocated. The final `pritunl` settings are used to make the deployment work behind an ELB.

## Important Considerations

* The `mongodb` chart should be installed first. We recommend using the official `mongodb` Helm chart for this. Alternatively, there's an option to run without an external database, but it isn't recommended for long term or production use.
* The charts should be installed with the names `pritunl` and `pritunl-mongodb` - doing anything else will break functionality unless you have explicitly changed the `mongo.serviceName` variable mentioned above.
* You may need to create a `pritunl` user within MongoDB depending on your setup. Check out https://docs.pritunl.com/docs/securing-mongodb.
