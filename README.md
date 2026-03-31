# Kafka KRaft Docker Compose Setup

This project provides a Docker Compose configuration to run Apache Kafka in **KRaft mode** (without ZooKeeper) together with a Kafka UI for easy management and monitoring.

## Prerequisites
- [Docker](https://docs.docker.com/get-docker/) (version 20.10+)
- [Docker Compose](https://docs.docker.com/compose/install/) (version 2.0+)

## Installation

1. Clone this repository or download the `compose.yaml` file into a directory of your choice.

2. Start the services:
   ```bash
   docker compose up -d
   ```

3. Verify that both containers are running and healthy:
   ```bash
   docker compose ps
   ```

4. Restart:
   ```bash
   docker compose down -v && docker compose up -d && docker compose logs -f 
   ```
   You should see `kafka-kraft` with status `healthy` and `kafka-ui` with status `up`.

## Usage

### Access Kafka UI
Open your browser and go to: [http://localhost:8085](http://localhost:8085)

### Manage Topics and Data from the Command Line
All Kafka administrative commands must be run inside the Kafka container using `docker exec`.

#### List Topics
```bash
docker exec -it kafka-kraft kafka-topics.sh --bootstrap-server localhost:9092 --list
```

#### Create a Topic
```bash
docker exec -it kafka-kraft kafka-topics.sh --bootstrap-server localhost:9092 --create --topic my-topic --partitions 3 --replication-factor 1
```

#### Example Create a Topic
```bash
docker exec -it kafka-kraft kafka-topics.sh --bootstrap-server localhost:9092 --create --topic test-topic --partitions 1 --replication-factor 1
```

#### Produce Messages
```bash
docker exec -it kafka-kraft kafka-console-producer.sh --bootstrap-server localhost:9092 --topic my-topic
```

#### Example Produce Messages
```bash
docker exec -it kafka-kraft kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test-topic
```
Type messages line by line and press **Ctrl+C** to exit.

#### Consume Messages
```bash
docker exec -it kafka-kraft kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-topic --from-beginning
```

#### Example Consume Messages
```bash
docker exec -it kafka-kraft kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --from-beginning
```

### Access Kafka from Your Host Machine
The external listener is configured on port `9094`. You can connect to Kafka using `localhost:9094` from any client on your host machine (e.g., `kcat`, Java applications).

Example with `kcat` (install separately):
```bash
kcat -P -b localhost:9094 -t my-topic
```

## Key Features
- **Apache Kafka 3.7.0** in KRaft mode – no ZooKeeper required.
- **Kafka UI** (provectuslabs/kafka-ui) for visual cluster management.
- **Multi‑listener configuration**:
    - Internal listener (`kafka:9092`) for container‑to‑container communication.
    - External listener (`localhost:9094`) for host‑to‑container communication.
    - Controller listener (`:9093`) for KRaft internal use.
- **Healthcheck** that verifies the broker is ready before starting the UI.
- **Automatic topic creation** enabled for convenience.

## Contribution Guidelines
Contributions are welcome! Please follow these steps:
1. Fork the repository.
2. Create a feature branch (`git checkout -b feature/amazing-feature`).
3. Commit your changes (`git commit -m 'Add some amazing feature'`).
4. Push to the branch (`git push origin feature/amazing-feature`).
5. Open a Pull Request.

## License
This project itself does not specify a license. The Docker images used are subject to their own licenses:
- [bitnamilegacy/kafka](https://hub.docker.com/r/bitnamilegacy/kafka) – Apache License 2.0
- [provectuslabs/kafka-ui](https://hub.docker.com/r/provectuslabs/kafka-ui) – Apache License 2.0

---
