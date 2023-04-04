# Capstone-project

Social Research Organisation

A non-profit that provides a website for social science researchers to obtain global development statistics. They store data in a MySQL database and the data is available through a PHP website that Shirley built. Shirley initially published the site through a commercial hosting company that provides limited support for technical issues and security.

Over the past year, Shirley’s website has grown in popularity. As a result of increased traffic, she started receiving complaints that the site is not as responsive as it used to be. She also experienced an attempted ransomware security breach. The security breach was unsuccessful, but her supervisor, Mateo Jackson, suggested that Shirley investigate new ways to host the website.

Shirley heard about Amazon Web Services (AWS), and initially moved her website and database to an EC2 instance that runs in a public subnet. She also runs an instance of MySQL on the same EC2 instance.

Shirley approached your team to make sure that her current design follows best practices. She wants to make sure that she has a robust and secure website. One of your colleagues started the process of migrating the site to a more secure implementation, but they were reassigned to another project. Your tasks are to complete the implementation, make sure that the website is secure, and confirm that the website returns data from the query page.

# Solution requirements:

•	Provide secure hosting of the MySQL database

•	Provide secure access for an administrative user

•	Provide anonymous access to web users

•	Run the website on a t2.micro EC2 instance, and provide Secure Shell (SSH) access to administrators

•	Provide high availability to the website through a load balancer

•	Store database connection information in the AWS Systems Manager Parameter Store

•	Provide automatic scaling that uses a launch template




# Existing resources:

Example VPC - 

	Public Subnet 1 10.0.0.0/24
	
	Public subnet 2 10.0.1.0/24
	
	Private Subnet 1 10.0.2.0/23
	
	Private Subnet 2 10.0.4.0/23
	

Example IGW


Security groups:

Bastion-SG – enable access to App

Example-DB – enable access to MySQL

Inventory App – Enable access to App

ALB SG – Port 80



  # Steps:
  

1. Create application load balancer
The resources that are already created consist of 2 public subnets which will host the application. In order to satisfy the requirement of ‘Provide high availability to the website through a load balancer’, first I will create a load balancer.

 
Load balancer configuration:

Listener – port 80

Apply to the public subnets in each AZ

Target group – AppTG



2. Create an autoscaling group

Next create an autoscaling group using the Example LT as a launch template. Must be in the Example VPC, select the 2 public subnets as this is where the application will be. This satisfies the requirement of providing automatic scaling that uses a launch template.
Next load application load balancer DNS into browser. This will show the website. When trying to query the website, it shows a connection error. It suggests there is no connection to a database. We now need to create a database in a private subnet.


3. Create RDS database subnet group
Create a subnet group in Example VPC in us-east-1a and us-east-1b selecting the two private subnets. 


4. Create RDS database 
Launch inside Example VPC, choose the database subnet group just created. Add ExampleSG as a security group. Create a username and password.


5. Systems manager – input new credentials 
Now that the database is created, use parameter store to store the database credentials. In the project outline we are given parameters used by the PHP application to connect to the database which will be used for this section.

The following parameters are used by the PHP application to connect to the database:

•	/example/endpoint – (to be added when database finishes creating)

•	/example/username - admin

•	/example/password - database

•	/example/database – exampledb



6. Connect to the bastion host via terminal

Now the credentials have been inputted into parameter store, now try to connect to the bastion host. This will be used to securely connect to the application instance.
Created file labsuser.pem and inputted the private key
Chmod 400 labsuser.pem
SSH into the private subnet

Cannot connect to the ec2 instance. This is because we need to adjust the security group to allow SSH. 


7. Change security groups to allow SSH
Here I found myself stuck for ages as I was changing the wrong security group. I firstly went to change the security group for the ExampleAPP, which still did not allow me access to the instance. However, I then remembered I needed to change the Inventory-App security group to allow SSH from the bastion host SG.

After I could access the instance, I checked whether the database had created and then inputted the endpoint finally into parameter store.


8. SSH into the new instance
After this change, I was able to SSH into the ExampleAPP instance. I now need to test the connection to the DB by inputting mysql -u admin -p --host exampledb.ch02qdvmfust.us-east-1.rds.amazonaws.com and entering the password.


9. We now need to dump the Countrydatadump.sql into the RDS instance 
Once I knew I could connect to the DB, I then needed to dump the data files into the new database.
I was having issues doing this step – I forgot to input the database name after the DB endpoint and then inputted the name incorrectly – I used exampledb instead of ExampleDB. Case sensitive! I was able to connect after this change


10. Connect to DB
To check whether the files were dumped correctly, I connected to the DB and entered ExampleDB. I then saw that the Countrydatadump file was there then went into the tables. It showed all the data was there.

11. Check the website again
I then went onto the website and refreshed the page. My database still wasn’t connected. I realised this was because I had put the incorrect database name in to parameter store  – I needed to change it from exampledb to ExampleDB. Once I changed this it worked!
