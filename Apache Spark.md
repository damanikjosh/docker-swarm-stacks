# Apache sparks with Python 3.10 API

The Docker Swarm stack YAML file provided represents a Spark cluster and a Jupyter notebook service. Here's a short documentation for this file:

**Spark Cluster Configuration:**

```yaml
version: '3.9'

x-spark:
  &spark-common
  image: apache/spark-py:v3.4.0
  user: root

  networks:
    - spark

services:
  master:
    <<: *spark-common
    hostname: master
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints: [node.role == manager]
    ports:
      - 8080:8080
      - 7077:7077
      - 6066:6066
    command: >
      bash -c "/opt/spark/sbin/start-master.sh -h master && tail -f /opt/spark/logs/*"

  worker:
    <<: *spark-common
    deploy:
      mode: global
    ports:
      - 8081:8081
    command: >
      bash -c "/opt/spark/sbin/start-worker.sh spark://master:7077 && tail -f /opt/spark/logs/*"


networks:
    spark:
      driver: overlay
      attachable: true

```

- The Spark cluster consists of a master node (`master`) and worker nodes (`worker`).
- The base image used for Spark components is `apache/spark-py:v3.4.0`, and the user is set to root.
- The services share a network named `spark`, which is an `overlay` network.
- The Spark master node (`master`) is deployed as a replicated service with one replica and is constrained to run on a node with the role of a manager.
- The Spark master node exposes ports `8080`, `7077`, and `6066`, which are mapped to the corresponding ports on the host.
- The Spark worker node (`worker`) is deployed as a `global` service, meaning there is one replica per available node.
- The Spark worker node exposes port `8081`, mapped to the host, and connects to the Spark master node at `spark://master:7077`.
- Both the Spark `master` and `worker` nodes execute a command that starts the respective Spark script and tails the log files.

**Jupyter Notebook Service:**

```yaml
services:
  notebook:
    image: jupyter/pyspark-notebook:python-3.10
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints: [node.role == manager]
    networks:
      - spark
    ports:
      - 8888:8888
      - 4040:4040
    volumes:
      - notebook:/home/jovyan/work
    environment:
      - JUPYTER_TOKEN=notebook

volumes:
  notebook:
```


- The Jupyter notebook service (`notebook`) uses the image `jupyter/pyspark-notebook:python-3.10`.
- It is deployed as a replicated service with one replica and constrained to run on a manager node.
- The Jupyter notebook service exposes ports `8888` and `4040`, which are mapped to the corresponding ports on the host.
- A named volume `notebook` is created and mounted at `/home/jovyan/work` inside the container, enabling persistent storage for notebook data.
- The environment variable `JUPYTER_TOKEN` is set to `notebook`, allowing authentication to the Jupyter notebook.

<sub>Description is generated using ChatGPT 💟</sub>
