Homework 2 — CI/CD with Jenkins, GitHub, Docker, Amazon ECR, and Docker Swarm
Project overview
This project demonstrates a CI/CD pipeline that builds a Docker image from a GitHub repository, pushes the image to a private Amazon Elastic Container Registry (ECR) repository, and deploys the image to a Docker Swarm cluster running on AWS EC2.
Repository and deployment details
GitHub repository: https://github.com/sortsom/homework-2-cicd
Branch: `main`
Jenkins job: `homework2-cicd`
AWS Region: `us-east-1` (US East, N. Virginia)
ECR repository: `homework2-app`
ECR URI: `753674273284.dkr.ecr.us-east-1.amazonaws.com/homework2-app`
Live application: http://18.215.154.25:8080
Swarm stack: `homework2`
Swarm service: `homework2_web`
> The live URL and temporary public access depend on the EC2 instance and its security-group rules remaining available. Do not put AWS credentials, private keys, or ngrok tokens in this repository.
Architecture
```text
Developer pushes to GitHub
          |
          v
GitHub push webhook
          |
          v
Jenkins job: homework2-cicd
          |
          +--> Checkout source
          +--> Build Docker image
          +--> Authenticate to Amazon ECR
          +--> Push image tagged with Git commit SHA
          +--> SSH to Swarm manager
                    |
                    v
              docker stack deploy
                    |
                    v
       Docker Swarm manager + worker
                    |
                    v
           Application on port 8080
```
Pipeline stages
Checkout — Jenkins checks out the source from SCM.
Build Docker Image — Docker builds the application image and tags it with a short Git commit SHA.
Push to Amazon ECR — Jenkins authenticates to ECR and pushes the tagged image to the private repository.
Deploy to Docker Swarm — Jenkins securely connects to the manager over SSH, transfers `docker-stack.yml`, authenticates Docker to ECR, and runs `docker stack deploy`.
Verify Deployment — Jenkins checks the services in the deployed stack.
The Jenkins job is configured to react to GitHub push webhooks. The GitHub webhook delivery was shown as successful, and Jenkins builds completed successfully.
Docker Swarm configuration
The stack file defines the application service with two replicas and a rolling-update policy:
Replicas: `2`
Update parallelism: `1`
Update delay: `10 seconds`
Update order: `start-first`
Published port: `8080` to container port `80`
Restart policy: restart on failure
The Swarm cluster was verified with one manager and one worker in `Ready` state. The service was reported as `2/2` replicas, and an HTTP check to `http://127.0.0.1:8080/` returned status `200`.
Repository files
The repository should contain these assignment deliverables:
`Dockerfile` — builds the application container.
`Jenkinsfile` — defines the CI/CD pipeline.
`docker-stack.yml` — defines the Swarm service and rolling-update policy.
Application source files — the web application.
`README.md` — project overview, architecture, configuration, and verification steps.
Confirm these files are committed to the GitHub repository before submission.
How to verify
Check Jenkins
Open Jenkins and select `homework2-cicd`.
Confirm the latest completed build is green.
Open the build and review each stage.
Check Amazon ECR
Open Amazon ECR in `us-east-1`.
Open Private registry → Repositories → `homework2-app` → Images.
Confirm a commit-SHA image tag is present.
Check Docker Swarm on the manager
```bash
sudo docker node ls
sudo docker stack services homework2
sudo docker service ps homework2_web
sudo docker service inspect homework2_web --format '{{json .Spec.UpdateConfig}}'
```
Check the application
```bash
curl -sS -o /dev/null -w "HTTP Status: %{http_code}\n" http://127.0.0.1:8080/
```
Expected result from the recorded test: `HTTP Status: 200`.
Git workflow
A feature branch was used to make a README change, and Pull Request #1 was merged into `main`. This demonstrates the feature-branch and pull-request requirement.
Security notes
Keep AWS access keys, EC2 private keys, Jenkins secrets, and ngrok authentication tokens out of Git.
Rotate credentials if they were exposed in screenshots, chat, or logs.
Restrict SSH (port 22) to a trusted source IP.
Allow Docker Swarm ports only between the manager and worker using a dedicated security-group reference or private network range: TCP 2377, TCP/UDP 7946, and UDP 4789.
Restrict application port 8080 to the intended audience unless public access is explicitly required for assessment.
Stop the ngrok tunnel when it is no longer needed; a tunnel can expose the Jenkins interface publicly.
Submission evidence checklist
[ ] GitHub repository URL and source files
[ ] Screenshot of successful Jenkins pipeline stages
[ ] Screenshot of GitHub webhook delivery
[ ] Screenshot of ECR repository and image tags
[ ] Screenshot of `docker node ls` showing manager and worker
[ ] Screenshot of `docker stack services homework2` showing `2/2`
[ ] Screenshot of the application returning HTTP 200 / loading in a browser
[ ] Screenshot or link to the merged feature-branch pull request
[ ] 1–2 page report
