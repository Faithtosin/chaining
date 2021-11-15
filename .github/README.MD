
## Pipelines

### Pull Request created pipeline (`PR-created.yml`)

This pipeline is triggered when a pull request is created.  

The branch is checked out and unit test is run against 4 different versions of Node (10.x, 12.x, 14.x, 15.x).

[SonarCloud](https://sonarcloud.io/summary/new_code?id=Faithtosin_chaining) also runs static code analysis using set quality gates and profiles to detect code vulnurabilities and code smells. 

Branch protection rules is also configured for the `main` branch, to make sure some required status checks pass before merging. These checks include Sonarcloud checks, and Unit test run in the `Pull Request created pipeline` for the 4 Node version. 

### Main Branch pipeline (`PR-merged.yml`)
#### build job
Since a pull request merge simulates a push action, this pipeline is triggered when there is a push to the `main` branch.  

The branch is checked out, task is installed, and AWS credentials are configured.
After the secure login to Amazon ECR, the docker image is built and pushed using commands in the Taskfile. The built image is tagged with the `github sha` and `main` and pushed to Amazon ECR repository.

#### deploy job
As an additional step, a `deploy job` was added to deploy the built docker image to Amazon ECS fargate. The job waits for successful deployment and stability of the new container before Changes visible at [cloudacademydemo.com](https://cloudacademydemo.com/).

### Release  pipeline (`Release-created.yml`)
Triggerd when a release is published. The workflow will not be run if the release has been targeted in the `main` branch.

The branch is checked out, task is installed, and AWS credentials are configured.
After the secure login to Amazon ECR, the docker image is built and pushed using commands in the Taskfile. The built image is tagged with the `version of the tag` of the published release. The built  pushed to Amazon ECR repository.

### Pull Request Labeler (`PR-labeler.yml`)
Adds labels to pull requests based on rules defined in the `labeler.yml`. Does not run on forks since Github Token has read & write permission. 




##
- AWS credentials  have only least required permissions and are stored securely using Github secrets. 
- Commands were added to the Taskfile to create a single point of entry even all CI actions.
- The `Main Branch pipeline` deploys to Amazon ECS fargate attached to an Application load balancer.  DNS attached with a TLS certificate. Deployed using Terraform.
