   # Rewardz-assignment


# WordPress High Availability on AWS Cloud:

  Here, you’ll be making use of a few AWS resources to set up WordPress High Availability on AWS Cloud. You’ll be using AWS Relational Database Service (RDS) to run the WordPress Database on a separate RDS instance. AWS Application Load Balancer (ALB) will be used as a single entry point for the website. And finally, AWS Elastic File System (EFS) will be used for storing all the WordPress config files, plugins, and WebPages.

  The wait is over, let’s get started. Follow the below-mentioned steps to set up WordPress High Availability on AWS Cloud.
   
      
# Create your MySQL Database : 

  First up, you’ll need to set up an RDS (Relational Database Service) My SQL Database instance to run the WordPress Database.
      Go to Amazon RDS in the AWS Console and click on the “Create database” button.
      As  WordPress uses MySQL, so choose the MySQL Database Engine to proceed.
      Scroll down to select your desired template.
      Now, provide the required Database details such as the name of your DB instance identifier, Master Username and Password for your DB, your desired DB instance class, etc.
After you’re done with the configuration, click on “Create Database"


# Create EC2 Instance :

   You will now need to create an EC2 (Elastic Compute Cloud) instance to run WordPress on.

Create an EC2 instance with the Amazon Linux 2 AMI (HVM) in the Virtual Private Cloud (VPC) in which you created RDS.
In the Security Group of RDS, open Port 3306 in inbound rules and add the EC2 instance Security Group Id in order to allow EC2 to connect to your RDS MySQL.

# Create EFS :
   
   Next, you are going to need the EFS (Elastic File System) to store your media files for WordPress High Availability.

Go to the EFS Console and click on “Create file system”.
Enter the name of EFS and select the VPC in which you have launched RDS and EC2.
After providing the required details, click on “Create”.
The file system is now created.
Select the file system and click on “Edit”.
Under the “Lifecycle management” option, select None.
Change the “Throughput Mode” to “Provisioned” and enter the desired value.
click "save"

   
   
# Attach EFS to EC2:

  Now, you need to attach this file system to the EC2 instance allowing it to store the data in the file system for WordPress High Availability.

Create a /var/www/html directory using the below command.
sudo mkdir -p /var/www/html/
Mount the file system using the below command.
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport <file_system_id>.efs.<region>.amazonaws.com:/ /var/www/html/


# Install the Apache Web Server:
To run WordPress, you’ll need to run a Web Server on your EC2 instance. It is recommended to use the Apache Web Server (open-source) for WordPress High Availability.

Run the below command in your terminal to install Apache on your EC2 instance.
sudo yum install -y httpd

Run the below command to start the Apache Web Server.
sudo systemctl start httpd

You can also enable the Apache Web Server to automatically start on boot. To do so, run the below command.
sudo systemctl enable httpd

Run the below command to check the status of httpd.
sudo systemctl status httpd

Enter <ec2_public_DNS> in your browser to visit the Apache Test Page. Make sure that you have opened Port 80 in the SG of EC2.

# Create Application ALB and Register EC2 in Target Group: 

Now, you will need to create an Elastic Load Balancer (ELB) to direct traffic to your servers for WordPress High Availability.

Go to the EC2 Console and click on “Load Balancers” located in the left navigation panel.
Click on the “Create Load Balancer” button and select “Application Load Balancer”.
Enter the name of the Load Balancer and select the VPC in which you have launched RDS and EC2. Provide the other necessary details required for configuration.
Now, create a new Target Group in configure routing. Select “Create New Target Group” and enter the name of TG. Select “Instance” as “Target Type” and “HTTP” as “Protocol”. Enter “80” in the Port field and scroll down to “Advanced health check settings”. Enter “200, 301” in the “Success codes”.


# Configure WordPress: 

  Now, you’ll need to edit a few areas of configuration for WordPress High Availability. To do so, open the wp-config.php file in your favorite text editor.

The Database configuration needs to be changed.

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'database_name_here' );
/** MySQL database username */
define( 'DB_USER', 'username_here' );
/** MySQL database password */
define( 'DB_PASSWORD', 'password_here' );
/** MySQL hostname */
define( 'DB_HOST', 'localhost' );
Replace the following terms with appropriate values.

DB_NAME: Name of the Database you created in RDS MySQL.
DB_USER: RDS MySQL Master Username.
DB_PASSWORD: RDS MySQL Password.
DB_HOST: RDS MySQL Host (click on your Database instance in RDS and you will get the connection endpoint).
The second configuration change is in the Authentication Unique Keys and Salts.

/**#@+
* Authentication Unique Keys and Salts.
*
* Change these to different unique phrases!
* You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
* You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
*/
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );
You can open this link and generate values for this configuration. You can then replace the entire section with the content from the link. After doing so, add the below line in the config.

define('FS_METHOD', 'direct');
You’re now all set to deploy your WordPress site.


# Deploy WordPress on AWS Cloud:  
  This step will make your Apache Web Server handle your WordPress requests.

You’ll first need to install the application dependencies needed for WordPress. To do so, run the below command in your terminal.
sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
Now, you need to copy your WordPress application files into the /var/www/html directory used by Apache Web Server.
# make sure you are at the the location where the wordpress directory is present
cd /home/ec2-user
ls
#output of ls should be: latest.tar.gz  wordpress
# copy the files
sudo cp -r wordpress/* /var/www/html/
Change the user group of /var/www/ to allow Apache (httpd) to access the files.
sudo chown -R apache:apache /var/www/
Now, open the /etc/httd/conf/httpd.conf file to make the following changes.

Change the following lines
<Directory />
   AllowOverride none
   Require all denied
</Directory>
to

<Directory />
   Options FollowSymLinks
   AllowOverride All
</Directory>

Finally, restart the Apache Web Server.
sudo systemctl restart httpd
You have now successfully installed WordPress on AWS Cloud.

# Make WordPress Highly Available :
  Go back to the AWS Console and select the EC2 instance that you have configured.
Go to Actions → Image and Templates → Create Image.
Enter the name and description of the image.
“Enable” the “No Reboot” check box.
Once you’re done, click on “Create Image”.
Once the image is available, launch the New EC2 instance.
After the new instance is launched, verify that the httpd service is running.
Run the ls /var/www/html command in the terminal to see if all the files are present under that directory.
Once you’re done with the verification, go to the Target Group that you have created in the Application Load Balancer (ALB) step. Register your new EC2 instances in that Target Group.
You have now successfully set up WordPress High Availability on AWS Cloud.


# Conclusion:

WordPress is a popular Content Management System (CMS) used for publishing blogs, running eCommerce sites, and many other use cases. WordPress High Availability ensures that your infrastructure continues to operate even when certain components of the system fail. This ensures minimal downtime and an enhanced experience for your audience.
