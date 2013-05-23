---
layout: post
title: "Quick Steps to Configure Amazon AutoScaling"
---

Setup Environment and Credentials
---------------------------------

    aptitude install openjdk-6-jre-headless
    cat <<-ACCESS > <PATH_TO_CREDENTIAL_FILE>
    AWSAccessKeyId=<YOUR_KEY_ID>
    AWSSecretKey=<YOUR_KEY_SECRET>
    ACCESS
    export AWS_CREDENTIAL_FILE=<PATH_TO_CREDENTIAL_FILE>
    export JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:bin/java::")


Install AutoScaling Command Line Tools
--------------------------------------

    wget http://ec2-downloads.s3.amazonaws.com/AutoScaling-2011-01-01.zip
    unzip AutoScaling-2011-01-01.zip
    export AWS_AUTO_SCALING_HOME=$( readlink -f AutoScaling-1.0.61.2 ) ## *CHECK VERSION BEFORE EXPORT VARIABLE*
    export PATH="$PATH:$AWS_AUTO_SCALING_HOME/bin"


Install CloudWatch Command Line Tool
------------------------------------


    wget http://ec2-downloads.s3.amazonaws.com/CloudWatch-2010-08-01.zip
    unzip CloudWatch-2010-08-01.zip
    export AWS_CLOUDWATCH_HOME=$( readlink -f CloudWatch-1.0.13.4 ) ## *CHECK VERSION BEFORE EXPORT VARIABLE*
    export PATH="$PATH:$AWS_CLOUDWATCH_HOME/bin"


Setup AutoScaling
-----------------

    as-create-launch-config <LAUNCH_CONFIG_NAME> --image-id ami-00000000 --instance-type m1.small --group <SECURITY_GROUPS_comma_separated> --key <YOUR_KEY_NAME.pem>

    as-create-auto-scaling-group <AUTO_SCALE_GROUP_NAME> --launch-configuration <LAUNCH_CONFIG_NAME> --availability-zones us-east-1a us-east-1d  --min-size 2 --max-size 5 --load-balancers <ELB_NAME>


    as-put-scaling-policy <SCALE_UP_POLICY_NAME> --auto-scaling-group <AUTO_SCALE_GROUP_NAME> --adjustment=1 --type ChangeInCapacity  --cooldown 300

    mon-put-metric-alarm <HIGH_CPU_ALARM_NAME>  --comparison-operator  GreaterThanThreshold  --evaluation-periods  1 --metric-name  CPUUtilization  --namespace  "AWS/EC2"  --period  600  --statistic Average --threshold  80 --dimensions "AutoScalingGroupName=<AUTO_SCALE_GROUP_NAME>" --alarm-actions <**POLICY-ARN_FROM_PREVIOUS_STEP**>


    as-put-scaling-policy <SCALE_DOWN_POLICY_NAME> --auto-scaling-group <AUTO_SCALE_GROUP_NAME>  --adjustment=-1 --type ChangeInCapacity  --cooldown 300

    mon-put-metric-alarm <LOW_CPU_ALARM_NAME>  --comparison-operator  LessThanThreshold  --evaluation-periods  1 --metric-name  CPUUtilization  --namespace  "AWS/EC2"  --period  600  --statistic Average --threshold  35 --dimensions "AutoScalingGroupName=<AUTO_SCALE_GROUP_NAME>" --alarm-actions <**POLICY-ARN_FROM_PREVIOUS_STEP**>



Destroying AutoScaling Group
----------------------------

    as-update-auto-scaling-group <AUTO_SCALE_GROUP_NAME> --min-size 0 --max-size 0
    as-delete-auto-scaling-group <AUTO_SCALE_GROUP_NAME>
    as-delete-launch-config <LAUNCH_CONFIG_NAME>

Check if environment is OK
--------------------------

    $ as-describe-scaling-activities
    ACTIVITY  eb20abac-d01f-434a-bfcb-fa1961328df4                        <AUTO_SCALE_GROUP_NAME>  InProgress
    ACTIVITY  cf2a0319-3f3b-4766-b91c-1c491e447087  2013-05-22T23:03:08Z  <AUTO_SCALE_GROUP_NAME>  Successful

    $ as-describe-launch-configs
    LAUNCH-CONFIG  <LAUNCH_CONFIG_NAME>  ami-00000000  m1.small

    $ as-describe-auto-scaling-groups
    AUTO-SCALING-GROUP  <AUTO_SCALE_GROUP_NAME>  <LAUNCH_CONFIG_NAME>  us-east-1e,us-east-1d  <ELB_NAME>  1  5  1  Default
    INSTANCE  i-00000000  us-east-1d  InService  Healthy  <LAUNCH_CONFIG_NAME>

    $ as-describe-auto-scaling-instances
    INSTANCE  i-00000000  <AUTO_SCALE_GROUP_NAME>  us-east-1d  InService  HEALTHY  <LAUNCH_CONFIG_NAME>

    $ as-describe-policies --headers
    SCALING-POLICY  GROUP-NAME  POLICY-NAME  SCALING-ADJUSTMENT        ADJUSTMENT-TYPE     COOLDOWN                    POLICY-ARN
    SCALING-POLICY  <AUTO_SCALE_GROUP_NAME>  <SCALE_DOWN_POLICY_NAME>  -1                  ChangeInCapacity  300       arn:aws:autoscaling:us-east-1:000000000000:scalingPolicy:00000000-0000-0000-0000-000000000000:autoScalingGroupName/<AUTO_SCALE_GROUP_NAME>:policyName/<SCALE_DOWN_POLICY_NAME>
    ALARM  ALARM-NAME            POLICY-NAME
    ALARM  <LOW_CPU_ALARM_NAME>  <SCALE_DOWN_POLICY_NAME>
    SCALING-POLICY  <AUTO_SCALE_GROUP_NAME>  <SCALE_UP_POLICY_NAME>    1                   ChangeInCapacity  300       arn:aws:autoscaling:us-east-1:000000000000:scalingPolicy:00000000-0000-0000-0000-000000000000:autoScalingGroupName/<AUTO_SCALE_GROUP_NAME>:policyName/<SCALE_UP_POLICY_NAME>
    ALARM  ALARM-NAME             POLICY-NAME
    ALARM	 <HIGH_CPU_ALARM_NAME>  <SCALE_UP_POLICY_NAME>
