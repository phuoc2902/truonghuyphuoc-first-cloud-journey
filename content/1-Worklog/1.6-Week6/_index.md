---
title: "Week 6 Worklog"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---
### Week 6 Objectives:

* Implement client-to-server Cart Synchronization logic upon successful user login.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | Define merge conflict resolution rules (e.g. sum quantities, update to latest pricing). | 20/05/2026 | 20/05/2026 |  |
| 3 | Write a backend service method that accepts a list of client items and performs database merge logic. | 21/05/2026 | 21/05/2026 |  |
| 4 | Modify frontend authentication flow to dispatch local storage cart items to the sync API. | 22/05/2026 | 22/05/2026 |  |
| 5 | Implement validation checks to handle items that are out of stock during synchronization. | 25/05/2026 | 25/05/2026 |  |
| 6 | Add clear-cart hooks on local storage to delete items only when sync API reports success. | 26/05/2026 | 26/05/2026 |  |

### Week 6 Achievements:

* Achieved seamless synchronization of shopping items when transitioning from guest to logged-in user.
* Minimized backend DB queries during sync by using bulk JPA saving operations.
* Local Storage is cleared automatically post-sync, preventing duplicate item issues.
