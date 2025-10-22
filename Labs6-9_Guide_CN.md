# CITS5503 Labs 6-9 实操指南

> 本指南以实验手册 (Lab6WebApplication.md、Lab7DevOps.md、Lab8ContainerAI.md、Lab9Rekognition.md) 为基础，总结完成实验所需的关键步骤。所有命令均假设你已经正确配置 AWS CLI，并具有必要的访问权限。请在执行命令前将示例中的占位符（如 `<Your-EC2-Public-IP>`、`<AWS-ACCOUNT-ID>` 等）替换为自己的实际信息。

---

## Lab 6 – 构建 Django Web 应用并接入负载均衡

### 1. 创建并登录 EC2 实例
1. 根据自己的学号查表确认分配的区域与 AMI。
2. 创建安全组并开放 22、80 端口：
   ```bash
   aws ec2 create-security-group --group-name <student-sg> --description "CITS5503 Lab6"
   aws ec2 authorize-security-group-ingress --group-name <student-sg> --protocol tcp --port 22 --cidr 0.0.0.0/0
   aws ec2 authorize-security-group-ingress --group-name <student-sg> --protocol tcp --port 80 --cidr 0.0.0.0/0
   ```
3. 创建 t2.micro Ubuntu 实例并绑定密钥对：
   ```bash
   aws ec2 run-instances --image-id <region-ami-id> \
     --count 1 --instance-type t2.micro \
     --key-name <student-key> --security-groups <student-sg>
   ```
4. 获取实例公网 IP 后通过 SSH 登录：
   ```bash
   ssh -i <student-key>.pem ubuntu@<Your-EC2-Public-IP>
   ```

### 2. 环境准备
```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y python3-venv nginx
sudo mkdir -p /opt/wwc/mysites && cd /opt/wwc/mysites
python3 -m venv myvenv
source myvenv/bin/activate
pip install django
```

### 3. 创建 Django 项目
```bash
django-admin startproject lab
cd lab
python3 manage.py startapp polls
```

修改 Django 文件：
- `polls/views.py`
- `polls/urls.py`
- `lab/urls.py`

确保 `polls` 应用返回 “Hello, world.” 页面，并在 `INSTALLED_APPS` 中注册。

### 4. 配置 nginx 与反向代理
编辑 `/etc/nginx/sites-enabled/default`：
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
重启 nginx：
```bash
sudo service nginx restart
```

### 5. 运行并验证
1. 启动 Django 开发服务器：`python3 manage.py runserver 8000`
2. 浏览器访问 `http://<Your-EC2-Public-IP>/polls/`，确认页面输出。
3. 按 Ctrl+C 停止服务器。

### 6. 创建并配置 ALB
1. 创建 Target Group（健康检查路径 `/polls/`）。
2. 创建 Application Load Balancer，选择至少两个子网并复用安全组。
3. 配置监听器 (HTTP:80) 并将 Target Group 注册至实例。
4. 访问 `http://<StudentID>-<random>.elb.<region>.amazonaws.com/polls/` 验证 ALB 转发。

### 7. 清理资源
停止 Django、终止 EC2、删除 ALB、Target Group 与安全组。

---

## Lab 7 – 使用 Fabric 实现远程自动化部署

### 1. 基础环境
1. 重复 Lab 6 的步骤创建新的 t2.micro 实例，并允许 22、80 端口。
2. 本地安装 Fabric：`pip install fabric invoke`。
3. 配置 `~/.ssh/config` 以便通过别名连接：
   ```text
   Host <student-alias>
       Hostname <Your-EC2-Public-DNS>
       User ubuntu
       IdentityFile ~/.ssh/<student-key>.pem
       StrictHostKeyChecking no
       UserKnownHostsFile /dev/null
   ```

### 2. Fabric 脚本关键函数
- `get_connection()`：返回基于别名的 SSH 连接。
- `setup_ec2()`：在远端创建目录、虚拟环境、安装依赖。
- `setup_django()`：同步 Django 项目文件（views、urls、settings）。
- `stop_server()` / `run_server()`：管理 Django 进程与日志。
- `check_site()`：使用 `curl` 验证网页响应。
- `deploy()`：调用上述函数形成自动化流水线。

脚本执行示例：
```python
from fabfile import deploy

if __name__ == "__main__":
    deploy()
```
执行后浏览器访问 `http://<Your-EC2-Public-IP>/polls/` 验证自动化部署成功。

### 3. 收尾
Fabric 完成验证后，关闭服务器并释放相关资源。

---

## Lab 8 – 构建用于 SageMaker Notebook 的容器

### 1. 编写 Dockerfile
```dockerfile
FROM python:3.10
RUN pip install jupyter boto3 sagemaker awscli
RUN mkdir /notebook
ENV JUPYTER_ENABLE_LAB=yes
ENV JUPYTER_TOKEN="CITS5503"
RUN jupyter notebook --generate-config
RUN echo "c.NotebookApp.ip = '0.0.0.0'" >> /root/.jupyter/jupyter_notebook_config.py
RUN wget -P /notebook https://raw.githubusercontent.com/zhangzhics/CITS5503_Sem2/master/Labs/src/LabAI.ipynb
WORKDIR /notebook
EXPOSE 8888
CMD ["jupyter", "notebook", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root"]
```

### 2. 构建镜像
```bash
docker build -t <student-id>-lab8 .
```

### 3. 运行容器并测试
```bash
docker run --rm -p 8888:8888 <student-id>-lab8
```
在浏览器打开 `http://127.0.0.1:8888/?token=CITS5503`，确认 Notebook 可用，最后停止容器。

---

## Lab 9 – 推送镜像至 ECR 并部署到 ECS

### 1. 使用 Boto3 创建或检查 ECR 仓库
```python
import boto3

def ensure_repo(repo_name: str) -> str:
    client = boto3.client("ecr")
    try:
        resp = client.describe_repositories(repositoryNames=[repo_name])
        return resp["repositories"][0]["repositoryUri"]
    except client.exceptions.RepositoryNotFoundException:
        resp = client.create_repository(repositoryName=repo_name)
        return resp["repository"]["repositoryUri"]

repository_uri = ensure_repo("<student-id>_ecr_repo")
print("ECR URI:", repository_uri)
```

### 2. 获取 Docker 登录命令
```python
import base64
import boto3

def get_login_command() -> str:
    client = boto3.client("ecr")
    token = client.get_authorization_token()["authorizationData"][0]
    username, password = base64.b64decode(token["authorizationToken"]).decode().split(":")
    return f"docker login -u {username} -p {password} {token['proxyEndpoint']}"

print(get_login_command())
```
执行输出的命令完成登录。

### 3. 推送镜像到 ECR
```bash
docker tag <student-id>-lab8:latest <AWS-ACCOUNT-ID>.dkr.ecr.<region>.amazonaws.com/<student-id>_ecr_repo:latest
docker push <AWS-ACCOUNT-ID>.dkr.ecr.<region>.amazonaws.com/<student-id>_ecr_repo:latest
```

### 4. 创建 ECS Task Definition
```python
import boto3

def register_task(image_uri: str, account_id: str, student_id: str):
    ecs = boto3.client("ecs")
    return ecs.register_task_definition(
        family=f"{student_id}-task-family",
        networkMode="awsvpc",
        requiresCompatibilities=["FARGATE"],
        cpu="256",
        memory="512",
        taskRoleArn=f"arn:aws:iam::{account_id}:role/{student_id}-task-role",
        executionRoleArn=f"arn:aws:iam::{account_id}:role/{student_id}-execution-role",
        containerDefinitions=[
            {
                "name": f"{student_id}-container",
                "image": image_uri,
                "essential": True,
                "portMappings": [
                    {"containerPort": 8888, "hostPort": 8888, "protocol": "tcp"}
                ],
                "environment": [
                    {"name": "JUPYTER_TOKEN", "value": "CITS5503"}
                ],
            }
        ],
    )
```

### 5. 部署至 ECS
1. 创建 VPC、子网、安全组（或复用现有网络资源）。
2. 创建 ECS 集群并选择 Fargate。
3. 基于上一步的 Task Definition 创建 Service，子网需启用公有 IP。
4. 通过 Service 暴露的负载均衡或公共 IP 访问 `8888` 端口，确认容器可用。

### 6. 清理
删除 ECS Service、Task Definition、ECR 镜像、ECR 仓库，并释放 IAM 角色与网络资源。

---

## 常见问题提示
- **权限不足**：检查 IAM 用户是否具备 EC2、ELB、ECR、ECS、IAM 等权限。
- **安全组未开放端口**：确保 22/80/8888 等端口在安全组和网络 ACL 中均已放行。
- **Django CSRF/ALLOWED_HOSTS 错误**：在 `settings.py` 中设置 `ALLOWED_HOSTS = ['*']`，并根据需要配置 `CSRF_TRUSTED_ORIGINS`。
- **Docker 推送失败**：确认已执行 ECR 登录命令且镜像名称与标签正确。

祝实验顺利完成！
