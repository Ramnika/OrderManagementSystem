# OrderManagementSystem
An order management system to process and take orders in an ecormmerce site

1. Instructions for Setting Up and Running the Application
Prerequisites:

Java 17 or higher.
MySQL or any compatible database.
Maven for building the project.
Postman or curl for testing the APIs.

Step 1: Clone the Repository
Clone the project repository from GitHub:

git clone https://github.com/your-username/orderManagementSystem.git
cd ecommerce-order-system

Step 2: Set Up the Database
Start MySQL and create a database named ecommerce:

CREATE DATABASE ecommerce;
Update the application.properties file with your database credentials:

spring.datasource.url=jdbc:mysql://localhost:3306/ecommerce?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=your_password

Step 3: Build and Run the Application
Build the project using Maven:

mvn clean install

Run the application:

mvn spring-boot:run
The application will start at http://localhost:portNumberSpecifiedInProperties.

2. Example API Requests and Responses
API Endpoints

Create an Order:

Endpoint: POST /orders

Request Body:

json
Copy
{
  "userId": "123",
  "orderId": "456",
  "itemIds": "[\"item1\", \"item2\"]",
  "totalAmount": 100.50
}

Example curl Command:

curl -X POST http://localhost:8080/orders \
-H "Content-Type: application/json" \
-d '{
  "userId": "123",
  "orderId": "456",
  "itemIds": "[\"item1\", \"item2\"]",
  "totalAmount": 100.50
}'

Response:

{
  "message": "Order received and queued for processing.",
  "orderId": "456"
}

Check Order Status:

Endpoint: GET /orders/{orderId}/status

Example curl Command:

curl -X GET http://localhost:8080/orders/456/status
Response:

{
  "orderId": "456",
  "status": "Pending"
}

Fetch Metrics:

Endpoint: GET /metrics

Example curl Command:

curl -X GET http://localhost:8080/metrics
Response:

{
  "totalOrdersProcessed": 3,
  "averageProcessingTimeSeconds": 5.4,
  "orderStatusCounts": {
    "Pending": 1,
    "Processing": 1,
    "Completed": 1
  }
}

3. Explanation of Design Decisions and Trade-Offs
Design Decisions

In-Memory Queue:
Used an in-memory queue (OrderQueue) to handle asynchronous order processing.
Trade-Off: In-memory queues are simple and fast but are not persistent. If the application crashes, queued orders will be lost.

Thread Pool for Order Processing:
Used a fixed thread pool (ExecutorService) to process orders concurrently.
Trade-Off: A fixed thread pool ensures controlled concurrency but may become a bottleneck under very high load.

Batch Processing:
Orders are saved in batches to reduce the number of database writes.
Trade-Off: Batch processing improves performance but introduces a delay in saving individual orders.


Trade-Offs

1) Scalability vs. Simplicity:
The system is designed for simplicity and ease of development. For higher scalability, we can consider using a distributed queue (e.g., RabbitMQ or Kafka) and a distributed database.

2) Performance vs. Durability:
In-memory queues and batch processing improve performance but reduce durability. For a production system, consider using persistent queues and databases.

4. Assumptions Made During Development
Order Processing Time:
Assumed that order processing takes between 1 and 10 seconds (simulated using Thread.sleep).

Order Statuses:
Assumed that orders have three possible statuses: Pending, Processing, and Completed.

Concurrency:
Assumed that the system needs to handle up to 1,000 concurrent orders. The thread pool size and database connection pool are configured accordingly.

Metrics Calculation:
Assumed that metrics (e.g., average processing time) are calculated based on completed orders only.

Error Handling:
Assumed that errors (e.g., invalid order IDs) are handled gracefully and logged for debugging.

5. Load Testing Instructions
   
To simulate 1,000 concurrent orders, use Locust:

Install Locust:

pip install locust
Create a locustfile.py: (Already present in resources folder)

from locust import HttpUser, task, between

class OrderUser(HttpUser):
    wait_time = between(1, 5)

    @task
    def create_order(self):
        self.client.post("/orders", json={
            "userId": "123",
            "orderId": "456",
            "itemIds": "[\"item1\", \"item2\"]",
            "totalAmount": 100.50
        })
Run Locust:

locust -f locustfile.py
Open the Locust web interface at http://localhost:8089 (port number mentioned in properties file) and start the test with 1,000 users.
