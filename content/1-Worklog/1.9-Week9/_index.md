---
title: "Week 9 Worklog"
date: 2024-01-01
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---
### Week 9 Objectives:

* Develop Message Consumers to process asynchronous email sending and notification delivery.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | Configure Spring Boot JavaMailSender properties with SMTP mail servers. | 10/06/2026 | 10/06/2026 | <https://spring.io/guides/gs/sending-email/> |
| 3 | Create EmailEventConsumer listener using @RabbitListener annotations to poll and process messages. | 11/06/2026 | 11/06/2026 | <https://spring.io/projects/spring-amqp> |
| 4 | Design responsive email templates using Thymeleaf HTML templates. | 12/06/2026 | 12/06/2026 | <https://www.thymeleaf.org/> |
| 5 | Develop NotificationEventConsumer listener to write alert logs directly into the database. | 15/06/2026 | 15/06/2026 |  |
| 6 | Run comprehensive end-to-end tests from frontend purchase to physical inbox receipt. | 16/06/2026 | 16/06/2026 |  |

### Week 9 Achievements:

* Asynchronous processing engine handles failures gracefully using retry backoff mechanisms.
* Dynamic Thymeleaf HTML emails are generated and sent successfully under sub-second timings.
* In-app notifications are stored and ready for real-time WebSockets integration.
