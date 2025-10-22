# Terminal Session Log

This log captures the commands attempted in the container along with their outputs so you can see what happens when running the Lab 6 AWS automation steps in this environment.

## Attempt to provision EC2 instance

```bash
aws ec2 run-instances --image-id ami-0eeab253db7e765a9 \
  --count 1 --instance-type t2.micro \
  --key-name 25500725-key --security-groups 25500725-sg
```

**Output**
```
bash: command not found: aws
```

## Attempt to install AWS CLI via pip

```bash
pip install awscli
```

**Output**
```
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'ProxyError('Cannot connect to proxy.', OSError('Tunnel connection failed: 403 Forbidden'))': /simple/awscli/
WARNING: Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'ProxyError('Cannot connect to proxy.', OSError('Tunnel connection failed: 403 Forbidden'))': /simple/awscli/
WARNING: Retrying (Retry(total=2, connect=None, read=None, redirect=None, status=None)) after connection broken by 'ProxyError('Cannot connect to proxy.', OSError('Tunnel connection failed: 403 Forbidden'))': /simple/awscli/
WARNING: Retrying (Retry(total=1, connect=None, read=None, redirect=None, status=None)) after connection broken by 'ProxyError('Cannot connect to proxy.', OSError('Tunnel connection failed: 403 Forbidden'))': /simple/awscli/
WARNING: Retrying (Retry(total=0, connect=None, read=None, redirect=None, status=None)) after connection broken by 'ProxyError('Cannot connect to proxy.', OSError('Tunnel connection failed: 403 Forbidden'))': /simple/awscli/
ERROR: Could not find a version that satisfies the requirement awscli (from versions: none)
ERROR: No matching distribution found for awscli
```

These errors show that the container does not have outbound internet access to install the AWS CLI, so the Lab 6 commands requiring AWS cannot be executed here.
