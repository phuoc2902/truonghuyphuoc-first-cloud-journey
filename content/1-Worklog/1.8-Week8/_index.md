---
title: "Week 8 Worklog"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---
### Week 8 Objectives:

* Implement Message Publisher pattern for asynchronous email and system notifications.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | Design serializable Event DTO payloads representing business activities like OrderSuccess and UserRegistered. | 03/06/2026 | 03/06/2026 |  |
| 3 | Write the EmailEventPublisher service utilizing RabbitTemplate to publish JSON payloads. | 04/06/2026 | 04/06/2026 | <https://spring.io/projects/spring-amqp> |
| 4 | Create the NotificationEventPublisher service for in-app alert triggers. | 05/06/2026 | 05/06/2026 |  |
| 5 | Refactor primary transaction flows (like checkout) to publish events instead of sending emails synchronously. | 08/06/2026 | 08/06/2026 |  |
| 6 | Run load tests to verify message publishing is non-blocking to HTTP requests. | 09/06/2026 | 09/06/2026 |  |

### Week 8 Achievements:

* Successfully decoupled core APIs from slow dependency operations (SMTP connections).
* Published event messages are transactional, rolling back if database operations fail.
* Reduced client checkout latency from seconds to milliseconds.
