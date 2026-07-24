# Spark Connect on Kubeflow Notebooks

Run interactive Spark jobs from Kubeflow Notebooks — in **Python or Scala** — against a
Spark Connect server that spawns executor pods on demand via native Kubernetes scheduling.

Executors scale up when work queues and terminate when idle. On EKS with Cluster
Autoscaler, nodes follow the same curve underneath.

```
Kubeflow Notebook  ──gRPC──>  Spark Connect Server  ──k8s API──>  Executor Pods
   (PySpark / Scala)              (long-running)                   (ephemeral)
```

---

## Contents

| Path | What it is |
|---|---|
| `spark-connect-service.yaml` | Spark Connect server Deployment + ClusterIP Service |
| `spark-scala-docker-image/Dockerfile` | Kubeflow notebook image with Almond (Scala 2.13) kernel |
| `sampleCode/pyspark.ipynb` | PySpark client example |
| `sampleCode/scala.ipynb` | Scala client example |

---

## Prerequisites

- Docker Desktop (Apple Silicon supported — images build for `linux/amd64`)
- `kind`, `kubectl`, `kustomize`, `helm`
- A Kubeflow install with Notebooks,
