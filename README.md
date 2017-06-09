# Running Tsung in Kubernetes

This project demonstrate one possible way to run Tsung in Kubernetes using `StatefulSet`.

## About Tsung

[Tsung] is an open-source multi-protocol distributed load testing tool written in [Erlang].
With proper setup, Tsung could generate millions of virtual users accessing target endpoints.

Typically we run Tsung in baremetal machines or virtual machines. In order to launch Tsung
in Kubernetes, we have to figure out a way to assign hostnames to Tsung pods because Tsung
master have to connect to slaves using their hostnames.

## About StatefulSet

[StatefulSet] is a beta feature added to Kubernetes in 1.5. It is a controller used to
provide unique identity to its Pods. Together with a headless service, we could assign dns
name to each pods in the StatefulSet.

## Demo

Here is a quick demo showing the process to launch a load test using Tsung in Kubernetes.
You could modify `tsung-config.yaml` to test your own systems.

### Create Namespace

```console
$ kubectl create namespace tsung
```

### Launch test target

We use nginx as a demo target

```console
$ kubectl create -f target.yaml --namespace tsung
```

### Set Tsung config

We will inject Tsung config to master pod using `ConfigMap`. Modify the settings if you like.

```console
$ kubectl create -f tsung-config.yaml --namespace tsung
```

### Launch Tsung slave

```console
$ kubectl create -f tsung-slave.yaml --namespace tsung
```

### Launch Tsung master

Tsung master will begin the test as soon as the Pod boots up. When the test ended,
the master process will keep running so that user could access the test report using
Tsung web interface.

```console
$ kubectl create -f tsung-master.yaml --namespace tsung
```

### Access Tsung web interface

```console
$ kubectl port-forward tsung-master-0 -n tsung 8091:8091
```

Then we could access the web interface at `http://localhost:8091`

### Cleanup

```console
$ kubectl delete namespace tsung
```

[Tsung]: http://tsung.erlang-projects.org/
[Erlang]: https://www.erlang.org/
[StatefulSet]: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
