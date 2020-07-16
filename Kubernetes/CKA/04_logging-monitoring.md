# Logging & Monitoring Progress: 9 / 9

## Section 4:76

### Monitor Cluster Components

Monitor resource consumption

#### node level metrics

-   nodes in clusters


-   how many are healthy
-   performance metrics (cpu, mem)

#### pods level metrics

-   pods on clusters
-   performance metrics (cpu, mem)

One metrics server per Kubernetes cluster (in-memory monitoring)

cAdvisor gathers metrics from pods and makes available

Download metrics server

    git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git

Install metrics server

    kubectl create -f .

Check node metrics

    kubectl top node

Check pod metrics

    kubectl top pods

## Section 4:79

### Managing Application Logs

Stream logs for pod

    kubectl logs -f event-simulator-pod
