# Labs 6-9

**Student ID:** 24400725  
**Student Name:** Yidan Xu

# Lab 6

## Set up an EC2 instance

### [1] Create an EC2 micro instance with Ubuntu and SSH into it

**1. Create an EC2 micro instance with Ubuntu**

**Step 1.** Find my region name which is : ap-southeast-2

**Step 2.** Create an EC2 micro instance with a created security group
```bash
aws ec2 run-instances --image-id ami-0eeab253db7e765a9 --count 1 --instance-type t2.micro --key-name 24400725-key --security-groups 24400725-sg
```
**Explanation:**
`aws ec2 run-instances` : This is the base command to launch one or more EC2 instances.
`--image-id ami-0eeab253db7e765a9` : Specifies the Amazon Machine Image (AMI) ID to use for the instance (Ubuntu 22.04 LTS).
`--count 1` : Specifies the number of instances to launch (1 instance).
`--instance-type t2.micro` : Defines the type of instance to launch (t2.micro is the smallest and most cost-effective instance type).
`--key-name 24400725-key` : Specifies the name of the SSH key pair to use for secure access to the instance.
`--security-groups 24400725-sg` : Defines the security group associated with the instance for firewall rules.
**Result:**
The id of the instance: i-05f8c8a1b2c3d4e5f

**Step 3.** Get the public ip address
```bash
aws ec2 describe-instances --instance-ids i-05f8c8a1b2c3d4e5f --query 'Reservations[0].Instances[0].PublicIpAddress' --output text
```
**Explanation:**
`aws ec2 describe-instances` : This command is used to retrieve information about one or more EC2 instances.
`--instance-ids i-05f8c8a1b2c3d4e5f` : Specifies the instance ID of the EC2 instance we want to query.
`--query 'Reservations[0].Instances[0].PublicIpAddress'` : This specifies a query to extract the specific information from the response.
Reservations[0] refers to the first reservation in the list.
Instances[0] refers to the first instance in the reservation.
PublicIpAddress is the specific field that returns the instance's public IP address.
`--output text` : Sets the output format to plain text instead of JSON.
**Result:**
<Your-EC2-Public-IP>

**2. SSH into the EC2 micro instance**

**Step 1.** Connect to the instance via ssh
```bash
ssh -i 24400725-key.pem ubuntu@<Your-EC2-Public-IP>
```
**Explanation:**
`ssh` : This is the command used to initiate a Secure Shell connection to a remote machine.
`-i 24400725-key.pem` : This specifies the identity file (private key) used for the SSH connection.
`ubuntu` : This is the default username for Ubuntu AMIs on EC2 instances.
`<Your-EC2-Public-IP>` : This is the public IP address of the EC2 instance we're trying to connect to.
**Result:**

### [2] Install the Python 3 virtual environment package

**Step 1:**
```bash
sudo apt-get update
```
**Explanation:**
`sudo apt-get update` : Updates the local package database with latest package information.
**Result:**

**Step 2:**
```bash
sudo apt-get upgrade
```
**Explanation:**
`sudo apt-get upgrade` : Upgrades all installed packages to their latest versions.
**Result:**

**Step 3:**
```bash
sudo apt-get install python3-venv
```
**Explanation:**
`sudo apt-get install python3-venv` : Installs Python 3 virtual environment tools.
**Result:**

**Step 4:** Change to root user
```bash
sudo bash
```
**Explanation:**
`sudo bash` : Starts a new Bash shell with root privileges.
**Result:**

### [3] Access a directory 

**Step 1:** Create a directory with a path `/opt/wwc/mysites`
```bash
mkdir -p /opt/wwc/mysites
```
**Explanation:**
`mkdir` : This command stands for "make directory" and is used to create new directories.
`-p` : This option tells mkdir to create the full directory structure, including any missing parent directories.
`/opt` : A standard directory used to store optional (third-party) software.
`/wwc` : A subdirectory you're creating under /opt.
`/mysites` : The actual directory you're creating within /wwc.
**Result:**

**Step 2:** cd into the directory
```bash
cd /opt/wwc/mysites
```
**Explanation:**
`cd` : Stands for "change directory." This command is used to navigate from one directory to another in a Linux file system.
`/opt/wwc/mysites` : This is the path of the directory to navigate to.
**Result:**

### [4] Set up a virtual environment

```bash
python3 -m venv myvenv
```
**Explanation:**
`python3` : This specifies that you are using Python 3. If you have both Python 2 and Python 3 installed, using python3 ensures that you're working with Python 3.
`-m venv` : This tells Python to run the venv module, which is used to create virtual environments.
`myvenv` : This is the name of the directory where the virtual environment will be created.
**Result:**

### [5] Activate the virtual environment

**Step 1:**
```bash
source myvenv/bin/activate
```
**Explanation:**
`source` : This command tells the shell to run a script in the current shell environment. In this case, it runs the activation script inside the virtual environment.
`myvenv/bin/activate` : This is the path to the activation script for the virtual environment.
**Result:**

**Step 2:**
```bash
pip install django
```
**Explanation:**
`pip` : This is the Python package installer. It is used to install and manage packages (or libraries) written in Python.
`install` : This tells pip to install the specified package, which in this case is django.
`django` : The name of the package we want to install.
**Result:**

**Step 3:**
```bash
django-admin startproject lab
```
**Explanation:**
`django-admin` : This is a command-line utility provided by Django. It is used to perform administrative tasks such as starting a new project, creating apps, running development servers, and more.
`startproject` : This is a subcommand of django-admin that initializes a new Django project. It creates the necessary directory structure and files for a new Django project.
`lab` : This is the name of the project we are creating. In this case, the project will be named lab.
**Result:**

**Step 4:**
```bash
cd lab
```
**Explanation:**
`cd` : This command stands for "change directory" and is used to navigate between directories in the terminal.
`lab` : This is the name of the directory we want to navigate into.
**Result:**

**Step 5:**
```bash
python3 manage.py startapp polls
```
**Explanation:**
`python3` : Specifies that you are using Python 3 to run the command.
`manage.py` : This is a command-line utility automatically created when you start a Django project. It is used to manage the Django project.
`startapp` : This is the Django subcommand that creates a new Django app within your project.
`polls` : This is the name of the new app we are creating. In this case, the app is called polls.
**Result:**

**Step 6:** Check
```bash
ls
```
**Explanation:**
`ls` : It is used to list the files and directories in the current working directory.
**Result:**

### [6] Install nginx

```bash
apt install nginx
```
**Explanation:**
`apt` : This is the package management tool used in Debian-based Linux distributions (such as Ubuntu) to manage the installation, update, and removal of software packages.
`install` : This tells apt to install the specified package.
`nginx` : This is the package name for the Nginx web server.
**Result:**

### [7] Configure nginx

**Step 1:** Edit `/etc/nginx/sites-enabled/default` file,
```bash
nano /etc/nginx/sites-enabled/default
```
**Explanation:**
`nano` : This is the command to open the nano text editor, a simple and user-friendly text editor that operates within the terminal.
`/etc/nginx/sites-enabled/default` : This is the path to the default Nginx site configuration file.
**Step 2:** Replace
```nginx
server {
listen 80 default_server;
listen [::]:80 default_server;
location / {
proxy_set_header X-Forwarded-Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_pass http://127.0.0.1:8000;
}
}
```
**Explanation:**
`listen 80 default_server` : This directive tells Nginx to listen on port 80 for HTTP requests and makes this server block the default server.
listen [::]:80 default_server : This directive tells Nginx to listen on IPv6 port 80 for HTTP requests.
`location /` : This directive defines a location block that matches all requests to the root path.
proxy_set_header X-Forwarded-Host $host : This sets the X-Forwarded-Host header to the original host header from the client request.
proxy_set_header X-Real-IP $remote_addr : This sets the X-Real-IP header to the client's IP address.
`proxy_pass http://127.0.0.1:8000` : This directive forwards all requests to the Django application running on localhost port 8000.
**Result:**

### [8] Restart nginx

```bash
service nginx restart
```
**Explanation:**
`service` : This is a command used to manage system services in Linux.
`nginx` : This specifies the name of the service to restart.
`restart` : This option stops the service and then starts it again, ensuring that any configuration changes take effect.
**Result:**

### [9] Access your EC2 instance

**Step 1:**
```bash
python3 manage.py runserver 8000
```
**Explanation:**
`python3` : Specifies that you are using Python 3 to run the command.
`manage.py` : This is a command-line utility automatically created when you start a Django project. It is used to manage the Django project.
`runserver` : This is the Django subcommand that starts the development server.
`8000` : This specifies the port number on which the development server will run.
**Result:**

**Step 2:** Check

**Step 3:** Quit the server

## Set up Django inside the created EC2 instance

### [1] Edit the following files (create them if not exist)

**Step 1:** edit polls/views.py and replace
```bash
vim polls/views.py
```
```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world.")
```
**Explanation:**
`vim polls/views.py` : This command opens the polls/views.py file in the vim text editor for editing.
`from django.http import HttpResponse` : This imports the HttpResponse class from Django's HTTP module, which is used to send HTTP responses.
def index(request): : This defines a function called index that takes a request parameter (standard Django view function signature).
return HttpResponse("Hello, world.") : This returns an HTTP response containing the text "Hello, world." which will be displayed in the browser.
**Result:**

**Step 2:** edit polls/urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```
**Explanation:**
`from django.urls import path` : This imports the path function from Django's URL routing system.
`from . import views` : This imports the views module from the current directory (polls app).
`urlpatterns` : This is a list that defines URL patterns for this Django app.
path('', views.index, name='index') : This creates a URL pattern that maps the empty string ('') to the index view function, with the name 'index' for referencing in templates.
**Result:**

**Step 3:** edit lab/urls.py
```python
from django.urls import include, path
from django.contrib import admin

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```
**Explanation:**
from django.urls import include, path : This imports both include and path functions from Django's URL routing system.
`from django.contrib import admin` : This imports Django's built-in admin functionality.
path('polls/', include('polls.urls')) : This creates a URL pattern that includes all URLs from the polls app under the '/polls/' prefix.
path('admin/', admin.site.urls) : This creates a URL pattern that includes Django's admin interface under the '/admin/' prefix.
**Result:**

### [2] Run the web server again

```bash
python3 manage.py runserver 8000
```
**Explanation:**
`python3` : Specifies that you are using Python 3 to run the command.
`manage.py` : This is a command-line utility automatically created when you start a Django project. It is used to manage the Django project.
`runserver` : This is the Django subcommand that starts the development server.
`8000` : This specifies the port number on which the development server will run.
**Result:**

### [3] Access the EC2 instance

http://<Your-EC2-Public-IP>/polls/
**Explanation:**
`http://` : This specifies the HTTP protocol for web access.
`<Your-EC2-Public-IP>` : This is the public IP address of your EC2 instance where the Django application is running.
`/polls/` : This is the URL path that corresponds to the polls app as configured in the Django URL patterns.
**Result:**

## Set up an ALB

### [1] Create an application load balancer

**Step 1:** Check AWS configure in the environment
```bash
aws configure
```
**Explanation:**
`aws configure` : This command displays or sets the AWS CLI configuration including access key, secret key, region, and output format.
**Result:**

**Step 2:** Check the region subnet where your EC2 instance resides

**Step 3:** Create a target group
```bash
aws elbv2 create-target-group \
--name 24400725 \
--protocol HTTP \
--port 80 \
--vpc-id <Your-VPC-ID> \
--target-type instance \
--health-check-protocol HTTP \
--health-check-path "/polls/"
```
**Explanation:**
`aws elbv2 create-target-group` : This command creates a target group for an Application Load Balancer.
`--name 24400725` : Specifies the name of the target group using the student ID.
`--protocol HTTP` : Sets the protocol to HTTP for communication with targets.
`--port 80` : Specifies port 80 for target communication.
`--vpc-id <Your-VPC-ID>` : Specifies the VPC ID where the targets are located.
`--target-type instance` : Sets the target type to instance (as opposed to IP or Lambda).
`--health-check-protocol HTTP` : Specifies HTTP protocol for health checks.
`--health-check-path "/polls/"` : Sets the health check path to "/polls/" which matches our Django app endpoint.
**Result:**

**Step 4:** Create the load balancer
```bash
aws elbv2 create-load-balancer \
--name 24400725 \
--subnets <Your-Subnet-ID-1> <Your-Subnet-ID-2> \
--security-groups <Your-Security-Group-ID> \
--scheme internet-facing \
--type application \
--ip-address-type ipv4
```
**Explanation:**
`aws elbv2 create-load-balancer` : This command creates a new Application Load Balancer.
`--name 24400725` : Specifies the name of the load balancer using the student ID.
`--subnets <Your-Subnet-ID-1> <Your-Subnet-ID-2>` : Specifies the subnet IDs where the load balancer will be deployed (across multiple availability zones).
`--security-groups <Your-Security-Group-ID>` : Specifies the security group ID for the load balancer.
`--scheme internet-facing` : Sets the load balancer scheme to internet-facing (accessible from the internet).
`--type application` : Specifies the type as Application Load Balancer (Layer 7).
`--ip-address-type ipv4` : Sets the IP address type to IPv4.
**Result:**

**Step 5:** Create a listener with a default rule Protocol: HTTP and Port 80 forwarding
```bash
aws elbv2 create-listener \
--load-balancer-arn <Your-LoadBalancer-ARN> \
--protocol HTTP \
--port 80 \
--default-actions Type=forward,TargetGroupArn=<Your-TargetGroup-ARN>
```
**Explanation:**
`aws elbv2 create-listener` : This command creates a listener for the Application Load Balancer.
`--load-balancer-arn` : Specifies the ARN (Amazon Resource Name) of the load balancer.
`--protocol HTTP` : Sets the protocol to HTTP for the listener.
`--port 80` : Specifies port 80 for the listener to listen on.
--default-actions Type=forward,TargetGroupArn=... : Sets the default action to forward traffic to the specified target group ARN.
**Result:**

**Step 6:** Choose the security group, allowing HTTP traffic.

**Step 7:** Add your instance as a registered target.
```bash
aws elbv2 register-targets \
--target-group-arn <Your-TargetGroup-ARN> \
--targets Id=<Your-Instance-ID>
```
**Explanation:**
`aws elbv2 register-targets` : This command registers targets (EC2 instances) with a target group.
`--target-group-arn` : Specifies the ARN of the target group where targets will be registered.
--targets Id=<Your-Instance-ID> : Specifies the instance ID of the EC2 instance to register as a target.
**Result:**

### [2] Health check

**Step 1:** For the target group, specify `/polls/` for a path for the health check.
```bash
aws elbv2 describe-target-groups --target-group-arns <Your-TargetGroup-ARN>
```
**Explanation:**
`aws elbv2 describe-target-groups` : This command retrieves information about one or more target groups.
`--target-group-arns` : Specifies the ARN of the target group to describe.
**Result:**

**Step 2:** Confirm the health check fetch the `/polls/` page every 30 seconds.
**Explanation:**
This step verifies that the health check is configured to check the `/polls/` endpoint every 30 seconds to ensure the target instance is healthy and responding correctly.
**Result:**

### [3] Access

http://24400725-666625381.ap-southeast-2.elb.amazonaws.com/polls/
**Explanation:**
`http://` : This specifies the HTTP protocol for web access.
`24400725-666625381.ap-southeast-2.elb.amazonaws.com` : This is the DNS name of the Application Load Balancer, which automatically routes traffic to healthy targets.
`/polls/` : This is the URL path that corresponds to the polls app as configured in the Django URL patterns.
**Result:**

Stop the Server:

## Web interface for CloudStorage application

### [1] Confirm Local Table Exists
```bash
aws dynamodb list-tables --endpoint-url http://localhost:8000
```
**Explanation:**
`aws dynamodb list-tables` : This command lists all DynamoDB tables in the current region.
`--endpoint-url http://localhost:8000` : This specifies the endpoint URL for the local DynamoDB instance running on port 8000.
**Result:**

### [2] Create an AWS DynamoDB table
```bash
aws dynamodb create-table \
--table-name 24400725CloudFiles \
--attribute-definitions AttributeName=fileName,AttributeType=S \
--key-schema AttributeName=fileName,KeyType=HASH \
--provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 \
--region ap-southeast-2
```
**Explanation:**
`aws dynamodb create-table` : This command creates a new DynamoDB table.
`--table-name 24400725CloudFiles` : Specifies the name of the table using the student ID as a prefix.
--attribute-definitions AttributeName=fileName,AttributeType=S : Defines the attributes for the table, where fileName is a String (S) type.
--key-schema AttributeName=fileName,KeyType=HASH : Defines the primary key schema with fileName as the hash key.
--provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 : Sets the provisioned capacity for read and write operations.
`--region ap-southeast-2` : Specifies the AWS region where the table will be created.
**Result:**

### [3] Import the Local Data into AWS
**Step 1** Edit a script copy_local_to_aws.py
```bash
python3 copy_local_to_aws.py
```
**Explanation:**
`python3` : Specifies that you are using Python 3 to run the command.
`copy_local_to_aws.py` : This is the name of the Python script that will copy data from the local DynamoDB to AWS DynamoDB.
**Result:**

### [4] Edit views.py
```python
# polls/views.py
import os
import boto3
from botocore.exceptions import ClientError
from django.http import JsonResponse, HttpResponse
AWS_REGION = os.getenv("AWS_DEFAULT_REGION", "ap-southeast-2")

TABLE_NAME = "24400725CloudFiles"
def index(request):
try:
dynamodb = boto3.resource("dynamodb", region_name=AWS_REGION)
table = dynamodb.Table(TABLE_NAME)
items, start_key = [], None
while True:
kwargs = {}
if start_key:
kwargs["ExclusiveStartKey"] = start_key
resp = table.scan(**kwargs)
items.extend(resp.get("Items", []))
start_key = resp.get("LastEvaluatedKey")
if not start_key:
break
return JsonResponse({"count": len(items), "items": items})
except ClientError as e:
return JsonResponse({"error": e.response["Error"]["Message"]}, status=500)
except Exception as e:
return JsonResponse({"error": str(e)}, status=500)
```
**Explanation:**
`import os` : This imports the os module for accessing environment variables.
`import boto3` : This imports the AWS SDK for Python to interact with AWS services.
`from botocore.exceptions import ClientError` : This imports the ClientError exception class for handling AWS API errors.
from django.http import JsonResponse, HttpResponse : This imports Django's HTTP response classes.
AWS_REGION = os.getenv("AWS_DEFAULT_REGION", "ap-southeast-2") : This sets the AWS region, defaulting to ap-southeast-2 if not specified in environment variables.
TABLE_NAME = "24400725CloudFiles" : This defines the name of the DynamoDB table using the student ID.
dynamodb = boto3.resource("dynamodb", region_name=AWS_REGION) : This creates a DynamoDB resource object.
table = dynamodb.Table(TABLE_NAME) : This gets a reference to the specific table.
table.scan(**kwargs) : This scans the table to retrieve all items, handling pagination with ExclusiveStartKey.
`JsonResponse` : This returns the data as JSON format suitable for web APIs.
**Result:**

### [5] Use template:

**Step 1** create a folder called templates under polls
```bash
mkdir templates
```
**Explanation:**
`mkdir templates` : This command creates a new directory named "templates" in the current location (polls directory).
**Result:**

**Step 2** add to the TEMPLATES section of lab/settings.py
```python
TEMPLATES = [
{
'BACKEND': 'django.template.backends.django.DjangoTemplates',
'DIRS': [
'polls/templates/'
],
```
**Explanation:**
`TEMPLATES` : This is a Django settings configuration that defines template engines and their directories.
`'BACKEND': 'django.template.backends.django.DjangoTemplates'` : This specifies Django's built-in template engine as the backend.
'DIRS': ['polls/templates/'] : This tells Django where to look for template files, specifying the polls/templates directory.
**Result:**

**Step 3** In the templates directory, add a file files.html with the following contents:
```bash
vim files.html
```
```html
<html>
<head>
    <title>Files</title>
</head>
<body>
    <h1>Files </h1>
    <ul>
        {% for item in items %}
          <li>{{ item.fileName }}</li>
    {% endfor %}
    </ul>
</body>
</html>
```
**Explanation:**
`vim files.html` : This command opens the files.html file in the vim text editor for editing.
`<html>` : This is the opening HTML tag that defines the document as HTML5.
`<title>Files</title>` : This sets the title of the web page that appears in the browser tab.
`<h1>Files </h1>` : This creates a level 1 heading displaying "Files".
{% for item in items %} : This is Django template syntax that loops through the items list passed from the view.
{{ item.fileName }} : This displays the fileName attribute of each item using Django template variable syntax.
{% endfor %} : This closes the for loop in Django template syntax.
**Results:**

### [6] Update views.py
```python
from django.shortcuts import render
from django.template import loader
from django.http import HttpResponse
import boto3
import json
from boto3.dynamodb.conditions import Key, Attr
from botocore.exceptions import ClientError

def index(request):
    template = loader.get_template('files.html')
    
    dynamodb = boto3.resource('dynamodb', region_name='ap-southeast-2',
                              aws_access_key_id='Your Access Key',
                              aws_secret_access_key='Your Secret')
    
    table = dynamodb.Table("24400725CloudFiles")
    
    items = []
    try:
        response = table.scan()
        
    except ClientError as e:
        print(e.response['Error']['Message'])
    else:    
        context = {'items': response['Items'] }
        
        return HttpResponse(template.render(context, request))
```
**Explanation:**
`from django.shortcuts import render` : Imports Django's render function for templates.
`from django.template import loader` : Imports Django template loader.
`from django.http import HttpResponse` : Imports Django HTTP response class.
`import boto3` : Imports AWS SDK for Python.
`import json` : Imports JSON module for data handling.
`from boto3.dynamodb.conditions import Key, Attr` : Imports DynamoDB condition classes.
`from botocore.exceptions import ClientError` : Imports AWS error handling.
template = loader.get_template('files.html') : Loads the HTML template file.
dynamodb = boto3.resource('dynamodb', region_name='ap-southeast-2', aws_access_key_id='Your Access Key', aws_secret_access_key='Your Secret') : Creates DynamoDB resource with credentials.
table = dynamodb.Table("24400725CloudFiles") : Gets reference to DynamoDB table.
items = [] : Initializes empty items list.
response = table.scan() : Scans table to retrieve all items.
context = {'items': response['Items']} : Creates context dictionary for template.
template.render(context, request) : Renders template with data.
**Result:**

### [4] Test the complete application

**Steps:**

1. Start the Django development server:
   ```bash
   python3 manage.py runserver 8000
   ```

2. Access the application and verify DynamoDB data is displayed:
   ```
   http://<EC2-IP>/polls/
   ```

**Explanation:**
- Testing confirms Django application successfully retrieves data from DynamoDB
- Verifies complete architecture functions correctly

**Result:**
- Application successfully displays DynamoDB data
- Complete architecture functioning properly

### [5] Cleanup and resource management

**Steps:**

1. Stop Django server with Ctrl+C

2. Terminate EC2 instance through AWS Console:
   - Select your instance
   - Actions → Instance State → Terminate

3. Delete Application Load Balancer:
   - Navigate to EC2 → Load Balancers
   - Select your ALB and delete

4. Clean up security groups if they're no longer needed

**Explanation:**
- Proper resource cleanup prevents ongoing charges
- Load balancers incur charges even when not actively used
- Following AWS best practices for resource management

**Result:**
- All AWS resources properly terminated
- No unexpected charges incurred

**Terminate all created resources:** S3 Bucket, DynamoDB table, EC2 instance, application load balancer, target group, security groups

# Lab 7

## [1] Create an EC2 instance
```bash
aws ec2 create-security-group --group-name 24400725-sg --description "security for development environment"
```
```bash
aws ec2 authorize-security-group-ingress --group-name 24400725-sg --protocol tcp --port 22 --cidr 0.0.0.0/0
```
```bash
aws ec2 run-instances --image-id ami-0eeab253db7e765a9 --count 1 --instance-type t2.micro --key-name 24400725-key --security-groups 24400725-sg --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=24400725}]'
```
**Result:**

## [2] Install and configure Fabric
**Step 1** Install Fabric
```bash
pip install fabric
```
**Explanation:**
`pip install fabric` : Installs the Fabric library, which is a Python library for streamlining SSH connections and application deployment.
**Result:**

**Step 2** Configure Fabric and replace it with the provided content
```bash
vim ~/.ssh/config
```
```
Host 24400725
Hostname <Your-EC2-Public-DNS>
User ubuntu
UserKnownHostsFile /dev/null
StrictHostKeyChecking no
PasswordAuthentication no
IdentityFile /home/ubuntu/24400725-key.pem
```
**Explanation:**
vim ~/.ssh/config : Opens the SSH configuration file for editing.
`Host 24400725` : Defines a host alias for easy connection.
`Hostname <Your-EC2-Public-DNS>` : Specifies the public DNS name of your EC2 instance.
`User ubuntu` : Sets the default username for this host.
`UserKnownHostsFile /dev/null` : Disables host key checking by sending known hosts to /dev/null.
`StrictHostKeyChecking no` : Disables strict host key checking for easier connections.
`PasswordAuthentication no` : Disables password authentication (uses key-based auth).
`IdentityFile /home/ubuntu/24400725-key.pem` : Specifies the path to the private key file.
**Result:**

**Step 3** Rely on the fabric code below to connect to your instance.
```bash
python3
>>> from fabric import Connection
>>> c = Connection('24400725')
>>> result = c.run('uname -s')
Linux
>>>
```
**Explanation:**
`python3` : Starts the Python 3 interactive interpreter.
`from fabric import Connection` : Imports the Connection class from Fabric.
c = Connection('24400725') : Creates a connection object using the host alias defined in SSH config.
result = c.run('uname -s') : Runs the 'uname -s' command on the remote server to get the operating system name.
`Linux` : The output showing the remote system is running Linux.
**Result:**

## [3] Use Fabric for automation
Edit a script to
```python
from fabric import Connection
from invoke import run as local
# ======== CONFIG ========
EC2_HOST = "<Your-EC2-Public-IP>"
EC2_USER = "ubuntu"
KEY_FILE = "/home/ubuntu/24400725-key.pem"
BASE_DIR = "/home/ubuntu/djangoapp"
VENV = f"{BASE_DIR}/venv"
PROJECT = "lab"
APP = "polls"
RUN_LOG = f"{BASE_DIR}/runserver.log"
# ========================

def get_connection():
    return Connection(
        host=EC2_HOST,
        user=EC2_USER,
        connect_kwargs={"key_filename": KEY_FILE},
    )
```
**Explanation:**
`from fabric import Connection` : Imports the Connection class from Fabric for SSH connections.
`from invoke import run as local` : Imports the run function for executing local commands.
EC2_HOST = "<Your-EC2-Public-IP>" : Sets the public IP address of your EC2 instance.
EC2_USER = "ubuntu" : Sets the username for SSH connections.
KEY_FILE = "/home/ubuntu/24400725-key.pem" : Specifies the path to the private key file.
BASE_DIR = "/home/ubuntu/djangoapp" : Sets the base directory for the Django application.
VENV = f"{BASE_DIR}/venv" : Sets the path to the Python virtual environment.
PROJECT = "lab" : Sets the Django project name.
APP = "polls" : Sets the Django application name.
RUN_LOG = f"{BASE_DIR}/runserver.log" : Sets the path to the server log file.
def get_connection(): : Defines a function to create SSH connections.
return Connection(...) : Returns a Connection object with specified parameters.
host=EC2_HOST : Sets the host IP address for the connection.
user=EC2_USER : Sets the username for the connection.
connect_kwargs={"key_filename": KEY_FILE} : Specifies the private key file for authentication.
**Result:**

def setup_ec2():
    c = get_connection()
    # System dependencies
    c.sudo("apt-get update -y")
    c.sudo("apt-get install -y python3-venv nginx")
    # Work dir
    c.run(f"mkdir -p {BASE_DIR}")
    c.sudo(f"chown -R {EC2_USER}:{EC2_USER} {BASE_DIR}")
    # Python virtualenv
    c.run(f"test -d {VENV} || python3 -m venv {VENV}")
    pip = f"{VENV}/bin/pip"
    django_admin = f"{VENV}/bin/django-admin"
    c.run(f"{pip} install --upgrade pip wheel")
    c.run(f"{pip} install 'django>=4,<6' gunicorn")
    # Create project if missing
    with c.cd(BASE_DIR):
        c.run(f"test -f manage.py || {django_admin} startproject {PROJECT} .")
    # --- Nginx setup --
    c.sudo("rm -f /etc/nginx/sites-enabled/default", warn=True)
    nginx_avail = "/etc/nginx/sites-available/django_default.conf"
    nginx_en = "/etc/nginx/sites-enabled/django_default.conf"

    nginx_conf = f"""server {{
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;
    location / {{
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $$host;
        proxy_set_header X-Real-IP $$remote_addr;
        proxy_set_header X-Forwarded-For $$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $$scheme;
    }}
    }}
    """
    c.run(f'echo "{nginx_conf.strip()}" | sudo tee {nginx_avail} >/dev/null')
    c.sudo(f"ln -sf {nginx_avail} {nginx_en}")
    c.sudo("nginx -t")
    c.sudo("systemctl restart nginx")
```
**Explanation:**
def setup_ec2(): : Defines a function to set up the EC2 instance.
c = get_connection() : Gets a connection to the remote server.
c.sudo("apt-get update -y") : Updates package lists with sudo privileges.
c.sudo("apt-get install -y python3-venv nginx") : Installs Python virtual environment and Nginx.
c.run(f"mkdir -p {BASE_DIR}") : Creates the base directory for the application.
c.sudo(f"chown -R {EC2_USER}:{EC2_USER} {BASE_DIR}") : Changes ownership of the directory to ubuntu user.
c.run(f"test -d {VENV} || python3 -m venv {VENV}") : Creates virtual environment if it doesn't exist.
pip = f"{VENV}/bin/pip" : Sets path to pip in virtual environment.
django_admin = f"{VENV}/bin/django-admin" : Sets path to django-admin in virtual environment.
c.run(f"{pip} install --upgrade pip wheel") : Upgrades pip and installs wheel.
c.run(f"{pip} install 'django>=4,<6' gunicorn") : Installs Django and Gunicorn.
with c.cd(BASE_DIR): : Changes to the base directory context.
c.run(f"test -f manage.py || {django_admin} startproject {PROJECT} .") : Creates Django project if manage.py doesn't exist.
c.sudo("rm -f /etc/nginx/sites-enabled/default", warn=True) : Removes default Nginx site.
nginx_conf = f"""...""" : Defines Nginx configuration as a string.
c.run(f'echo "{nginx_conf.strip()}" | sudo tee {nginx_avail} >/dev/null') : Writes Nginx config to file.
c.sudo(f"ln -sf {nginx_avail} {nginx_en}") : Creates symbolic link to enable the site.
c.sudo("nginx -t") : Tests Nginx configuration.
c.sudo("systemctl restart nginx") : Restarts Nginx service.
**Result:**

def setup_django():
    c = get_connection()
    py = f"{VENV}/bin/python"
    # Create app if not exists
    with c.cd(BASE_DIR):
        c.run(f"test -d {BASE_DIR}/{APP} || {py} manage.py startapp {APP}")
    # Write views.py
    c.run(f'cat > {BASE_DIR}/{APP}/views.py <<EOF\nfrom django.http import HttpResponse\n\ndef index(request):\n    return HttpResponse("Hello, world.")\nEOF')
    # Write app urls.py
    c.run(f'cat > {BASE_DIR}/{APP}/urls.py <<EOF\nfrom django.urls import path\nfrom . import views\n\nurlpatterns = [\n    path("", views.index, name="index"),\n]\nEOF')

    # Project urls.py
    c.run(f'cat > {BASE_DIR}/{PROJECT}/urls.py <<EOF\nfrom django.contrib import admin\nfrom django.urls import include, path\n\nurlpatterns = [\n    path("polls/", include("polls.urls")),\n    path("admin/", admin.site.urls),\n]\nEOF')

    # Fix settings.py
    settings = f"{BASE_DIR}/{PROJECT}/settings.py"
    c.run(f"sed -i '/\\\\nINSTALLED_APPS/d' {settings}", warn=True)
    # Always set ALLOWED_HOSTS = ['*']
    c.run(
        f"grep -q '^ALLOWED_HOSTS' {settings} "
        f"&& sed -i \"s/^ALLOWED_HOSTS.*/ALLOWED_HOSTS = ['*']/\" {settings} "
        f"|| echo \"ALLOWED_HOSTS = ['*']\" >> {settings}"
    )
    # Ensure CSRF_TRUSTED_ORIGINS exists
    csrf_line = f"CSRF_TRUSTED_ORIGINS = ['http://{EC2_HOST}']"
    c.run(f"grep -q 'CSRF_TRUSTED_ORIGINS' {settings} || echo \"{csrf_line}\" >> {settings}")
    # Ensure polls app registered
    c.run(f"grep -q \"'{APP}'\" {settings} || sed -i \"/INSTALLED_APPS = \\[/ a\\\n'{APP}',\" {settings}")
    # Run migrations
    with c.cd(BASE_DIR):
        c.run(f"{py} manage.py migrate")
```
**Explanation:**
def setup_django(): : Defines a function to set up the Django application.
c = get_connection() : Gets a connection to the remote server.
py = f"{VENV}/bin/python" : Sets path to Python in virtual environment.
with c.cd(BASE_DIR): : Changes to the base directory context.
c.run(f"test -d {BASE_DIR}/{APP} || {py} manage.py startapp {APP}") : Creates Django app if it doesn't exist.
c.run(f'cat > {BASE_DIR}/{APP}/views.py <<EOF...') : Creates views.py file with a simple view.
c.run(f'cat > {BASE_DIR}/{APP}/urls.py <<EOF...') : Creates app URLs configuration.
c.run(f'cat > {BASE_DIR}/{PROJECT}/urls.py <<EOF...') : Creates project URLs configuration.
settings = f"{BASE_DIR}/{PROJECT}/settings.py" : Sets path to Django settings file.
c.run(f"sed -i '/\\\\nINSTALLED_APPS/d' {settings}", warn=True) : Removes any duplicate INSTALLED_APPS lines.
c.run(...ALLOWED_HOSTS...) : Sets ALLOWED_HOSTS to allow all hosts.
csrf_line = f"CSRF_TRUSTED_ORIGINS = ['http://{EC2_HOST}']" : Sets CSRF trusted origins.
c.run(f"grep -q 'CSRF_TRUSTED_ORIGINS' {settings} || echo \"{csrf_line}\" >> {settings}") : Adds CSRF_TRUSTED_ORIGINS if not exists.
c.run(f"grep -q \"'{APP}'\" {settings} || sed -i \"/INSTALLED_APPS = \\[/ a\\\n'{APP}',\" {settings}") : Adds polls app to INSTALLED_APPS.
with c.cd(BASE_DIR): : Changes to the base directory context.
c.run(f"{py} manage.py migrate") : Runs Django database migrations.
**Result:**

def stop_server():
    c = get_connection()
    c.run("pkill -f 'manage.py runserver' || true", warn=True)
```
**Explanation:**
def stop_server(): : Defines a function to stop the Django development server.
c = get_connection() : Gets a connection to the remote server.
c.run("pkill -f 'manage.py runserver' || true", warn=True) : Kills any running Django development server processes.
**Result:**

def run_server():
    c = get_connection()
    with c.cd(BASE_DIR):
        c.run(f"mv {RUN_LOG} {RUN_LOG}.1 2>/dev/null || true", warn=True)
        c.run(f"nohup {VENV}/bin/python manage.py runserver 0.0.0.0:8000 > {RUN_LOG} 2>&1 &", pty=False)
    print(f" Django started. Logs: tail -n 50 {RUN_LOG}")
```
**Explanation:**
def run_server(): : Defines a function to start the Django development server.
c = get_connection() : Gets a connection to the remote server.
with c.cd(BASE_DIR): : Changes to the base directory context.
c.run(f"mv {RUN_LOG} {RUN_LOG}.1 2>/dev/null || true", warn=True) : Backs up the previous log file.
c.run(f"nohup {VENV}/bin/python manage.py runserver 0.0.0.0:8000 > {RUN_LOG} 2>&1 &", pty=False) : Starts Django server in background.
print(f" Django started. Logs: tail -n 50 {RUN_LOG}") : Prints information about server startup.
**Result:**

def check_site():
    local(f"curl -I http://{EC2_HOST}/polls/ || true")
    local(f"curl http://{EC2_HOST}/polls/ || true")
```
**Explanation:**
def check_site(): : Defines a function to check if the website is accessible.
local(f"curl -I http://{EC2_HOST}/polls/ || true") : Makes a HEAD request to check if the site is accessible.
local(f"curl http://{EC2_HOST}/polls/ || true") : Makes a GET request to retrieve the page content.
**Result:**

def deploy():
    setup_ec2()
    setup_django()
    stop_server()
    run_server()
    check_site()
```
**Explanation:**
def deploy(): : Defines a function to deploy the complete application.
setup_ec2() : Calls the setup_ec2 function to configure the server.
setup_django() : Calls the setup_django function to configure Django.
stop_server() : Stops any existing Django server.
run_server() : Starts the Django development server.
check_site() : Checks if the website is accessible.
**Result:**

**Step 4** Run the Fabric script
```python
deploy()
```
**Explanation:**
deploy() : Executes the complete deployment process including EC2 setup, Django configuration, and server startup.
**Result:**

if __name__ == "__main__":
    deploy()
```
**Explanation:**
if __name__ == "__main__": : Checks if the script is being run directly (not imported).
deploy() : Executes the deployment process when the script is run directly.
**Result:**

**Terminate all created resources:** EC2 instance, security groups

<div style="page-break-after: always;"></div>

# Lab 8

## Create a Dockerfile and build a Docker image

### [1] Create your Dockerfile:
**Step 1:** Create a Dockerfile and replace it with the provided content
```bash
vim Dockerfile
```
```dockerfile
FROM python:3.10
RUN pip install jupyter boto3 sagemaker awscli
RUN mkdir /notebook
# Use a sample access token
ENV JUPYTER_ENABLE_LAB=yes
ENV JUPYTER_TOKEN="CITS5503"
# Allow access from ALL IPs
RUN jupyter notebook --generate-config
RUN echo "c.NotebookApp.ip = '0.0.0.0'" >> /root/.jupyter/jupyter_notebook_config.py
# Copy the ipynb file
RUN wget -P /notebook https://raw.githubusercontent.com/zhangzhics/CITS5503_Sem2/master/Labs/src/LabAI.ipynb
WORKDIR /notebook
EXPOSE 8888
CMD ["jupyter", "notebook", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root"]
```
**Explanation:**
`vim Dockerfile` : Opens a new file called Dockerfile for editing.
`FROM python:3.10` : Specifies the base image as Python 3.10.
`RUN pip install jupyter boto3 sagemaker awscli` : Installs required Python packages for Jupyter, AWS services, and SageMaker.
`RUN mkdir /notebook` : Creates a directory called /notebook for storing Jupyter notebooks.
ENV JUPYTER_ENABLE_LAB=yes : Sets environment variable to enable JupyterLab.
ENV JUPYTER_TOKEN="CITS5503" : Sets a sample access token for Jupyter.
`RUN jupyter notebook --generate-config` : Generates Jupyter configuration files.
RUN echo "c.NotebookApp.ip = '0.0.0.0'" >> /root/.jupyter/jupyter_notebook_config.py : Configures Jupyter to accept connections from any IP.
`RUN wget -P /notebook https://raw.githubusercontent.com/zhangzhics/CITS5503_Sem2/master/Labs/src/LabAI.ipynb` : Downloads the LabAI notebook file.
`WORKDIR /notebook` : Sets the working directory to /notebook.
`EXPOSE 8888` : Exposes port 8888 for Jupyter access.
CMD ["jupyter", "notebook", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root"] : Sets the default command to start Jupyter notebook.
**Result:**

**Step 2:** Build your dockerfile:
```bash
docker build -t 24400725-lab8 .
```
**Explanation:**
`docker build` : Command to build a Docker image from a Dockerfile.
`-t 24400725-lab8` : Tags the image with name "24400725-lab8" using your student ID.
`.` : Specifies the build context as the current directory (where Dockerfile is located).
**Result:**

<div style="page-break-after: always;"></div>

# Lab 9

## Prepare ECR via Boto3 scripts on your local machine

### [1] use Boto3 script to create a ECR repository
```python
import boto3

def create_or_check_repository(repository_name):
    ecr_client = boto3.client('ecr')
    try:
        response = ecr_client.describe_repositories(repositoryNames=[repository_name])
        repository_uri = response['repositories'][0]['repositoryUri']
    except ecr_client.exceptions.RepositoryNotFoundException:
        response = ecr_client.create_repository(repositoryName=repository_name)
        repository_uri = response['repository']['repositoryUri']
    return repository_uri

repository_name = '24400725' + '_ecr_repo'
repository_uri = create_or_check_repository(repository_name)
print("ECR URI:", repository_uri)
```
**Explanation:**
`import boto3` : Imports the AWS SDK for Python to interact with AWS services.
def create_or_check_repository(repository_name): : Defines a function to create or check if an ECR repository exists.
ecr_client = boto3.client('ecr') : Creates an ECR client for interacting with Amazon Elastic Container Registry.
try: : Begins a try block to attempt to describe an existing repository.
response = ecr_client.describe_repositories(repositoryNames=[repository_name]) : Attempts to get information about an existing repository.
repository_uri = response['repositories'][0]['repositoryUri'] : Extracts the repository URI from the response.
except ecr_client.exceptions.RepositoryNotFoundException: : Catches the exception when repository doesn't exist.
response = ecr_client.create_repository(repositoryName=repository_name) : Creates a new repository if it doesn't exist.
repository_uri = response['repository']['repositoryUri'] : Extracts the URI from the newly created repository.
return repository_uri : Returns the repository URI.
repository_name = '24400725' + '_ecr_repo' : Creates repository name using your student ID.
repository_uri = create_or_check_repository(repository_name) : Calls the function to create or get the repository.
print("ECR URI:", repository_uri) : Prints the repository URI.
**Result:**

### [2] Retrieve authorisation
```python
import boto3
import base64

def get_docker_login_cmd():
    ecr_client = boto3.client('ecr')
    token = ecr_client.get_authorization_token()
    username, password = base64.b64decode(token['authorizationData'][0]['authorizationToken']).decode().split(':')
    registry = token['authorizationData'][0]['proxyEndpoint']
    return f"docker login -u {username} -p {password} {registry}"

print(get_docker_login_cmd())
```
**Explanation:**
`import boto3` : Imports the AWS SDK for Python.
`import base64` : Imports the base64 module for decoding authorization tokens.
def get_docker_login_cmd(): : Defines a function to get Docker login command for ECR.
ecr_client = boto3.client('ecr') : Creates an ECR client.
token = ecr_client.get_authorization_token() : Gets authorization token from ECR.
username, password = base64.b64decode(...).decode().split(':') : Decodes the authorization token and splits into username and password.
registry = token['authorizationData'][0]['proxyEndpoint'] : Gets the registry endpoint.
return f"docker login -u {username} -p {password} {registry}" : Returns the Docker login command string.
print(get_docker_login_cmd()) : Prints the Docker login command.
**Result:**

### [3] Login
**Explanation:**
Execute the Docker login command printed in the previous step to authenticate with ECR.
**Result:**

## Push a local Docker image onto ECR

### [1] Tag Image
```bash
docker tag 24400725-lab8:latest <Your-AWS-Account-ID>.dkr.ecr.ap-southeast-2.amazonaws.com/24400725_ecr_repo:latest
```
**Explanation:**
`docker tag` : Command to tag a Docker image.
`24400725-lab8:latest` : Source image name and tag (the image built in Lab 8).
`<Your-AWS-Account-ID>.dkr.ecr.ap-southeast-2.amazonaws.com/24400725_ecr_repo:latest` : Target ECR repository URI with latest tag.
**Result:**

### [2] Push to ECR
```bash
docker push <Your-AWS-Account-ID>.dkr.ecr.ap-southeast-2.amazonaws.com/24400725_ecr_repo:latest
```
**Explanation:**
`docker push` : Command to push a Docker image to a registry.
`<Your-AWS-Account-ID>.dkr.ecr.ap-southeast-2.amazonaws.com/24400725_ecr_repo:latest` : The ECR repository URI to push the image to.
**Result:**

## Deploy your Docker image onto ECS

### [1] Create a task definition for an ECS task: add environment variables
```python
import boto3

def create_ecs_task_definition(
    client, image_uri, account_id, task_role_name, execution_role_name, student_id,
    environment_dict=None, port=8888, cpu='256', memory='512'
):
    task_role_arn = f'arn:aws:iam::{account_id}:role/{task_role_name}'
    execution_role_arn = f'arn:aws:iam::{account_id}:role/{execution_role_name}'
    env_list = [{'name': k, 'value': v} for k, v in (environment_dict or {}).items()]
    
    response = client.register_task_definition(
        family=f'{student_id}-task-family',
        networkMode='awsvpc',
        requiresCompatibilities=['FARGATE'],
        cpu=cpu,
        memory=memory,
        taskRoleArn=task_role_arn,
        executionRoleArn=execution_role_arn,
        containerDefinitions=[
            {
                'name': f'{student_id}-container',
                'image': image_uri,
                'essential': True,
                'portMappings': [
                    {
                        'containerPort': port,
                        'hostPort': port,
                        'protocol': 'tcp'
                    },
                ],
            }
        ]
    )
    return response
```
**Explanation:**
`import boto3` : Imports the AWS SDK for Python.
def create_ecs_task_definition(...): : Defines a function to create ECS task definition.
client : ECS client for AWS API calls.
image_uri : URI of the Docker image to deploy.
account_id : AWS account ID for IAM role ARNs.
task_role_name : Name of the IAM role for the task.
execution_role_name : Name of the IAM role for task execution.
student_id : Student ID used for naming resources.
environment_dict : Optional dictionary of environment variables.
port : Port number for the container (default 8888 for Jupyter).
cpu : CPU units for the task (default '256').
memory : Memory in MB for the task (default '512').
task_role_arn : Constructs ARN for the task role.
execution_role_arn : Constructs ARN for the execution role.
env_list : Converts environment dictionary to ECS format.
response = client.register_task_definition(...) : Registers the task definition with ECS.
family : Sets the task definition family name using student ID.
networkMode='awsvpc' : Sets network mode to VPC.
requiresCompatibilities=['FARGATE'] : Specifies Fargate launch type.
containerDefinitions : Defines container specifications.
name : Sets container name using student ID.
image : Sets the Docker image URI.
essential=True : Marks container as essential (task stops if container fails).
portMappings : Maps container port to host port.
**Result:**

