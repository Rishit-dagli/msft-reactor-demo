# Microsft Reactor Demo

## Demo 1

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

## Demo 2

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
