Project: spark-hudi — Copilot instructions for AI coding agents

- Goal: help contributors run and develop a local Spark + Hudi environment (MinIO-backed S3A) and iterate on PySpark/Scala jobs and notebooks.

Quick architecture summary
- Services: MinIO (S3 compatible object store), Spark master, multiple Spark workers, Spark history server, and a Jupyter container. Defined in `docker-compose.yml`.
- Spark image: built from `spark/Dockerfile` which downloads Spark 3.5.2, installs Hadoop AWS and Hudi jars into `/home/spark/jars`, configures event log dir and history server, and provides `entrypoint.sh` to start jupyter/shell/pyspark.
- App code directory: host `spark/app` is mounted into containers at `/home/spark/app`. Event logs are stored in `spark/event_logs` and mounted into containers.

How to run locally (developer workflow)
- Start all services: in repository root run `docker-compose up --build` (compose file at repo root).
- Useful service targets: `jupyter` container runs Jupyter Notebook on host port 8888; `spark-master` exposes 7077 (master) and 8080 (master UI); history server on 18080; MinIO console on 9001.
- To open an interactive shell inside the image: the Dockerfile's entrypoint supports commands: `jupyter`, `spark-shell`, `pyspark`, `bashed`.

Project-specific conventions and patterns
- Logging/history: Spark event logs are persisted to `spark/event_logs` (configured by `spark/Dockerfile` and `conf/core-site.xml`). When adding jobs, prefer enabling event logging to make them visible in the history server.
- Jars: Additional jars required by Hudi, Avro, and Hadoop AWS are downloaded into `spark/jars` at image build time. When adding libraries, either add `curl` lines in `Dockerfile` or mount jars via `downloads/` and adjust Dockerfile to COPY them.
- S3/MinIO: The project uses MinIO as S3. Environment variables used in `docker-compose.yml`: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `S3_ENDPOINT`. Code that uses S3A should set the endpoint and credentials accordingly (see `conf/core-site.xml` for examples).
- Non-root user: the image runs as a non-root `spark` user (UID/GID 1000). When creating files in mounted volumes, ensure host permissions are compatible (or run builds/tests that tolerate uid mapping).

Key files to consult when modifying behavior
- `docker-compose.yml` — service roles, mounts, network and ports.
- `spark/Dockerfile` — how the runtime is constructed (Spark version, Java, jars, user, event log settings).
- `spark/entrypoint.sh` — how containers are started and which runtime flags are used (notably how history-server and notebooks are started, and the default spark-shell/pyspark jars).
- `spark/conf/core-site.xml` — S3A/Hadoop configurations used by the image.
- `spark/app` — place to add notebooks, PySpark jobs, or helper scripts. Mounts into containers as `/home/spark/app`.

Editing guidance for AI agents
- When changing Spark versions or adding jars, update `spark/Dockerfile` and ensure jars are placed in `/home/spark/jars` (either via curl or COPY). Preserve the non-root user creation and the event log configuration lines.
- For runtime changes (memory, master URL), prefer editing `docker-compose.yml` environment variables (e.g., `MASTER_URL`, `SPARK_DRIVER_MEMORY`) rather than hardcoding in entrypoint.
- Preserve `entrypoint.sh` CLI interface (jupyter, spark-shell, pyspark, bashed). If adding modes, keep the command-case structure and document new options here.

Testing and verification
- After edits to Dockerfile/docker-compose, run `docker-compose up --build` and verify: Jupyter at http://localhost:8888, Spark master UI at http://localhost:8080, MinIO console at http://localhost:9001.
- Validate Hudi integration by running a minimal PySpark job that writes a Hudi table to S3A pointing at MinIO (example notebooks live in `spark/app` if present).

Limitations and assumptions (inferred)
- This repository focuses on a local dev environment for experimenting with Spark + Hudi using MinIO; it is not wired for production deployments.
- Java runtimes: Dockerfile installs OpenJDK 17 at build time but sets JAVA_HOME pointing to Java 11 path; be careful if changing Java version — confirm the correct JAVA_HOME in the image.

If anything here is unclear or you'd like examples (a minimal Hudi write notebook, or a sample `core-site.xml` for MinIO S3A), tell me which example and I'll add it.