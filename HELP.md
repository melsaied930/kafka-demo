# Help & Troubleshooting

## Common Issues

### 1. Kafka UI shows "No Data" or does not display topics
**Symptom:** The UI loads but shows an empty cluster, or an error about being unable to connect.

**Cause:** The Kafka broker advertises a listener that is not reachable from the UI container. By default, the UI uses `kafka:9092` as the bootstrap server. If the broker advertises `localhost:9092`, the UI will try to connect to its own localhost instead of the Kafka container.

**Solution:** Ensure the advertised listener matches the internal service name. The corrected configuration uses:
- `KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092,EXTERNAL://localhost:9094`

If you are still experiencing issues, check the UI logs:
```bash
docker logs kafka-ui
```

If you are still experiencing issues, check the UI logs errors:
```bash
docker logs kafka-ui 2>&1 | grep -i error
```

### 2. `kafka-topics.sh: command not found`
**Symptom:** Running `kafka-topics.sh` on the host machine fails.

**Cause:** Kafka binaries are not installed on your host. They exist only inside the Kafka container.

**Solution:** Always prefix commands with `docker exec -it kafka-kraft`. Example:
```bash
docker exec -it kafka-kraft kafka-topics.sh --bootstrap-server localhost:9092 --list
```

### 3. Unable to produce/consume from host machine
**Symptom:** Clients on the host cannot connect to `localhost:9092`.

**Cause:** The internal listener (`localhost:9092`) is not exposed for external access.

**Solution:** Use the external listener on port `9094`. Update your client to use `localhost:9094`. If you need to change the external port, modify the `ports` mapping in `compose.yaml` and adjust the `EXTERNAL` advertised listener accordingly.

### 4. Container fails to start with port conflicts
**Symptom:** Error like `port is already allocated`.

**Solution:** Stop any existing services using ports `9092`, `9093`, `9094`, or `8085`. You can change the host ports by editing the `ports` section in `compose.yaml`.

## Frequently Asked Questions

### How do I create a topic?
```bash
docker exec -it kafka-kraft kafka-topics.sh --bootstrap-server localhost:9092 --create --topic my-topic --partitions 1 --replication-factor 1
```

### How do I delete a topic?
```bash
docker exec -it kafka-kraft kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic my-topic
```

### Can I change the Kafka version?
Yes. Update the `image` tag for the `kafka` service (e.g., `bitnamilegacy/kafka:3.8.0`). Ensure the version is compatible with KRaft mode.

### How do I persist Kafka data?
By default, data is stored inside the container. To make it persistent, add a volume mount to the `kafka` service:
```yaml
volumes:
  - kafka_data:/bitnami/kafka
```
And define the volume at the bottom of the compose file.

### How can I access Kafka from another Docker container?
Use the internal service name `kafka:9092` as the bootstrap server. Ensure the other container is on the same `kafka-net` network.

## Detailed Configuration Options

### Kafka Environment Variables
| Variable | Description |
|----------|-------------|
| `KAFKA_ENABLE_KRAFT` | Enables KRaft mode (no ZooKeeper). |
| `KAFKA_CFG_NODE_ID` | Unique node ID for the broker. |
| `KAFKA_CFG_PROCESS_ROLES` | Roles this node plays (`broker,controller`). |
| `KAFKA_CFG_CONTROLLER_LISTENER_NAMES` | Listener name used for controller communication. |
| `KAFKA_CFG_LISTENERS` | Comma‑separated list of listeners and their ports. |
| `KAFKA_CFG_ADVERTISED_LISTENERS` | Listeners advertised to clients. |
| `KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP` | Maps listener names to security protocols. |
| `KAFKA_CFG_INTER_BROKER_LISTENER_NAME` | Listener used for broker‑to‑broker communication. |
| `KAFKA_CFG_CONTROLLER_QUORUM_VOTERS` | List of controller nodes (format: `id@host:port`). |
| `ALLOW_PLAINTEXT_LISTENER` | Allows plaintext listeners without SSL. |
| `KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE` | Automatically create topics when produced to. |

### Ports
| Port (Host) | Port (Container) | Purpose |
|-------------|-----------------|---------|
| 9092 | 9092 | Internal listener (used by UI and other containers). |
| 9093 | 9093 | Controller listener (KRaft internal). |
| 9094 | 9094 | External listener (for host access). |
| 8085 | 8080 | Kafka UI web interface. |

## Additional Resources
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Kafka UI GitHub Repository](https://github.com/provectus/kafka-ui)
- [Bitnami Kafka Image Documentation](https://github.com/bitnami/containers/tree/main/bitnami/kafka)