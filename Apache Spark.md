# Apache sparks with Python 3.10 API

Spark stacks
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
If you need pyspark notebook, add this service block
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
