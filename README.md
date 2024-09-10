# Flux Manager

Flux Manager is a Go application designed to automate YAML distribution and kubeconfig management across multiple Kubernetes clusters. It replaces existing bash scripts to offer improved flexibility, scalability, and maintainability.

## Key Features

- **YAML Distribution:** Distributes static and templated YAML files to specified cluster folders.
- **Kubeconfig Management:** Manages kubeconfig files, including webhook key insertion and context name standardization.
- **Cluster Context Management:** Standardizes context names and facilitates context switching for per-cluster updates.
- **Data Handling:** Reads cluster details from a `clusters.csv` file for streamlined operations.
- **Script Conversion:** Converts existing bash script functionality to Go, including Flux installation and upgrades.
- **Error Handling and Logging:** Implements robust error handling and detailed logging for all operations.
- **Performance Optimization:** Optimized for efficient performance under varying loads.
- **Testing and Validation:** Comprehensive unit and integration tests ensure functionality correctness.

## Requirements

- Go 1.16 or later
- Kubernetes clusters
- `clusters.csv` file with cluster details

## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/your-repo/flux-manager.git
   ```

2. Navigate to the project directory:
   ```bash
   cd flux-manager
   ```

3. Build the application:
   ```bash
   go build -o flux-manager
   ```

## Usage

1. Ensure your `clusters.csv` file is correctly formatted with cluster names and additional details.

2. Run the application:
   ```bash
   ./flux-manager
   ```

3. Follow the prompts to distribute YAML files and manage kubeconfig files.

## Configuration

- YAML files and kubeconfig paths can be configured in the `config.yaml` file.
- Customize context names and other settings as needed.

## Testing

Run unit tests to ensure functionality:
```bash
go test ./...
```

## Documentation

Detailed documentation is available in the `docs` folder, including setup instructions, configuration options, and usage examples.

## Cluster Configuration

Example of a cluster configuration file (`backend-services-flux-blueprint-cluster01.yaml`):

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tL
    server: https://k-2bvk8irdtbvf4b2ko92tttfmd72i41du-kapi.ssnc-corp.cloud:6443
  name: k-2bvk8irdtbvf4b2ko92tttfmd72i41du
contexts:
- context:
    cluster: k-2bvk8irdtbvf4b2ko92tttfmd72i41du
    user: webhook
  name: webhook@k-2bvk8irdtbvf4b2ko92tttfmd72i41du
current-context: webhook@k-2bvk8irdtbvf4b2ko92tttfmd72i41du
kind: Config
preferences: {}
users:
- name: webhook
  user:
    token: ${SSCCLOUD_APIKEY}
```

Replace `${SSCCLOUD_APIKEY}` with your actual token for authentication.

## Bootstrap Script

The `bootstrap_flux.sh` script is used to bootstrap Flux for multiple clusters.

```bash
#!/usr/bin/env bash

declare -A clusters
clusters["cluster1"]="$KUBECONFIG_DIR/backend-services-flux-blueprint-cluster01.yaml ./clusters/backend-services-flux-blueprint-cluster01"
clusters["cluster2"]="$KUBECONFIG_DIR/backend-services-flux-blueprint-cluster02.yaml ./clusters/backend-services-flux-blueprint-cluster02"
clusters["cluster3"]="$KUBECONFIG_DIR/backend-services-flux-blueprint-cluster03.yaml ./clusters/backend-services-flux-blueprint-cluster03"

bootstrap_flux() {
    local cluster_name=$1
    local kubeconfig_path=$2
    local flux_path=$3

    export KUBECONFIG=$kubeconfig_path

    echo "Using kubeconfig: $KUBECONFIG"
    echo "Bootstrapping Flux for $cluster_name at path $flux_path..."

    kubectl config use-context $cluster_name

    flux bootstrap github \
        --owner=cloud \
        --repository=backend-services-flux-blueprint-deploy \
        --branch=main \
        --path=$flux_path \
        --hostname=code.ssnc.dev \
        --token-auth \
        --namespace=flux-system

    unset KUBECONFIG
}

for cluster in "${!clusters[@]}"; do
    read kubeconfig_path flux_path <<< "${clusters[$cluster]}"
    bootstrap_flux "$cluster" "$kubeconfig_path" "$flux_path"
done
```

By using Flux Manager, you can enhance your automation capabilities for managing Kubernetes clusters, ensuring more reliable and scalable operations.