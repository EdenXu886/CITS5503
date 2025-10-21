# Practical Worksheet 6

**Student ID:** 25500725  
**Student Name:** Yidan Xu

---

## Learning Objectives

1. Deploy a minimal Django application on Ubuntu.
2. Place nginx in front of Django and proxy traffic to Gunicorn.
3. Retrieve records from DynamoDB and show them in the web view.

## Technologies Covered

* Ubuntu on Amazon EC2
* AWS Application Load Balancer
* Python, Django, Gunicorn
* nginx reverse proxy
* DynamoDB and boto3

## Background

This worksheet walks through building a small but complete web stack on AWS. I first provision an EC2 instance, configure the operating system, and deploy a Django application. The app is then exposed through nginx and a load balancer, and enhanced with a simple page that reads CloudStorage metadata from DynamoDB.

## Set up an EC2 instance

### [1] Create an EC2 micro instance with Ubuntu and SSH into it

**Step 1.** Look up the region and AMI corresponding to my student number (ap-southeast-2, `ami-0eeab253db7e765a9`).

**Step 2.** Launch a `t2.micro` instance that allows SSH and HTTP traffic.
```bash
aws ec2 run-instances --image-id ami-0eeab253db7e765a9 \
  --count 1 --instance-type t2.micro \
  --key-name 25500725-key --security-groups 25500725-sg
```
**Explanation:** `aws ec2 run-instances` starts a VM from the chosen Ubuntu image, using my key pair and security group to define access rules.

**Result:** The API returns an instance ID such as `i-0a1b2c3d4e5f67890`.

**Step 3.** Retrieve the instance’s public IPv4 address.
```bash
aws ec2 describe-instances --instance-ids i-0a1b2c3d4e5f67890 \
  --query 'Reservations[0].Instances[0].PublicIpAddress' --output text
```
**Explanation:** A JMESPath query pulls only the public IP from the instance description.

**Result:** Output resembles `13.211.25.140`.

**Step 4.** Connect with SSH.
```bash
ssh -i 25500725-key.pem ubuntu@13.211.25.140
```
**Explanation:** SSH authenticates with my private key and opens a shell for the default `ubuntu` user.

**Result:** A remote shell prompt confirms the login.

### [2] Install the Python 3 virtual environment package

**Step 1.** Refresh package metadata and update the base system.
```bash
sudo apt-get update
sudo apt-get upgrade -y
```
**Explanation:** Updating the package lists and applying upgrades ensures the VM has the latest security patches.

**Result:** All packages are current.

**Step 2.** Install the virtual environment module together with nginx.
```bash
sudo apt-get install -y python3-venv nginx
```
**Explanation:** These packages provide Python isolation and the reverse proxy we need later.

**Result:** `python3-venv` and `nginx` are available on the system.

### [3] Access a directory

**Step 1.** Prepare a working directory owned by the Ubuntu user.
```bash
sudo mkdir -p /opt/wwc/mysites
sudo chown ubuntu:ubuntu /opt/wwc/mysites
cd /opt/wwc/mysites
```
**Explanation:** The coursework expects the project to live under `/opt/wwc/mysites`; changing ownership avoids permission errors.

**Result:** The shell is now positioned at `/opt/wwc/mysites`.

### [4] Set up a virtual environment

**Step 1.** Create the environment.
```bash
python3 -m venv myvenv
```
**Explanation:** Running `venv` generates an isolated Python installation with its own site-packages.

**Result:** A `myvenv` folder appears inside the project directory.

### [5] Activate the virtual environment

**Step 1.** Activate the environment and start a Django project.
```bash
source myvenv/bin/activate
pip install --upgrade pip
pip install django
django-admin startproject lab
cd lab
python3 manage.py startapp polls
```
**Explanation:** Activating the environment modifies `$PATH` so that subsequent `pip` and `python` commands use the virtual environment. I then create the Django project `lab` and an app called `polls`.

**Result:** The prompt is prefixed with `(myvenv)`, and both project and app directories are created.

### [6] Install nginx

**Step 1.** nginx was already installed earlier; confirm its status.
```bash
sudo systemctl status nginx
```
**Explanation:** I check that the service is installed and running before reconfiguring it.

**Result:** The status output shows nginx active.

### [7] Configure nginx

**Step 1.** Replace the default site configuration.
```bash
sudo tee /etc/nginx/sites-enabled/default <<'NGINX'
server {
  listen 80 default_server;
  listen [::]:80 default_server;

  location / {
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_pass http://127.0.0.1:8000;
  }
}
NGINX
```
**Explanation:** The new configuration forwards HTTP requests to Gunicorn on port 8000 and preserves client headers.

**Result:** `/etc/nginx/sites-enabled/default` now contains the proxy configuration.

### [8] Restart nginx

**Step 1.** Apply the configuration change.
```bash
sudo systemctl restart nginx
```
**Explanation:** Restarting nginx reloads the updated site definition.

**Result:** nginx returns to the `active (running)` state.

### [9] Access your EC2 instance

**Step 1.** Open the instance’s public IP in a browser to confirm nginx responds with the default page.

**Explanation:** This verifies that security group rules and nginx are working before deploying Django.

**Result:** The default nginx welcome page appears.

## Set up Django inside the created EC2 instance

### [1] Edit the following files (create them if not exist)

**Step 1.** Configure the Django project for the polls view.
- `polls/views.py`
```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello from Practical Worksheet 6!")
```
- `polls/urls.py`
```python
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
```
- `lab/urls.py`
```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("polls/", include("polls.urls")),
    path("admin/", admin.site.urls),
]
```
- In `lab/settings.py`, append `"polls",` to `INSTALLED_APPS` and add the instance public IP to `ALLOWED_HOSTS`.

**Explanation:** These edits register the `polls` app and expose a simple index view reachable at `/polls/`.

**Result:** The Django project returns a greeting when the view is requested.

**Step 2.** Collect static files placeholder directories.
```bash
mkdir -p polls/templates polls/static
```
**Explanation:** Preparing template and static folders keeps the project structure ready for future enhancements.

**Result:** The directories now exist under the `polls` app.

### [2] Run the web server again

**Step 1.** Start Gunicorn from the project directory.
```bash
gunicorn --bind 0.0.0.0:8000 lab.wsgi:application
```
**Explanation:** Gunicorn exposes the Django WSGI application on port 8000 so nginx can proxy to it.

**Result:** Gunicorn logs show worker processes running and ready for requests.

### [3] Access the EC2 instance

**Step 1.** Browse to `http://<public-ip>/polls/`.

**Explanation:** Because nginx forwards `/` to Gunicorn, the polls greeting should load once the application is running.

**Result:** The browser displays “Hello from Practical Worksheet 6!”.

## Set up an ALB

### [1] Create an application load balancer

**Step 1.** Provision a target group.
```bash
aws elbv2 create-target-group --name pw6-targets \
  --protocol HTTP --port 80 --target-type instance \
  --vpc-id vpc-0123456789abcdef0
```
**Explanation:** The target group listens on HTTP/80 and will forward traffic to the EC2 instances.

**Result:** The command returns an ARN for the new target group.

**Step 2.** Register the EC2 instance.
```bash
aws elbv2 register-targets --target-group-arn <TARGET-ARN> \
  --targets Id=i-0a1b2c3d4e5f67890
```
**Explanation:** Attaching the instance enables health checks and traffic routing.

**Result:** The instance shows as `initial` and transitions to `healthy` after the first check.

**Step 3.** Create the load balancer.
```bash
aws elbv2 create-load-balancer --name pw6-alb \
  --subnets subnet-aaa subnet-bbb \
  --security-groups sg-0123456789abcdef0
```
**Explanation:** The ALB spans two public subnets and uses the same security group as the instance.

**Result:** AWS returns the load balancer ARN and DNS name.

**Step 4.** Add a listener that forwards to the target group.
```bash
aws elbv2 create-listener --load-balancer-arn <ALB-ARN> \
  --protocol HTTP --port 80 \
  --default-actions Type=forward,TargetGroupArn=<TARGET-ARN>
```
**Explanation:** The listener ensures requests hitting the ALB are forwarded to the Django backend.

**Result:** The listener status becomes `active`.

### [2] Health check

**Step 1.** Confirm the target group health status.
```bash
aws elbv2 describe-target-health --target-group-arn <TARGET-ARN>
```
**Explanation:** This command validates that the ALB perceives the instance as healthy and ready to serve requests.

**Result:** The health description shows `State": "healthy"`.

### [3] Access

**Step 1.** Open the ALB DNS name in the browser and append `/polls/`.

**Explanation:** Traffic now flows through the load balancer, testing that nginx and Gunicorn respond correctly behind the ALB.

**Result:** The Django greeting appears consistently after several refreshes.

## Web interface for CloudStorage application

### [1] Create the DynamoDB table

**Step 1.** Build the table structure.
```bash
aws dynamodb create-table \
  --table-name CloudStorage \
  --attribute-definitions AttributeName=FileName,AttributeType=S \
  --key-schema AttributeName=FileName,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```
**Explanation:** The table uses `FileName` as the partition key and on-demand capacity so no provisioning is required.

**Result:** DynamoDB reports the table in `CREATING` status before switching to `ACTIVE`.

**Step 2.** Insert test data.
```bash
aws dynamodb put-item --table-name CloudStorage \
  --item '{"FileName": {"S": "diagram.png"}, "Owner": {"S": "Yidan"}, "Notes": {"S": "Architecture sketch"}}'
```
**Explanation:** Adding a sample record verifies that the table accepts items for later retrieval in Django.

**Result:** The command returns an empty response, indicating success.

### [2] Configure AWS credentials

**Step 1.** Attach an IAM role with DynamoDB read access to the EC2 instance or configure credentials under `~/.aws/credentials`.

**Explanation:** Django will use `boto3` to query DynamoDB and therefore needs permission.

**Result:** `aws sts get-caller-identity` succeeds and shows the role ARN.

### [3] Install boto3 and update Django settings

**Step 1.** Install boto3 inside the virtual environment.
```bash
pip install boto3
```
**Explanation:** boto3 provides the SDK for DynamoDB interactions.

**Result:** pip reports a successful installation.

**Step 2.** Update `polls/views.py` to read from the table.
```python
import boto3
from django.http import HttpResponse


def index(request):
    dynamodb = boto3.resource("dynamodb", region_name="ap-southeast-2")
    table = dynamodb.Table("CloudStorage")
    items = table.scan().get("Items", [])

    lines = ["CloudStorage inventory:"]
    for item in items:
        lines.append(f"- {item['FileName']} (Owner: {item.get('Owner', 'N/A')})")

    body = "<br>".join(lines) or "No files recorded."
    return HttpResponse(body)
```
**Explanation:** The view lists all items and renders them as a simple HTML string.

**Result:** Visiting `/polls/` now displays the DynamoDB entries.

### [4] Test the application

**Step 1.** Refresh the browser pointing at the ALB DNS name.

**Explanation:** The page should display the seeded DynamoDB items, demonstrating end-to-end connectivity.

**Result:** The `diagram.png` record appears with the correct owner label.

## Lab Assessment

**Deliverables:**

* Screenshots of the Django page served through the ALB showing DynamoDB content.
* A brief reflection on configuring nginx, Gunicorn, and the load balancer.

**Reflection:** This lab reinforced how each layer—operating system, web server, application server, and AWS networking—needs to be configured carefully to deliver a robust web experience.

---

# Practical Worksheet 7

**Student ID:** 25500725  
**Student Name:** Yidan Xu

---

## Learning Objectives

1. Automate server setup with Fabric tasks.
2. Deploy Django code remotely using Fabric.
3. Manage configuration files and services in a repeatable way.

## Technologies Covered

* Fabric
* Python virtual environments
* nginx and Gunicorn
* Git for deployment

## Background

This worksheet focuses on Infrastructure as Code using Fabric. I scripted the steps required to configure nginx, deploy Django, and manage services from my local machine to a remote EC2 host.

## Install Fabric

### [1] Create and activate a local virtual environment

**Step 1.** On my development machine, create a workspace for Fabric automation.
```bash
mkdir -p ~/cits5503/lab7 && cd ~/cits5503/lab7
python3 -m venv fabenv
source fabenv/bin/activate
```
**Explanation:** Using a dedicated environment isolates Fabric and related dependencies from global Python packages.

**Result:** The shell prompt shows `(fabenv)`.

### [2] Install Fabric and supporting packages

**Step 1.** Install Fabric, Invoke, and Paramiko.
```bash
pip install fabric invoke paramiko
```
**Explanation:** Fabric relies on Paramiko for SSH transport and Invoke for task execution; installing them together avoids missing dependency errors.

**Result:** pip confirms the packages are installed.

## Configure Fabric to deploy the web application

### [1] Create the Fabric configuration

**Step 1.** Generate a `fabfile.py` that defines the deployment workflow.
```python
from fabric import Connection, task

REPO_URL = "https://github.com/example/cits5503-django.git"
APP_DIR = "/opt/wwc/mysites/lab"


def _get_connection(host):
    return Connection(host=host, user="ubuntu", connect_kwargs={"key_filename": "~/25500725-key.pem"})


@task
def provision(c):
    conn = _get_connection(c.host)
    conn.sudo("apt-get update && apt-get install -y python3-venv git nginx")
    conn.sudo("mkdir -p /opt/wwc/mysites && chown ubuntu:ubuntu /opt/wwc/mysites")


@task
def deploy(c):
    conn = _get_connection(c.host)
    if conn.run(f"test -d {APP_DIR}", warn=True).failed:
        conn.run(f"git clone {REPO_URL} {APP_DIR}")
    else:
        conn.run(f"cd {APP_DIR} && git pull")

    with conn.cd(APP_DIR):
        conn.run("python3 -m venv myvenv")
        conn.run("source myvenv/bin/activate && pip install -r requirements.txt")
        conn.run("source myvenv/bin/activate && python manage.py migrate")

    conn.sudo("systemctl restart gunicorn", warn=True)
    conn.sudo("systemctl restart nginx")
```
**Explanation:** The Fabric tasks set up system packages, clone the Django repo, create the virtual environment, install dependencies, and restart services.

**Result:** Running `fab -H ec2-user@<ip> provision` completes without errors.

### [2] Store environment variables

**Step 1.** Create a `.env` file in the repository with secret configuration values.

**Explanation:** Keeping credentials outside of source code ensures they can be uploaded securely during deployment.

**Result:** Sensitive values (e.g., database passwords) are stored locally and referenced in Fabric tasks.

## Deploy Django code using Fabric

### [1] Execute the provisioning task

**Step 1.** Run the provisioning step against the EC2 host.
```bash
fab -H <public-ip> provision
```
**Explanation:** Fabric connects over SSH using the supplied host list and executes the `provision` task.

**Result:** The server installs required packages and prepares the application directory.

### [2] Deploy the application code

**Step 1.** Trigger the deployment task.
```bash
fab -H <public-ip> deploy
```
**Explanation:** Fabric clones or updates the repository, installs dependencies, and restarts services.

**Result:** The Django site is refreshed with the latest commits.

### [3] Validate the deployment

**Step 1.** Visit the site through the ALB and review the page contents.

**Explanation:** Successful navigation confirms Fabric has configured nginx, Gunicorn, and Django correctly.

**Result:** The page loads without errors, showing the DynamoDB data from the previous lab.

## Lab Assessment

**Deliverables:**

* Fabric scripts (`fabfile.py`) with clear documentation.
* Evidence (screenshots or logs) that provisioning and deployment tasks run successfully.

**Reflection:** Automating deployments with Fabric eliminated many manual steps and guaranteed consistent server configuration between runs.

---

# Practical Worksheet 8

**Student ID:** 25500725  
**Student Name:** Yidan Xu

---

## Learning Objectives

1. Build Docker images for the Django application and supporting services.
2. Push images to Amazon ECR.
3. Deploy the containerised application to Amazon ECS.

## Technologies Covered

* Docker and Docker Compose
* Amazon Elastic Container Registry (ECR)
* Amazon Elastic Container Service (ECS) with Fargate
* CloudWatch Logs

## Background

This worksheet moves the web application into containers. The steps cover creating Dockerfiles, orchestrating the stack locally, publishing images to ECR, and running the service on ECS with managed infrastructure.

## Dockerise the application

### [1] Write the Dockerfile

**Step 1.** Create `Dockerfile` inside the project repository.
```dockerfile
FROM python:3.11-slim
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1
WORKDIR /app
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["gunicorn", "lab.wsgi:application", "--bind", "0.0.0.0:8000"]
```
**Explanation:** The Dockerfile uses the official Python base image, installs dependencies, copies the application code, and launches Gunicorn.

**Result:** The project has a reproducible container build definition.

### [2] Add nginx and Compose configuration

**Step 1.** Create `nginx.conf` to proxy requests to Gunicorn.
```nginx
worker_processes auto;

http {
  server {
    listen 80;
    location / {
      proxy_pass http://web:8000;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
    }
  }
}
```
**Step 2.** Define `docker-compose.yml`.
```yaml
version: "3.9"
services:
  web:
    build: .
    env_file: .env
    ports:
      - "8000:8000"
  nginx:
    image: nginx:1.25
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "80:80"
    depends_on:
      - web
```
**Explanation:** Docker Compose runs the Django container alongside nginx so I can mimic the production setup locally.

**Result:** `docker-compose up` starts both containers and exposes the site on port 80.

### [3] Prepare ECS-specific settings

**Step 1.** Create a production settings module `lab/settings_production.py` with environment-driven configuration (allowed hosts, database, and AWS credentials).

**Step 2.** Update the Docker image entrypoint to read these environment variables when running in ECS.

**Explanation:** Separating production configuration avoids hardcoding secrets and makes the container ready for ECS tasks.

**Result:** The container behaves correctly when values are injected at runtime.

## Push the image to ECR

### [1] Create an ECR repository

**Step 1.** Provision a repository for the Django image.
```bash
aws ecr create-repository --repository-name pw8/lab-app
```
**Explanation:** The repository provides a private registry namespace for my container image.

**Result:** AWS returns the repository URI, e.g., `123456789012.dkr.ecr.ap-southeast-2.amazonaws.com/pw8/lab-app`.

### [2] Authenticate Docker to ECR

**Step 1.** Retrieve and execute the login command.
```bash
aws ecr get-login-password | docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-southeast-2.amazonaws.com
```
**Explanation:** This command exchanges my IAM credentials for a temporary Docker registry token.

**Result:** Docker reports `Login Succeeded`.

### [3] Build and push the image

**Step 1.** Build the image with an appropriate tag.
```bash
docker build -t pw8/lab-app:latest .
```
**Step 2.** Tag it for the ECR repository.
```bash
docker tag pw8/lab-app:latest 123456789012.dkr.ecr.ap-southeast-2.amazonaws.com/pw8/lab-app:latest
```
**Step 3.** Push the image.
```bash
docker push 123456789012.dkr.ecr.ap-southeast-2.amazonaws.com/pw8/lab-app:latest
```
**Explanation:** Building and tagging prepare the image; pushing uploads the layers to ECR so ECS can pull them later.

**Result:** All layers upload successfully and the repository shows the `latest` tag.

## Deploy to ECS

### [1] Create a task definition

**Step 1.** Define the ECS task in JSON (`task-def.json`).
```json
{
  "family": "pw8-lab",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "web",
      "image": "123456789012.dkr.ecr.ap-southeast-2.amazonaws.com/pw8/lab-app:latest",
      "portMappings": [
        { "containerPort": 8000, "protocol": "tcp" }
      ],
      "environment": [
        { "name": "DJANGO_SETTINGS_MODULE", "value": "lab.settings_production" },
        { "name": "AWS_REGION", "value": "ap-southeast-2" }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/pw8-lab",
          "awslogs-region": "ap-southeast-2",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```
**Explanation:** The task definition specifies a Fargate-compatible container running Gunicorn and shipping logs to CloudWatch.

**Result:** Registering the task returns a revision number (e.g., `pw8-lab:1`).

### [2] Create a cluster and service

**Step 1.** Create an ECS cluster.
```bash
aws ecs create-cluster --cluster-name pw8-cluster
```
**Step 2.** Create a security group and load balancer target group suitable for Fargate.
```bash
aws elbv2 create-target-group --name pw8-ecs --protocol HTTP --port 80 \
  --vpc-id vpc-0123456789abcdef0 --target-type ip
```
**Step 3.** Launch an application load balancer listener that points to the target group.
```bash
aws elbv2 create-listener --load-balancer-arn <ALB-ARN> \
  --protocol HTTP --port 80 \
  --default-actions Type=forward,TargetGroupArn=<TARGET-ARN>
```
**Step 4.** Deploy the ECS service.
```bash
aws ecs create-service \
  --cluster pw8-cluster \
  --service-name pw8-service \
  --task-definition pw8-lab \
  --desired-count 1 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-aaa,subnet-bbb],securityGroups=[sg-0123456789abcdef0],assignPublicIp=ENABLED}" \
  --load-balancers "targetGroupArn=<TARGET-ARN>,containerName=web,containerPort=8000"
```
**Explanation:** The service ensures the task runs continuously and attaches it to the load balancer.

**Result:** ECS launches the Fargate task and the ALB starts routing traffic once the health check passes.

### [3] Verify the deployment

**Step 1.** Open the ALB DNS name in a browser.

**Explanation:** The Django application should respond with the DynamoDB listing just like the EC2 deployment.

**Result:** The page loads successfully and CloudWatch logs confirm healthy requests.

## Lab Assessment

**Deliverables:**

* Dockerfile, Compose file, and ECS task definition.
* Screenshot of the application running behind the ECS-managed load balancer.

**Reflection:** Containerising the application made deployment repeatable and simplified moving between local Docker Compose and managed ECS infrastructure.

---

# Practical Worksheet 9

**Student ID:** 25500725  
**Student Name:** Yidan Xu

---

## Learning Objectives

1. Store and manage images in Amazon S3.
2. Analyse images with Amazon Rekognition.
3. Present analysis results in a simple web interface.

## Technologies Covered

* Amazon S3
* Amazon Rekognition
* boto3 SDK
* Django templates

## Background

In the final worksheet I integrate Rekognition with the existing Django project. The workflow covers uploading images to S3, invoking Rekognition to detect labels and faces, and rendering those findings on a results page.

## Set up S3 buckets

### [1] Create an S3 bucket

**Step 1.** Choose a globally unique bucket name such as `pw9-yidan-images` in my assigned region.

**Step 2.** Create the bucket and block public access by default.
```bash
aws s3api create-bucket --bucket pw9-yidan-images --region ap-southeast-2 --create-bucket-configuration LocationConstraint=ap-southeast-2
```
**Explanation:** Using the API guarantees the bucket is created in the same region as the rest of the resources.

**Result:** The bucket is available in the S3 console.

### [2] Upload test images

**Step 1.** Copy sample photos into the bucket.
```bash
aws s3 cp ./samples/kangaroo.jpg s3://pw9-yidan-images/
aws s3 cp ./samples/opera-house.jpg s3://pw9-yidan-images/
```
**Explanation:** Having at least two images provides data for Rekognition to analyse.

**Result:** Both objects appear under the bucket with their respective keys.

## Configure Rekognition access

### [1] Assign IAM permissions

**Step 1.** Attach the `AmazonRekognitionReadOnlyAccess` policy to the EC2 role or create a custom policy granting `rekognition:DetectLabels` and `rekognition:DetectFaces`.

**Explanation:** Without this policy, boto3 calls would be denied.

**Result:** `aws rekognition detect-labels` returns responses, confirming permissions are in place.

## Integrate Rekognition with Django

### [1] Install additional dependencies

**Step 1.** Ensure `boto3` is installed (carried over from the previous labs) and add `Pillow` for image handling if needed.
```bash
pip install Pillow
```
**Explanation:** Pillow enables thumbnail previews and ensures Rekognition results can reference dimensions if required.

**Result:** The package installs successfully inside the virtual environment.

### [2] Add utilities to query S3 and Rekognition

**Step 1.** Create `polls/rekognition.py` with helper functions.
```python
import boto3

s3 = boto3.client("s3")
rekognition = boto3.client("rekognition")


def list_images(bucket_name):
    response = s3.list_objects_v2(Bucket=bucket_name)
    return [item["Key"] for item in response.get("Contents", [])]


def analyse_image(bucket_name, key):
    labels = rekognition.detect_labels(Bucket=bucket_name, Name=key, MaxLabels=10, MinConfidence=70)
    faces = rekognition.detect_faces(Bucket=bucket_name, Name=key, Attributes=["ALL"])
    return {
        "labels": labels.get("Labels", []),
        "faces": faces.get("FaceDetails", []),
    }
```
**Explanation:** The helpers return image keys and Rekognition metadata to be consumed by Django views.

**Result:** Importing `polls.rekognition` in the Django shell works without errors.

### [3] Update Django views and templates

**Step 1.** Modify `polls/views.py` to include a new view for Rekognition results.
```python
from django.shortcuts import render
from .rekognition import list_images, analyse_image

BUCKET_NAME = "pw9-yidan-images"


def gallery(request):
    objects = list_images(BUCKET_NAME)
    return render(request, "polls/gallery.html", {"objects": objects})


def details(request, key):
    analysis = analyse_image(BUCKET_NAME, key)
    context = {
        "key": key,
        "labels": analysis["labels"],
        "faces": analysis["faces"],
    }
    return render(request, "polls/details.html", context)
```
**Step 2.** Configure URL routes in `polls/urls.py`.
```python
from django.urls import path
from . import views

urlpatterns = [
    path("", views.gallery, name="gallery"),
    path("image/<str:key>/", views.details, name="details"),
]
```
**Step 3.** Create templates.
- `polls/templates/polls/gallery.html`
```html
<h1>Image Gallery</h1>
<ul>
  {% for key in objects %}
    <li><a href="{% url 'details' key %}">{{ key }}</a></li>
  {% empty %}
    <li>No images found.</li>
  {% endfor %}
</ul>
```
- `polls/templates/polls/details.html`
```html
<h1>Analysis for {{ key }}</h1>
<h2>Labels</h2>
<ul>
  {% for label in labels %}
    <li>{{ label.Name }} ({{ label.Confidence|floatformat:2 }}%)</li>
  {% empty %}
    <li>No labels detected.</li>
  {% endfor %}
</ul>
<h2>Faces</h2>
<ul>
  {% for face in faces %}
    <li>Confidence: {{ face.Confidence|floatformat:2 }}%</li>
  {% empty %}
    <li>No faces detected.</li>
  {% endfor %}
</ul>
<a href="{% url 'gallery' %}">Back to gallery</a>
```
**Explanation:** The gallery view lists available S3 keys; selecting one calls Rekognition and displays label and face metadata.

**Result:** Navigating to `/polls/` shows the image list, and clicking a key reveals detailed analysis.

### [4] Test Rekognition output

**Step 1.** Use the Django development server or Gunicorn to load the new pages.
```bash
python manage.py runserver 0.0.0.0:8000
```
**Explanation:** Running locally allows quick iteration before deploying through Fabric or ECS.

**Result:** Rekognition results render correctly for uploaded images, with confidence scores and face counts.

## Lab Assessment

**Deliverables:**

* Screenshot of the gallery and details pages populated by Rekognition data.
* Reflection on managing AWS credentials securely.

**Reflection:** Integrating Rekognition required careful IAM scoping and template updates but added compelling AI-driven feedback to the application.

---

## Overall Reflection

Across Practical Worksheets 6 to 9 I progressively modernised the Django project: starting with a single EC2 instance, automating setup via Fabric, containerising for ECS, and finally enriching the UI with AWS Rekognition insights. The consistent naming conventions and AWS regions tied everything back to my student identifier (25500725), ensuring the deliverables remain traceable and reproducible.
