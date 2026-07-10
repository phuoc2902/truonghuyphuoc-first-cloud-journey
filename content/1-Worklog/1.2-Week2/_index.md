---
title: "Week 2 Worklog"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---
### Week 2 Objectives:

* Configure Axios interceptors for distributed Request ID tracing and automatic JWT token handling.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | Research Axios Interceptor design patterns and token refresh flow architectures. | 22/04/2026 | 22/04/2026 | <https://axios-http.com/docs/interceptors> |
| 3 | Configure basic Axios instance and implement Request Interceptor to inject a unique RequestID header. | 23/04/2026 | 23/04/2026 | <https://axios-http.com/docs/interceptors> |
| 4 | Develop Response Interceptor to capture HTTP 401 Unauthorized errors and handle user authentication loss. | 24/04/2026 | 24/04/2026 |  |
| 5 | Write async logic to call the refresh token endpoint and transparently retry previous failed API requests. | 27/04/2026 | 27/04/2026 |  |
| 6 | Mock HTTP responses and execute unit tests on the Axios client configuration. | 28/04/2026 | 28/04/2026 |  |

### Week 2 Achievements:

* Fully configured central Axios client ready for remote server communication.
* Implemented request tracing by attaching unique RequestID identifiers on all outbound calls.
* Robust authentication refresh mechanism created to silently renew expired user access tokens.
