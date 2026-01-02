## Create a Variable.tf file:
In this file, specify all credentials and always add this file to .gitignore before pushing to github
```
/*
variable "region" {
  default = "us-east-1"
}

variable "accesskey" {
  default = "AKIAQV6DOBG5D2NF3QGH"
}

variable "secretkey" {
  default = "/JbImdXWmmtQmU3Q2qx1RZxNXthbs12tlA8twRVo"
}

variable "dbname" {
  default = "appdatabase"
}

variable "username" {
  default = "admin"
}

variable "password" {
  default = "db*pass123"
}
*/
```

- Region
- AWS access key
- AWS secret access key

## Create a provider.tf file:
In this file, specify aws provider

## Create a vpc.tf file:
Configure a VPC named AppVPC with a CIDR block of 10.0.0.0/16 and enable both DNS support and DNS hostnames.

Create two subnets within AppVPC, one in us-east-1a with CIDR 10.0.1.0/24 and another in us-east-1b with CIDR 10.0.2.0/24.

## Create a sg.tf file:
Set up a security group within AppVPC that allows web traffic on TCP ports 22, 80, 443 and 3306 from anywhere. Also, it should allow external traffic to all IPs.

## Create a network-interface.tf file:
Create two network interfaces - nw-interface1 and nw-interface2.
Both of the interfaces should use WebTrafficSG as the security group, while the nw-interface1 should use AppSubnet1 and nw-interface2 should use AppSubnet2, respectively.

## Create an igw-rt.tf file:
Attach the network (AppVPC) to any Internet Gateway. Tag this gateway as AppInternetGateway.
Also, create a route table for the VPC AppVPC. Tag this table as AppRouteTable. Create an associated output for this ID named route_table_ID.

Create a route in your AWS infrastructure to allow internet access. The route should be associated with the route table named AppRouteTable and should direct traffic to the internet gateway named AppInternetGateway.
Set the destination CIDR block to 0.0.0.0/0 to allow all outbound traffic.

Associate two subnets, AppSubnet1 and AppSubnet2, with the route table named AppRouteTable to ensure that the subnets use this route table for their traffic routing.

## Edit network-interface.tf file:
To ensure that our future EC2 instances get assigned a public IP address, create two Elastic IP (EIP) resources and attach them to one network interface each - nw-interface1 andnw-interface2


## Create instance.tf file:
Create two EC2 instances within AppVPC, one in each subnet (AppSubnet1 and AppSubnet2), using the ami-06c68f701d8090592 AMI and t2.micro instance type.
Create a key-pair for the EC2 instances called my-ec2-key. Store it in /root. Use this key-pair for both the EC2 instances.

```
aws ec2 create-key-pair \
  --key-name my-ec2-key \
  --query 'KeyMaterial' \
  --output text > /root/my-ec2-key.pem

chmod 400 /root/my-ec2-key.pem
```
Tag the instances with Name as WebServer1 (AppSubnet1) and WebServer2 (AppSubnet2), respectively.
Attach the appropriate network interfaces to each instance according to their subnet ID.

Add two outputs to the configuration that contain the instance IDs of the created EC2 instances.

## Create rds.tf file:
We will now be provisioning an RDS database instance. We want this instance to be accessible from the security group of the web servers.
Create a database subnet group called app-db-subnet-group, which should include the subnets within the VPC AppVPC.

Now, provision an RDS instance in AppVPC. The database should be accessible from the WebServer security group and have the following specs:

```
Allocated storage: 20
Engine: mysql
Engine version: 8.0.33
Instance class: db.t3.micro
Database name: appdatabase(call from variable.tf)
Username: admin(call from variable.tf)
Password: db*pass123(call from variable.tf)
Database subnet group: app_db_subnet_group
VPC security group ID: ID of WebTrafficSG
Ensure that the database is publicly accessible. Tag the RDS instance with Name as AppDatabase.
```

```
ssh -i my-ec2-key.pem ec2-user@<public_IP>
install mysql
mysql -h <DB_endpoint> -P 3306 -u admin -p
```

