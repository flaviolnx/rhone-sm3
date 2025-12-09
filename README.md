<H1>----------------ìnicio Instalação Istio------------------------</H1>

1 - Instalar Operator Service Mesh 3.0

2 - Criar Projeto istio-system

3 - Criar Projeto istio-cni

3 - Criat Istio CRD com o seguinte spec:

<code>
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
</code>


4 - criar IstioCNI no projeto istio-cni

5 - criar namespace bookinfo

    oc new-project bookinfo

6 - adicionar labels ao namespace bookinfo 

    oc label namespace bookinfo istio-discovery=enabled istio-injection=enabled

7 - fazer o deploy da aplicação bookinfo 
 
    oc apply -f https://raw.githubusercontent.com/openshift-service-mesh/istio/release-1.24/samples/bookinfo/platform/kube/bookinfo.yaml -n bookinfo

8 - criar o gateway 
  
    oc apply -n bookinfo -f https://raw.githubusercontent.com/istio-ecosystem/sail-operator/main/chart/samples/ingress-gateway.yaml

9 - configurar app bookinfo para usar o gateway 
 
    oc apply -f https://raw.githubusercontent.com/openshift-service-mesh/istio/release-1.24/samples/bookinfo/networking/bookinfo-gateway.yaml -n bookinfo

10 - expor o serviço do gateway 

    oc expose svc istio-ingressgateway -n bookinfo
  
 <H1>----------------Final Instalação Istio------------------------</H1>

 
<H1> ----------------Inicio Observabilidade------------------------</H1>
 
1 - Habilitar Monitoring, core e user workload 

https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html/monitoring/configuring-core-platform-monitoring

2 - Criar service monitor e pod monitor para o namespace do control plane e todos os namespaces das apps que estarão nas mesh - 

    oc apply -f 01-control-plane-service-monitor.yaml -n istio-system && oc apply -f 02-control-plane-pod-monitor.yaml -n istio-system

3 - Instalar o Operador do GrafanaTempo

4 - Instalar o Operador do OpenTelemetry

5 - Criar o secret tempo-secret - 

    oc apply -f 03-tempo-s3-secret.yaml -n tempo

6 - Criar Service Account otel-colector-deployment no namespace istio-system 

    oc create sa otel-collector-deployment

7 - aplicar cluster role e clusterrolebinding para a service account 

    oc apply -f 04-otel-collector-cluster-role.yaml && oc apply -f 05-otel-collector-cluster-role-binding.yaml -n istio-system

8 - Instalar o tempo stack 

    oc apply -f 06-tempo-stack.yaml

9 - Instalar OpenTelemetry Collector 
  
    oc apply -f 07-otel-istio.yaml

10 - configurar control plane (istio resource) para enviar traces para o collector

<code>
  values:
    meshConfig:
      enableTracing: true
      extensionProviders:
      - name: otel
        opentelemetry:
          port: 4317
          service: otel-collector.istio-system.svc.cluster.local
</code>          

11 - criar objeto telemetry no namespace do control plane 
    
    oc apply -f 08-telemetry.yaml

12 - Instalar cluster observability operator

13 - Criar distributed-tracing ui plugin

14 - Criar service monitor/podmonitor no namespace da aplicação(criar isso no namespace de todas as aplicações que fizerem parte da mesh, sem isso o Kiali não gera o gráfico de tráfego) 

15 - gerar carga para a aplicação book info

16 - verificar os traces na url interna do openshift

17 - Instalar operator do kiali

18 - configurar cluster role binding de cluster-monitoring-view para o kiali

    oc apply -f 09-kiali-cluster-role-binding.yaml

19 - Instalar o kiali - 

    oc apply -f 10-kiali.yaml

20 - gerar tráfego para aplicação

21 - observar no kiali
