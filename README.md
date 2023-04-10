# Capstone-project

The capstone project sets the scene of a researcher from Non-profit organisation, Shirley Rodriguez, wanting to improve the performance of the Non-profit's website. She also has concerns about the security of the website. It consists of a MySQL database and a PHP web application residing on a single EC2 instance in a public subnet.

Shirley has recently experienced an increase in popularity on her website and needs to upgrade its current set up.

# Solution Requirements

- Provide secure hosting of the MySQL database
- Provide secure access for an administrative user
- Provide anonymous access to web users
- Run the website on a t2.micro EC2 instance, and provide Secure Shell (SSH) access to administrators
- Provide high availability to the website through a load balancer
- Store database connection information in the AWS Systems Manager Parameter Store
- Provide automatic scaling that uses a launch template

# Step 1 - Inspect the current VPC
![p2jvlb0iy1bgcshkj9hl](https://user-images.githubusercontent.com/129402830/230879298-023cb2cc-e03d-422e-b12f-d2e25d5bbfce.png)


The following resources have already been created:

'Example' VPC which consisted of:
- 2x Public Subnets (containing 1 Bastion Host)
- 2x Private Subnets
- Internet Gateway

Several Security Groups:
- Bastion-SG
- Example-DBSG
- InventoryAppSG
- Application Load Balancer SG

- Example-LT - An EC2 launch template that contains the SQL Dump file and PHP application

# 2. Create an application load balancer
As this was my first cloud project after the academy, I was eager to dive straight in. Seeing the list of requirements was quite overwhelming to begin with, but once I understood and visualised the goal architecture, it felt like a great challenge to busy myself with.

In order to satisfy the requirement of ‘Provide high availability to the website through a load balancer’, first I created a load balancer.

I created the application load balancer in Example VPC, creating a target group which routes traffic to the two public subnets as this is where the application will be hosted.

# 3. Create an autoscaling group
Next I created an autoscaling group using the Example LT as a launch template. A new instance called 'ExampleApp' was created which contained the SQL Dump file and PHP application.

This satisfies the requirement of providing automatic scaling that uses a launch template.

Next, I tested the website to see how it was functioning. To do this I loaded the application load balancer DNS into browser. It works! Woo!

However, when trying to use the query function of the website it shows a connection error.

![e3in5fv5s6vh1go9j2yj](https://user-images.githubusercontent.com/129402830/230879018-0c2be24c-9f9b-4457-888d-ecd88bb80003.jpg)

It suggests there is no connection to a database. Next job - create the RDS database!

# 4. Create an RDS Database subnet group
Create a subnet group in Example VPC in us-east-1a and us-east-1b selecting the two private subnets.

# 5. Create RDS Database
I launched the RDS Database inside the private subnets of Example VPC. I attached the ExampleDBSG security group.

By hosting the Database in private subnets and adding security groups, this satisfies the 'Provide secure hosting of the MySQL database'. Plus, bonus points for making the RDS database multi-AZ as it boosts availability.

# 6. Systems Manager
Now that I have the database up and running, I need somewhere to store the connection information. Luckily, this fulfils the solution requirement 'Store database connection information in the AWS Systems Manager Parameter Store' so this part was relatively straight forward.

In the project outline we are given parameters used by the PHP application to connect to the database which I am using for this section.

I will customise the following parameters which will be the PHP application to connect to the database:

- /example/endpoint
- /example/username
- /example/password
- /example/database

Below is what my parameters looked like. Note - I had to wait until the RDS database had finished configuring before inputting the database endpoint.

![4eltchqssdaq1sw90a44](https://user-images.githubusercontent.com/129402830/230878946-7dd211a9-dbcc-4bfb-9f97-0caf94ac85ce.png)


# 7. Connecting to the Bastion Host
ALB check, ASG check, RDS check...

By this point I was hit with the case of severe mind blank and had to quickly go over everything I had done so far. I then realised I needed a way to connect securely to the instances. Then I remembered that a Bastion host was one of the resources that was already created - how handy!

I connected to the Bastion host instance via EC2 instance connect. By doing this, I can securely connect to the 'ExampleApp' application instance and the RDS database instance. This also satisfies the 'provide Secure Shell (SSH) access to administrators' requirement.

![jdtiaovu9qfj5soo84d2](https://user-images.githubusercontent.com/129402830/230878887-6278f6a3-a1f3-4626-bc55-4e7abf20a158.jpg)


....However, when trying to SSH into the ExampleApp instance, nothing happened. Then it dawned on me. Security groups.

# 8. Adjust inbound security group rules
In order to allow SSH from the Bastion host into the application instance, the inbound security group rules must be adjusted.

After I could access the instance, I checked whether the database had been created and then inputted the endpoint into parameter store.

# 9. SSH into the new instance
After this change, I was able to SSH into the application instance. I now need to test the connection to the database. The image below shows how I achieved this:

![fjtwgiwugv88tmzxn1gl](https://user-images.githubusercontent.com/129402830/230878697-a5f7d6f0-03b8-4c23-946c-58cc2e943573.jpg)


Note: the end of the code is the DNS of the RDS database

Once I had entered the password I was able to successfully connect to the database.


# 10. Dumping the Countrydatadump.sql into RDS
Once I knew I could connect to the RDS database, I then needed to dump the data files into the new database.

Now here's the interesting part (aka the part I was most apprehensive about). I had experience doing this before in a previous lab during the academy, however, I had to quickly refresh my knowledge before delving into this.

After some light research, I attempted the data dump. I then proceeded to make the same error several times. After several failed attempts, I realised I was missing out the database name at the end of the command.

After being overjoyed at the fact I'd discovered my error, I then continued the saga and made a new error. This time it was I was typing in exampledb when I should have been typing in ExampleDB!

I then connected to the database again to check whether the files had been dumped correctly.

![6p55dovnsz6o8ef4irs5](https://user-images.githubusercontent.com/129402830/230878498-55bcc09b-8a45-40f1-ad5c-d2f724425b26.jpg)


# 11. Check the website again
I then went onto the website and refreshed the page.

Did you think I'd finished there? Oh, no. Yet another error. When I tried to use the query function again it greeted me with this message:

![avr6v5t4u9uun5i00zpx](https://user-images.githubusercontent.com/129402830/230878454-35c4308e-22bd-4cd3-a2d6-034ba7e4f4b9.jpg)


My database still wasn’t connected. HUH?!

Then I realised - the database name was incorrect. (remember earlier the issue with exampledb vs ExampleDB? Yeah, uh..)

Thankfully I quickly realised I inputted the database name wrong into parameter store and changed it from exampledb to ExampleDB. Once I changed this...

![cafkb94w0tlrin4t7dq4](https://user-images.githubusercontent.com/129402830/230878369-64af677c-a35d-4018-898a-ef0e3fb7ae26.jpg)


Success!

My solution:

![2vgfxljizhhgrta30sme](https://user-images.githubusercontent.com/129402830/230878300-2d3efbd5-1e14-44b1-98b0-6a84d90cc604.png)



# Extra features I implemented to increase security
After I finished this project I still wasn't happy with the security of the architecture. I then decided to add a few features to satisfy this.

1. Reconfigured the Autoscaling group
I adjusted the existing autoscaling group to sit in the private subnets. This would mean the application instances would be hosted in private subnets instead of public subnets they were in previously.

2. Reconfigured the Load Balancer
I then adjusted the load balancer to direct traffic to the private subnets by adjusting the target groups.

3. Add a NAT gateway
I added a NAT gateway in one of the public subnets to allow the instances in private subnets connect to the internet.

4. Adjusted the private route table
This was crucial in order to direct internet traffic via the NAT gateway.

5. Delete the old application instance
I first tried to test the webpage in my browser and at first it didn't work. I was slightly confused for a few minutes before realising I needed to delete the old application instances. After I did this, success!

Final architecture:

![8lj2ruix2nxgtkc9mge8](https://user-images.githubusercontent.com/129402830/230878184-cb3d9daf-2a16-4072-8935-be3ab00b65f8.png)

By placing the application instances in the private subnets and using a NAT gateway it has greatly increased the security of the website. This is because the NAT gateway prevents the internet from establishing connections to the instances.

# Conclusion
Every project I do is an opportunity to learn and grow from the experience. I thoroughly enjoyed this project and it has made me even more excited to continue building my skills in new and exciting projects.

This was a great learning experience and a perfect opportunity to put everything I have learnt in the past 12 weeks on the Digital Futures Cloud Engineering course into practice. After seeing what I'm capable of doing during this project, I'm so excited to see what else I can achieve outside of the academy!
