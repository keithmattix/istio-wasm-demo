# Istio WASM Demo

## Prerequisites

1. [Install k3d](https://k3d.io/v5.6.0/#installation)
2. [Install istioctl](https://istio.io/latest/docs/setup/install/istioctl/)
3. [Instal Helm](https://helm.sh/docs/intro/install/)
4. [Enable WASM Workloads in Docker Desktop](https://docs.docker.com/desktop/wasm/) (if applicable)

## Demo

1. `k3d cluster create wasm-cluster --image ghcr.io/deislabs/containerd-wasm-shims/examples/k3d:v0.9.0 -p "8081:80@loadbalancer" --agents 2`
2. `kubectl apply -f https://github.com/deislabs/containerd-wasm-shims/raw/main/deployments/workloads/runtime.yaml`
3. `kubectl apply -f https://github.com/deislabs/containerd-wasm-shims/raw/main/deployments/workloads/workload.yaml`
4. Wait ~15 seconds for resources to be Ready
5. `curl -v http://127.0.0.1:8081/spin/hello`
6. Install Istio: `istioctl install --set profile=default --skip-confirmation`
7. Install Prometheus with: `kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.19/samples/addons/prometheus.yaml`
   1. *Note: this is a demo prometheus install, not for production use*
8. Install Kiali: `helm install --namespace istio-system --set auth.strategy="anonymous" --repo https://kiali.org/helm-charts kiali-server kiali-server`

9. Label the default namespace with `istio-injection=enabled`: `kubectl label namespace default istio-injection=enabled`
10. Deploy WASM application YAML in this repo: `kubectl apply -f ./product-api.yaml`
    1.  Notice that there are 2 Ready "containers" within the product-api pod. One of them is the Istio sidecar!
11. Deploy the Istio `sleep` sample application: `kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.19/samples/sleep/sleep.yaml`
12. Start a loop to curl the product-api service from the sleep pod:

```shell
while true
do
kubectl exec -it deploy/sleep -- curl http://product-api:5001/v1-get-item-types
sleep 1
done
```
13. In a separate terminal, port-forward the kiali service: `kubectl port-forward svc/kiali 20001:20001 -n istio-system`
14. View the kiali graph (make sure to stretch out the range to >1min so the graph doesn't reset)

