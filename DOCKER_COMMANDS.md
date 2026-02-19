# 🐳 Docker Commands – Backend Developer Cheat Sheet

---

## 🔍 1. Check Containers

### Show running containers
docker ps

### Show all containers (including stopped)
docker ps -a

---

## ▶ 2. Start / Stop Containers

### Start container
docker start <container-name>

### Stop container
docker stop <container-name>

### Restart container
docker restart <container-name>

---

## ❌ 3. Remove Containers

### Remove stopped container
docker rm <container-name>

### Force remove running container
docker rm -f <container-name>

---

## 📦 4. Run New Container

### Run container (detached mode)
docker run -d --name <name> -p <host-port>:<container-port> <image>

Example (Kafka):
docker run -d --name kafka -p 9092:9092 confluentinc/cp-kafka

---

## 📜 5. View Logs

### See container logs
docker logs <container-name>

### Follow logs live
docker logs -f <container-name>

---

## 🖥 6. Execute Commands Inside Container

### Open bash shell
docker exec -it <container-name> bash

### Run Kafka topic list
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --list

---

## 📚 7. Kafka Specific Commands

### List topics
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --list

### Create topic
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --create --topic employee-topic --partitions 1 --replication-factor 1

### Describe topic
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --describe --topic employee-topic

### Console producer
docker exec -it kafka kafka-console-producer --bootstrap-server localhost:9092 --topic employee-topic

### Console consumer
docker exec -it kafka kafka-console-consumer --bootstrap-server localhost:9092 --topic employee-topic --from-beginning

---

## 🧹 8. Clean Everything

### Stop all containers
docker stop $(docker ps -aq)

### Remove all containers
docker rm $(docker ps -aq)

---

# 💡 Important Concepts

- Image → Blueprint
- Container → Running instance
- Port Mapping → host:container
- docker exec → Run command inside container
- docker logs → Debug issues

