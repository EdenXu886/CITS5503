# Practical Worksheet 8

**Student ID:** 25500725  
**Student Name:** Yidan Xu

> **Screenshot reminder:** Replace every _Insert your terminal screenshot for this step here._ line with the real capture from your terminal session.

## Learning Objectives

1. Use Amazon ECR and Amazon ECS together with AWS Secrets Manager to host a managed Jupyter workload.
2. Produce a Docker image that contains the lab’s notebook, AWS SDK tooling, and a ready-to-run Jupyter environment.
3. Launch a SageMaker hyperparameter tuning job from that containerised environment.
4. Ship the container to ECR and orchestrate it on ECS so the notebook remains accessible on demand.

## Technologies Covered

* AWS CLI and Boto3 for scripted AWS interactions.
* Amazon ECR for storing and versioning container images.
* Amazon ECS on Fargate for serverless container execution.
* AWS Secrets Manager for securely injecting credentials.
* Amazon SageMaker for running tuning jobs.
* Docker, Python, and Jupyter Notebook tooling.

## Background

The worksheet walks through preparing a container workspace locally, publishing it to AWS, and then connecting to that running container to execute SageMaker tuning tasks. Every section below mirrors the lab rubric while expanding the shell commands and boto3 snippets I executed.

## Create a Dockerfile and build a Docker image

### [1] Draft the Dockerfile

**Step 1.** Create the Dockerfile described in the lab brief.
```bash
echo 'FROM python:3.10

RUN pip install jupyter boto3 sagemaker awscli
RUN mkdir /notebook

ENV JUPYTER_ENABLE_LAB=yes
ENV JUPYTER_TOKEN="CITS5503"

RUN jupyter notebook --generate-config
RUN echo "c.NotebookApp.ip = '0.0.0.0'" >> /root/.jupyter/jupyter_notebook_config.py

RUN wget -P /notebook https://raw.githubusercontent.com/zhangzhics/CITS5503_Sem2/master/Labs/src/LabAI.ipynb

WORKDIR /notebook
EXPOSE 8888
CMD ["jupyter", "notebook", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root"]' > Dockerfile
```
**Explanation:**
- Starts from the official Python 3.10 base so the image has a predictable runtime.
- Installs the required packages (`jupyter`, `boto3`, `sagemaker`, `awscli`) inside the image.
- Preloads the notebook referenced by the lab using `wget` to keep the container self-contained.
- Configures Jupyter to listen on all interfaces and exposes port `8888` for later port forwarding.
- Stores the lab token in `JUPYTER_TOKEN`, letting me authenticate to the notebook without editing config files manually.

**Result:** Dockerfile saved in the current working directory.
> _Insert your terminal screenshot for this step here._

### [2] Build the container image

**Step 1.** Build the image with the student-specific tag.
```bash
docker build -t 25500725-lab8 .
```
**Explanation:**
- `docker build` reads the Dockerfile and produces a local image snapshot.
- Tagging with `25500725-lab8` keeps the image identifiable during subsequent pushes to ECR.

**Result:** Image `25500725-lab8:latest` appears in the local Docker cache.
> _Insert your terminal screenshot for this step here._

### [3] Smoke-test the notebook locally

**Step 1.** Start the container and publish the notebook port.
```bash
docker run -p 8888:8888 25500725-lab8
```
**Explanation:**
- Runs the image locally and forwards host port `8888` to the container so the Jupyter UI is reachable.
- Confirms the notebook loads before investing time in AWS deployment.

**Result:** The terminal prints Jupyter startup logs including the access URL with the `CITS5503` token.
> _Insert your terminal screenshot for this step here._

**Step 2.** Visit the notebook in a browser to verify the lab notebook downloaded successfully.
```text
URL: http://127.0.0.1:8888/?token=CITS5503
```
**Explanation:**
- Opening the URL validates that the container bundles the notebook and accepts the configured token.
- The rendered notebook confirms the dataset exploration cells are available for later SageMaker work.

**Result:** Browser displays `LabAI.ipynb` within Jupyter Lab.
> _Insert your terminal screenshot for this step here._

## Prepare ECR via Boto3 scripts on your local machine

### [1] Discover or create the ECR repository

**Step 1.** Run the helper script to create the repository if it does not exist.
```python
import boto3

def ensure_repository(repo_name):
    ecr = boto3.client('ecr')
    try:
        details = ecr.describe_repositories(repositoryNames=[repo_name])
        return details['repositories'][0]['repositoryUri']
    except ecr.exceptions.RepositoryNotFoundException:
        response = ecr.create_repository(repositoryName=repo_name)
        return response['repository']['repositoryUri']

repository_name = '25500725_ecr_repo'
uri = ensure_repository(repository_name)
print('ECR URI:', uri)
```
**Explanation:**
- Uses boto3 to call ECR APIs so the whole procedure stays in code, matching the worksheet specification.
- Gracefully handles the repository already existing to avoid duplicate resources.

**Result:** Terminal prints the repository URI, e.g. `123456789012.dkr.ecr.ap-southeast-2.amazonaws.com/25500725_ecr_repo`.
> _Insert your terminal screenshot for this step here._

### [2] Generate the Docker login command

**Step 1.** Request an authorization token from ECR and convert it to a Docker login command.
```python
import boto3
import base64

def build_login_command():
    ecr = boto3.client('ecr')
    token = ecr.get_authorization_token()
    auth_data = token['authorizationData'][0]
    user, password = base64.b64decode(auth_data['authorizationToken']).decode().split(':')
    return f"docker login -u {user} -p {password} {auth_data['proxyEndpoint']}"

print(build_login_command())
```
**Explanation:**
- Retrieves the one-time credentials necessary for Docker to push into the private registry.
- Decodes the Base64 string and formats the command that must be run in a shell session.

**Result:** Shell displays the `docker login` command string.
> _Insert your terminal screenshot for this step here._

**Step 2.** Execute the emitted login command to authenticate Docker with ECR.
```bash
docker login -u AWS -p <ECR_PASSWORD> https://123456789012.dkr.ecr.ap-southeast-2.amazonaws.com
```
**Explanation:**
- Substitutes the password from the script output and logs Docker in to the registry endpoint.
- Required before any `docker push` succeeds.

**Result:** Docker reports `Login Succeeded`.
> _Insert your terminal screenshot for this step here._

**Step 3.** (WSL 2 only) Patch DNS if AWS endpoints cannot resolve.
```bash
sudo rm /etc/resolv.conf
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
sudo chattr +i /etc/resolv.conf
```
**Explanation:**
- Overrides the default WSL resolver configuration when encountering `no such host` errors.
- Locks the file to prevent Windows from reverting the DNS server during the session.

**Result:** Subsequent ECR commands resolve without DNS failures.
> _Insert your terminal screenshot for this step here._

## Push a local Docker image onto ECR

### [1] Tag the image with the remote repository URI

**Step 1.** Apply the tag required for the push.
```bash
docker tag 25500725-lab8:latest 123456789012.dkr.ecr.ap-southeast-2.amazonaws.com/25500725_ecr_repo:latest
```
**Explanation:**
- Associates the local image with the ECR repository URI returned earlier.
- Leaving the `:latest` tag allows ECS task definitions to pull the newest build automatically.

**Result:** `docker images` now shows both the original and ECR-qualified tags.
> _Insert your terminal screenshot for this step here._

### [2] Push the image to ECR

**Step 1.** Upload the tagged image.
```bash
docker push 123456789012.dkr.ecr.ap-southeast-2.amazonaws.com/25500725_ecr_repo:latest
```
**Explanation:**
- Transfers each layer to the managed registry so the image can be consumed by ECS tasks.
- Progress output confirms the upload and identifies skipped layers if they already exist.

**Result:** Push completes with a digest such as `sha256:...`.
> _Insert your terminal screenshot for this step here._

## Deploy your Docker image onto ECS

### [1] Register the task definition

**Step 1.** Run the boto3 helper to create or update the ECS task definition.
```python
import boto3

def register_task(image_uri, student_id, account_id, task_role, execution_role, environment=None):
    ecs = boto3.client('ecs')
    container_def = {
        'name': f'{student_id}-lab8-container',
        'image': image_uri,
        'essential': True,
        'portMappings': [{'containerPort': 8888, 'protocol': 'tcp'}],
    }
    if environment:
        container_def['environment'] = [
            {'name': key, 'value': value} for key, value in environment.items()
        ]
    return ecs.register_task_definition(
        family=f'{student_id}-lab8-task',
        cpu='256',
        memory='512',
        networkMode='awsvpc',
        requiresCompatibilities=['FARGATE'],
        executionRoleArn=f'arn:aws:iam::{account_id}:role/{execution_role}',
        taskRoleArn=f'arn:aws:iam::{account_id}:role/{task_role}',
        containerDefinitions=[container_def],
    )

response = register_task(
    image_uri='123456789012.dkr.ecr.ap-southeast-2.amazonaws.com/25500725_ecr_repo:latest',
    student_id='25500725',
    account_id='123456789012',
    task_role='Lab8TaskRole',
    execution_role='Lab8ExecutionRole',
    environment={'AWS_DEFAULT_REGION': 'ap-southeast-2'}
)
print('Task definition ARN:', response['taskDefinition']['taskDefinitionArn'])
```
**Explanation:**
- Chooses `awsvpc` networking and Fargate compatibility per the worksheet.
- Injects optional environment variables (e.g., AWS region) without hard-coding secrets in the image.
- Returns the ARN used when launching ECS tasks.

**Result:** Terminal displays the registered task definition ARN.
> _Insert your terminal screenshot for this step here._

### [2] Launch the service or standalone task

**Step 1.** Create the ECS cluster if it does not already exist.
```bash
aws ecs create-cluster --cluster-name 25500725-lab8-cluster
```
**Explanation:**
- Establishes a logical group of capacity for Fargate tasks.
- Idempotent operation—returns existing cluster details when it already exists.

**Result:** Cluster ARN printed to the terminal.
> _Insert your terminal screenshot for this step here._

**Step 2.** Start a Fargate task using the registered definition.
```bash
aws ecs run-task \
  --cluster 25500725-lab8-cluster \
  --launch-type FARGATE \
  --task-definition 25500725-lab8-task \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-abc123],securityGroups=[sg-abc123],assignPublicIp=ENABLED}"
```
**Explanation:**
- Specifies the subnet and security group to allow outbound internet and inbound notebook access.
- Enables a public IP so the notebook endpoint can be accessed without a load balancer.

**Result:** Task transitions to `PENDING` then `RUNNING` status with an assigned public IP in the response payload.
> _Insert your terminal screenshot for this step here._

### [3] Retrieve the notebook endpoint and connect

**Step 1.** Describe the running task to obtain the public IP.
```bash
aws ecs describe-tasks --cluster 25500725-lab8-cluster --tasks <TASK_ARN> \
  --query 'tasks[0].attachments[0].details[?name==`publicIPv4Address`].value' --output text
```
**Explanation:**
- Filters the response down to the network interface detail containing the public IPv4 address.
- `--output text` prints only the address, simplifying copy-paste for the browser.

**Result:** Command outputs an IPv4 address such as `54.206.10.45`.
> _Insert your terminal screenshot for this step here._

**Step 2.** Open the notebook through the ECS task.
```text
URL: http://54.206.10.45:8888/?token=CITS5503
```
**Explanation:**
- Uses the same token configured inside the Dockerfile to authenticate.
- Confirms the ECS orchestration works the same as the local container run.

**Result:** Notebook launches in the browser, ready for SageMaker steps.
> _Insert your terminal screenshot for this step here._

## Use SageMaker from the container

### [1] Configure AWS credentials within the notebook

**Step 1.** Store credentials in AWS Secrets Manager and mount them as environment variables.
```python
import boto3

def fetch_secret(secret_name):
    client = boto3.client('secretsmanager')
    payload = client.get_secret_value(SecretId=secret_name)
    return payload['SecretString']

credentials = fetch_secret('25500725/sagemaker/credentials')
print('Secret retrieved')
```
**Explanation:**
- Reads the lab-provided secret that contains the AWS access key and secret key.
- Keeps sensitive data out of the notebook source while enabling boto3 to authenticate.

**Result:** Notebook output confirms the secret was fetched (without printing the actual keys).
> _Insert your terminal screenshot for this step here._

**Step 2.** Persist the credentials to the default AWS config files within the container session.
```python
import json
import os

creds = json.loads(credentials)
os.makedirs(os.path.expanduser('~/.aws'), exist_ok=True)
with open(os.path.expanduser('~/.aws/credentials'), 'w') as fh:
    fh.write('[default]\n')
    fh.write(f"aws_access_key_id={creds['aws_access_key_id']}\n")
    fh.write(f"aws_secret_access_key={creds['aws_secret_access_key']}\n")
```
**Explanation:**
- Parses the JSON secret and writes it into the credentials file expected by boto3 and the SageMaker SDK.
- Ensures subsequent API calls run without embedding keys in every script cell.

**Result:** `~/.aws/credentials` populated inside the running container.
> _Insert your terminal screenshot for this step here._

### [2] Launch the hyperparameter tuning job

**Step 1.** Define the training image and estimator in the notebook.
```python
import sagemaker
from sagemaker.estimator import Estimator

session = sagemaker.Session()
role = 'arn:aws:iam::123456789012:role/SageMakerExecutionRole'
training_image = sagemaker.image_uris.retrieve('linear-learner', session.boto_region_name)

estimator = Estimator(
    image_uri=training_image,
    role=role,
    instance_count=1,
    instance_type='ml.m4.xlarge',
    volume_size=5,
    max_run=3600,
    output_path=f's3://25500725-sagemaker-output/'
)
```
**Explanation:**
- Pulls the built-in Linear Learner algorithm image that SageMaker hosts.
- Configures compute size and output bucket following the lab’s recommended defaults.

**Result:** Estimator object instantiated without errors.
> _Insert your terminal screenshot for this step here._

**Step 2.** Configure the hyperparameter tuning job.
```python
from sagemaker.tuner import HyperparameterTuner, ContinuousParameter

tuner = HyperparameterTuner(
    estimator,
    objective_metric_name='validation:rmse',
    hyperparameter_ranges={'wd': ContinuousParameter(1e-6, 1e-3)},
    metric_definitions=[{'Name': 'validation:rmse', 'Regex': 'validation:rmse=(.*?);'}],
    max_jobs=4,
    max_parallel_jobs=2,
    strategy='Bayesian'
)
```
**Explanation:**
- Sets up a Bayesian tuning strategy with search bounds matching the worksheet guidance.
- Registers a regex metric extractor so the tuner can parse the training logs.

**Result:** Tuner object ready to launch the job.
> _Insert your terminal screenshot for this step here._

**Step 3.** Start the tuning run on SageMaker.
```python
tuner.fit({'train': 's3://25500725-sagemaker-datasets/train/',
           'validation': 's3://25500725-sagemaker-datasets/validation/'})
```
**Explanation:**
- Points SageMaker to the training and validation datasets hosted in S3.
- Submits the tuning job that will iterate through the defined hyperparameter space.

**Result:** Notebook prints the tuning job ARN and transitions into `InProgress` state.
> _Insert your terminal screenshot for this step here._

### [3] Monitor and capture results

**Step 1.** Poll the tuning job until it completes.
```python
import time

sm_client = boto3.client('sagemaker')

def wait_for_completion(job_name):
    while True:
        job = sm_client.describe_hyper_parameter_tuning_job(HyperParameterTuningJobName=job_name)
        status = job['HyperParameterTuningJobStatus']
        print('Status:', status)
        if status in ('Completed', 'Failed', 'Stopped'):
            return job
        time.sleep(60)

summary = wait_for_completion(tuner.latest_tuning_job.name)
```
**Explanation:**
- Uses the SageMaker API to monitor job progress and exits when a terminal state is reached.
- Prints intermediate status updates which are helpful for screenshots.

**Result:** Output shows status transitions ending in `Completed` with the best training job name.
> _Insert your terminal screenshot for this step here._

**Step 2.** Retrieve the top-performing hyperparameters.
```python
best_job = summary['BestTrainingJob']['TrainingJobName']
best = sm_client.describe_training_job(TrainingJobName=best_job)
print('Best job metrics:', best['FinalMetricDataList'])
print('Best job hyperparameters:', best['HyperParameters'])
```
**Explanation:**
- Inspects the winning training job to document metric scores and the tuned weight decay value.
- Provides evidence for the worksheet submission’s analysis section.

**Result:** Notebook prints the metric dictionary and chosen hyperparameters.
> _Insert your terminal screenshot for this step here._

## Clean up resources

### [1] Stop the ECS workload

**Step 1.** Stop the running ECS task.
```bash
aws ecs stop-task --cluster 25500725-lab8-cluster --task <TASK_ARN>
```
**Explanation:**
- Shuts down the Fargate task to avoid additional charges once screenshots are collected.

**Result:** Task status changes to `STOPPED`.
> _Insert your terminal screenshot for this step here._

### [2] Remove AWS artefacts

**Step 1.** Deregister old task definition revisions.
```bash
aws ecs deregister-task-definition --task-definition 25500725-lab8-task:1
```
**Explanation:**
- Keeps the account tidy by removing unused revisions that ECS no longer needs.

**Result:** Command returns a JSON confirmation payload.
> _Insert your terminal screenshot for this step here._

**Step 2.** Delete the ECR repository when no longer required.
```bash
aws ecr delete-repository --repository-name 25500725_ecr_repo --force
```
**Explanation:**
- `--force` removes all image layers before deleting the repository to avoid retention costs.

**Result:** Repository deletion acknowledged by the CLI.
> _Insert your terminal screenshot for this step here._

**Step 3.** Tear down temporary IAM roles or secrets if they were created only for this lab.
```bash
aws iam delete-role --role-name Lab8ExecutionRole
aws iam delete-role --role-name Lab8TaskRole
aws secretsmanager delete-secret --secret-id 25500725/sagemaker/credentials --force-delete-without-recovery
```
**Explanation:**
- Prevents unused roles from lingering with permissions in the AWS account.
- Removes the temporary secret so credentials are not left lying around.

**Result:** Each command returns confirmation of the deletion.
> _Insert your terminal screenshot for this step here._

## Reflection

- Containerising the notebook forced me to manage dependencies declaratively, making the ECS launch deterministic.
- Publishing to ECR and reusing the URI across boto3 scripts ensured ECS pulled the exact image I validated locally.
- Automating the SageMaker tuning workflow via notebooks highlighted how Secrets Manager and IAM roles work together to secure credentials.
- The cleanup steps were essential to avoid ongoing resource costs after capturing the required evidence for the worksheet.
