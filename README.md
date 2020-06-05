# 1. 3-tier-architecture

Link: [Designing a three tier architecture](https://medium.com/the-andela-way/designing-a-three-tier-architecture-in-aws-e5c24671f124#:~:text=Introduction&text=A three-tier architecture is,and the data storage layer.)

# 2. Navigating through CloudFormation

##  2.1 Navigating the UI

1. Services -> CloudFormation
   ![](assets/a001_starting_page.PNG)
2. We will try building a template from scratch. Create template in Designer
   <img src="assets/a002_template_designer.PNG" style="zoom:50%;" />
3. Rename the file
   ![](assets/a003_three_tier_template.PNG)

## 2.2 Testing CF scripts

1. First configure your `AWS Access Key ID`, `AWS Secret Access Key` and `Default region name` on the CLI

   ```shell
   aws configure
   ```

2. Then run the following command. Link: [Validating CF templates](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-validate-template.html)

   ```shell
   aws cloudformation validate-template --template-body file://three_tier_stack.yaml
   ```

   

# 3. Objective 1: Deploying a VPC, 2AZs and their subnets

## 3.1 Add the VPC

1. On the left panel "Resource Types", grab the VPC and drag it in
   <img src="assets/a004_create_vpc.PNG" style="zoom:50%;" />

2. Rename the VPC to something more logical. Naming rules are [a-zA-Z0-9]
   <img src="assets/a005_rename_vpc.PNG" style="zoom:50%;" />

3. Then we need to add the `CidrBlock` property to it (Compulsory) this has to be done in the `Template` section. We can also add a `Tag` for the VPC. Link: [VPC CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-vpc.html)

   ```yaml
   Resources:
     vpcApSoutheast1ThreeTierStack:
    Type: 'AWS::EC2::VPC'
       Properties: 
         CidrBlock: 10.0.0.0/16
         Tags:
          - Key: Name
            Value: vpc-ap-southeast-1-d-three-tier-Stack
   ```
   
   

## 3.2 Add the AZs and Subnets

Note: There is no AZ component, we just have to configure the subnet CIDR and properties

Since we are deploying in ap-southeast-1, there are 3 Availability Zones to choose from 

1. ap-southeast-1a (apse1-az2). This will contain 
   - public-subnet-1. CIDR (10.0.1.0/24)
   - private-subnet-3. CIDR (10.0.3.0/24)
2. ap-southeast-1b (apse1-az1). This will contain 
   - public-subnet-2. CIDR (10.0.2.0/24)
   - private-subnet-4. CIDR (10.0.4.0/24)
3. ap-southeast-1c (apse1-az3). We don't need this AZ

<u>**In Cloud Designer**</u>

1. From the Resource types -> EC2 -> Subnet. Add 4 subnets and rename them accordingly
   <img src="assets/a006_create_subnets.PNG" style="zoom:50%;" />

2. Assign subnets to AZs. Link: [subnet AZ Cloudformation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getavailabilityzones.html) and [Nested Functions with Short Form YAML](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-select.html)

   ```yaml
   publicSubnet1:
       Type: 'AWS::EC2::Subnet'
       Properties:
         VpcId: !Ref vpcApSoutheast1ThreeTierStack
         AvailabilityZone: !Select
           - 0
           - Fn::GetAZs: !Ref 'AWS::Region'
       Metadata:
         'AWS::CloudFormation::Designer':
           id: e30d239b-b3ea-4ffb-8d01-2c09354147cf
   ```
   
3. Add the IPs for the subnets and their tags. Link: [subnet properties Cloudformation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnet.html#cfn-ec2-subnet-availabilityzone)

   ```yaml
   publicSubnet1:
       Type: 'AWS::EC2::Subnet'
       Properties:
         VpcId: !Ref vpcApSoutheast1ThreeTierStack
         CidrBlock: 10.0.1.0/24
         AvailabilityZone: !Select
           - 0
           - Fn::GetAZs: !Ref 'AWS::Region'
         Tags:
           - Key: Name
             Value: public-subnet-1
       Metadata:
         'AWS::CloudFormation::Designer':
           id: e30d239b-b3ea-4ffb-8d01-2c09354147cf
   ```
   
   

