![](https://s3-us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/sts-sign-in-images/media/aws-logo.png)

# Working with AWS CloudFormation


© 2018 Amazon Web Services, Inc. and its affiliates. All rights reserved. This work may not be reproduced or redistributed, in whole or in part, without prior written permission from Amazon Web Services, Inc. Commercial copying, lending, or selling is prohibited.

Errors or corrections? Email us at <aws-course-feedback@amazon.com>.

Other questions? Contact us at <https://aws.amazon.com/contact-us/aws-training/>

## Overview

This lab, designed for new AWS users.  It walks you through the authoring of a CloudFormation template and launching a CloudFormation stack.  You will learn about key objects in CloudFormation, including: template authoring with Resources, Parameters, and Mappings.  In this lab, you will launch an EC2 static web server into a VPC environment.  At the end of this lab, you will have a better understanding of the core components of CloudFormation template authoring.

In this exercise you will create a single EC2 server in an existing VPC.  The EC2 server will be inside of an autoscaling group to showcase one possible HA setup.  There are several footnotes to mark in this lab:

* You will launch a Linux system with a small web server running on it.  This lab is not Linux specific, it is only used as a demonstration.  You can use any system.
* This is not a fully baked HA solution.  It will only launch a new instance in response to the original instance failing.  Addressing to the instance will change, and there will be a small downtime while the instance relaunches.

The examples in this exercise will be deliberately simplistic, because most people start a template of their own from a downloaded sample such as the ones at <http://aws.amazon.com/cloudformation/aws-cloudformation-templates/>. It’s easier to modify an existing template than to create a template from scratch. This lab introduces the most important parts of an intermediate AWS CloudFormation template over several steps, explaining each element along the way.

## Topics covered

By the end of this lab, you will be able to:

* Create and execute a basic AWS CloudFormation template that defines various resources, including Amazon EC2 instances and security groups.
* Create a parameters section of a CloudFormation template to request dynamic inputs at runtime.
* Reference region-specific AMIs and other conditional data using the Mapping section.

## Technical knowledge prerequisites

To successfully complete this lab, you should be familiar with text editors and locating files after you have saved them.

**What Is Amazon AWS CloudFormation?**

AWS CloudFormation gives developers and systems administrators an easy way to create and manage a collection of related AWS resources, provisioning and updating them in an orderly and predictable fashion.

You can use the AWS CloudFormation sample templates or create your own templates to describe the AWS resources, and any associated dependencies or runtime parameters, required to run your application. You don’t need to figure out the order for provisioning AWS services or the subtleties of making those dependencies work. AWS CloudFormation takes care of this for you. After the AWS resources are deployed, you can modify and update them in a controlled and predictable way, in effect applying version control to your AWS infrastructure the same way you do with your software.

You can deploy and update a template and its associated collection of resources (called a stack) by using the AWS Management Console, AWS Command Line Interface, or APIs. AWS CloudFormation is available at no additional charge, and you pay only for the AWS resources needed to run your applications.

## Start Lab

Notice the lab properties below the lab title:

 * **setup -** The estimated time to set up the lab environment.
 * **access -** The time the lab will run before automatically shutting down.
 * **completion -** The estimated time the lab should take to complete.

1. [1]Click **START LAB** to launch your lab. If you are prompted for a token, use the one distributed to you (or credits you've purchased).

	![](https://s3-us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/sts-sign-in-images/media/start-lab.png)

	A status bar shows the progress of the lab environment creation process (the AWS Management Console is accessible during lab resource creation, but your AWS resources may not be fully available until the process is complete).
	
	![](https://s3-us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/sts-sign-in-images/media/loading.png)

2. [2]Click **OPEN CONSOLE**, which will automatically log you in to the AWS Console.

	![](https://s3-us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/sts-sign-in-images/media/open-console.png)

Please do not change the Region unless instructed.

---------------------------------------

### Common login errors

**Error : Federated login credentials**

![](https://s3-us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/sts-sign-in-images/media/federatedloginerror.png)

If you see this message:

   * Close the browser tab to return to your initial lab window
   * Wait a few seconds
   * Click Open Console again

You should now be able to access the AWS Management Console.

---------------------------------------

**Error: You must first log out**

![](https://s3-us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/sts-sign-in-images/media/logouterror.png)

If you see this message:

   * Click To logout, click here
   * Close the browser tab to return to your initial Qwiklabs window
   * Click Open Console again

---------------------------------------

## Create a VPC with AWS CloudFormation

3. [3]Download the [demovpc.template](https://s3.amazonaws.com/workday-cfn-immersionday/demovpc.template) file and save it to your computer.

4. [4]In the **AWS Management Console**, on the **Services** menu, click **CloudFormation**.

5. [5]Click **Create Stack**.

6. [6]In the **Choose a template** section, select <i class="fa fa-dot-circle-o" aria-hidden="true"></i> **Upload a template to Amazon S3**.

7. [7]Click **Browse**.

8. [8]Browse to and select your **demovpc.template** file.

9. [9]Click **Next**.

10. [10]On the **Specify Details** page, configure the following:

* For **Stack name**, enter `vpcStack`
* Click **Next**.

11. [11]On the **Options** page, click **Next**.


12. [12]On the **Review** page, review the parameters, and then click **Create**.

This will create the stack.

## Preparation

13. [13]Open a text editor on your computer.

* Make sure this is not a word processing program like Microsoft Word.  It will need to be a text editor.
* Use Notepad on a Windows system, or TextEditor on a Mac.
* You can also download other text editors like Sublime Text 3, or Notepad++.

14. [14]Create and save a new file called **ec2static.template**.  Save this file to a place where you can easily locate it later.

15. [15]Edit the file and put these two lines at the top.

```
AWSTemplateFormatVersion : "2010-09-09"
Description : "CloudFormation Template to create a wordpress server in existing vpc."
```

These lines are not required, but the Template Format Version is often used, and it is always a good idea to have a description for your cloudformation template.  These will be lines 1 and 2 in your template.

## Create the AutoScaling Group

16. [16]At line 3, add in the following **Resources** section.

```
Resources :
```

This is the area where you will set up the various components for AWS to create.

17. [17]Starting at line 4, add these lines.  Make sure you keep the indentation as it appears below.

```
  MyASG :
    Type : "AWS::AutoScaling::AutoScalingGroup"
    Properties :
      AutoScalingGroupName : "MyASG"
      DesiredCapacity : "1"
      LaunchConfigurationName : “”
      MaxSize : "1"
      MinSize : "1"
      VPCZoneIdentifier :
        - “”
        - “”
      Tags :
        - Key : "Name"
          Value : “”
          PropagateAtLaunch : "true"
        - Key : "Project"
          Value : "CloudFormation Immersion Day"
          PropagateAtLaunch : "true"
```

These properties are the minimum necessary to launch the autoscaling group.  For a comprehensive list of all available resources and their corresponding properties visit: ![]https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html.  Notice there are a few values that are just quotes.  These are values you will use CloudFormation templating features to fill out.  Let’s start with line 17 and the empty quotes after the value with the Key of ‘Name’.

This is a value that may change depending on circumstances, where the instances will be launched, what environment it may launch in, etc.  You will create a parameter to enter this value at the launch time of the CloudFormation stack.

18. [18]Add the following lines to the template at line 3.

```
Parameters : 
  ServerName :
    Type : "String"
    Description : "The name of the server."
    Default : "EC2 Static Web Server"
```

This will force CloudFormation to ask for a ServerName at the stack launch time, and default to the value of EC2 Static Server.  You can change this to anything you want at launch time.

19. [19]To place the value of this input into the tag field, update line 22 so it looks like:

```
          Value : !Ref ServerName
```

The **!Ref** is an intrinsic function that refers to whatever value follows it.  This will take the value entered into the ServerName parameter and place it into the value of the tag.

Let’s evaluate lines 18 and 19.  These placeholders are for the subnets that you will launch the server into.  You would not want to hard code these into a template as subnet targets will be different if you are launching this into different VPCs, different accounts, or different regions.  You will create a parameter for the subnets.  This will be a comma separated list of the available subnets to launch into.

20. [20]Add the following code at line 8 between the **Resources** line and the end of the **ServerName** parameter.

```
  Subnets :
    Type : "String"
    Description : "Comma Separated List of Target Subnets for the EC2 Server."
```

21. [21]Now find line 21 and 22.  Replace line 21 with:

```
        - !Select [0, !Split [ ",", !Ref Subnets]]
```

22. [22]And line 22 with:

```
        - !Select [1, !Split [ ",", !Ref Subnets]]
```

The **!Ref Subnets** is grabbing the value entered for the parameter.  The **!Split** is separating the value at the comma, and putting the values for the subnets into a list.  The **!Select** is grabbing the individual values of the subnets out of that list.

## Create a Launch Configuration

Notice the **""** at the value for line 17.  We need to enter the launch configuration here.  Since we do not have any launch configurations available, we will need to create one.

23. [23]Add this code at the end of your template, it should be line 30.

```
  MyLC :
    Type : "AWS::AutoScaling::LaunchConfiguration"
    Properties :
      ImageId: ""
      InstanceType : ""
      AssociatePublicIpAddress : True
      SecurityGroups :
        - ""
      UserData : !Base64 "#include\nhttps://awstechbootcamp.s3.amazonaws.com/bootstrap.sh"
```

24. [24]Add a new parameter for the **InstanceType** property at line 11.

```
  InstanceType :
    Type : "String"
    Default : "t2.micro"
    AllowedValues :
      - "t2.micro"
      - "m3.medium"
      - "c5.large"
    Description : "Enter the Instance Type for the WordPress Instance"
```

Notice the **AllowedValues** section here.  This option constrains the values that the user can enter when the stack is launched.

25. [25]Modify the value for the reference at line 42.

```
      InstanceType : !Ref InstanceType
```

**ImageId** at line 41 is a bit of a special case.  You could create another parameter and have the users track down the ami at launch time and input it.  This can cause copy/paste errors, grabbing the wrong ID from a confusing list and possible add errors to the launch process, especially because AMI ids change from region to region.  In this case you are going to use a mapping.  A mapping is a key value set that can switch between multiple values depending on circumstances.

26. [26]Add the following mapping to the template at line 19.

```
Mappings :
  RegionMap :
    us-east-1 :
      "ami" : "ami-1853ac65"
    us-west-1 :
      "ami" : "ami-bf5540df"
    us-west-2 :
      "ami" : "ami-d874e0a0"
    us-east-2 :
      "ami" : "ami-25615740"
```

The mapping will allow the use of different AMIs depending on the region where you launch the CloudFormation.

27. [27]Place the proper **AMI ID** into the value at line 51 by using the FindInMap intrinsic function.

```
      ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", "ami"]
```

The **!Ref “AWS::Region”** function returns the value of the region the CloudFormation stack is running, and the **FindInMap** looks up the mapping we entered earlier and returns the value.

28. [28]Place the reference for the Launch Configuration in line 35.

```
      LaunchConfigurationName : !Ref MyLC
```

## Add a Security Group

Line 55 shows a **“”** for the security groups.  Since we don’t have any security groups already created for this, you will need to create a new one.

29. [29]Add a new resource at the end of the template at line 57.

```
  MySG :
    Type : "AWS::EC2::SecurityGroup"
    Properties :
      GroupName : "StaticEC2SecurityGroup"
      GroupDescription : "Inbound Access Rules for Static EC2 Web Access"
      SecurityGroupIngress :
        - 
          CidrIp : "0.0.0.0/0"
          Description : "Inbound SSH Access from the Internet"
          FromPort : "22"
          IpProtocol : "tcp"
          ToPort : "22"
        - 
          CidrIp : "0.0.0.0/0"
          Description : "Inbound HTTP Access from the Internet"
          FromPort : "80"
          IpProtocol : "tcp"
          ToPort : "80"
      VpcId : ""
```

Since there is a missing value for the **VpcId**, you will need to enter that.  

30. [30]Add another parameter at line 19.

```
  VPC :
    Type : "String"
    Description : "The VPC ID for the target VPC"
```

31. [31]Replace the reference for the VPC ID at line 78.

```
      VpcId : !Ref VPC
```

32. [32]Finally, add the reference for the security group into line 58.

```
        - !Ref MySG
```

Now you are ready to launch the CloudFormation Template.Once complete, it will look like this:

```
AWSTemplateFormatVersion : "2010-09-09"
Description : "CloudFormation Template to create a wordpress server in existing vpc."
Parameters : 
  ServerName :
    Type : "String"
    Description : "The name of the server."
    Default : "EC2 Static Web Server"
  Subnets :
    Type : "String"
    Description : "Comma Separated List of Target Subnets for the EC2 Server."
  InstanceType :
    Type : "String"
    Default : "t2.micro"
    AllowedValues :
      - "t2.micro"
      - "m3.medium"
      - "c5.large"
    Description : "Enter the Instance Type for the WordPress Instance"
  VPC :
    Type : "String"
    Description : "The VPC ID for the target VPC"
Mappings :
  RegionMap :
    us-east-1 :
      "ami" : "ami-1853ac65"
    us-west-1 :
      "ami" : "ami-bf5540df"
    us-west-2 :
      "ami" : "ami-d874e0a0"
    us-east-2 :
      "ami" : "ami-25615740"
Resources :
  MyASG :
    Type : "AWS::AutoScaling::AutoScalingGroup"
    Properties :
      AutoScalingGroupName : "MyASG"
      DesiredCapacity : "1"
      LaunchConfigurationName : !Ref MyLC
      MaxSize : "1"
      MinSize : "1"
      VPCZoneIdentifier :
        - !Select [0, !Split [ ",", !Ref Subnets]]
        - !Select [1, !Split [ ",", !Ref Subnets]]
      Tags :
        - Key : "Name"
          Value : !Ref ServerName
          PropagateAtLaunch : "true"
        - Key : "Project"
          Value : "CloudFormation Immersion Day"
          PropagateAtLaunch : "true"
  MyLC :
    Type : "AWS::AutoScaling::LaunchConfiguration"
    Properties :
      ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", "ami"]
      InstanceType : !Ref InstanceType
      AssociatePublicIpAddress : True
      SecurityGroups :
        - !Ref MySG
      UserData : !Base64 "#include\nhttps://awstechbootcamp.s3.amazonaws.com/bootstrap.sh"
  MySG :
    Type : "AWS::EC2::SecurityGroup"
    Properties :
      GroupName : "StaticEC2SecurityGroup"
      GroupDescription : "Inbound Access Rules for Static EC2 Web Access"
      SecurityGroupIngress :
        - 
          CidrIp : "0.0.0.0/0"
          Description : "Inbound SSH Access from the Internet"
          FromPort : "22"
          IpProtocol : "tcp"
          ToPort : "22"
        - 
          CidrIp : "0.0.0.0/0"
          Description : "Inbound HTTP Access from the Internet"
          FromPort : "80"
          IpProtocol : "tcp"
          ToPort : "80"
      VpcId : !Ref VPC
```

## Launch Your Template

33. [33]In the **AWS Management Console**, on the **Services** menu, click **CloudFormation**.

34. [34]Click on the **vpcstack** you created earlier.  Find the **Resources** tab in the bottom window pane.

35. [35]Copy the subnet IDs, as well as the VPC ID.  You will need these values in a moment

 * The subnet ID will start with **subnet-** and be followed by an eight character identifier.
 * The VPC ID will start with **vpc-** and be followed by an eight character identifier.
 * Copy these values onto a new notepad page, or anywhere accessible to you later.

36. [36]Click **Create Stack**.

37. [37]In the **Choose a template** section, select <i class="fa fa-dot-circle-o" aria-hidden="true"></i> **Upload a template to Amazon S3**.

38. [38]Click **Browse**.

39. [39]Browse to and select your **ec2static.template** file.

40. [40]Click **Next**.

41. [41]On the **Specify Details** page, configure the following:

* For **Stack name**, enter `staticstack`
* For **ServerName**, either keep the default name, or give your server a personalized name.
* For **InstanceType**, click on the drop down, and view your available instance types.  Keep the choice on t2.micro.
* For **Subnets**, enter the two subnets you copied in step 35, separated by a comma.  The value should look similar to: **subnet-12345678,subnet-87654321**
* For **VPC**, enter the VPC ID you copied in step 35.  It should look similar to: **vpc-12345678**
* Click **Next**.

42. [42]On the **Options** page, click **Next**.


43. [43]On the **Review** page, review the parameters, and then click **Create**.

This will create the stack.

## Check the Results

44. [44]Once the CloudFormation stack has completed, click on the **Services** menu and find **EC2**.

45. [45]Click on the **Instances** menu option on the left.

You will find your instance here with the name you specified in the parameter list, or the default name of **My EC2 Server**.

46. [46]Click on the instance, and in the details pane below, copy the **Puclic DNS (IPv4)** value.

* The value will start with **ec2** and continue to **amazonaws.com**

47. [47]Paste the value into a browser and see your wordpress site!

48. [48]Click on the **Actions** button, hover over **Instance State** and click on **Terminate**.

Give it a few minutes, and a new instance will start up, fulfilling the autoscaling group desired limit of 1.

## Conclusion

Congratulations! You’ve finished this lab on configuring auto scaling. In this lab, you:

* Created and executed a basic AWS CloudFormation template that defines various resources, including Amazon EC2 instances and security groups.
* Used the Parameters section of an AWS CloudFormation template to request dynamic inputs at runtime.
* Supplied an initialization script to an Amazon EC2 instance by including User Data in your AWS CloudFormation template.
* Referenced region-specific AMIs and other conditional data using the Mapping section.

## End Lab

Follow these steps to close the console, end your lab, and evaluate the experience.

49. [49]Return to the AWS Management Console.

50. [50]On the navigation bar, click **&lt;yourusername&gt;@&lt;AccountNumber&gt;**, and then click **Sign out**.

51. [51]Click **END LAB**.

52. [52]Click **OK**.

53. [53]\(Optional\) Select the applicable number of stars, type a comment, and then click **SUBMIT**.

	**Note:** The number of stars indicates the following:

 * 1 star = Very dissatisfied
 * 2 stars = Dissatisfied
 * 3 stars = Neutral
 * 4 stars = Satisfied
 * 5 stars = Very satisfied

## Additional Resources

* For more information about AWS CloudFormation, see <http://aws.amazon.com/documentation/cloudformation/>.
* For more information about AWS Training and Certification, see <http://aws.amazon.com/training/>.
* For more AWS Self-Paced Labs, see <http://run.qwiklabs.com>.

For feedback, suggestions, or corrections, please email us at <aws-course-feedback@amazon.com>.

