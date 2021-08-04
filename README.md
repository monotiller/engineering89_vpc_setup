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
        2. Assign to internet gateway
    2. Private: `10.212.2.0`
        1. Assign to private acl
    3. Bastion: `10.212.3.0`
        1. Assign to public acl
        2. Assign to internet gateway
6. Create 3 security groups:
    1. app:
        1. Inbound:

| Name | Security group rule ID | IP version            | Type | Protocol | Port range | Source | Description |                         |
|------|------------------------|-----------------------|------|----------|------------|--------|-------------|-------------------------|
|      | –                      | sgr-02d0dff61fca657a1 | IPv4 | HTTP     | TCP        | `80`     | `0.0.0.0/0`   | Allows users to connect |
|      | –                      | sgr-0680b6d60cffe5b47 | IPv4 | SSH      | TCP        | `22`     | `[your ip]/32` | Allows us to SSH in     |
        2. Outbound:

| Name | Security group rule ID | IP version            | Type | Protocol    | Port range | Destination | Description |                                |
|------|------------------------|-----------------------|------|-------------|------------|-------------|-------------|--------------------------------|
|      | –                      | sgr-099431bf7eb8cf289 | IPv4 | All traffic | All        | All         | `0.0.0.0/0`   | Allows all traffic leaving the |
    2. Bastion
        1. Inbound:

| Name | Security group rule ID | IP version            | Type | Protocol | Port range | Source | Description      |                                                                    |
|------|------------------------|-----------------------|------|----------|------------|--------|------------------|--------------------------------------------------------------------|
|      | –                      | sgr-0446df729793b5a88 | IPv4 | SSH      | TCP        | `22`     | `90.240.26.195/32` | We only want the ssh set up since this talks between us and the db |
        2. Outbound:

| Name | Security group rule ID | IP version            | Type | Protocol    | Port range | Destination | Description |   |
|------|------------------------|-----------------------|------|-------------|------------|-------------|-------------|---|
|      | –                      | sgr-008207190a3f5c202 | IPv4 | All traffic | All        | All         | `0.0.0.0/0`   | – |
    3. db
        1. Inbound: link to your app and you db security groups on port `27017` and `22` respectively
        2. Outbound: Leave as default
7. Head on over to EC2 and spin up 3 new instances:
    1. App
    2. DB
    3. Bastion
8. For each step, select the matching VPC and corresponding subnet & security group for each instance
9. Make sure that for app and bastion you allow for a public IP to be automatically assigned
