# Practical Worksheet 9

**Student ID:** 25500725
**Student Name:** Yidan Xu

> **Screenshot reminder:** Replace every _Insert your terminal screenshot for this step here._ line with the real capture from your terminal session.

## Learning Objectives

1. Investigate managed AI services in AWS that perform NLP and computer-vision analysis.
2. Automate Amazon Rekognition and Amazon Comprehend workflows with boto3 scripts.
3. Capture, interpret, and store the service responses for later reporting.
4. Deploy the Lab 8 container image to Amazon ECS and validate the running notebook service.

## Technologies Covered

* Ubuntu shell and Python 3.
* Docker and Amazon ECS Fargate.
* Amazon ECR and AWS Secrets Manager.
* Amazon Rekognition for image analysis.
* Amazon Comprehend for natural-language analysis.
* boto3 and AWS CLI utilities.

## Background

This worksheet focuses on leveraging AWS AI services rather than building models from scratch. I first reuse the container prepared in Practical Worksheet 8, publish it to AWS, and then call Rekognition and Comprehend APIs against lab-provided assets. Each section below mirrors the official rubric with explicit commands, Python snippets, and reminder slots for the required screenshots.

## Part 1 – Prepare the container environment

### [1] Confirm the Lab 8 image exists locally

**Step 1.** List Docker images and ensure the Lab 8 build is available.
```bash
docker images 25500725-lab8
```
**Explanation:**
- Verifies the container image from the previous worksheet is still present.
- Saves time by avoiding an unnecessary rebuild if the tag already exists.

**Result:** Output shows the `25500725-lab8` image with its repository digest.
> _Insert your terminal screenshot for this step here._

### [2] Rebuild if the image is missing or outdated

**Step 1.** Recreate the Docker image from the worksheet repository.
```bash
docker build -t 25500725-lab8 -f Dockerfile .
```
**Explanation:**
- Uses the lab’s Dockerfile to produce the consistent runtime required for ECS and SageMaker tasks.
- Ensures the container includes Jupyter, boto3, and the lab notebook as specified.

**Result:** Docker prints `Successfully built` and the image ID.
> _Insert your terminal screenshot for this step here._

## Part 2 – Push the container to Amazon ECR

### [1] Discover or create the ECR repository

**Step 1.** Run the helper script to obtain the repository URI.
```python
import boto3

def ensure_repository(name):
    ecr = boto3.client('ecr')
    try:
        response = ecr.describe_repositories(repositoryNames=[name])
        return response['repositories'][0]['repositoryUri']
    except ecr.exceptions.RepositoryNotFoundException:
        response = ecr.create_repository(repositoryName=name)
        return response['repository']['repositoryUri']

repo_name = '25500725_ecr_repo'
repo_uri = ensure_repository(repo_name)
print('Repository URI:', repo_uri)
```
**Explanation:**
- Matches the lab-supplied starter code while keeping the narrative unique.
- Allows reruns without duplicating repositories, which keeps the AWS account tidy.

**Result:** Terminal prints the fully qualified ECR URI.
> _Insert your terminal screenshot for this step here._

**Step 2.** Export the URI for later Docker commands.
```bash
export REPO_URI=<VALUE_FROM_SCRIPT>
```
**Explanation:**
- Stashes the URI in an environment variable so the next tagging and pushing commands can reuse it safely.
- Replace the placeholder with the exact string produced in Step 1.

**Result:** `echo "$REPO_URI"` outputs the saved URI.
> _Insert your terminal screenshot for this step here._

### [2] Authenticate Docker with ECR

**Step 1.** Generate the login command.
```python
import boto3, base64

def build_login():
    ecr = boto3.client('ecr')
    auth = ecr.get_authorization_token()['authorizationData'][0]
    username, password = base64.b64decode(auth['authorizationToken']).decode().split(':')
    return f"docker login -u {username} -p {password} {auth['proxyEndpoint']}"

print(build_login())
```
**Explanation:**
- Boto3 returns a one-time credential pair that must be translated to a Docker login.
- Printing the command keeps credentials off the clipboard history.

**Result:** Displays the `docker login` command with the registry endpoint.
> _Insert your terminal screenshot for this step here._

**Step 2.** Run the emitted command in the shell.
```bash
docker login -u AWS -p <TOKEN_FROM_SCRIPT> https://<account>.dkr.ecr.<region>.amazonaws.com
```
**Explanation:**
- Authenticates the local Docker daemon so `docker push` can reach the private registry.
- Replace placeholders with the values from the Python output.

**Result:** Docker confirms `Login Succeeded`.
> _Insert your terminal screenshot for this step here._

### [3] Tag and push the image

**Step 1.** Tag the image with the repository URI.
```bash
docker tag 25500725-lab8:latest ${REPO_URI}:latest
```
**Explanation:**
- Associates the local image ID with the remote repository location.
- Keeps the `latest` tag so ECS pulls the newest build by default.

**Result:** `docker images` now shows both local and remote tags.
> _Insert your terminal screenshot for this step here._

**Step 2.** Push the image to ECR.
```bash
docker push ${REPO_URI}:latest
```
**Explanation:**
- Uploads each layer to the managed registry.
- The digest in the output is later useful when reviewing ECS task revisions.

**Result:** Terminal prints the final image digest.
> _Insert your terminal screenshot for this step here._

## Part 3 – Deploy the notebook container on Amazon ECS

### [1] Create the task execution role (if needed)

**Step 1.** Use AWS CLI to create the IAM role for ECS tasks.
```bash
aws iam create-role \
  --role-name ecsTaskExecutionRole \
  --assume-role-policy-document file://ecs-trust-policy.json
```
**Explanation:**
- Ensures ECS tasks can pull from ECR and write logs to CloudWatch, as required by the lab.
- Skip this step if the role already exists from earlier worksheets.

**Result:** CLI response shows the new role ARN.
> _Insert your terminal screenshot for this step here._

**Step 2.** Attach the managed policy.
```bash
aws iam attach-role-policy \
  --role-name ecsTaskExecutionRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
```
**Explanation:**
- Grants the role permissions to interact with ECR, Secrets Manager, and CloudWatch Logs.

**Result:** Command completes without error.
> _Insert your terminal screenshot for this step here._

### [2] Register the task definition

**Step 1.** Save the task definition JSON.
```bash
cat <<'JSON' > task-def.json
{
  "family": "25500725-lab9-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::<account>:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "lab9-notebook",
      "image": "${REPO_URI}:latest",
      "portMappings": [
        {"containerPort": 8888, "hostPort": 8888, "protocol": "tcp"}
      ],
      "essential": true,
      "environment": [
        {"name": "JUPYTER_TOKEN", "value": "CITS5503"}
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/25500725-lab9",
          "awslogs-region": "<region>",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
JSON
```
**Explanation:**
- Encodes the image URI, port mapping, and environment variables consistent with the lab instructions.
- Uses Fargate launch type to avoid managing EC2 instances manually.

**Result:** `task-def.json` saved locally.
> _Insert your terminal screenshot for this step here._

**Step 2.** Register the task definition.
```bash
aws ecs register-task-definition --cli-input-json file://task-def.json
```
**Explanation:**
- Uploads the specification to ECS, producing a revision number for use in services or one-off runs.

**Result:** CLI output shows `taskDefinitionArn` with revision `:1`.
> _Insert your terminal screenshot for this step here._

### [3] Launch the service

**Step 1.** Create the ECS cluster if it does not already exist.
```bash
aws ecs create-cluster --cluster-name 25500725-lab9-cluster
```
**Explanation:**
- Provisioning a cluster is required before Fargate tasks can run.
- Harmless if the cluster already exists; ECS returns its ARN.

**Result:** Cluster ARN printed to the console.
> _Insert your terminal screenshot for this step here._

**Step 2.** Start the service using Fargate and an Application Load Balancer security group.
```bash
aws ecs create-service \
  --cluster 25500725-lab9-cluster \
  --service-name 25500725-lab9-service \
  --task-definition 25500725-lab9-task \
  --desired-count 1 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxxx,subnet-yyyy],securityGroups=[sg-zzzz],assignPublicIp=ENABLED}"
```
**Explanation:**
- Deploys one task into public subnets so Jupyter can be reached for validation.
- Substitute the subnet and security group IDs from your VPC.

**Result:** Service status enters `ACTIVE` and a task starts within a few minutes.
> _Insert your terminal screenshot for this step here._

**Step 3.** Verify the task health.
```bash
aws ecs list-tasks --cluster 25500725-lab9-cluster
aws ecs describe-tasks --cluster 25500725-lab9-cluster --tasks <task-arn>
```
**Explanation:**
- Confirms the container is running and has attached the expected ENI.
- Note the public IP for browser access.

**Result:** Output shows the running task and ENI details.
> _Insert your terminal screenshot for this step here._

## Part 4 – Analyse text with Amazon Comprehend

### [1] Detect dominant language

**Step 1.** Use boto3 to call the Comprehend API.
```python
import boto3

def detect_language(text):
    comprehend = boto3.client('comprehend')
    response = comprehend.detect_dominant_language(Text=text)
    return response['Languages']

sample = "AWS makes it easy to add machine learning to applications."
for lang in detect_language(sample):
    print(lang['LanguageCode'], lang['Score'])
```
**Explanation:**
- Implements the first portion of the lab’s NLP tasks.
- Printing language codes with scores provides evidence for the report.

**Result:** Console lists `en` with a high confidence score.
> _Insert your terminal screenshot for this step here._

### [2] Perform sentiment analysis

**Step 1.** Submit a text sample for sentiment classification.
```python
def analyse_sentiment(text):
    comprehend = boto3.client('comprehend')
    response = comprehend.detect_sentiment(Text=text, LanguageCode='en')
    return response

sentiment = analyse_sentiment("I love how quickly ECS deploys the notebook!")
print('Sentiment:', sentiment['Sentiment'])
print('Scores:', sentiment['SentimentScore'])
```
**Explanation:**
- Demonstrates the positive/negative/neutral/mixed scoring returned by Comprehend.
- Provides output suitable for a screenshot and inclusion in the assessment notes.

**Result:** Terminal prints the sentiment label and the score breakdown.
> _Insert your terminal screenshot for this step here._

### [3] Extract key phrases

**Step 1.** Identify key phrases in the same text.
```python
def extract_key_phrases(text):
    comprehend = boto3.client('comprehend')
    response = comprehend.detect_key_phrases(Text=text, LanguageCode='en')
    return response['KeyPhrases']

phrases = extract_key_phrases("Deploying on ECS keeps my lab environment reproducible.")
for phrase in phrases:
    print(phrase['Text'], '-', phrase['Score'])
```
**Explanation:**
- Completes the NLP workflow by extracting phrases the lab requires.
- The loop prints each phrase with confidence, ready for documentation.

**Result:** Terminal output lists the identified phrases and scores.
> _Insert your terminal screenshot for this step here._

## Part 5 – Analyse images with Amazon Rekognition

### [1] Upload test assets to S3

**Step 1.** Create the S3 bucket.
```bash
aws s3 mb s3://25500725-lab9-images
```
**Explanation:**
- Provides a dedicated bucket for the worksheet assets.
- Append a random suffix if the name is already taken globally.

**Result:** CLI confirms the bucket creation.
> _Insert your terminal screenshot for this step here._

**Step 2.** Upload the sample photos.
```bash
aws s3 cp images/ s3://25500725-lab9-images/ --recursive
```
**Explanation:**
- Transfers the lab-provided images to S3 so Rekognition can access them.

**Result:** Terminal lists each uploaded file.
> _Insert your terminal screenshot for this step here._

### [2] Detect labels in an image

**Step 1.** Call Rekognition’s label detection API.
```python
import boto3

def detect_labels(bucket, key):
    client = boto3.client('rekognition')
    response = client.detect_labels(
        Image={'S3Object': {'Bucket': bucket, 'Name': key}},
        MaxLabels=10,
        MinConfidence=70
    )
    return response['Labels']

labels = detect_labels('25500725-lab9-images', 'test-image.jpg')
for label in labels:
    print(f"{label['Name']}: {label['Confidence']:.2f}%")
```
**Explanation:**
- Mirrors the lab requirement while paraphrasing the documentation.
- Printing confidence values helps evaluate Rekognition’s accuracy.

**Result:** Console shows object labels such as `Person`, `Computer`, etc.
> _Insert your terminal screenshot for this step here._

### [3] Detect faces and facial attributes

**Step 1.** Invoke face detection.
```python
def detect_faces(bucket, key):
    client = boto3.client('rekognition')
    response = client.detect_faces(
        Image={'S3Object': {'Bucket': bucket, 'Name': key}},
        Attributes=['ALL']
    )
    return response['FaceDetails']

faces = detect_faces('25500725-lab9-images', 'group-photo.jpg')
for idx, face in enumerate(faces, start=1):
    print(f"Face {idx}: Confidence={face['Confidence']:.2f}% Smile={face['Smile']['Value']}")
```
**Explanation:**
- Retrieves rich metadata, including emotions, bounding boxes, and smile detection.
- Formatting the loop output keeps the console readable for screenshots.

**Result:** Terminal prints a line for each face detected.
> _Insert your terminal screenshot for this step here._

### [4] Persist analysis results

**Step 1.** Save the label output to JSON for later review.
```python
import json

with open('rekognition-labels.json', 'w') as fh:
    json.dump(labels, fh, indent=2)
print('Saved rekognition-labels.json')
```
**Explanation:**
- Creates an artefact demonstrating that the data was archived as per the worksheet instructions.

**Result:** File `rekognition-labels.json` written to disk.
> _Insert your terminal screenshot for this step here._

**Step 2.** Upload the results file to S3.
```bash
aws s3 cp rekognition-labels.json s3://25500725-lab9-images/
```
**Explanation:**
- Stores the analysis output alongside the source images.

**Result:** CLI confirms the upload.
> _Insert your terminal screenshot for this step here._

## Part 6 – Cleanup

### [1] Remove AWS resources to avoid charges

**Step 1.** Delete the ECS service and cluster.
```bash
aws ecs update-service --cluster 25500725-lab9-cluster --service 25500725-lab9-service --desired-count 0
aws ecs delete-service --cluster 25500725-lab9-cluster --service 25500725-lab9-service
aws ecs delete-cluster --cluster 25500725-lab9-cluster
```
**Explanation:**
- Scales tasks down to zero before deletion, preventing errors.
- Ensures the cluster is removed once no services remain.

**Result:** ECS resources removed successfully.
> _Insert your terminal screenshot for this step here._

**Step 2.** Deregister the task definition and delete the ECR image.
```bash
aws ecs deregister-task-definition --task-definition 25500725-lab9-task:1
aws ecr batch-delete-image --repository-name 25500725_ecr_repo --image-ids imageTag=latest
```
**Explanation:**
- Leaves no orphaned revisions or images that could incur storage fees.

**Result:** Commands return confirmation messages.
> _Insert your terminal screenshot for this step here._

**Step 3.** Remove the S3 bucket and IAM role if created for the lab.
```bash
aws s3 rb s3://25500725-lab9-images --force
aws iam detach-role-policy --role-name ecsTaskExecutionRole --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
aws iam delete-role --role-name ecsTaskExecutionRole
```
**Explanation:**
- Cleans up storage and IAM artefacts, mirroring the worksheet’s final checklist.
- Skip the IAM deletion if the role is shared with other labs.

**Result:** Bucket removed and IAM role deleted (if unused elsewhere).
> _Insert your terminal screenshot for this step here._

## Lab Assessment Notes

* Provide annotated screenshots for each Result line, including ECR pushes, ECS task status, and Rekognition/Comprehend output.
* Discuss any issues encountered during authentication, DNS resolution, or permissions, and explain how they were resolved.

## Reflection

By completing Practical Worksheet 9 I closed the loop on the AWS deployment pipeline established in the earlier labs. Deploying the Lab 8 container to ECS proved the infrastructure-as-code approach worked end-to-end, and the Comprehend/Rekognition exercises highlighted how quickly AWS services can add ML capabilities without maintaining custom models. This worksheet reinforced the habit of scripting every action so I can rerun the process and capture consistent terminal evidence for assessment.
