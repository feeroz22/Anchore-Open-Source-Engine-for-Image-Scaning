# Anchore-Open-Source-Engine-for-Image-Scaning
Anchore Open Source Engine for Image Scanning

# Overview
The Anchore Engine is distributed as a Docker Image available from DockerHub that can be scaled horizontally to handle hundreds of thousands of images.
A PostgreSQL database is required to provide persistent storage for the Anchore Engine, this database can be run as a Docker Container or provided as an external service to be accessed by the Anchore Engine.
For detailed requirements for the database, network and storage please refer to the System Requirements page.
The Engine is comprised of six smaller micro-services that can be deployed in a single container or scaled out to handle load.
Core Services
•	API Service
•	Catalog Service
•	Queuing Service
•	Policy Engine Service
•	Kubernetes Webhook Service
Workers
•	Image Analyzer service
# Anchore Engine
In short: a system to help automate the description and enforcement of requirements on the content of docker containers. With a bit more detail? Anchore Engine is a docker container static analysis and policy-based compliance tool that automates the inspection, analysis, and evaluation of images against user-defined checks to allow high confidence in container deployments by ensuring workload content meets the required criteria. Anchore Engine ultimately provides a policy evaluation result for each image: pass/fail against policies defined by the user. Additionally, the way that policies are defined and evaluated allows the policy evaluation itself to double as an audit mechanism that allows point-in-time evaluations of specific image properties and content attributes.
# How does it work
Anchore takes a data-driven approach to analysis and policy enforcement. The system essentially has discrete phases for each image analyzed:
•	Fetch the image content and extract it, but never execute it
•	Analyze the image by running a set of Anchore analyzers over the image content to extract and classify as much metadata as possible
•	Save the resulting analysis in the database for future use and audit
•	Evaluate policies against the analysis result, including vulnerability matches on the artifacts discovered in the image.
•	Update to the latest external data used for policy evaluation and vulnerability matches (we call this external data sync a feed sync), and automatically update image analysis results against any new data found upstream.
•	Notify users of changes to policy evaluations and vulnerability matches
•	Repeat 5 & 6 on intervals to ensure latest external data and updated image evaluations
# Install Docker Compose
On Linux, you can download the Docker Compose binary from the Compose repository release page on GitHub. Follow the instructions from the link, which involve running the curl command in your terminal to download the binaries. These step by step instructions are also included below.
1.	Run this command to download the latest version of Docker Compose:
 
The above command is an example, and it may become out-of-date. To ensure you have the latest version, check the Compose repository release page on GitHub.
If you have problems installing with curl, see Alternative Install Options tab above.
2.	Apply executable permissions to the binary:
 
# Note: If the command docker-compose fails after installation, check your path. You can also create a symbolic link to /usr/bin or any other directory in your path.
For example:
 
1.	Optionally, install command completion for the bash and zsh shell.
2.	Test the installation.
 

# Anchor Installation with Docker Compose
The Anchore Engine can be run using Docker Compose. A sample Docker Compose file is provided, that runs both the Anchore Engine and PostgreSQL database as containers.
1.	Set up a working directory for your configuration and data volumes
 
2.	Download the docker-compose.yaml and config.yaml files from the scripts/docker-compose directory of the github project.
 

3.	Your ~/aevolume directory should now look like this: 

 

4.	(Optional) Review the default docker-compose.yaml and config.yaml files to modify any particular settings for your environment (not necessary for quickstart)
5.	Run 'docker-compose pull' to instruct Docker to download the required container images from DockerHub. 

 

6.	Start the Anchore Engine 
Note: This command should be run from the directory container docker-compose.yaml 
 

7.	Verify that your DB and service containers are up and then run an anchore-cli command to verify system status: 

 
 

8.	The first time you run anchore-engine, it will take some time to perform its initial data feed sync (vulnerability data download).  Subsequently, anchore-engine will only sync data changes and thus you will only have to wait the very first time you start the engine.  You can watch the status of your feed sync with anchore-cli:mmand to verify system status: 

 
1.	As soon as all the feeds show a non-zero RecordCount, then the feeds are all synced, and the system is ready to generate vulnerability reports.  You can add images right away, but you will not see any vulnerability scan results until the vulnerability data feeds are synced.
2.	Start using the anchore-engine service to analyze images - a short example follows

 

9.	Stopping the Anchore Engine 
Note: This command should be run from the directory containing docker-compose.yaml 

 













# Installing the Anchore CLI 
The Anchore CLI is published as a Python Package that can be installed from source from the Python PyPIpackage repository on any platform supporting PyPi.

# Installing Anchore CLI on CentOS and Red Hat Enterprise Linux

 

 

# Setting the Path
Once installed the anchore-cli utility has been installed you may need to adjust your PATH to ensure that the anchore-cli executable is in the user's path.
The install location is system dependent, governed by PIP and may vary based the distribution on which you are running.
The most common default locations are:

Apple macOS: $HOME/Library/Python/2.7/bin
Linux: $/HOME/.local/bin

 


 


# Configuring the Anchore CLI
By default, the Anchore CLI will try to connect to the Anchore Engine at http://localhost/v1 with no authentication. 

The username, password and URL for the server can be passed to the Anchore CLI using one of three methods:

## a.	Command Line Parameters
The following command line parameters are used to configure the Anchore CLI to connect to and authenticate with the Anchore Engine.

--u    TEXT   Username      eg. admin
--p    TEXT   Password       eg. foobar
--url  TEXT   Service URL   eg. http://localhost:8228/v1
--insecure Skip certificate validation checks (optional)

These connection parameters should be passed before any other commands.
eg.

$ anchore-cli --u admin --p foobar --url http://test.example.com:8228/v1

## b.	Environment Variables
Rather than passing command line parameters for every call to the Anchore CLI they can be stored as environment variables

 ![image](https://user-images.githubusercontent.com/46320181/55561541-9eee4300-56f2-11e9-8098-71f8c1b96c66.png)

 ![image](https://user-images.githubusercontent.com/46320181/55561550-a1e93380-56f2-11e9-93dd-f26c86c090ea.png)


# Scanning Images on Amazon Elastic Container Registry (ECR)
Amazon AWS typically uses keys instead of traditional usernames & passwords. These keys consist of an access key ID and a secret access key. While it is possible to use the aws ecr get-login command to create an access token, this will expire after 12 hours so it is not appropriate for use with Anchore Engine, otherwise a user would need to update their registry credentials regularly. So when adding an Amazon ECR registry to Anchore Engine you should pass the aws_access_key_id and aws_secret_access_key. 
For example:
$ anchore-cli registry add /
             1234567890.dkr.ecr.us-east-1.amazonaws.com /
             MY_AWS_ACCESS_KEY_ID /
             MY_AWS_SECRET_ACCESS_KEY /
             --registry-type=awsecr
The registry-type parameter instructs Anchore Engine to handle these credentials as AWS credentials rather than traditional usernames and passwords. Currently the Anchore Engine supports two types of registry authentication standard username and password for most Docker V2 registries and Amazon ECR. In this example we specified the registry type on the command line however if this parameter is omitted then the CLI will attempt to guess the registry type from the URL which uses a standard format.
The Anchore Engine will use the AWS access key and secret access keys to generate authentication tokens to access the Amazon ECR registry, the Anchore Engine will manage regeneration of these tokens which typically expire after 12 hours.
In addition to supporting AWS access key credentials Anchore also supports the use of IAM roles for authenticating with Amazon ECR if the Anchore Engine is run on an EC2 instance.
In this case you can configure the Anchore Engine to inherit the IAM role from the EC2 instance hosting the engine.
When launching the EC2 instance that will run the Anchore Engine you need to specify a role that includes the AmazonEC2ContainerRegistryReadOnly policy.

![image](https://user-images.githubusercontent.com/46320181/55561569-a9104180-56f2-11e9-84e4-271df7f5fc3c.png)
![image](https://user-images.githubusercontent.com/46320181/55561583-ae6d8c00-56f2-11e9-981f-169c2b078cde.png)
![image](https://user-images.githubusercontent.com/46320181/55561592-b62d3080-56f2-11e9-8da2-4596b38eb6c2.png) 

  
Give a name to the role and add this role to the Instance you are launching.
On the running EC2 instance you can manually verify that the instance has inherited the correct role by running the following command:

 ![image](https://user-images.githubusercontent.com/46320181/55561600-b9c0b780-56f2-11e9-9816-18eafd2328e5.png)
 
Running the following command lists the defined registries.

 ![image](https://user-images.githubusercontent.com/46320181/55561612-bd543e80-56f2-11e9-9f8a-1d8802aa53b6.png)

Add a repository to the Anchore Engine:
![image](https://user-images.githubusercontent.com/46320181/55704680-0c44f100-59dd-11e9-904a-675467674851.png)

Add an image to the Anchore Engine:
![image](https://user-images.githubusercontent.com/46320181/55561656-d6f58600-56f2-11e9-84c1-359eea195693.png)
 
# Errors:

1)	Docker service need to be in running state on instance.

 ![image](https://user-images.githubusercontent.com/46320181/55561665-da890d00-56f2-11e9-94c2-251e7dafc051.png)

2)	Images need to be add before passing the get parameter in command.

![image](https://user-images.githubusercontent.com/46320181/55561674-df4dc100-56f2-11e9-82d3-717fd0338535.png)

 


## References:

https://anchore.com/blog/anchore-engine/
https://github.com/anchore/anchore-engine
https://anchore.freshdesk.com/support/solutions/articles/36000020729-install-with-docker-compose

