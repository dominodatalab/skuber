# Fork note

This is a fork of doriordan/skuber maintained by Domino Data Lab. The default branch here is ddl-master that includes
some changes for our own uses of the library. E.g. plugins to support publishing to our internal repo while we wait on
the upstream to issue new releases. We will eagerly merge our feature branches into this branch while they are pending
review at the upstream project.

# Skuber

Skuber is a Scala client library for [Kubernetes](http://kubernetes.io). It provides a fully featured, high-level and strongly typed Scala API for managing Kubernetes cluster resources (such as Pods, Services, Deployments, ReplicaSets, Ingresses  etc.) via the Kubernetes REST API server

## Prerequisites

The client supports v1.0 through v1.3 of the Kubernetes API, so should work against all supported Kubernetes releases in use today.

You need Java 8 to run Skuber.

## Release

You can include the current Skuber release in your application by adding the following to your `sbt` project:

    resolvers += Resolver.url(
      "bintray-skuber",
      url("http://dl.bintray.com/oriordan/skuber"))(
      Resolver.ivyStylePatterns)

    libraryDependencies += "io.doriordan" %% "skuber" % "1.3.0"

The current release has been built for Scala 2.11 - other Scala language versions such as 2.12 will be supported in future.

## Building

Building the library from source is very straightforward. Simply run `sbt test`in the root directory of the project to build the library (and examples) and run the unit tests to verify the build.

## Quick Start

The quickest way to get started with Skuber:

- If you don't already have Kubernetes installed, then follow the instructions [here](https://github.com/kubernetes/minikube) to install minikube, which is now the recommended way to run Kubernetes locally.

- Ensure Skuber configures itself from the default Kubeconfig file (`$HOME/.kube/config`) : 

	`export SKUBER_CONFIG=file` 

- Try one or more of the examples: if you have cloned this repository run `sbt` in the top-level directory to start sbt in interactive mode and then:

```
    > project examples

    > run
    [warn] Multiple main classes detected.  Run 'show discoveredMainClasses' to see the list

    Multiple main classes detected, select one to run:

    [1] skuber.examples.deployment.DeploymentExamples
    [2] skuber.examples.fluent.FluentExamples
    [3] skuber.examples.guestbook.Guestbook
    [4] skuber.examples.scale.ScaleExamples
    [5] skuber.examples.ingress.NginxIngress

    Enter number: 
```

For other Kubernetes setups, see the [Configuration guide](docs/Configuration.md) for details on how to tailor the configuration for your clusters security, namespace and connectivity requirements.

## Example Usage

The code block below illustrates the simple steps required to create a replicated nginx service on a Kubernetes cluster that can be accessed by clients (inside or outside the cluster) at a stable address.

It creates a [Replication Controller](http://kubernetes.io/docs/user-guide/replication-controller/) that ensures five replicas ([pods](http://kubernetes.io/docs/user-guide/pods/)) of an nginx container are always running in the cluster, and exposes these to clients via a [Service](http://kubernetes.io/docs/user-guide/services/) that automatically proxies any request received on port 30001 on any node of the cluster to port 80 of one of the currently running nginx replicas.

    import skuber._
    import skuber.json.format._

    val nginxSelector  = Map("app" -> "nginx")
    val nginxContainer = Container("nginx",image="nginx").exposePort(80)
    val nginxController= ReplicationController("nginx",nginxContainer,nginxSelector)
    	.withReplicas(5)
    val nginxService = Service("nginx")
    	.withSelector(nginxSelector)
    	.exposeOnNodePort(30001 -> 80) 

    import scala.concurrent.ExecutionContext.Implicits.global

    val k8s = k8sInit

    val createOnK8s = for {
      svc <- k8s create nginxService
      rc  <- k8s create nginxController
    } yield (rc,svc)

    createOnK8s onComplete {
      case Success(_) => System.out.println("Successfully created nginx replication controller & service on Kubernetes cluster")
      case Failure(ex) => System.err.println("Encountered exception trying to create resources on Kubernetes cluster: " + ex)
    }

    k8s.close

## Features

- Comprehensive Scala case class representations of Kubernetes kinds (e.g. `Pod`, `Service` etc.)
- Reading and writing Kubernetes resources in the standardised Kubernetes JSON format
- The API has `create`, `get`, `delete`, `list`, `update`, and `watch` operations on Kubernetes resources
  - These are asynchronous operations (i.e. the API operations return their results in [Futures](http://docs.scala-lang.org/overviews/core/futures.html))
  - Each operations results in a HTTP request to the Kubernetes REST API server
  - Strongly-typed API e.g. `k8s get[Deployment]("myDepl")` returns a value of type `Future[Deployment]`
- Support for the full `core` Kubernetes API group 
- Support for the `extensions` API group (e.g. `HorizontalPodAutoScaler`,`Ingress`) 
- Fluent API helps build or modify the desired specification (`spec`) of a Kubernetes resource
- Watching Kubernetes objects and kinds returns `Iteratees` for reactive processing of events from the cluster
- Highly [configurable](docs/Configuration.md) via kubeconfig files or programmatically
- Full support for [label selectors](http://kubernetes.io/docs/user-guide/labels)
	- includes a mini-DSL for building expressions from both set-based and equality-based requirements.

See the [programming guide](docs/GUIDE.md) for more details.

## Status

The coverage of the core Kubernetes API functionality by Skuber is extensive.

Support of more recent extensions group functionality is not yet entirely complete:  full support (with examples) is included for [Deployments](http://kubernetes.io/docs/user-guide/deployments/), [Horizontal pod autoscaling](http://kubernetes.io/docs/user-guide/horizontal-pod-autoscaling/) and [Ingress / HTTP load balancing](http://kubernetes.io/docs/user-guide/ingress/); support for other [Extensions API group](http://kubernetes.io/docs/api/#api-groups) features including [Daemon Sets](http://kubernetes.io/docs/admin/daemons/) and [Jobs](http://kubernetes.io/docs/user-guide/jobs/) is added over time.

## License

This code is licensed under the Apache V2.0 license, a copy of which is included [here](LICENSE.txt).
