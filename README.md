# Hosting Moodle™ on AWS

### Version 2.0.2

## Overview

This repository provides set of CloudFormation nested templates that deploy a highly available, elastic, and scalable [Moodle™ 4.4](https://docs.moodle.org) environment on AWS. Moodle™ offers a learning platform that provides educators, administrators and learners a single robust, secure and integrated system for personalized learning environment. 

These nested templates can be used to deploy Moodle™ on AWS using [Amazon Virtual Private Cloud (Amazon VPC)](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html), [Amazon Elastic Compute Cloud (Amazon EC2)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html), [Auto Scaling](http://docs.aws.amazon.com/autoscaling/latest/userguide/WhatIsAutoScaling.html), [Elastic Load Balancing (Application Load Balancer)](http://docs.aws.amazon.com/elasticbalancing/latest/application/introduction.html), [Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_AuroraOverview.html), [Amazon ElastiCache](http://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/WhatIs.html), [Amazon Elastic File System (Amazon EFS)](http://docs.aws.amazon.com/efs/latest/ug/whatisefs.html), [Amazon CloudFront](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html), [Amazon Route 53](http://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html), [Amazon Certificate Manager (Amazon ACM)](http://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html)  with [AWS CloudFormation](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html) in YAML format. 

This architecture is expansive enough to meet the needs of large institutions / organizations. Smaller organizations can choose to run a subset of the template to meet their needs. These templates can also be run individually and may be modified.

This template currently uses [Moodle™ 4.4](https://download.moodle.org/download.php/stable404/moodle-4.4.tgz) stable version downloaded directly from [download.moodle.org](https://download.moodle.org/releases/latest/). Details for downloading are available in the [templates/03-pipelinehelper.yaml](templates/03-pipelinehelper.yaml) template file.

## Deployment guide

Read the [reference architecture](#architecture) and the steps below to understand the deployment scope and options. While following the steps and guidelines to deploy Moodle™, pay careful attention to the parameters and their descriptions.

### Pre-requisites
1) Select an [AWS Region](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-available-regions) (for example: us-east-1) for your deployment.
2) Give a meaningful `Stack Name` that does `not have` any `special characters` including hyphen(-) Eg: `MoodleDevDeploy` or `MoodleProd`
2) If you plan to use HTTPS, you must [create](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-public.html) or [import](https://docs.aws.amazon.com/acm/latest/userguide/import-certificate.html) your certificate into Amazon Certificate Manager (ACM) and provide its [ARN](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) when deploying the CloudFormation stack.
3) Alternatively, if you plan to use an SSL Certificate with [Amazon CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html), you must create or import your certificate into Amazon Certificate Manager in the us-east-1 region before launching Moodle™ and provide it's ARN when deploying the CloudFormation stack.

### Steps
1) Deploy the 00-main.yaml stack. You can also click the `Launch Stack` button below to launch the stack in your logged-in AWS Account.
2) After the stack deployment completes, you will see a `DNS Name` entry under the `Outputs` tab under the main CloudFormation template. This DNS Name value will be your Moodle™ app URL. You can configure aliases or CNAMEs to point to this DNS Name if you want to customize this.
3) Navigate to the Moodle™ application URL to complete the installation. 
    1) *NOTE: You may encounter a 504 Gateway Timeout or CloudFront error on the final step of the Moodle™ application installation wizard (after configuring the administrator password). You can safely ignore this error and refresh the page to complete the installation.*  
    2) You may also see "Installation must be finished from the original IP address, sorry." If this is the case, update your database and set the `lastip` field of the `mdl_user` table to the internal IP address of your Application Load Balancer which can be found under the `Network Interfaces` section of the `EC2` section of the AWS Console. To update the value in the database, run these commands on the EC2 web server:

            psql -h <hostname> -U<Username> 
            update mdl_user set lastip='<ip address>';

4) Once the Moodle™ installation wizard completes successfully, you need to update the value of the SSM Parameter `IsMoodleSetupCompleted` to 'Yes'.
    1) In your main `CloudFormation` template, check `Outputs` tab to see parameter `IsMoodleSetupCompleted`. Click the link under `Value` to get details of the parameter.
    3) Edit the parameter and change the value to `Yes`.
    4) Go back to `Outputs` tab to see link for `MoodleCodePipeline`. Click on the link to open Code Pipeline. Click on the `Release Change` button. This will re-run the deployment pipeline and update the Moodle™ configurations post-installation, in order to adjust the auto-scaling configuration and the session cache configuration.
    
5) This template can optionally deploy [Amazon ElastiCache](https://aws.amazon.com/elasticache/) as the Moodle™ Session and/or Application cache(s). When this feature is activated, you still need to configure the Application Cache within Moodle™ after deployment (see [how-to](#application-caching) guide). The cache endpoint is listed under the CloudFormation `Output` tab as `ApplicationCacheServerEndpoint`.

*`NOTE:` To connect to your EC2 web servers, select an EC2 Instance and click on the `Connect` button in the AWS Console. Open the `Session Manager` tab and click on the `Connect` button. Note that this feature uses the [AWS SSM Agent](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-getting-started-instance-profile.html) that is installed on the instances, allowing you to connect to EC2 Instances without opening the SSH port to Internet traffic. An alternative approach to connect to your instances would be to enable the bastion host through the CloudFormation stack parameters.

### Launch the CloudFormation Template

You can launch this CloudFormation template in different AWS Regions. Below are links to help you get started quickly, but note that you can always change the region yourself once you are in the AWS Console.

| AWS Region Code | Name | Launch |
| --- | --- | --- 
| us-east-1 |US East (N. Virginia)| [![cloudformation-launch-stack](images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=Moodle&templateURL=https://s3.amazonaws.com/aws-refarch/moodle/al2023/templates/00-main.yaml) |
| us-east-2 |US East (Ohio)| [![cloudformation-launch-stack](images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks/new?stackName=Moodle&templateURL=https://s3.amazonaws.com/aws-refarch/moodle/al2023/templates/00-main.yaml) |
| us-west-2 |US West (Oregon)| [![cloudformation-launch-stack](images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=Moodle&templateURL=https://s3.amazonaws.com/aws-refarch/moodle/al2023/templates/00-main.yaml) |
| eu-west-1 |EU (Ireland)| [![cloudformation-launch-stack](images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/new?stackName=Moodle&templateURL=https://s3.amazonaws.com/aws-refarch/moodle/al2023/templates/00-main.yaml) |
| eu-central-1 |EU (Frankfurt)| [![cloudformation-launch-stack](images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=eu-central-1#/stacks/new?stackName=Moodle&templateURL=https://s3.amazonaws.com/aws-refarch/moodle/al2023/templates/00-main.yaml) |
| ap-southeast-1 |AP (Singapore)| [![cloudformation-launch-stack](images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-southeast-1#/stacks/new?stackName=Moodle&templateURL=https://s3.amazonaws.com/aws-refarch/moodle/al2023/templates/00-main.yaml) |
| ap-southeast-2 |AP (Sydney)| [![cloudformation-launch-stack](images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-southeast-2#/stacks/new?stackName=Moodle&templateURL=https://s3.amazonaws.com/aws-refarch/moodle/al2023/templates/00-main.yaml) |
| ap-south-1 |India (Mumbai)| [![cloudformation-launch-stack](images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-south-1#/stacks/new?stackName=Moodle&templateURL=https://s3.amazonaws.com/aws-refarch/moodle/al2023/templates/00-main.yaml) |
| ca-central-1 |Canada (Central))| [![cloudformation-launch-stack](images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=ca-central-1#/stacks/new?stackName=Moodle&templateURL=https://s3.amazonaws.com/aws-refarch/moodle/al2023/templates/00-main.yaml) |

## Architecture

The following sections describe the architecture and its components. This architecture uses a similar approach to the one used in the [WordPress Reference Architecture](https://github.com/awslabs/aws-refarch-wordpress).

![](images/aws-refarch-moodle-architecture.png)

### AWS Certificate Manager

AWS Certificate Manager lets you easily provision, manage, and deploy Secure Sockets Layer/Transport Layer Security (SSL/TLS) certificates for use with AWS services. You should use SSL/TLS to protect data in transit, including sessions and passwords. If you plan to use SSL/TLS, you must create or import a certificate using AWS Certificate Manager before you deploy the template. In addition, if wish to use CloudFront and host Moodle™ in a region other than us-east-1, you must create or import the certificate in both us-east-1 and the region you are hosting Moodle™ in. CloudFront requires certificates in the us-east-1 region.

### Application Load Balancer

The Application Load Balancer distributes incoming application traffic across multiple EC2 instances in multiple Availability Zones. You achieve high availability by clustering multiple Moodle™ servers behind this load balancer. You can review Moodle™'s overview of [Server Clustering](https://docs.moodle.org/en/Server_cluster) before proceeding.

### Amazon Autoscaling

Amazon EC2 Auto Scaling helps ensure that the appropriate number of Amazon EC2 instances are available to handle the load of the application. The template configures autoscaling based on CPU utilization. An additional instance is added when the average CPU utilization exceeds 75% for three minutes and removed when the average CPU utilization is less than 25% for three minutes. Based on the instance type, cache configuration, and other factors, you may find that other metrics are better predictors of load. You can change the metrics to better meet your operational needs.

*`Note:` that the installation wizard causes spikes in CPU that could cause the cluster to scale unexpectedly. To avoid an issue with this during installation, initial deployment starts with minimum and maximum autoscaling values of 1. Once you complete the Moodle™ installation wizard, update the SSM parameter `IsMoodleSetupCompleted` and run the Moodle™ pipeline, the minimum and maximum autoscaling values will be updated according to your parameters.*

### Amazon Elastic File System (EFS)

Amazon Elastic File System (Amazon EFS) provides simple, scalable file storage in the AWS Cloud. Using EFS makes Moodle™ operations and management (shared files, updates, patches, etc.) easier. However, Moodle™ performance may suffer when the application code itself is run from mounted volumes like EFS. Moodle™ recommends `dirroot` to be on local or high-performance storage. This template follows that recommendation, and uses a combination of Elastic Block Storage (EBS) and EFS for storage. Each web server in the Moodle™ Cluster employs the following directory structure:
```
$CFG->dirroot = '/var/www/moodle/html'        #Stored on root EBS volume
$CFG->localcachedir = '/var/www/moodle/local' #Stored on root EBS volume 

$CFG->dataroot = '/var/www/moodle/data'       #Stored on shared EFS filesystem
$CFG->cachedir = '/var/www/moodle/cache'      #Stored on shared EFS filesystem
$CFG->tempdir = '/var/www/moodle/temp'        #Stored on shared EFS filesystem
```

With [elastic](https://docs.aws.amazon.com/efs/latest/ug/performance.html#elastic) throughput type, Amazon EFS automatically scales throughput performance up or down to meet the needs of your workload activity. You don't need to specify or provision the throughput capacity to meet your application needs. 

*Moodle™ recommends the `dirroot` be set as read only for the apache process in a clustered environment [[Reference]](https://docs.moodle.org/400/en/Server_cluster#.24CFG-.3Edirroot). You should not install plugins to a server cluster from the admin page. `Moodle™ recommends manually installing plugins on each server during planned maintenance`. To follow the infrastructure-as-code methodology, installation/upgrade of plugins can be managed using AWS CodePipeline scripts. See the `.pipeline` folder inside your AWS CodeCommit Moodle™ repository.

### AWS CodePipeline
This CloudFormation templates use AWS Services to create a CI/CD pipeline to help manage your Moodle™ environment. AWS CodeCommit will host a git repository for your Moodle™ environment. It initially pulls the source from [download.moodle.org.](https://download.moodle.org/download.php/stable404/moodle-4.4.tgz). It also adds files required to automate the deployment pipeline. You can explore these files under the `.pipeline` folder.
This template also creates an AWS CodePipeline configuration that build artifacts to deploy on EC2 with autoscaling groups using AWS CodeBuild and AWS CodeDeploy. It can optionally support a BLUE_GREEN deployment.

*You can customize the overall pipeline for your Moodle™ setup.*

### AWS Systems Manager - Parameter Store
This template also uses the Parameter Store to host Moodle™ environment configurations parameters like the database endpoint, the database credentials, the application  and session cache endpoints, etc. This allows easy management of these configuration parameters. You can change these parameters and refresh your deployment to quickly implement them. 

### `Caching`
---
Caching can have a dramatic impact on Moodle™'s performance. This template configures various forms of caching including OPcache, CloudFront and ElastiCache. 

#### OPcache

[PHP OPcache](https://www.php.net/manual/book.opcache.php) speeds up PHP execution by caching precompiled scripts in memory. This template configures OPcache as described [here](https://docs.moodle.org/400/en/OPcache).

#### Amazon ElastiCache

Amazon ElastiCache for `Memcached` is a Memcached-compatible in-memory key-value store service that can be used as a cache or a data store. Moodle™ recommends that you `don't use the same memcached server for both sessions and MUC` [Refer](https://docs.moodle.org/26/en/Session_handling). Events triggering MUC caches to be purged leads to MUC purging the memcached server]. This template configures two ElastiCache clusters, one for session caching and one for application caching.

This template also allows you to create `Amazon ElastiCache for Redis` as Redis compatible in-memory key-value store service that can be used as a cache or a data store.

#### Session Caching

Moodle™ recommends that you [store user sessions in one shared memcached server](https://docs.moodle.org/en/Server_cluster#Performance_recommendations). The template configures session caching as described [here](https://docs.moodle.org/en/Session_handling#Memcached_session_driver). 

*`Note:` This template doesn't configure the Session Cache during initial deployment. It waits for you to finish the initial Moodle™ installation wizard and update the Parameter `IsMoodleSetupCompleted` value to `Yes` in the SSM Parameter Store. Once the installation is completed, you need to run the Moodle™ pipeline to enable session caching and finalize the other remaining configuration.*

#### Application Caching

The template deploys an ElastiCache cluster for application caching, `but the application caching must be configured after the Moodle installation is completed`. You can configure [memcached](https://docs.moodle.org/en/Caching#Memcached) or [Redis](https://docs.moodle.org/en/Redis_cache_store) by filling in the auto-discovery endpoint to the list of Servers under both Store Configuration and Enable Clustered Servers (see image below). You can find the `ApplicationCacheServerEndpoint` address in the `Outputs` of the CloudFormation stack. Finally, scroll to the bottom of the caching administration page in Moodle™ and set ElastiCache as the default store for application caching. 

![](images/aws-refarch-moodle-caching.png)
![](images/aws-refarch-moodle-caching-redis.png)

#### Amazon CloudFront 

Amazon CloudFront is a global content delivery network (CDN) service that securely delivers data, videos, applications, and APIs to your viewers with low latency and high transfer speeds. It also helps in caching content closer to user's geography and reduces loads on the web servers.

### Amazon Route 53

Amazon Route 53 is a highly available and scalable cloud Domain Name System (DNS) service. The template will optionally configure a Route53 alias that points to either the Application Load Balancer or CloudFront. If you are using another DNS system, you should create a CNAME record in your DNS system to reference either the Application Load Balancer or CloudFront (if deployed). If you don't have access to DNS you can leave Domain Name blank and the template will configure Moodle™ to use the auto-generated Application Load Balancer domain name. 

---

## License

This library is licensed under the Apache 2.0 License. 

Portions copyright.

- Moodle™ is licensed under the General Public License (GPLv3 or later) from the Free Software Foundation.
- OPcache is licensed under PHP License, version 3.01.

Please see [LICENSE](LICENSE) for applicable license terms and [NOTICE](NOTICE) for applicable notices.

The word Moodle and associated Moodle logos are trademarks or registered trademarks of Moodle Pty Ltd or its related affiliates.
