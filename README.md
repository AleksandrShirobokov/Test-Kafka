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
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
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
      KAFKA_REST_BOOTSTRAP_SERVERS: kafka:9092
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
## После выполнения compose файла проверяю работоспособность контейнера
### Перехожу на порт kafka-ui:
  
![Снимок экрана (109)](https://github.com/AleksandrShirobokov/Test-Kafka/assets/69298696/dc4070c8-5937-43ff-a608-906ccf981d51)

- **Брокер:**

![Снимок экрана (110)](https://github.com/AleksandrShirobokov/Test-Kafka/assets/69298696/f9bae42e-761d-4052-ae1e-b3b02b406043)

- **Топики:**

![Снимок экрана (111)](https://github.com/AleksandrShirobokov/Test-Kafka/assets/69298696/da1bb951-0233-49ab-b5ed-fab357ddb43c)



## Для отправки сообщения выполняю следующий запрос:

```
curl -X POST -H "Content-Type: application/vnd.kafka.json.v2+json" \
--data '{"records":[{"value":{"key1":"value1"}}]}' \
http://localhost:8082/topics/test-topic
```


## Проверяю в kafka-ui:
- **Созданный test-topic:**
  
![Снимок экрана (8)](https://github.com/AleksandrShirobokov/Test-Kafka/assets/69298696/8e299881-f260-4f6c-a077-26e2d2812ad9)

- **Полученное сообщение:**

![Снимок экрана (9)](https://github.com/AleksandrShirobokov/Test-Kafka/assets/69298696/1ad612be-5148-4833-bf9e-23c8cd777c44)

## Также использую скрипт python для отправки сообщения:

```
#!/usr/bin/env python3
import requests
import json

url = "http://localhost:8082/topics/test-topic" #Адрес Rest

headers = {
    "Content-Type": "application/vnd.kafka.json.v2+json" #Заголовки
}

data = {
    "records": [
        {"value": {"message": "Проверка доставки сообщения"}}  #Данные сообщ.
    ]
}

data_json = json.dumps(data) #Перевод в json

response = requests.post(url, headers=headers, data=data_json) #POST запрос

if response.status_code == 200:  #Статус ответа
    print("Message sent successfully!")
else:
    print(f"Failed to send message. Status code: {response.status_code}, Response: {response.text}")
```

- **Запрос в CLI:**

![Снимок экрана (112)](https://github.com/AleksandrShirobokov/Test-Kafka/assets/69298696/81e71d9a-d342-4689-a903-47b5f7d5a25e)

- **Сообщение в test-topic:**

![Снимок экрана (113)](https://github.com/AleksandrShirobokov/Test-Kafka/assets/69298696/86a5a698-e9bd-4def-9ef3-b5d692ca514d)

## Работа с новой группой слушателей:

- **Cоздаю новую группу *consumer_new* с *my_topic*:**

```
curl -X POST -H "Content-Type: application/vnd.kafka.v2+json"
--data '{"name": "my_instance", "format": "json", "auto.offset.reset": "earliest"}'
http://localhost:8082/consumers/consumer_new
```

- **Подписываюсь на новую группу *consumer_new*:**

```
curl -X POST -H "Content-Type: application/vnd.kafka.v2+json"
--data '{"topics":["my_topic"]}' http://localhost:8082/consumers/consumer_new/instances/my_instance/subscription
```  

- **Отправляю POST-запрос с сообщением в *my_topic*:**

```
curl -X POST -H "Content-Type: application/vnd.kafka.json.v2+json"
--data '{"records":[{"value":{"message":"Проверка связи"}}]}' http://localhost:8082/topics/my_topic
```

- **Отправляю GET для получения отправленного сообщения:**

```
curl -X GET -H "Accept: application/vnd.kafka.json.v2+json"
http://localhost:8082/consumers/consumer_new/instances/my_instance/records
```

- **После получения сообщения удаляю *consumer_new:***

```
 curl -X DELETE -H "Content-Type: application/vnd.kafka.v2+json"
http://localhost:8082/consumers/consumer_new/instances/my_instance
```

## Результат работы в cli:

![Снимок экрана туц](https://github.com/AleksandrShirobokov/Test-Kafka/assets/69298696/fb8c47ae-d21e-4a0d-896e-87f71b5d4ee5)

## Отслеживание работы в kafka-ui:

- **Группа *consumer_new* и *my_topic*:**

![Снимок экрана (12)](https://github.com/AleksandrShirobokov/Test-Kafka/assets/69298696/5b8f56ce-0d32-4c76-b17a-93073e91d833)

- **Полученное сообщение:**

![Снимок экрана (13)](https://github.com/AleksandrShirobokov/Test-Kafka/assets/69298696/b0e78f4d-463a-40b1-ba95-fe30d1c7d170)

- **Отписка от *consumer_new*(статус группы переходит в empty):**

![Снимок экрана (14)](https://github.com/AleksandrShirobokov/Test-Kafka/assets/69298696/4b7ab16e-2bdb-428b-bcd3-a682902b6029)

  
