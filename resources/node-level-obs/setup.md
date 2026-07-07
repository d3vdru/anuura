# Node-Level Observability Setup

This layer collects GPU metrics from each node with NVIDIA DCGM Exporter and stores them in Prometheus. Grafana can then be used to view dashboards backed by the Prometheus data source.

## 1. Start DCGM Exporter on each GPU node

Run one DCGM Exporter instance on every node that should report GPU metrics. The exporter listens on port `9400`, so make sure that port is reachable from the node where Prometheus will run.

Replace `node_name` with the node's hostname or another stable name that should appear in the exported metrics.

### Docker nodes

Use Docker on nodes where Docker is available:

```bash
docker run -d \
  --gpus all \
  --cap-add SYS_ADMIN \
  --rm \
  -p 9400:9400 \
  -e DCGM_HOSTNAME="node_name" \
  nvcr.io/nvidia/k8s/dcgm-exporter:4.5.3-4.8.2-distroless
```

To use the system hostname automatically:

```bash
docker run -d \
  --gpus all \
  --cap-add SYS_ADMIN \
  --rm \
  -p 9400:9400 \
  nvcr.io/nvidia/k8s/dcgm-exporter:4.5.3-4.8.2-distroless
```

### Turning nodes with Singularity

Use Singularity on Turning nodes:

```bash
SINGULARITYENV_DCGM_HOSTNAME="node_name" \
singularity run --nv \
  docker://nvcr.io/nvidia/k8s/dcgm-exporter:4.5.3-4.8.2-distroless
```

To keep the exporter running in the background and use the system hostname automatically:

```bash
singularity run --nv \
  docker://nvcr.io/nvidia/k8s/dcgm-exporter:4.5.3-4.8.2-distroless 
```

Singularity uses the host network by default, so the exporter should be available on port `9400` of the node where it is started.

## 2. Configure Prometheus targets

On the node where the observability dashboard will run, edit `prometheus.yml` and list every node running DCGM Exporter under the `dcgm-exporter` scrape job.

Example:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'dcgm-exporter'
    static_configs:
      - targets:
          - 'node1_ip:9400'
          - 'node2_ip:9400'
          - 'node3_ip:9400'
        labels:
          cluster: 'gpu-cluster'
```

Use IP addresses or DNS names that are reachable from the Prometheus host.

## 3. Launch Prometheus and Grafana

From this directory, start the observability services:

```bash
docker compose up -d
```

This launches:

- Prometheus with `prometheus.yml` mounted as its scrape configuration.
- Grafana with persistent storage in the `grafana_data` Docker volume.

The compose file uses host networking. After the services start, Prometheus should be available on port `9090` and Grafana should be available on port `3000` on the dashboard node.

## 4. Verify collection

Check that DCGM Exporter is responding on each node:

```bash
curl http://node1_ip:9400/metrics
```

Check that Prometheus has loaded the targets:

1. Open Prometheus on `http://dashboard_node:9090`.
2. Go to **Status > Targets**.
3. Confirm that each `dcgm-exporter` target is `UP`.

In Grafana, add Prometheus as a data source using:

```text
http://localhost:9090
```

Then import or create dashboards using the DCGM metrics exposed by the exporters.

NVIDIA provides an official DCGM Exporter Grafana dashboard with ID `12239`. After the Prometheus instance is added as a Grafana data source, import the dashboard from:

```text
https://grafana.com/grafana/dashboards/12239-nvidia-dcgm-exporter-dashboard/
```

## 5. Remaining work

- Dump metric logs from Prometheus into the NAS server for longer-term storage and analysis.
- Future work can provide deeper integration with job-level observability through Weights & Biases, following the DCGM Exporter and W&B workflow described at:

```text
https://wandb.ai/dimaduev/dcgm/reports/Monitoring-GPU-cluster-performance-with-NVIDIA-DCGM-Exporter-and-Weights-Biases--Vmlldzo0MDYxMTA1
```
