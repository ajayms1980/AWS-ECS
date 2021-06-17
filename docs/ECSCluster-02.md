## ECS Cluster 
<ul>
  <li>Clusters are Region-specific.</li>
  <li>Logical Grouping of Tasks and Services .</li>
  <li>A cluster may contain a mix of tasks hosted on AWS Fargate, Amazon EC2 instances, or external instances. </li>
  <li>ECS Agent (Docker Container) allows container instances to connect to your cluster.</li>
  <li>ECS container agent is included in the Amazon ECS-optimized AMIs, but you can also install it on any Amazon EC2 instance that supports the Amazon ECS specification. </li>
  <li> ECS container agent is only supported on Amazon EC2 instances.</li>
</ul>  

![image](https://user-images.githubusercontent.com/84008107/121775756-2a9f6c00-cba7-11eb-9396-7f99fcfe57ce.png)

### ECS Cluster Setup

### Step1:-Create A Role for the service -> Elastic Container Service
 It will show below use case

##### Select your use case as per predefined policies
<ul> 
  <li><b>EC2 Role for Elastic Container Service</b>:-Allows EC2 instances in an ECS cluster to access ECS. <br /><b> Policy:- </b> <i><u>AmazonEC2ContainerServiceforEC2Role</u></i><br /> <b>Role Name:-</b><u><i>ecs_ec2_Role1</u></i></li>
  <li><b>Elastic Container Service</b>:-Allows ECS to create and manage AWS resources on your behalf. <br /><b> Policy:- </b> <i><u>AmazonEC2ContainerServiceRole</u></i><br /> <b>Role Name:-</b><u><i>ecs_resource_Role2</u></i></li></li>
  <li><b>Elastic Container Service Autoscale</b>:-Allows Auto Scaling to access and update ECS services. <br /><b> Policy:- </b> <i><u>AmazonEC2ContainerServiceAutoscaleRole </u></i><br /> <b>Role Name:-</b><u><i>ecs_autoscaling_Role3</u></i></li></li>
  <li><b>Elastic Container Service Task</b>:-Allows ECS tasks to call AWS services on your behalf.<br /><b> Policy:- </b> <i><u>AmazonECSTaskExecutionRolePolicy </u></i><br /> <b>Role Name:-</b><u><i>ecs_task_Role4</u></i></li></li>
</ul>  

### Step2:- Create AWS Infrastructure 
Create a cloudformation file to create the following VPC Infrastructure

<ul>
  <li>VPC</li>
  <li>2 subnets in 2 different AZs </li>
  <li> Internet Gateway</li>
  <li> Routing Tables</li>
</ul>

<b> Create below cloudformation file to setup above infrastructure. file name is core-infrastructure-setup.yml </b>

      AWSTemplateFormatVersion: '2010-09-09'
      Description: VPC and subnets as base for an ECS cluster
      Parameters:
        EnvironmentName:
          Type: String
          Default: ecs-course

      Mappings:
        SubnetConfig:
          VPC:
            CIDR: '172.16.0.0/16'
          PublicOne:
            CIDR: '172.16.0.0/24'
          PublicTwo:
            CIDR: '172.16.1.0/24'

      Resources:
        VPC:
          Type: AWS::EC2::VPC
          Properties:
            EnableDnsSupport: true
            EnableDnsHostnames: true
            CidrBlock: !FindInMap ['SubnetConfig', 'VPC', 'CIDR']

        PublicSubnetOne:
          Type: AWS::EC2::Subnet
          Properties:
            AvailabilityZone:
               Fn::Select:
               - 0
               - Fn::GetAZs: {Ref: 'AWS::Region'}
            VpcId: !Ref 'VPC'
            CidrBlock: !FindInMap ['SubnetConfig', 'PublicOne', 'CIDR']
            MapPublicIpOnLaunch: true
        PublicSubnetTwo:
          Type: AWS::EC2::Subnet
          Properties:
            AvailabilityZone:
               Fn::Select:
               - 1
               - Fn::GetAZs: {Ref: 'AWS::Region'}
            VpcId: !Ref 'VPC'
            CidrBlock: !FindInMap ['SubnetConfig', 'PublicTwo', 'CIDR']
            MapPublicIpOnLaunch: true

        InternetGateway:
          Type: AWS::EC2::InternetGateway
        GatewayAttachement:
          Type: AWS::EC2::VPCGatewayAttachment
          Properties:
            VpcId: !Ref 'VPC'
            InternetGatewayId: !Ref 'InternetGateway'
        PublicRouteTable:
          Type: AWS::EC2::RouteTable
          Properties:
            VpcId: !Ref 'VPC'
        PublicRoute:
          Type: AWS::EC2::Route
          DependsOn: GatewayAttachement
          Properties:
            RouteTableId: !Ref 'PublicRouteTable'
            DestinationCidrBlock: '0.0.0.0/0'
            GatewayId: !Ref 'InternetGateway'
        PublicSubnetOneRouteTableAssociation:
          Type: AWS::EC2::SubnetRouteTableAssociation
          Properties:
            SubnetId: !Ref PublicSubnetOne
            RouteTableId: !Ref PublicRouteTable
        PublicSubnetTwoRouteTableAssociation:
          Type: AWS::EC2::SubnetRouteTableAssociation
          Properties:
            SubnetId: !Ref PublicSubnetTwo
            RouteTableId: !Ref PublicRouteTable

      Outputs:
        VpcId:
          Description: The ID of the VPC that this stack is deployed in
          Value: !Ref 'VPC'
          Export:
            Name: !Sub ${EnvironmentName}:VpcId
        PublicSubnetOne:
          Description: Public subnet one
          Value: !Ref 'PublicSubnetOne'
          Export:
            Name: !Sub ${EnvironmentName}:PublicSubnetOne
        PublicSubnetTwo:
          Description: Public subnet two
          Value: !Ref 'PublicSubnetTwo'
          Export:
            Name: !Sub ${EnvironmentName}:PublicSubnetTwo


<b> Command to create above cloudformation stack </b>

    aws cloudformation create-stack --capabilities CAPABILITY_IAM --stack-name ecs-core-infrastructure --template-body file://./core-infrastructure-setup.yml

### Step 3:- Setup a Cluster 
<ul>
  <li> Create Cluster ( by Clicking on Cluster and then click on Create Cluster Button ) </li>
  <li> Select cluster template:-> Select <b> EC2 Linux + Networking </b></li>
  <li> Provide Following Details to template </br>
       1. Provide Cluster name ( I am giving cluster name as mycluster ) </br>
       2. Provisioning Model :- On-Demand Instance </br>
       3. EC2 instance type* :- t2.small ( you can change as per your requirement)</br>
       4. Number of instances*:- 1 ( you can change as per your requirement) </br>
       5. EC2 AMI ID*:- AMAZON LINUX 2 AMI </br>
       6. Root EBS Volume Size (GiB):- 30 GB </br>
       7. Key pair : (Select any valid key pair) </br>
       8. VPC : (Select the VPC which get created above with cloud formation)
       9. Subnet: (Select the subnet which get created for VPC mentioned in previous step)
       10.

       

       

  </li>
</ul>  
