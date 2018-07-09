# SecurityMonkey AWS CloudFormation Template

Stand up an AWS CloudFormation stack running [SecurityMonkey](https://github.com/Netflix/security_monkey) v0.8.0 (2017-02-21)

This CloudFormation stack creates a single EC2 instance in a public subnet of a VPC, running SecurityMonkey and storing data in an Postgresql RDS DB in MultiAZ configuration. Both the EC2 instance and RDS db will be created and configured with security groups. A VPC and subnets will be also created.

## AWS resources it creates
- VPC with a public subnet in AZ1 and 2 RDS subnets in AZ1 and AZ2.
- An EC2 instance with Security Monkey installed: you can see all steps to install Security Monkey in the template, a self signed SSL certificate is created as part of the insallation script.
- RDS Postgresql MultiAZ instance
- Security Group for Security Monkey Server allowing SSH and HTTPS traffic from the specified network at deployment time. Another SG for RDS to allow the EC2 instance talk to the RDS instance.


This template downloads, installs and configures Security Monkey (master branch) and all its dependences on top of an official Ubuntu 14.04LTS AMI.

## Requirements

- AWS Account
- Access to the the AWS Admin Console UI.
- Key to deploy EC2 instances
- Preconfigured SecurityMonkeyInstanceProfile role. Follow all of these instructions: https://github.com/Netflix/security_monkey/blob/master/docs/quickstart.rst#setup-iam-roles
- Accept the Ubuntu terms and conditions in AWS Marketplace here http://aws.amazon.com/marketplace/pp?sku=b3dl4415quatdndl4qa6kcu45

## Stack Parameters

You must provide values for the following parameters (look at the template for descriptions):

- AZ1
- AZ2
- SecurityMonkeyInstanceProfileRole
- SecurityMonkeyInstanceType
- KeyName
- RDSDBName
- RDSUsername
- RDSPassword
- RDSPort
- RDSInstanceType
- RDSAllocatedStorage
- FromNetwork
- SMLogLevel
- SMMailDefaultSender
- SMSecurityTeamEmail
- SMUseSMTP
- SMMailServer
- SMMailPort
- SMMailUserName
- SMMailPassword

To customize the security profile of your stack, also override the default values of those parameters.

## After deployment steps

- SSH to your Security Monkey Server and add your admin user using the Security Monkey manage.py tool.
```bash
ssh -i yourkey.pem ubuntu@PUBLIC_IP
```
- Add an admin user to Security Monkey:
```bash
export SECURITY_MONKEY_SETTINGS=/usr/local/src/security_monkey/env-config/config-deploy.py
cd /usr/local/src/security_monkey
python manage.py create_user you@example.com Admin
```
- Add your first AWS account (this can also be done through the web app):
```bash
python manage.py add_account --number AWSaccountID --name AccountName
```
- Each time you add an account securitymonkeyscheduler has to be restarted manually:
```bash
$ sudo supervisorctl
supervisor> restart securitymonkeyscheduler
```
- If you haven't set up the email configuration properly at deployment you can do it by editing /usr/local/src/security_monkey/env-config/config-deploy.py and then restarting supervisor.
```bash
$ sudo service supervisor restart
```
- Security Monkey logs are in /var/log/security_monkey
- Supervisor logs are in /var/log/supervisor

## Production considerations

- Instance where Security Monkey Server runs will have a Public IP address to make the access easier but it is not mandatory and not recommended for production.
- Security Monkey Log Level should not be DEBUG, INFO will be fine.
- Use you own SSL certificate instead of a self signed certificate.
- Read this: http://securitymonkey.readthedocs.io/en/latest/quickstart.html#productionalizing-security-monkey


## Additional information
- For more information about Security Monkey please refer to the official documentation site: http://securitymonkey.readthedocs.io/en/latest/

## Thanks
Part of this template is inspired on another template from https://github.com/trumant/securitymonkey_cloudformation but updated without using Ruby and with few more options in order to allow Security Monkey configuration from the template deployment form.
