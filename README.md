# Start Istio Installation

1 - Install Service Mesh Operator 3.0

2 - Create Project istio-system

3 - Create Project istio-cni

4 - Create Istio CRD with the following spec:

```yaml
spec:
  namespace: istio-system
  updateStrategy:
    inactiveRevisionDeletionGracePeriodSeconds: 30
    type: InPlace
  values:
    meshConfig:
      discoverySelectors:
        - matchLabels:
            istio-discovery: enabled
  version: v1.24.5
```

5 - Create IstioCNI in the istio-cni project

6 - Create namespace bookinfo oc new-project bookinfo

7 - Add labels to the bookinfo namespace

`oc label namespace bookinfo istio-discovery=enabled istio-injection=enabled`

8 - Deploy the bookinfo application

` oc apply -f https://raw.githubusercontent.com/openshift-service-mesh/istio/release-1.24/samples/bookinfo/platform/kube/bookinfo.yaml -n bookinfo`

9 - Create the gateway

`oc apply -n bookinfo -f https://raw.githubusercontent.com/istio-ecosystem/sail-operator/main/chart/samples/ingress-gateway.yaml`

10 - Configure bookinfo app to use the gateway

`oc apply -f https://raw.githubusercontent.com/openshift-service-mesh/istio/release-1.24/samples/bookinfo/networking/bookinfo-gateway.yaml -n bookinfo`

11 - Expose the gateway service 

`oc expose svc istio-ingressgateway -n bookinfo`

# End Istio Installation

Start Observability

1 - Enable Monitoring, core and user workload [https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html/monitoring/configuring-core-platform-monitoring]()

2 - Create service monitor and pod monitor for the control plane namespace and all app namespaces that will be in the mesh

`oc apply -f 01-control-plane-service-monitor.yaml -n istio-system && oc apply -f 02-control-plane-pod-monitor.yaml -n istio-system`

3 - Install the Grafana Tempo Operator

4 - Install the OpenTelemetry Operator

5 - Create the tempo-secret secret

`oc apply -f 03-tempo-s3-secret.yaml -n tempo`

6 - Create Service Account otel-collector-deployment in the istio-system namespace

 `oc create sa otel-collector-deploymen`t

7 - Apply cluster role and clusterrolebinding for the service account

`oc apply -f 04-otel-collector-cluster-role.yaml && oc apply -f 05-otel-collector-cluster-role-binding.yaml -n istio-system`

8 - Install the tempo stack

`oc apply -f 06-tempo-stack.yaml`

9 - Install OpenTelemetry Collector

`oc apply -f 07-otel-istio.yaml`

10 - Configure control plane (Istio resource) to send traces to the collector

```yaml
  values:
    meshConfig:
      enableTracing: true
      extensionProviders:
      - name: otel
        opentelemetry:
          port: 4317
          service: otel-collector.istio-system.svc.cluster.local
```

11 - Create telemetry object in the control plane namespace

`oc apply -f 08-telemetry.yaml`

12 - Install cluster observability operator

13 - Create distributed-tracing ui plugin

14 - Create service monitor/pod monitor in the application namespace (create this in the namespace of all applications that are part of the mesh, without it, Kiali will not generate the traffic graph)

15 - Generate load for the book info application

16 - Check the traces at the openshift internal URL

17 - Install Kiali operator

18 - Configure cluster role binding of cluster-monitoring-view for kiali

`oc apply -f 09-kiali-cluster-role-binding.yaml`

19 - Install Kiali

oc apply -f 10-kiali.yaml

20 - Generate traffic for the application

21 - Observe in Kiali
