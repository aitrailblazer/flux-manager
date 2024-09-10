# README for Flux Manager

## Overview

Flux Manager is a Go application designed to automate YAML distribution and kubeconfig management across multiple Kubernetes clusters. It replaces existing bash scripts, offering improved flexibility, scalability, and maintainability.

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

## Bootstrap Script

The `bootstrap_flux.sh` script is used to bootstrap Flux for multiple clusters.

```bash
#!/usr/bin/env bash

# Define an associative array for clusters, their specific kubeconfig, and Flux paths
declare -A clusters
clusters["cluster1"]="$KUBECONFIG_DIR/backend-services-flux-blueprint-cluster01.yaml ./clusters/backend-services-flux-blueprint-cluster01"
clusters["cluster2"]="$KUBECONFIG_DIR/backend-services-flux-blueprint-cluster02.yaml ./clusters/backend-services-flux-blueprint-cluster02"
clusters["cluster3"]="$KUBECONFIG_DIR/backend-services-flux-blueprint-cluster03.yaml ./clusters/backend-services-flux-blueprint-cluster03"

# Function to bootstrap Flux
bootstrap_flux() {
    local cluster_name=$1
    local kubeconfig_path=$2
    local flux_path=$3

    # Set KUBECONFIG to the specific file
    export KUBECONFIG=$kubeconfig_path

    echo "Using kubeconfig: $KUBECONFIG"
    echo "Bootstrapping Flux for $cluster_name at path $flux_path..."

    # Switch context to the cluster
    kubectl config use-context $cluster_name

    # Flux bootstrap command tailored for each cluster
    flux bootstrap github \
        --owner=cloud \
        --repository=backend-services-flux-blueprint-deploy \
        --branch=main \
        --path=$flux_path \
        --hostname=code.ssnc.dev \
        --token-auth \
        --namespace=flux-system

    # Reset KUBECONFIG if needed, or unset to use default
    unset KUBECONFIG
}

# Loop through each cluster and bootstrap Flux
for cluster in "${!clusters[@]}"; do
    read kubeconfig_path flux_path <<< "${clusters[$cluster]}"
    bootstrap_flux "$cluster" "$kubeconfig_path" "$flux_path"
done
```

## Cluster Configuration File: `clusters.csv`

The `clusters.csv` file contains details about each cluster:

```csv
name,update,environment
uat-sre-kc,true,UAT
uat-sre-stl,false,UAT
managed-k8s-capi-uat-kc,false,UAT
managed-k8s-capi-uat-stl,false,UAT
oculus,false,oculus
cloud-services-bootstrap-bar,false,PROD
cloud-services-bootstrap-cam,false,PROD
cloud-services-capv-stl,false,PROD
cloud-services-capv-wc,false,PROD
cloud-services-lon,false,PROD
cloud-services-vie,false,PROD
cloud-services-wal,false,PROD
cloud-services-ykt,false,PROD
managed-k8s-capi-bootstrap-bar,false,PROD
managed-k8s-capi-bootstrap-cam,false,PROD
managed-k8s-capi-prod-cam,false,PROD
managed-k8s-capi-prod-kc,false,PROD
managed-k8s-capi-prod-lon,false,PROD
managed-k8s-capi-prod-stl,false,PROD
managed-k8s-capi-prod-vie,false,PROD
managed-k8s-capi-prod-wales,false,PROD
managed-k8s-capi-prod-ykt,false,PROD
```

## Bootstrap Script: `flux-bootstrap-csv.sh`

The `flux-bootstrap-csv.sh` script automates the process of switching contexts and bootstrapping Flux based on the entries in `clusters.csv`.

```bash
#!/bin/bash

# Path to the CSV file
CSV_FILE="clusters.csv"
TEMP_FILE="temp.csv"

# Create a temporary file to store updated CSV content
touch $TEMP_FILE

# Read the CSV file line by line
while IFS=, read -r name execute rest_of_line; do
    
    # Skip the header row if it exists 
    if [[ "$name" == "name" ]]; then 
        echo "$name,$execute,$rest_of_line" >> $TEMP_FILE 
        continue 
    fi 

    if [[ "$execute" == "true" ]]; then 
        # Switch cluster context 
        echo "Switching to cluster $name" 
        kubectl config use-context $name 
        
        # Run the flux bootstrap command with the name from the CSV 
        echo "$GITHUB_TOKEN" | flux bootstrap git --url=https://code.ssnc.dev/cloud/backend-services-flux-deploy --branch=main --path=clusters/"$name" --token-auth 

        # Set execute to false after running the command 
        execute="false" 
    fi 

    # Write the line to the temporary file 
    echo "$name,$execute,$rest_of_line" >> $TEMP_FILE 

done < "$CSV_FILE"

# Replace the original CSV file with the updated content 
mv $TEMP_FILE $CSV_FILE 
echo "**NOTE** Your cluster context has probably changed!"
```

By using Flux Manager along with these scripts and configurations, you can enhance your automation capabilities for managing Kubernetes clusters efficiently.