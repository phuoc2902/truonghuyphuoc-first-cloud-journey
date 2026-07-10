---
title: "Week 12 Worklog"
date: 2024-01-01
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---
### Week 12 Objectives:

* Automate Docker packaging and push release images to GitLab Container Registry.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | Write a multi-stage Dockerfile containing build environment and final lightweight JRE runner. | 01/07/2026 | 01/07/2026 | <https://docs.docker.com/develop/develop-images/multistage-build/> |
| 3 | Configure package stage using Docker-in-Docker (dind) executor to compile container images. | 02/07/2026 | 02/07/2026 | <https://docs.gitlab.com/ee/ci/docker/using_docker_build.html> |
| 4 | Set up environment variables in GitLab UI to store container registry credentials securely. | 03/07/2026 | 03/07/2026 |  |
| 5 | Push compiled Docker image tagged with Git commit hash to GitLab Container Registry. | 06/07/2026 | 06/07/2026 |  |
| 6 | Conduct a complete end-to-end deployment verification of the dockerized image. | 07/07/2026 | 07/07/2026 |  |

### Week 12 Achievements:

* DevOps flow fully completed: commit -> build -> test -> docker build -> image registry push.
* Containerized application package size minimized using multi-stage builds (reducing footprints by 60%).
* Standardized deployments ready for continuous delivery (CD) integration.
