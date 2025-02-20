# Cab Booking System with Kafka Integration

## Overview
This project consists of two Spring Boot applications:
1. **cab-book-driver** - Responsible for updating cab locations and publishing them to a Kafka topic.
2. **cab-book-user** - Listens for location updates from Kafka and processes them.

Kafka is used as the messaging system to facilitate real-time communication between the driver and user services.

---

## Project Structure

### cab-book-driver
- **KafkaConfig**: Defines Kafka topic configuration.
- **AppConstant**: Stores application constants.
- **CabLocationController**: REST controller to simulate cab location updates.
- **CabLocationService**: Sends updated locations to Kafka.
- **CabBookDriverApplication**: Main Spring Boot application class.

### cab-book-user
- **LocationService**: Listens to Kafka messages from the `cab-location` topic.
- **CabBookUserApplication**: Main Spring Boot application class.

---

## Kafka Configuration

### Kafka Topic Creation
The `KafkaConfig` class in `cab-book-driver` defines a Kafka topic named `cab-location`:

```java
@Bean
public NewTopic topic() {
    return TopicBuilder.name(AppConstant.CAB_LOCATION).build();
}
```

### Publishing to Kafka
The `CabLocationService` sends location updates to the Kafka topic:

```java
public boolean updateLocation(String location){
    kafkaTemplate.send(AppConstant.CAB_LOCATION, location);
    return true;
}
```

### Consuming from Kafka
The `LocationService` in `cab-book-user` listens for messages from the `cab-location` topic:

```java
@KafkaListener(topics = "cab-location", groupId = "user-group")
public void cabLocation(String location){
    System.out.println(location);
}
```

---

## API Endpoints

### Driver Service
#### Update Location
- **Endpoint:** `PUT /location`
- **Function:** Simulates location updates for the cab and sends them to Kafka.
- **Implementation:**
  ```java
  @PutMapping
  public ResponseEntity updateLocation() throws InterruptedException {
      int range = 100;
      while (range>0){
          cabLocationService.updateLocation(Math.random() + "," + Math.random());
          Thread.sleep(1000);
          range --;
      }
      return new ResponseEntity<>(Map.of("message","location updated"), HttpStatus.OK);
  }
  ```

---

## Running the Project

### Prerequisites
1. Install **Kafka** and **Zookeeper** (or use a managed Kafka service).
2. Ensure **Java 17+** and **Spring Boot** are installed.
3. Set up Kafka topic `cab-location`.

### Steps
1. Start **Zookeeper** and **Kafka Broker**:
   ```sh
   zookeeper-server-start.sh config/zookeeper.properties
   kafka-server-start.sh config/server.properties
   ```
2. Run `cab-book-driver`:
   ```sh
   mvn spring-boot:run -f cab-book-driver
   ```
3. Run `cab-book-user`:
   ```sh
   mvn spring-boot:run -f cab-book-user
   ```
4. Trigger cab location updates using the **PUT** API request to `/location`.
5. Check the console of `cab-book-user` to see location updates.

---

## Conclusion
This project demonstrates a **real-time location update system** using **Kafka and Spring Boot**. The driver service publishes location updates, while the user service consumes and processes them.

