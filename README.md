# VPC Setup
## Before we begin
I was assigned the private IP address `10.212.0.0` for this task so anywhere you see that, replace it with your own private IP!

Please note also that to keep things consistent we will be using:
- `10.212.1.0` to reference the public subnet
- `10.212.2.0` to reference the private subnet
- `10.212.3.0` to reference the Bastion's subnet 

## Steps for creation
Feel free to name anything anything, however, where I mention "public", "private" and "Bastion", it is recommended you choose those as the suffixes to your naming convention to help you better keep track of what's doing what!

1. Create a new VPC the private IP address as the IPv4 CIDR block
2. Create an internet gateway
3. Create a route table

    |   Destination    |   Target                    |   Status  |   Propagated  |
    |------------------|-----------------------------|-----------|---------------|
    |   `10.212.0.0/16`  |   local                     |   Active  |   No          |
    |   `0.0.0.0/0`      |   Your internet gateway id  |   Active  |   No          |
4. Create 3 ACLs
    1. Public:
        1. Inbound:

        | Rule number | Type       | Protocol | Port range   | Source       | Allow/Deny |
        |-------------|------------|----------|--------------|--------------|------------|
        | 100         | HTTP (80)  | TCP (6)  | `80`           | `0.0.0.0/0`    | Allow      |
        | 115         | SSH (22)   | TCP (6)  | `22`           | `[your ip]/32` | Allow      |
        | 120         | Custom TCP | TCP (6)  | `1024 - 65535` | `0.0.0.0/0`    | Allow      |    
		2. Outbound:
    
        | Rule number | Type       | Protocol | Port range   | Destination   | Allow/Deny |
        |-------------|------------|----------|--------------|---------------|------------|
        | 100         | HTTP (80)  | TCP (6)  | `80`           | `0.0.0.0/0`     | Allow      |
        | 110         | Custom TCP | TCP (6)  | `27017`        | `10.212.2.0/24` | Allow      |
        | 120         | Custom TCP | TCP (6)  | `1024 - 65535` | `0.0.0.0/0`     | Allow      |
    2. Private: 
        1. Inbound:
        
        | Rule number | Type       | Protocol | Port range   | Source        | Allow/Deny |
        |-------------|------------|----------|--------------|---------------|------------|
        | 100         | Custom TCP | TCP (6)  | `27017`        | `10.212.1.0/24` | Allow      |
        | 110         | Custom TCP | TCP (6)  | `1024 - 65535` | `0.0.0.0/0`     | Allow      |
        | 120         | SSH (22)   | TCP (6)  | `22`           | `10.212.3.0/24` | Allow      |
        2. Outbound:
        
        | Rule number | Type       | Protocol | Port range   | Destination   | Allow/Deny |
        |-------------|------------|----------|--------------|---------------|------------|
        | 100         | HTTP (80)  | TCP (6)  | `80`           | `0.0.0.0/0`     | Allow      |
        | 110         | Custom TCP | TCP (6)  | `1024 - 65535` | `10.212.1.0/24` | Allow      |
        | 120         | Custom TCP | TCP (6)  | `1024 - 65535` | `10.212.3.0/24` | Allow      |
    3. Bastion:
        1. Inbound:
        
        | Rule number | Type       | Protocol | Port range   | Source        | Allow/Deny |
        |-------------|------------|----------|--------------|---------------|------------|
        | 100         | SSH (22)   | TCP (6)  | `22`           | `0.0.0.0/0`     | Allow      |
        | 110         | Custom TCP | TCP (6)  | `1024 - 65535` | `10.212.2.0/24` | Allow      |
        2. Outbound:
        
        | Rule number | Type       | Protocol | Port range   | Destination   | Allow/Deny |
        |-------------|------------|----------|--------------|---------------|------------|
        | 100         | SSH (22)   | TCP (6)  | `22`           | `10.212.2.0/24` | Allow      |
        | 110         | Custom TCP | TCP (6)  | `1024 - 65535` | `0.0.0.0/0`     | Allow      |
5. Create 3 subnets in the eu-west-1a availability zone:
    1. Public: `10.212.1.0`
        1. Assign to public acl
        2. Assign to your route table
    2. Private: `10.212.2.0`
        1. Assign to private acl
    3. Bastion: `10.212.3.0`
        1. Assign to public acl
        2. Assign to your route table
6. Create 3 security groups:
    1. app:
        1. Inbound:

        | Name | Security group rule ID | IP version            | Type | Protocol | Port range | Source | Description |                         |
        |------|------------------------|-----------------------|------|----------|------------|--------|-------------|-------------------------|
        |      | –                      | [Will be unique to your setup] | IPv4 | HTTP     | TCP        | `80`     | `0.0.0.0/0`   | Allows users to connect |
        |      | –                      | [Will be unique to your setup] | IPv4 | SSH      | TCP        | `22`     | `[your ip]/32` | Allows us to SSH in     |
        2. Outbound:

        | Name | Security group rule ID | IP version            | Type | Protocol    | Port range | Destination | Description |                                |
        |------|------------------------|-----------------------|------|-------------|------------|-------------|-------------|--------------------------------|
        |      | –                      | [Will be unique to your setup] | IPv4 | All traffic | All        | All         | `0.0.0.0/0`   | Allows all traffic leaving the |
    2. Bastion
        1. Inbound:

        | Name | Security group rule ID | IP version            | Type | Protocol | Port range | Source | Description      |                                                                    |
        |------|------------------------|-----------------------|------|----------|------------|--------|------------------|--------------------------------------------------------------------|
        |      | –                      | [Will be unique to your setup] | IPv4 | SSH      | TCP        | `22`     | `[your ip]/32` | We only want the ssh set up since this talks between us and the db |
        2. Outbound:

        | Name | Security group rule ID | IP version            | Type | Protocol    | Port range | Destination | Description |   |
        |------|------------------------|-----------------------|------|-------------|------------|-------------|-------------|---|
        |      | –                      | [Will be unique to your setup] | IPv4 | All traffic | All        | All         | `0.0.0.0/0`   | – |
    3. db
        1. Inbound: link to your app and you db security groups on port `27017` and `22` respectively
        2. Outbound: Leave as default
7. Head on over to EC2 and spin up 3 new instances:
    1. App
    2. DB
    3. Bastion
8. For each step, select the matching VPC and corresponding subnet & security group for each instance
9. Make sure that for app and Bastion you allow for a public IP to be automatically assigned

## SSH in to the DB from Bastion
In order to access the DB you will have to go via your Bastion instance. Here's how:

1. SSH in to your Bastion instance
2. If you set a key for your DB instance you'll need to upload that to your Bastion instance now
3. SSH in to your DB instance from within your Bastion instance!