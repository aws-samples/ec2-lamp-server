## ec2-lamp-server
Sample [AWS CloudFormation](https://aws.amazon.com/cloudformation/) template to provision [Amazon EC2](https://aws.amazon.com/ec2/) instance with LAMP or LAPP stack.

## Description
[LAMP](https://aws.amazon.com/what-is/lamp-stack/) is an acronym for Linux, Apache, MySQL/MariaDB and PHP. It is a common open source web platform for many of the web's popular applications.  A variation is the LAPP stack which include Linux, Apache, PostgreSQL (instead of MySQL) and PHP. 

This repo provides starter CloudFormation template to provision single EC2 LAMP or LAPP server instance. The EC2 instance can be used for software development, or deployment of applications that use LAMP stack for use cases where factors such as high availablility (HA) and scalability are not a primary priority. 

For use cases that require high performance, reliability, scalability and high availability, users should re-architect their LAMP applications. Some resources that can help with the design includes:
- [AWS Well-Architected](https://aws.amazon.com/architecture/well-architected/)
- [AWS Architecture Center](https://aws.amazon.com/architecture) including [Scaling PHP Applications on AWS](https://d1.awsstatic.com/architecture-diagrams/ArchitectureDiagrams/scaling-PHP-applications-on-AWS-ra.pdf)
- [Moodle Reference Architecture](https://docs.aws.amazon.com/architecture-diagrams/latest/moodle-learning-management-system-on-aws/moodle-learning-management-system-on-aws.html)
- [Best Practices for WordPress on AWS](https://docs.aws.amazon.com/whitepapers/latest/best-practices-wordpress/reference-architecture.html)

## Overview of features
The template installs the following
- Graphical desktop with [NICE DCV](https://aws.amazon.com/hpc/dcv/) server for secure remote access
- [Apache web server](https://www.apache.org/)
- Choice of [MySQL](https://www.mysql.com/), [MariaDB](https://mariadb.org/) and [PostgreSQL](https://www.postgresql.org/) database server
- [PHP 8.1](https://www.php.net/releases/8.1/en.php) with common PHP extensions
- [Redis](https://redis.io/) and [Memcached](https://memcached.org/) in memory database
- [SSM Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html) for secure remote terminal access
- [CodeDeploy](https://aws.amazon.com/codedeploy/) agent
- [CloudWatch](https://aws.amazon.com/cloudwatch/) agent
- [NFS](https://aws.amazon.com/efs/) client
- [Amazon FSx for Lustre](https://aws.amazon.com/fsx/lustre/) client
- [AWS CLI v2](https://aws.amazon.com/blogs/developer/aws-cli-v2-is-now-generally-available/) with [auto-prompt](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-parameters-prompting.html)
- (Optional) S3 bucket access : access to specific S3 bucket via [EC2 IAM role](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
- [MountPoint for Amazon S3](https://aws.amazon.com/blogs/aws/mountpoint-for-amazon-s3-generally-available-and-ready-for-production-workloads/) 
- [Certbot](https://certbot.eff.org/)
- (Optional) Route 53 hosted zone access : access to specific Route 53 hosted zone via [EC2 IAM role](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) for use with certbot-dns-route53 plugin

  
Note that use of cloudformation template indicates acceptance of license agreements of all software that is installed in the EC2 instance. 


### PHP Configuration
Based on public articles about PHP performance (many thanks to the authors), the following changes were made to PHP configuration

- PHP [OPcache](https://www.php.net/manual/en/book.opcache.php) and [JIT](https://php.watch/versions/8.0/JIT) enabled: from [PHP 8.x: A Deep Dive for General PHP Performance Improvement Features](https://accesto.com/blog/php-performance-improvement-features/)
- [FastCGI Process Manager (FPM)](https://www.php.net/manual/en/install.fpm.php): from [PHP-FPM Cuts Web App Loading Times by 300%](https://www.cloudways.com/blog/php-fpm-on-cloud/) 
- [Apache MPM Event](https://httpd.apache.org/docs/2.4/mod/event.html) module
- Redis session store: from [Highly Performant PHP Sessions with Redis](https://levelup.gitconnected.com/highly-performant-php-sessions-with-redis-b2dc17b4f4e4)

## Deployment via CloudFormation console
Download desired .yaml file based on the operating system ([Amazon Linx 2](https://aws.amazon.com/amazon-linux-2/) or [Ubuntu Linux 22.04 LTS server](https://releases.ubuntu.com/jammy/)) 

Login to AWS [CloudFormation console](https://console.aws.amazon.com/cloudformation/home#/stacks/create/template). Choose **Create Stack**, **Upload a template file**, **Choose File**, select your .YAML file and choose **Next**. Specify a **Stack name** and specify parameters values. 

EC2 section
- `processorArchitecture`: Intel/AMD or [Graviton ARM64](https://aws.amazon.com/ec2/graviton/). Default is `Graviton ARM64 (aarch64)`
- `instanceType`: EC2 [instance types](https://aws.amazon.com/ec2/instance-types/). Do ensure value matches CPU architecture. Default is `t4g.large`
- `ec2Name`: EC2 instance name 
- `ec2KeyPair`: [EC2 key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) name. Value is mandatory
- `volumeSize`: [Amazon EBS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html) volume size

VPC section
- `vpcID`: [VPC](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) with internet connectivity. Select [default VPC](https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html) if unsure
- `subnetID`: subnet with internet connectivity. Select subnet in default VPC if unsure. If you specify a different `instanceType`, ensure that it is available in AZ subnet you select. 
- `assignStaticIP`: associates a static public IPv4 address using [Elastic IP address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html). Default is `Yes`
- `displayPublicIP`: set this to `No` if you provision EC2 instance in a subnet that will not receive [public IP address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-instance-addressing.html#concepts-public-addresses). EC2 private IP will be displayed in CloudFormation Outputs section instead. Default is `Yes`

LAMP configuration
- `databaseOption`: option to install database engine of choice;  MySQL, MariaDB, PostgreSQL or none. For Amazon Linux 2, template will use [MySQL Community Edition](https://www.mysql.com/products/community/) repository to install MySQL Server with randomly generated password stored in `\var\log\mysql.log` file
- `s3BucketName` (optional): name of [Amazon S3](https://aws.amazon.com/s3/) bucket to grant EC2 instance to via IAM policy as per [Writing IAM Policies: How to Grant Access to an Amazon S3 Bucket](https://aws.amazon.com/blogs/security/writing-iam-policies-how-to-grant-access-to-an-amazon-s3-bucket/).  Leave text field empty not to grant access. A `*` value will grant EC2 instance to all S3 buckets in your AWS account and is not recommended. Default is empty.
- `r53ZoneID` (optional): [Amazon Route 53](https://aws.amazon.com/route53/) hosted zone ID to grant EC2 instance to. This is to be used if your domain DNS is hosted by Route 53 and you want to use Certbot to get SSL/TLS certificate using [dns_route53](https://certbot-dns-route53.readthedocs.io/) plugin

Remote Administration
- `ingressIPv4`: allowed IPv4 source prefix, e.g. `1.2.3.4/32`. Get source IP from [https://checkip.amazonaws.com](https://checkip.amazonaws.com)
- `ingressIPv6`: allowed IPv6 source prefix. Use `::1/128` to block all incoming IPv6 access


### CloudFormation Outputs
The following are available on **Outputs** section 
- `SSMSessionManager`: SSM Session Manager URL link to change login user password. Password change command is in *Description* field.
- `DCVwebConsole`: NICE DCV web browser client URL link. Login as the user specified in *Description* field# 
- `EC2Instance`: EC2 console URL link to start/stop your EC2 instance or to get the latest IPv4 (or IPv6 if enabled) address.
- `WebUrl`: EC2 web server URL link

#Native NCIE DCV clients can be downloaded from [https://download.nice-dcv.com/](https://download.nice-dcv.com/).
Web browser client can be disabled by removing `nice-dcv-web-viewer` package.


## Using Certbot to obtain LetsEncrypt certificate
Do visit [Let's Encrypt](https://letsencrypt.org/) site if you are not familiar with [Certbot](https://certbot.eff.org/)

### Using Certbot with apache plugin
Ensure `assignStaticIP` is configured to `Yes` in your CloudFormation stack and a DNS entry is associated with your EC2 instance IP address.

- From terminal, run the below command and follow instructions
  ```
  sudo certbot --apache
  ```

### Using Certbot with dns_route53 plugin
Ensure that you have granted Route 53 hosted zone access by specifying `r53ZoneID` value in your CloudFormation stack, and a DNS entry is associated with your EC2 instance IP address.

- From terminal, execute the below command and follow instructions.  
  ```
  sudo certbot certonly --dns-route53
  ```
  More information from [certbot-dns-route53 documentation site](https://certbot-dns-route53.readthedocs.io)

- After certificate is issued, execute command
  ```
  sudo chmod 755 /etc/letsencrypt/live
  ```

- Modify Apache SSL file (`/etc/httpd/conf.d/ssl.conf` for Amazon Linux 2, `/etc/apache2/sites-available/default-ssl.conf` for Ubuntu Linux) as follows
  ```
  SSLCertificateFile /etc/letsencrypt/live/<CERT-NAME>/cert.pem
  SSLCertificateKeyFile /etc/letsencrypt/live/<CERT-NAME>/privkey.pem
  SSLCertificateChainFile /etc/letsencrypt/live/<CERT-NAME>/chain.pem
  ```
  Replace `<CERT-NAME>` with the actual value in your `/etc/letsencrypt/live` folder.

- Restart Apache
  ```
  sudo systemctl restart httpd
  ```

More information from [Certbot documentation site](https://eff-certbot.readthedocs.io/en/stable/using.html#where-are-my-certificates)


## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

