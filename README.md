# Cluster Monitoring Operator

The Cluster Monitoring Operator manages and updates the Prometheus-based cluster monitoring stack deployed on top of OpenShift.

It contains the following components:

* [Prometheus Operator](https://github.com/coreos/prometheus-operator)
* [Prometheus](https://github.com/prometheus/prometheus)
* [Alertmanager](https://github.com/prometheus/alertmanager) cluster for cluster and application level alerting
* [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)
* [node_exporter](https://github.com/prometheus/node_exporter)

The deployed Prometheus Operator is intended to be used only for cluster-level monitoring.
As such, the deployed Prometheus instance (`prometheus-k8s`) is responsible for monitoring and alerting on cluster and OpenShift components; it should not be extended to monitor user applications.
**Important:** The Prometheus Operator managed by the Cluster Monitoring Operator will by default only look for `ServiceMonitor` resources in `openshift-monitoring` namespace.

Users interested in leveraging Prometheus for application monitoring on OpenShift should consider using [OLM](https://github.com/operator-framework/operator-lifecycle-manager) to easily deploy a Prometheus Operator and setup new Prometheus instances to monitor and alert on their applications.

Alertmanager is a cluster-global component for handling alerts generated by all Prometheus instances deployed in that cluster.

Metrics are collected from the following components:

* [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)
* [node_exporter](https://github.com/prometheus/node_exporter)
* Kubelets
* API server
* Prometheus (just `prometheus-k8s` for now)
* Alertmanager

## Adding new metrics to be sent via telemetry

To add new metrics to be sent via telemetry, simply add a selector that matches the time-series to be sent in [manifests/0000_50_cluster_monitoring_operator_04-config.yaml](manifests/0000_50_cluster_monitoring_operator_04-config.yaml).

Documentation on the data sent can be found in the [data collection documentation](Documentation/data-collection.md).

## Roadmap

* Monitor etcd
* Adapt Tectonic inherited alerts with OpenShift operational knowledge

## Testing

- **Unit tests**: `make test-unit`

- **End-to-end tests**: `make test-e2e`

## Contributing

### Notes
Cluster Monitoring Operator is part of OpenShift and therefore follows the [OpenShift Life Cycle](https://access.redhat.com/support/policy/updates/openshift)

You should keep this in mind when decding in which release you want your feature or fix.

### Technical procedure
Before you get started, you have to perform the following *mandatory* steps:
* Open an bug in Bugzilla
* Fork this repository

Very little things are defined explictly in this repository but most things are imported from upstream projects instead.
Therefore you should have [jsonnet bundler](https://github.com/jsonnet-bundler/jsonnet-bundler) installed and [udpated](https://github.com/coreos/kube-prometheus#update-jb).

Assuming you have made your changes upstream ([see an example change](https://github.com/kubernetes-monitoring/kubernetes-mixin/pull/466/files)), you can now go ahead
and update the dependency here:

```
cd jsonnet
jb update
```

Now make sure that you only update or adjust the dependency you need to and commit that update

```
git add -p jsonnet/jsonnet.lock.json
git commit -m 'jsonnet: <meaningful message about what you did>'
git push
git checkout jsonnet/jsonnet.lock.json
```

The last step is to regenerate all assets.
This is easiest done in a container using the following command;

```
make generate-in-docker
```
or if you are on a Mac

```
MAKE=make make generate-in-docker
```

At this point, you should follow a standard git workflow:

* review your changes using `git diff`
* add all generated files in one commit
* push to your branch
* open a Pull Request

If your Pull Request is related to a Bugzilla ticket, it should have the following structure: "Bug 123456: this is the exact problem or fix"

## Release

Before a new OpenShift release happens make sure to pin the dependencies to the release branches:

1. In [kube-prometheus](https://github.com/coreos/kube-prometheus/) cut a release.
2. In this repo set the "version" in `jsonnet/jsonnetfile.json` to the release branches for all the dependencies.
