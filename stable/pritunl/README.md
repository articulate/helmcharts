# Pritunl Helm Chart

This project will install a Pritunl server-side deployment in a Kubernetes cluster. It was written with Amazon EKS in mind.

This creates a `Deployment` of Pritunl clustered with a default replica count of `3`. The Pritunl instances will use the associated `mongodb` database instance to function in a cluster without further configuration.

Don't skip over the "Important Considerations" section at the end before installing!

**NOTE:** Install a `mongodb` chart first! Don't skip this step. Once that's done, you can install the `pritunl` chart using the typical commands. The chart, by default, expects `mongodb` to be reachable via `pritunl-mongodb.{{ .Release.Namespace }}.svc.cluster.local`.

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

|         Parameter    |                                   Description                                                                                                             |     Default      |
|----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------|----------------  |
| `image.registry`     | Image repository where the `pritunl` image is hosted.                                                                                                     | `docker.io`      |
| `image.repository`   | Image name path.                                                                                                                                          | `eb129/pritunl`  |
| `image.tag`          | Image tag.                                                                                                                                                | `latest`         |
| `image.pullPolicy`   | Image pull policy.                                                                                                                                        | `Always`         |
| `mongoService`       | The DNS name of the `mongodb` `Service` running in the cluster.                                                                                           | `pritunl-mongodb`|
| `ports.http`         | The port that the pod will listen for HTTP requests on.                                                                                                   | `80`             |
| `ports.vpn`          | The port that the pod will listen for inbound VPN connection requests on.                                                                                 | `1194`           |
| `ports.webui`        | The port that the pod will listen for the Pritunl user interface on.                                                                                      | `443`            |
| `privileged.enabled` | Whether or not the containers should be privileged. Should be enabled unless there's a good reason not to be.                                             | `true`           |
| `replicaCount`       | The number of pods that will run in the `ReplicaSet` as a part of the `Deployment`. This is effectively how many HA nodes you want.                       | `3`              |
| `service.annotations`| The annotations required on the `Service` when using this behind an Amazon Elastic Load Balancer. The values are configurable, see the `values.yaml` file.| See `values.yaml`|
| `service.type`       | The type declaration for the `Service`. Can be `NodePort` or `LoadBalancer`.                                                                              | `LoadBalancer`   |
| `tty.enabled`        | Allocate a TTY for the Pritunl containers. This needs to be on so you can gain access to the Pritunl pods for troubleshooting, etc.                       | `true`           |

## Installation

`helm install pritunl --name pritunl`

To test the associated manifests, the following commands can be run independently from within the Helm directory where the chart is stored:
`helm install --debug --dry-run --name pritunl ./pritunl`

## Pritunl Configuration

The entrypoint for the Docker image is a script called `start_pritunl.sh` that executes the following commands:

* `pritunl set-mongodb mongodb://$MONGO_HOST:27017/pritunl`
* `pritunl set app.reverse_proxy true`
* `pritunl set app.server_ssl false` => If you're testing this on your local machine without deploying to AWS, this should be set to `true`.
* `pritunl set app.server_port 1194`
* `pritunl start`

In this case, `$MONGO_HOST` is a variable passed in by Kubernetes that is configurable in Pritunl's `values.yaml` file as `mongoService`. This variable should reflect the resolvable service name of `pritunl-mongodb`. If any of this is changed, **be sure to update this variable**.

The next three options are necessary to allow Pritunl to function behind an ELB.

If there are other commands that need to be run, it is recommended to rebuild the Docker image and update the entrypoint script.

## Important Considerations

* The `mongodb` chart should be installed first. We recommend using the official `monbodb` Helm chart for this.
* The charts should be installed with the names `pritunl` and `pritunl-mongodb` - doing anything else will break functionality unless you have explicitly changed the `mongoService` variable mentioned above.
* You may need to create a `pritunl` user within MongoDB depending on your setup. Check out https://docs.pritunl.com/docs/securing-mongodb.
