# Microsft Reactor Demo

## Demo 1: Run a simple Rust app locally with Wasm

1. Install `wasmtime`

```sh
curl https://wasmtime.dev/install.sh -sSf | bash
```

2. Install rust with rustup

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

3. Install the `wasm32-wasi` target

```sh
rustup target add wasm32-wasi
```

4. Run the example

```sh
cd hello-world
cargo build --target wasm32-wasi --release
wasmtime target/wasm32-wasi/release/hello-world.wasm 
cd ..
```

## Demo 2: Run a Rust app locally

1. Install the `wasm32-wasi` target

```sh
rustup target add wasm32-wasi
```

2. Build the example

```sh
cd tensorflow-mobilenet-v2
cargo build --target wasm32-wasi --release
```

3. Run the example

```sh
wasmtime target/wasm32-wasi/release/example-tensorflow-mobilenet-v2.wasm --dir=.
```

4. AOT Compilation

Let's now do an inference but this time with AOT compilation. Notice closely the time differences!

```sh
wasmtime compile target/wasm32-wasi/release/example-tensorflow-mobilenet-v2.wasm
wasmtime example-tensorflow-mobilenet-v2.cwasm --allow-precompiled --dir=.
cd ..
```

## Demo 3: WebAssembly Text Format

1. Install `wabt`

```sh
wget https://github.com/WebAssembly/wabt/releases/download/1.0.29/wabt-1.0.29-ubuntu.tar.gz
tar zxf wabt-1.0.29-ubuntu.tar.gz
```

2. Convert Wasm to WAT

```sh
./wabt-1.0.29/bin/wasm2wat hello-world/target/wasm32-wasi/release/hello-world.wasm --output=hello-world.wat
```

## Demo 4: WASI Node pools on AKS

1. Register the `WasmNodePoolPreview` feature

```sh
az feature register --namespace "Microsoft.ContainerService" --name "WasmNodePoolPreview"
```

Running this command should show that the feature is registered:
    
```sh
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/WasmNodePoolPreview')].{Name:name,State:properties.state}"
```

Refresh the registration of the `ContainerService` resource:

```sh
az provider register --namespace Microsoft.ContainerService
```

2. Install the `aks-preview` Azure CLI extension

```sh
az extension add --name aks-preview
```

3. Create a new AKS cluster

```sh
az group create --name wasmRG -l westus
az aks create --resource-group wasmRG --name myAKScluster2
```

4. Add a WASM/WASI node pool to an existing AKS Cluster

```sh
az aks nodepool add \
    --resource-group wasmRG \
    --cluster-name myAKScluster2 \
    --name mywasipool \
    --node-count 1 \
    --workload-runtime wasmwasi
```

Verify the runtime:

```sh
az aks nodepool show -g reactor-demo --cluster-name myAKScluster2 -n mywasipool | jq '.workloadRuntime'
```

5. Configure `kubectl`

```sh
az aks get-credentials -n myAKScluster2 -g wasmRG
```

5. Get the internal IP of the WASI node

```sh
kubectl get nodes -o wide
```

6. Run a simple deployment

```sh
kubectl apply -f wasi-deployment/wasi-example.yaml
```

7. Create a reverse proxy

Update the `values.yaml` according to the internal IP of the load balancer which you can get with `kubectl get svc`.

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install hello-wasi bitnami/nginx -f wasi-depoyment/values.yaml
kubectl get svc
```

8. Try the deployment

```sh
curl EXTERNAL_IP/hello
```