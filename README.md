## ec2-lamp-server
Sample [AWS CloudFormation](https://aws.amazon.com/cloudformation/) templates to provision [Amazon EC2](https://aws.amazon.com/ec2/) instance with LAMP, LAPP or LEMP stack.

## Description
[LAMP](https://aws.amazon.com/what-is/lamp-stack/) is an acronym for the operating system, Linux; the web server, Apache; the database server, MySQL; and the programming language, PHP. It is a common open source web platform for many of the web's popular applications.  Variations include LAPP which uses PostgreSQL as the database server and LEMP which uses Nginx as web server. 

This repo provides starter CloudFormation template to provision a EC2 LAMP, LAPP or LEMP server instance. The EC2 instance can be used for software development, or deployment of LAMP/LAPP/LEMP stack based applications for use cases where factors such as high availablility (HA) and scalability are not a primary priority. 

For use cases that require high performance, reliability, scalability and high availability, users should re-architect their LAMP applications. Some resources that can help with the design includes:
- [AWS Well-Architected](https://aws.amazon.com/architecture/well-architected/)
- [AWS Architecture Center](https://aws.amazon.com/architecture) including [Scaling PHP Applications on AWS](https://d1.awsstatic.com/architecture-diagrams/ArchitectureDiagrams/scaling-PHP-applications-on-AWS-ra.pdf)
- [Moodle Reference Architecture](https://docs.aws.amazon.com/architecture-diagrams/latest/moodle-learning-management-system-on-aws/moodle-learning-management-system-on-aws.html)
- [Best Practices for WordPress on AWS](https://docs.aws.amazon.com/whitepapers/latest/best-practices-wordpress/reference-architecture.html)

## Overview of features
The template provides the following features:
- Graphical desktop with [NICE DCV](https://aws.amazon.com/hpc/dcv/) server for secure remote access
- [Apache](https://www.apache.org/) or [Nginx](https://www.nginx.com/) web server
- [MySQL](https://www.mysql.com/), [MariaDB](https://mariadb.org/) or [PostgreSQL](https://www.postgresql.org/) database server
- [PHP 8.1](https://www.php.net/releases/8.1/en.php) with common PHP extensions
- [Redis](https://redis.io/) and [Memcached](https://memcached.org/) in memory database
- [SSM Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html) for secure remote terminal access
- [AWS CodeDeploy](https://aws.amazon.com/codedeploy/) agent
- [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) agent
- [NFS](https://aws.amazon.com/efs/) client
- [Amazon FSx for Lustre](https://aws.amazon.com/fsx/lustre/) client
- [AWS CLI v2](https://aws.amazon.com/cli/) with [auto-prompt](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-parameters-prompting.html)
- [MountPoint for Amazon S3](https://aws.amazon.com/blogs/aws/mountpoint-for-amazon-s3-generally-available-and-ready-for-production-workloads/) 
- (Optional) [Amazon S3](https://aws.amazon.com/s3/) bucket access via [EC2 IAM role](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
- [Certbot](https://certbot.eff.org/)
- (Optional) [Amazon Route 53](https://aws.amazon.com/route53/) hosted zone access via [EC2 IAM role](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) for use with certbot-dns-route53 plugin

  
Note that use of cloudformation template indicates acceptance of license agreements of all software that is installed in the EC2 instance. 


### PHP Configuration
Based on public articles about PHP performance (many thanks to the authors), the following changes were made to PHP configuration

- PHP [OPcache](https://www.php.net/manual/en/book.opcache.php) and [JIT](https://php.watch/versions/8.0/JIT) enabled: from [Make your PHP 8 apps twice as fast (OPCache & JIT)](https://medium.com/@edouard.courty/make-your-php-8-apps-twice-as-fast-opcache-jit-8d3542276595)
- [FastCGI Process Manager (FPM)](https://www.php.net/manual/en/install.fpm.php): from [PHP-FPM Cuts Web App Loading Times by 300%](https://www.cloudways.com/blog/php-fpm-on-cloud/) 
- [Apache MPM Event](https://httpd.apache.org/docs/2.4/mod/event.html)
- Redis session store: from [Highly Performant PHP Sessions with Redis](https://levelup.gitconnected.com/highly-performant-php-sessions-with-redis-b2dc17b4f4e4)

CloudFormation default processor architecture option is Graviton as per [arm64 vs x86_64 for php](https://fraudmarc.com/post/arm64-vs-x86-64-for-php). 


## Deployment via CloudFormation console
Download .yaml file for your operating system ([Amazon Linux 2](https://aws.amazon.com/amazon-linux-2/) or [Ubuntu Linux 22.04 LTS server](https://releases.ubuntu.com/jammy/)) 

Login to AWS [CloudFormation console](https://console.aws.amazon.com/cloudformation/home#/stacks/create/template). Choose **Create Stack**, **Upload a template file**, **Choose File**, select your .YAML file and choose **Next**. Edit a **Stack name** and specify parameters values. 

EC2
- `processorArchitecture`: Intel/AMD or [Graviton ARM64](https://aws.amazon.com/ec2/graviton/). Default is `Graviton ARM64 (aarch64)`
- `instanceType`: EC2 [instance types](https://aws.amazon.com/ec2/instance-types/). Do ensure type matches processor architecture. Default is `t4g.large` [burstable instance type](https://aws.amazon.com/ec2/instance-types/t4/). For best performance, use [M6g](https://aws.amazon.com/ec2/instance-types/m6g/) or [M7g](https://aws.amazon.com/ec2/instance-types/m7g/)
- `ec2Name`: EC2 instance name 
- `ec2KeyPair`: [EC2 key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) name. [Create key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html) if necessary
- `volumeSize`: [Amazon EBS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html) volume size

VPC
- `vpcID`: [VPC](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) with internet connectivity. Select [default VPC](https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html) if unsure
- `subnetID`: subnet with internet connectivity. Select subnet in default VPC if unsure
- `assignStaticIP`: associates a static public IPv4 address using [Elastic IP address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html). Default is `Yes`
- `displayPublicIP`: set this to `No` if you provision EC2 instance in a subnet that will not receive [public IP address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-instance-addressing.html#concepts-public-addresses). EC2 private IP will be displayed in CloudFormation Outputs section instead. Default is `Yes`

LAMP configuration
- `webOption`: option to select Apache or Nginx web server
- `databaseOption`: option to install either MySQL, MariaDB, PostgreSQL database server or none. MySQL option for Amazon Linux 2 use [MySQL Community Edition](https://www.mysql.com/products/community/) repository, where MySQL root password can be retrieved using the command `sudo grep password /var/log/mysqld.log`.
- `s3BucketName` (optional): name of [Amazon S3](https://aws.amazon.com/s3/) bucket to grant EC2 instance to [via IAM policy](https://aws.amazon.com/blogs/security/writing-iam-policies-how-to-grant-access-to-an-amazon-s3-bucket/).  Leave text field empty not to grant access. A `*` value will grant the EC2 instance access to all S3 buckets in your AWS account and is not recommended. Default is empty.
- `r53ZoneID` (optional): [Amazon Route 53](https://aws.amazon.com/route53/) hosted zone ID to grant access to. Enable this if your DNS is on Route 53 and you want to use Certbot with [certbot-dns-route53](https://certbot-dns-route53.readthedocs.io/) plugin to get HTTPS certificates. A `*` value will grant access to all Route 53 zones in your AWS account. Permission is restricted to TXT DNS records only using [resource record set permissions](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-permissions.html). Default is empty. 

Remote Administration
- `ingressIPv4`: allowed IPv4 source prefix to SSH and NICE DCV ports, e.g. `1.2.3.4/32`. Get source IP from [https://checkip.amazonaws.com](https://checkip.amazonaws.com). Default is `0.0.0.0/0`
- `ingressIPv6`: allowed IPv6 source prefix to SSH and NICE DCV ports. Use `::1/128` to block all incoming IPv6 access. Default is `::/0`


### CloudFormation Outputs
The following are available on **Outputs** section 
- `SSMSessionManager`: SSM Session Manager URL link. Use this for terminal access and to change login user password. Password change command is in *Description* field.
- `DCVwebConsole`: NICE DCV web browser client URL link#. Login as the user specified in *Description* field 
- `EC2Instance`: EC2 console URL link to start/stop your EC2 instance or to get the latest IPv4 (or IPv6 if enabled) address.
- `WebUrl`: EC2 web server URL link

#Native NICE DCV clients can be downloaded from [https://download.nice-dcv.com/](https://download.nice-dcv.com/).
Web browser client can be disabled by removing `nice-dcv-web-viewer` package.


## Using Certbot to obtain HTTPS certificate
Please refer to [Certbot site](https://certbot.eff.org/pages/about) if you are not familiar and/or [need help](https://certbot.eff.org/pages/help) with this tool.  

Create a DNS A record entry that resolves to your EC2 instance IP address, and ensure `assignStaticIP` is configured to `Yes` in your CloudFormation stack. 

### Using Certbot with certbot-dns-route53 plugin
Ensure that you have granted Route 53 hosted zone access by specifying `r53ZoneID` value in your CloudFormation stack

- From terminal, execute the below command and follow instructions.  
  ```
  sudo certbot certonly --dns-route53
  ```
  More information from [certbot-dns-route53 documentation site](https://certbot-dns-route53.readthedocs.io)


- Modify your web server configuration file to use the issued cert.
  - Apache
  
  Edit either `/etc/httpd/conf.d/ssl.conf` (Amazon Linux) or `/etc/apache2/sites-available/default-ssl.conf` (Ubuntu Linux), and replace the existing entries with the following 
  ```
  SSLCertificateFile /etc/letsencrypt/live/<CERT-NAME>/fullchain.pem
  SSLCertificateKeyFile /etc/letsencrypt/live/<CERT-NAME>/privkey.pem
  ```
  Replace `<CERT-NAME>` with the actual value in your `/etc/letsencrypt/live` folder.
  - Nginx 
  
  Edit `/etc/nginx/nginx.conf` and replace the existing entries with the following
  ```
  ssl_certificate "/etc/letsencrypt/live/<CERT-NAME>/fullchain.pem";
  ssl_certificate_key "/etc/letsencrypt/live/<CERT-NAME>/privkey.pem";
  ```
  Replace `<CERT-NAME>` with the actual value in your `/etc/letsencrypt/live` folder.
  
- Verify your configuration
  - Apache 
  ```
  sudo apachectl -t
  ```
  - Nginx
  ```
  sudo nginx -t
  ```
  
- Restart web server
  - Apache: Amazon Linux 
  ```
  sudo systemctl restart httpd
  ```
  - Apache: Ubuntu Linux 
  ```
  sudo systemctl restart apache2
  ```
  - Nginx: Amazon Linux / Ubuntu Linux
  ```
  sudo systemctl restart nginx
  ```



### Using Certbot with apache plugin

- From terminal, run the below command and read instructions carefully
  ```
  sudo certbot --apache
  ```
  You may need to reconfigure your Apache site settings after certificate is issued

### Using Certbot with nginx plugin

- From terminal, run the below command and read instructions carefully
  ```
  sudo certbot --nginx
  ```
  You may need to reconfigure your Nginx site settings after certificate is issued


Refer to [Certbot documentation site](https://eff-certbot.readthedocs.io/en/stable/using.html#where-are-my-certificates) for more information


## Securing your EC2 instance
To futher secure your EC2 instance, you may want to
- Disable SSH inbound from the internet by modifying your Security Groups. You can use SSM Session Manager or NICE DCV to remote into your console
- Use [Amazon CloudFront](https://aws.amazon.com/cloudfront/) with [AWS WAF](https://aws.amazon.com/waf/) to protect your EC2 from DDoS attacks. 


## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

