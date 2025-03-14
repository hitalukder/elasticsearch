# Automating Elasticsearch Vector Database Deployment on OpenShift with Helm

## Overview

If you're building an AI-powered search, RAG application, or recommendation system, vector similarity search is essential. Elasticsearch is a powerful open-source vector database that combines traditional text and vector search, improving accuracy and relevance. Setting up a production-ready Elasticsearch cluster is complex, requiring the configuration of multiple components and making installation challenging.

That's where Helm comes in. This guide simplifies the process using Helm, a package manager for Kubernetes that can be used to automate Elasticsearch deployment. We'll walk through provisioning an Elasticsearch vector database using Helm on an OpenShift cluster.

## Prerequisites

Before you begin, ensure you have:

- An OpenShift Cluster with admin access
- OpenShift CLI (`oc`) installed ([Download Here](https://access.redhat.com/downloads/content/290))
- Helm installed on your local machine ([Installation Guide](https://helm.sh/docs/intro/install/))

## Cluster Sizing Guide

### How Many Shards Do You Really Need?

Number of Shards = (Source Data + Room to Grow) * (1 + Indexing Overhead) / Desired Shard Size

Example:
For 150GB of data expected to double (100% growth):

```sh
(150GB + 150GB) x 1.10 / 25GB = 14 shards (rounded up)
```

### How Much Storage Do You Need?

```sh
Your Data x (1 + Number of Replicas) x 1.45
```

Example:

```sh
200 GB x (1 + 1) x 1.45 = 580 GB (approximately)
```

### Sample Cluster Configuration

Let's build a minimum setup with 4 data nodes with 1 replica per shard, 3 master nodes and 4 ML nodes for a high-performance vector database indexing and retrieval for a total of 580 GB document size.

#### Data Nodes

- **Total Storage**: 580 GB
- **Compute & Memory**: 12 vCPU and 46 GB of memory
- **Each Node**: 3 CPUs, 12 GB memory, 150 GB storage
- **JVM Heap**: 6 GB (50% of memory)

#### Master Nodes

- **Each Node**: 2-4 CPUs, 8GB memory, 50–100 GB storage
- **JVM Heap**: 4GB (50% of memory)

#### ML Nodes (for embeddings, if applicable)

- **4 ML Nodes**: 16 CPUs, 32GB RAM per node
- **Total Storage**: 50GB–100GB
- **Simultaneous embeddings**: 64 (16 threads per allocation * 4 nodes)

## Configuration

The following configurations are available in the [`values.yaml`](values.yaml) file and modify the values file based on your use case. A production like configuration also added in the repo. [`prod-values.yaml`](prod-values.yaml)

### Cluster Configuration

- **clusterName**: Defines the name of the Elasticsearch cluster.
- **elasticsearchVersion**: Specifies the version of Elasticsearch to deploy.
- **storageClassName**: The storage class to use for all nodes in the cluster.

### Node Configurations

The Helm chart allows configuring different types of Elasticsearch nodes:

#### Master Node

- **replica_count**: Number of master nodes.
- **cpu_limit / cpu_request**: CPU resource allocation.
- **memory_limit / memory_request**: Memory resource allocation.
- **storage_size**: Storage allocation.
- **java_options**: JVM settings for Elasticsearch.

#### Data Node

- **replica_count**: Number of data nodes.
- **cpu_limit / cpu_request**: CPU resource allocation.
- **memory_limit / memory_request**: Memory resource allocation.
- **storage_size**: Storage allocation.
- **java_options**: JVM settings for Elasticsearch.

#### Ingest Node

- **required**: Set to `true` to enable ingest nodes.
- **replica_count**: Number of ingest nodes.
- **cpu_limit / cpu_request**: CPU resource allocation.
- **memory_limit / memory_request**: Memory resource allocation.
- **storage_size**: Storage allocation.
- **java_options**: JVM settings for Elasticsearch.

#### Machine Learning (ML) Node

- **required**: Set to `true` to enable ML nodes.
- **replica_count**: Number of ML nodes.
- **cpu_limit / cpu_request**: CPU resource allocation.
- **memory_limit / memory_request**: Memory resource allocation.
- **storage_size**: Storage allocation.
- **java_options**: JVM settings for Elasticsearch.

### Custom TLS Certificate

For securing access to the Elasticsearch instance via OpenShift routes:

- **required**: Set to `true` to use a custom TLS certificate.
- **crt**: Base64-encoded certificate.
- **key**: Base64-encoded private key.

## Deployment

### Clone the Helm Chart Repository

```sh
git clone https://github.com/your-repo/elasticsearch-helm.git
cd elas
```

### Modify Values File

Open the repository in a code editor and update `values.yaml` based on your use case. Sample values files:

- **Basic version**: `values.yaml`
- **Production-like version**: `prod-values.yaml`

### Deploy Elasticsearch

#### Basic Deployment

```sh
helm install vector-db .
```

#### Production-Like Deployment

```sh
helm install vector-db -f prod-values.yaml
```

#### Updating Values

```sh
helm upgrade vector-db .
```

### Verify Deployment

```sh
oc get Elasticsearch
```

If successful, the health status should be **green** and phase **Ready**.

### To uninstall

```sh
helm uninstall vector-db
```

## Accessing the Elasticsearch Cluster

### Set Cluster Name

```sh
export CLUSTER_NAME=<your cluster name>
```

### Retrieve Credentials

```sh
echo $(oc get secret ${CLUSTER_NAME}-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')
```

### Find URLs for Kibana and Elasticsearch

```sh
oc get route
export ES_USER=elastic
export ES_PASSWORD=$(oc get secret ${CLUSTER_NAME}-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')
export ES_URL=$(oc get routes route-${CLUSTER_NAME} -o jsonpath='https://{.spec.host}')
```

### Verify Elasticsearch Installation

```sh
curl -u "${ES_USER}:${ES_PASSWORD}" -k "${ES_URL}"
```

Expected Output:

```json
{
  "name" : "es-sample-es-ml-0",
  "cluster_name" : "es-sample",
  "cluster_uuid" : "CbcJtY3URqqWf4N_V0ykHQ",
  "version" : {
    "number" : "8.12.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "lucene_version" : "9.9.1",
  }
}
```

## Conclusion

This guide provides a comprehensive approach to deploying Elasticsearch as a vector database on OpenShift using Helm. Proper planning, sizing, and monitoring will ensure a scalable and efficient AI-powered search system.

## Additional Resources

- [Elasticsearch Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- [Kubernetes ReplicaSet Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)

## License

This project is licensed under the MIT License.

