---
title: "Week 7 Worklog"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---
### Week 7 Objectives:

* Install, configure, and initialize Message Queue (RabbitMQ/Kafka) infrastructure.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | Compare RabbitMQ vs Kafka and choose RabbitMQ for AMQP standard compliance and routing features. | 27/05/2026 | 27/05/2026 |  |
| 3 | Write docker-compose templates to launch a RabbitMQ instance with Management interface. | 28/05/2026 | 28/05/2026 | <https://www.docker.com/> |
| 4 | Define connection factory properties, exchanges, queues, and binding keys inside Spring Boot configuration. | 29/05/2026 | 29/05/2026 | <https://spring.io/projects/spring-amqp> |
| 5 | Configure Jackson Message Converter to automatically serialize DTO payloads into JSON format. | 01/06/2026 | 01/06/2026 |  |
| 6 | Verify exchange-to-queue bindings via RabbitMQ Management HTTP Console. | 02/06/2026 | 02/06/2026 |  |

### Week 7 Achievements:

* Established a local RabbitMQ container running successfully within the system environment.
* Configured Spring Boot AMQP integration settings enabling direct message routing capabilities.
* Created JSON message parsing standard used across all publish/subscribe interactions.
