# Test-Kafka

### Создаю docker-compose.yml файл с такими сервисами как:
- **1.Zookeeper**: *используется для координации и управления распределенными системами. В контексте Kafka, он используется для управления метаданными, отслеживания состояния брокеров и управляемых топиков и разделов (partitions).*

- **2.Kafka**: *распределенная система обмена сообщениями между серверными приложениями в режиме реального времени.*

- **3.Kafka REST Proxy**: *предоставляет HTTP API для взаимодействия с Kafka. Это позволяет производить и потреблять сообщения в Kafka через RESTful API, что удобно для интеграции с веб-приложениями и другими системами, которые поддерживают HTTP*

- **4.Kafka-ui**: *это веб-интерфейс для управления и мониторинга Kafka кластеров. Он предоставляет удобный интерфейс для просмотра топиков, сообщений, потребителей и других метрик Kafka.*

```
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.2.1
    hostname: zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-server:7.2.1
    hostname: kafka
    container_name: kafka
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
      - "9997:9997"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_CONFLUENT_LICENSE_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_CONFLUENT_BALANCER_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_JMX_PORT: 9997
      KAFKA_JMX_HOSTNAME: kafka

  kafka-rest:
    image: confluentinc/cp-kafka-rest:7.2.1
    hostname: kafka-rest
    container_name: kafka-rest
    depends_on:
      - kafka
    ports:
      - "8082:8082"
    environment:
      KAFKA_REST_HOST_NAME: kafka-rest
      KAFKA_REST_BOOTSTRAP_SERVERS: 'kafka:9092'
      KAFKA_REST_LISTENERS: http://0.0.0.0:8082

  kafka-ui:
    container_name: kafka-ui
    image: provectuslabs/kafka-ui:latest
    ports:
      - "8080:8080"
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:9092
      KAFKA_CLUSTERS_0_ZOOKEEPER: zookeeper:2181
      DYNAMIC_CONFIG_ENABLED: 'true'
```
### После выполнения compose файла проверяю работоспособность контейнера
- **Перехожу на порт kafka-ui:**
  
![Снимок экрана (109)](https://github.com/AleksandrShirobokov/Test-Kafka/assets/69298696/dc4070c8-5937-43ff-a608-906ccf981d51)

- **Брокер:**

![Снимок экрана (110)](https://github.com/AleksandrShirobokov/Test-Kafka/assets/69298696/f9bae42e-761d-4052-ae1e-b3b02b406043)

- **Топики:**

![Снимок экрана (111)](https://github.com/AleksandrShirobokov/Test-Kafka/assets/69298696/da1bb951-0233-49ab-b5ed-fab357ddb43c)
