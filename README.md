# transit-gateway-bandwidth-test

### Transit Gateway Bandwidth Testing via iPerf3

Proof of Concept to showcase the inter-VPC network speeds achieved with Transit Gateway.

### About

AWS Transit Gateway is a service that enables customers to connect their Amazon Virtual Private Clouds (VPCs) and their on-premises networks to a single gateway. As you grow the number of workloads running on AWS, you need to be able to scale your networks across multiple accounts and Amazon VPCs to keep up with the growth.

With AWS Transit Gateway, you only have to create and manage a single connection from the central gateway in to each Amazon VPC, on-premises data center, or remote office across your network. Transit Gateway acts as a hub that controls how traffic is routed among all the connected networks which act like spokes. This hub and spoke model significantly simplifies management and reduces operational costs because each network only has to connect to the Transit Gateway and not to every other network. Any new VPC is simply connected to the Transit Gateway and is then automatically available to every other network that is connected to the Transit Gateway. This ease of connectivity makes it easy to scale your network as you grow.

[via ![AWS Transit Gateway](https://aws.amazon.com/transit-gateway/)]

Transit Gateway offers the following Performance and Limits:

**Maximum bandwidth (burst) per VPC, Direct Connect gateway, or peered Transit Gateway connection: 50 Gbps**

This GitHub Repository offers a cloudformation template to test this limit.

----

## Architecture

![Stack-Resources](https://github.com/CYarros10/transit-gateway-bandwidth-test/blob/master/images/architecture-design-pattern.png)

----

## Tutorial

### Step 1: Deploy Cloudformation

- Download/Clone git repo
- Go to Cloudformation Console and deploy cloudformation/master.yml
- For Parameters, choose the public key you want for SSH access. Leave the rest as defaults.
- Click next, then create.

### Step 2: Setup Cloudwatch Dashboard

- Go to Cloudwatch console
- Select Metrics -> Transit Gateway -> Per Transit Gateway Metrics -> Transit Gateway ID (if you don't know, its in VPC console) -> BytesIn + BytesOut
- Select Graphed Metrics Tab
- Specify Statistic = Sum, Period = Second
- Configure the graph title, window range, and auto-refresh to your desired settings.
- If you'd like to add to a Cloudwatch Dashboard: Select Actions -> Add to Dashboard

## Conclusion

Based on the Cloudwatch metrics over time, you should see, on average, around 35 GB/s of traffic between the two VPCs.  Network bandwidth will occasionally reach 50 GB/s.


## FAQs

- what is iPerf3?

https://iperf.fr/

- What's happening during cloudformation deployment?

The cloudformation template deploys two VPCs, both with public subnets and route tables, and a transit gateway with both VPCs attached.  Because Transit Gateway has a limit of 50GB/s and EC2s have a limit of 5 GB/s bandwidth, it is necessary to deploy at least 10 EC2s in both VPCs. One VPC will have ~10 EC2s acting as iPerf3 servers that accept traffic, the other VPC will have ~10 EC2s acting as iPerf3 clients that will send traffic.  Transit Gateway allows us to use the Private IP address of the iPerf3 server EC2s, ensuring that no traffic is publicly facing.

- How does the EC2 instance send traffic?

Upon creation, each EC2 instance will pull code from this github repository via the User Data.  The iperf3-log-to-s3.sh script does the iperf3 client to server traffic command.  The results of the command are spit into a client.json log file and uploaded to S3 bucket for archived analysis. A cronjob task is set to run iperf3-log-to-s3.sh every minute, ensuring that traffic is continuously being sent from each EC2 client to an EC2 server.