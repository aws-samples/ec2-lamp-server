## ec2-lamp-server
[AWS CloudFormation](https://aws.amazon.com/cloudformation/) template to provision [Amazon EC2](https://aws.amazon.com/ec2/) instance with PHP, Apache/Nginx, MySQL/MariaDB/PostgreSQL, i.e. LAMP, LEMP, LAPP or LEPP stack. 

## Description
[LAMP](https://aws.amazon.com/what-is/lamp-stack/) is an acronym for the operating system, Linux; the web server, Apache; the database server, MySQL (or MariaDB); and the programming language, PHP. It is a common open source web platform for many of the web's popular applications.  Variations include LEMP which replaces web server with Nginx, LAPP which replaces database server with PostgreSQL, and LEPP which uses Nginx and PostgreSQL. According to [W3Techs](https://w3techs.com/), PHP is used by more than [70%](https://w3techs.com/technologies/overview/programming_language), and either Nginx or Apache is used by more than [60%](https://w3techs.com/technologies/overview/web_server) of websites as of 2023.

This repo provides CloudFormation template to provision a EC2 instance with option to specify PHP version, web server engine (Apache or Nginx) and database engine (MySQL, MariaDB or PostgreSQL) to install. The EC2 instance can be used for software development or deployment of PHP based web applications such as [WordPress](https://wordpress.org/) and [Moodle](https://moodle.org/). 


## Architecture Diagram
<img alt="image" src="ec2-lamp-server.png">

## Overview of features
The template provides the following features:
- Choice of [Amazon Linux 2023](https://aws.amazon.com/linux/amazon-linux-2023/), [Amazon Linux 2](https://aws.amazon.com/amazon-linux-2/), [Ubuntu 22.04](https://ubuntu.com/aws) and [Ubuntu Pro 22.04](https://aws.amazon.com/about-aws/whats-new/2023/04/amazon-ec2-ubuntu-pro-subscription-model/)
- [NICE DCV](https://aws.amazon.com/hpc/dcv/) remote desktop streaming (Amazon Linux 2 and Ubuntu Linux)
- [Apache](https://www.apache.org/) or [Nginx](https://www.nginx.com/) web server
- [MySQL](https://www.mysql.com/), [MariaDB](https://mariadb.org/) or [PostgreSQL](https://www.postgresql.org/) database server
- [PHP 8](https://www.php.net/releases/8.1/en.php) with common PHP extensions (imagick, apcu, memcached, redis, igbinary, msgpack, lzf, lz4, zstd, libsodium, zip, etc)
- [Composer](https://getcomposer.org/)
- [Redis](https://redis.io/) and [Memcached](https://memcached.org/) in memory database
- [EC2 Instance Connect](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-linux-inst-eic.html) for SSH access
- [SSM Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html) for secure shell access 
- [AWS CodeDeploy](https://aws.amazon.com/codedeploy/) agent
- [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) agent
- [NFS](https://aws.amazon.com/efs/) client
- [AWS CLI v2](https://aws.amazon.com/cli/) with [auto-prompt](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-parameters-prompting.html)
- [MountPoint for Amazon S3](https://aws.amazon.com/s3/features/mountpoint/) for mounting S3 bucket as local file system
- (Optional) [Amazon S3](https://aws.amazon.com/s3/) bucket for use with Mountpoint with S3
- [Certbot](https://certbot.eff.org/) with apache, nginx and certbot-dns-route53 plugins
- (Optional) [Amazon Route 53](https://aws.amazon.com/route53/) hosted zone access for use with certbot-dns-route53 plugin
- [EC2 IAM role](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) for [NICE DCV license verification](https://docs.aws.amazon.com/dcv/latest/adminguide/setting-up-license.html#setting-up-license-ec2), [AWS Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/setup-instance-permissions.html), [CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create-iam-roles-for-cloudwatch-agent.html#create-iam-roles-for-cloudwatch-agent-roles), [Mountpoint for S3](https://github.com/awslabs/mountpoint-s3/blob/main/doc/CONFIGURATION.md#iam-permissions), [Code Deploy](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonEC2RoleforAWSCodeDeployLimited.html), [X-Ray](https://docs.aws.amazon.com/xray/latest/devguide/security_iam_service-with-iam.html#xray-permissions-aws) and [Route 53](https://certbot-dns-route53.readthedocs.io/en/stable/)
- (Optional) [AWS Backup](https://aws.amazon.com/backup/) to backup EC2 instance data
- (Optional) [Webmin](https://webmin.com/) web-based system administration



## Notice
Although this repository is released under the [MIT-0](LICENSE) license, its CloudFormation template uses features from 
[MySQL Community Edition](https://www.mysql.com/products/community/) and [Webmin](https://webmin.com/) which are licensed under [GPL](https://www.mysql.com/products/community/) and [BSD-3-Clause](https://webmin.com/about/) license respectively. 

Usage indicates acceptance of [NICE DCV EULA](https://www.nice-dcv.com/eula.html) and license agreements of all software that is installed in the EC2 instance. 


## Deployment via CloudFormation console
Download .yaml file for the desired operating system ([Amazon Linux 2](https://github.com/aws-samples/ec2-lamp-server/blob/main/AmazonLinux-2-LAMP-server.yaml), [Amazon Linux 2023](https://github.com/aws-samples/ec2-lamp-server/blob/main/AmazonLinux-2023-LAMP-server.yaml) or [Ubuntu/Ubuntu Pro 22.04 server](https://github.com/aws-samples/ec2-lamp-server/blob/main/UbuntuLinux-2204-LAMP-server.yaml)) 

Login to AWS [CloudFormation console](https://console.aws.amazon.com/cloudformation/home#/stacks/create/template). Choose **Create Stack**, **Upload a template file**, **Choose File**, select your .YAML file and choose **Next**. Enter a **Stack name** and specify parameters values.

EC2 instance
- `ec2Name`: EC2 instance name 
- `ec2KeyPair`: [EC2 key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) name. [Create key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html) if necessary
- `processorArchitecture`: Intel/AMD x86_64 or Graviton arm64. Default is `Graviton (arm64)`.
- `instanceType`: EC2 [instance types](https://aws.amazon.com/ec2/instance-types/). Do ensure type matches processor architecture. Default is `t4g.large` [burstable instance type](https://aws.amazon.com/ec2/instance-types/t4/). For best performance, consider [M6g](https://aws.amazon.com/ec2/instance-types/m6g/) or [M7g](https://aws.amazon.com/ec2/instance-types/m7g/) for general purpose workloads

Network
- `vpcID`: [VPC](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) with internet connectivity. Select [default VPC](https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html) if unsure
- `subnetID`: subnet with internet connectivity. Select subnet in default VPC if unsure
- `displayPublicIP`: set this to `No` if your EC2 instance will not receive [public IP address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-instance-addressing.html#concepts-public-addresses). EC2 private IP will be displayed in CloudFormation Outputs section instead. Default is `Yes`
- `assignStaticIP`: associates a static public IPv4 address using [Elastic IP address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html). Default is `Yes`

Remote Administration
- `ingressIPv4`: allowed IPv4 internet source prefix to SSH and NICE DCV ports, e.g. `1.2.3.4/32`. You can get your source IP from [https://checkip.amazonaws.com](https://checkip.amazonaws.com). Use `127.0.0.1/32` to block incoming access from public internet. Default is `0.0.0.0/0`. 
- `ingressIPv6`: allowed IPv6 internet source prefix to SSH and NICE DCV ports. Use `::1/128` to block all incoming IPv6 access. Default is `::/0`
- `allowSSHport`: allow inbound SSH from `ingressIPv4` and `ingressIPv6`. Option does not affect [EC2 Instance Connect](https://aws.amazon.com/blogs/compute/new-using-amazon-ec2-instance-connect-for-ssh-access-to-your-ec2-instances/) access. Default is `Yes`
- `installDCV`: install graphical desktop environment and [NICE DCV](https://aws.amazon.com/hpc/dcv/) server. Default is `Yes`
- `installWebmin`: install [Webmin](https://webmin.com/) web-based system administration tool. Default is `No`

LAMP
- `webOption`: `Apache`, `Nginx` web server or `none`.
- `phpVersion`: PHP version to install or `none`. PHP version 8.2 and higher for Ubuntu will use [ppa:ondrej/php](https://launchpad.net/~ondrej/+archive/ubuntu/php/) PPA 
- `databaseOption`: `MySQL`, `MariaDB`, `PostgreSQL` database server or `none`. MySQL option for Amazon Linux will attempt to use [MySQL Community Edition](https://www.mysql.com/products/community/) repository, where MySQL root password can be retrieved using the command `sudo grep password /var/log/mysqld.log`. Select `none` if using external database such as [Amazon RDS](https://aws.amazon.com/rds/).
- `s3BucketName` (optional): name of [Amazon S3](https://aws.amazon.com/s3/) bucket to grant EC2 instance access using [IAM policy](https://aws.amazon.com/blogs/security/writing-iam-policies-how-to-grant-access-to-an-amazon-s3-bucket/).  Leave text field empty not to grant access. A `*` value will grant the EC2 instance access to all S3 buckets in your AWS account and is usually not recommended. Default is empty.
- `r53ZoneID` (optional): [Amazon Route 53](https://aws.amazon.com/route53/) hosted zone ID to grant access to. Enable this if your DNS is on Route 53 in your AWS account and you want to use Certbot with [certbot-dns-route53](https://certbot-dns-route53.readthedocs.io/) plugin to get HTTPS certificate. A `*` value will grant access to all Route 53 zones in your AWS account. Permission is restricted to **_acme-challenge.\*** TXT DNS records using [resource record set permissions](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-permissions.html). Default is empty. 

EBS
- `volumeSize`: [Amazon EBS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html) volume size
- `volumeType`: [EBS General Purpose Volume](https://aws.amazon.com/ebs/general-purpose/) type

AWS Backup
- `enableBackup`: EC2 data protection with AWS Backup. Default is `Yes`
- `scheduleExpression`: start time of backup using [CRON expression](https://en.wikipedia.org/wiki/Cron#CRON_expression). Default is 1 am
- `scheduleExpressionTimezone`: timezone in which the schedule expression is set. Default is `Etc/UTC`
- `deleteAfterDays`:  number of days after backup creation that a recovery point is deleted. Default is `7`

### CloudFormation Outputs
The following are available on **Outputs** section 
- `EC2console`: EC2 console URL link to start/stop your EC2 instance or to get the latest IPv4 (or IPv6 if enabled) address.
- `EC2instanceConnect`: [EC2 Instance Connect](https://aws.amazon.com/blogs/compute/new-using-amazon-ec2-instance-connect-for-ssh-access-to-your-ec2-instances/) URL link.  Functionality is only available under [certain conditions](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-connect-prerequisites.html).
- `SSMsessionManager` or `SSMsessionManagerDCV`: [SSM Session Manager](https://aws.amazon.com/blogs/aws/new-session-manager/) URL link
- `WebUrl`: EC2 web server URL link
- `DCVwebConsole`: NICE DCV web browser client URL link#. Login as the user specified in *Description* field 
- `WebminUrl` (where applicable): Webmin URL link. Set the root password by running `sudo passwd root` using `EC2instanceConnect`, `SSMsessionManager` or SSH session first.  

#Native NICE DCV clients can be downloaded from [https://download.nice-dcv.com/](https://download.nice-dcv.com/).
Web browser client can be disabled by removing `nice-dcv-web-viewer` package.


### PHP performance configuration
Based on public articles about PHP performance (many thanks to the authors), the following enhancements were made:

- Default processor architecture option is [Graviton](https://aws.amazon.com/ec2/graviton/) as per [arm64 vs x86_64 for php](https://fraudmarc.com/post/arm64-vs-x86-64-for-php)
- PHP [OPcache](https://www.php.net/manual/en/book.opcache.php) and [JIT](https://php.watch/versions/8.0/JIT) enabled: from [Make your PHP 8 apps twice as fast (OPCache & JIT)](https://medium.com/@edouard.courty/make-your-php-8-apps-twice-as-fast-opcache-jit-8d3542276595)
- [FastCGI Process Manager (FPM)](https://www.php.net/manual/en/install.fpm.php): from [PHP-FPM Cuts Web App Loading Times by 300%](https://www.cloudways.com/blog/php-fpm-on-cloud/)
- [Apache MPM Event](https://httpd.apache.org/docs/2.4/mod/event.html): from [Apache Performance Tuning: MPM Modules](https://www.liquidweb.com/kb/apache-performance-tuning-apache-mpm-modules/#best)
- Redis session store: from [Highly Performant PHP Sessions with Redis](https://levelup.gitconnected.com/highly-performant-php-sessions-with-redis-b2dc17b4f4e4)
- Serialisation (igbinary,msgpack) and compression (lzf,zstd,lz4) extensions to reduce Redis/Memcached network traffic: from [Strategies for Reducing Big Redis Traffic in Laravel](https://world.hey.com/otar/strategies-for-reducing-big-redis-traffic-in-laravel-a168f96a)
- PHP [OPcache file cache](https://www.php.net/manual/en/opcache.configuration.php#ini.opcache.file-cache) configured as per [PHP Opcache file cache](https://patrickkerrigan.uk/blog/php-opcache-file-cache/) but not enabled. To enable, edit `/etc/php.d/10-opcache.ini` (Amazon Linux) or `/etc/php/8.1/fpm/php.ini` (Ubuntu) file to uncomment the line beginning with `opcache.file_cache=/var/www/.opcache` and restart php-fpm.



## Using Cloudwatch agent
[Amazon CloudWatch agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html) is installed, and enables collection of [EC2 system-level metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/metrics-collected-by-CloudWatch-agent.html) and [AWS X-Ray](https://aws.amazon.com/xray/) traces. 

The template configures agent to collect memory utilization metrics. You can configure Cloudwatch agent to collect other data as follows.

### Create agent configuration file
Create agent configuration file. You can use [agent configuration file wizard](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create-cloudwatch-agent-configuration-file-wizard.html):
```
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```
### Start Cloudwatch agent
After `config.json` file is created, [start CloudWatch agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/install-CloudWatch-Agent-on-EC2-Instance-fleet.html#start-CloudWatch-Agent-EC2-fleet):
```
sudo systemctl enable amazon-cloudwatch-agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json
```

Refer to [How do I install and configure the unified CloudWatch agent to push metrics and logs from my EC2 instance to CloudWatch?](https://repost.aws/knowledge-center/cloudwatch-push-metrics-unified-agent) for more details.


## Using Certbot to obtain TLS certificate
Refer to [Certbot site](https://certbot.eff.org/pages/about) for [help](https://certbot.eff.org/pages/help) with this tool.  

Create a DNS record entry that resolves to your EC2 instance IP address, and ensure `assignStaticIP` is configured to `Yes` in your CloudFormation stack. 


### Using certbot-dns-route53 plugin 

Ensure that you have granted Route 53 hosted zone access by specifying `r53ZoneID` value in your CloudFormation stack. From terminal, run the below command based on installed web server type and follow instructions.

- Apache
  ```
  sudo certbot --dns-route53 --installer apache
  ```
- Nginx
  ```
  sudo certbot --dns-route53 --installer nginx
  ```
  

### Using apache plugin

- From terminal, run the below command and follow instructions.
  ```
  sudo certbot --apache
  ```
  
### Using nginx plugin

- From terminal, run the below command and follow instructions.
  ```
  sudo certbot --nginx
  ```
 


## About EC2 instance

### Well-architected
To improve performance, reliability, scalability, high availability and functionality, EC2 instance can be extended to use other services such as [Amazon RDS](https://aws.amazon.com/rds/), [Amazon S3](https://aws.amazon.com/s3/), [Amazon ElastiCache](https://aws.amazon.com/elasticache/) and [Amazon EFS](https://aws.amazon.com/efs/), and with [AWS SDK for PHP](https://aws.amazon.com/sdk-for-php/). Some useful resources that can help with the integration include:
- [AWS Well-Architected](https://aws.amazon.com/architecture/well-architected/)
- [AWS Architecture Center](https://aws.amazon.com/architecture) including [Scaling PHP Applications on AWS](https://d1.awsstatic.com/architecture-diagrams/ArchitectureDiagrams/scaling-PHP-applications-on-AWS-ra.pdf)
- [PHP on AWS](https://aws.amazon.com/developer/language/php/)
- [Moodle Reference Architecture](https://docs.aws.amazon.com/architecture-diagrams/latest/moodle-learning-management-system-on-aws/moodle-learning-management-system-on-aws.html)
- [Best Practices for WordPress on AWS](https://docs.aws.amazon.com/whitepapers/latest/best-practices-wordpress/reference-architecture.html)

### Securing
To futher secure your EC2 instance, you may want to
- Remove NICE DCV web browser client
- Restrict remote administration access to your IP address only (`ingressIPv4` and `ingressIPv6`)
- Disable SSH access from public internet. Use [EC2 Instance Connect](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-connect-methods.html#ec2-instance-connect-connecting-console) or [SSM Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-sessions-start.html#start-ec2-console) for in-browser terminal access. If you have [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) and [Session Manager plugin for the AWS CLI](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html) installed, you can start a session using [AWS CLI](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-sessions-start.html#sessions-start-cli) or [SSH](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-sessions-start.html#sessions-start-ssh)
- Enable [AWS Backup](https://aws.amazon.com/backup). Enable [AWS Backup Vault Lock](https://aws.amazon.com/blogs/storage/enhance-the-security-posture-of-your-backups-with-aws-backup-vault-lock/) for enhanced data protection. Refer to [documentation](https://docs.aws.amazon.com/aws-backup/latest/devguide/restoring-ec2.html) for instructions on restoring EC2 instance. 
- Use [Amazon CloudFront](https://aws.amazon.com/cloudfront/) with [AWS WAF](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-web-awswaf.html) to protect your instance from DDoS attacks and common web threats. The AWS blog [Accelerate and protect your websites using Amazon CloudFront and AWS WAF](https://aws.amazon.com/blogs/networking-and-content-delivery/accelerate-and-protect-your-websites-using-amazon-cloudfront-and-aws-waf/) and [CloudFront dynamic websites](https://github.com/aws-samples/amazon-cloudfront-dynamic-websites) CloudFormation template may help with CloudFront distribution setup. When using CloudFront, you can restrict your EC2 instance HTTP and HTTPS port access to CloudFront IPs only. The CloudFormation template creates additional inbound HTTP and HTTPS security groups with [AWS-managed prefix list for Amazon CloudFront](https://aws.amazon.com/blogs/networking-and-content-delivery/limit-access-to-your-origins-using-the-aws-managed-prefix-list-for-amazon-cloudfront/) as source where possible.
- Enable [Amazon Inspector](https://aws.amazon.com/inspector/) to scan EC2 instance for software vulnerabilities and unintended network exposure
- Enable [Amazon GuardDuty](https://aws.amazon.com/guardduty/) security monitoring service with [Malware Protection](https://docs.aws.amazon.com/guardduty/latest/ug/malware-protection.html) to detect the potential presence of malware in EBS volumes



## Clean Up
To remove created resources, you will need to
- [Disable](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_ChangingDisableAPITermination.html) EC2 instance termination protection
- [Delete](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html) CloudFormation stack


## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

