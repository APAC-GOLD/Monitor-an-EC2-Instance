# Monitor an EC2 Instance

Lab overview
Logging and monitoring are techniques implemented to achieve a common goal. They work together to help ensure that a system’s performance baselines and security guidelines are always met.

Logging refers to recording and storing data events as log files. Logs contain low-level details that can give you visibility into how your application or system performs under certain circumstances. From a security standpoint, logging helps security administrators identify red flags that are easily overlooked in their system.

Monitoring is the process of analyzing and collecting data to help ensure optimal performance. Monitoring helps detect unauthorized access and helps align your services’ usage with organizational security.

In this lab, you create an Amazon CloudWatch alarm that initiates when the Amazon Elastic Compute Cloud (Amazon EC2) instance exceeds a specific central processing unit (CPU) utilization threshold. You create a subscription using Amazon Simple Notification Service (Amazon SNS) that sends an email to you if this alarm is goes off. You log in to the EC2 instance and run a stress test command that causes the CPU utilization of the EC2 instance to reach 100 percent.

This test simulates a malicious actor gaining control of the EC2 instance and spiking the CPU. CPU spiking has various possible causes, one of which is malware.

OBJECTIVES
After completing this lab, you should be able to:

Create an Amazon SNS notification
Configure a CloudWatch alarm
Stress test an EC2 instance
Confirm that an Amazon SNS email was sent
Create a CloudWatch dashboard
DURATION
This lab requires approximately 60 minutes to complete.

LAB ENVIRONMENT
The lab environment includes one preconfigured EC2 instance named Stress Test with an attached AWS Identity and Access Management (IAM) role that you can use to connect via AWS Systems Manager session manager.

All backend components, such as Amazon EC2, IAM roles, and some AWS services, have been built into the lab already.

Start lab
To launch the lab, at the top of the page, choose Start lab.
 You must wait for the provisioned AWS services to be ready before you can continue.

To open the lab, choose Open Console.
You are automatically signed in to the AWS Management Console in a new web browser tab.

 Do not change the Region unless instructed.

COMMON SIGN-IN ERRORS
Error: You must first sign out


If you see the message, You must first log out before logging into a different AWS account:

Choose the click here link.
Close your Amazon Web Services Sign In web browser tab and return to your initial lab page.
Choose Open Console again.
Error: Choosing Start Lab has no effect
In some cases, certain pop-up or script blocker web browser extensions might prevent the Start Lab button from working as intended. If you experience an issue starting the lab:

Add the lab domain name to your pop-up or script blocker’s allow list or turn it off.
Refresh the page and try again.
Task 1: Configure Amazon SNS
In this task, you create an SNS topic and then subscribe to it with an email address.

Amazon SNS is a fully managed messaging service for both application-to-application (A2A) and application-to-person (A2P) communication.

In the AWS Management Console, enter SNS in the search  bar, and then choose Simple Notification Service.

On the left, choose the  button, choose Topics, and then choose Create topic.

On the Create topic page in the Details section, configure the following options:

Type: Choose Standard.
Name: Enter 

MyCwAlarm
Choose Create topic.

On the MyCwAlarm details page, choose the Subscriptions tab, and then choose Create subscription.

On the Create subscription page in the Details section, configure the following options:

Topic ARN: Leave the default option selected.
Protocol: From the dropdown list, choose Email.
Endpoint: Enter a valid email address that you can access.
Choose Create subscription.
In the Details section, the Status should be Pending confirmation. You should have received an AWS Notification - Subscription Confirmation email message at the email address that you provided in the previous step.

Open the email that you received with the Amazon SNS subscription notification, and choose Confirm subscription.

Go back to the AWS Management Console. In the left navigation pane, choose Subscriptions.

The Status should now be  Confirmed.

SUMMARY OF TASK 1
In this task, you created an SNS topic and then created a subscription for the topic by using an email address. This topic is now able to send alerts to the email address that you associated with the Amazon SNS subscription.

Task 2: Create a CloudWatch alarm
In this task, you view some metrics and logs stored within CloudWatch. You then create a CloudWatch alarm to initiate and send an email to your SNS topic if the Stress Test EC2 instance increases to more than 60 percent CPU utilization.

CloudWatch is a monitoring and observability service built for DevOps engineers, developers, site reliability engineers (SREs), IT managers, and product owners. CloudWatch provides you with data and actionable insights to monitor your applications, respond to system-wide performance changes, and optimize resource utilization. CloudWatch collects monitoring and operational data in the form of logs, metrics, and events. You get a unified view of operational health and gain visibility of your AWS resources, applications, and services running on AWS and on premises.

In the AWS Management Console, enter Cloudwatch in the search  bar, and then choose it.

In the left navigation pane, choose the  Metrics dropdown list, and then choose All metrics.

CloudWatch usually takes 5-10 minutes after the creation of an EC2 instance to start fetching metric details.

On the Metrics page, choose EC2, and choose Per-Instance Metrics.

From this page, you can view all the metrics being logged and the specific EC2 instance for the metrics.

Select the check box with CPUUtilization as the Metric name for the Stress Test EC2 instance.
The following image shows the metrics and instance that you should select.


*The following CloudWatch metrics are shown for the Stress Test: StatusCheckFailed_System, StatusCheckFailed_Instance, NetworkPacketsIn, NetworkPacketsOut, CPUUtilization, and NetworkIn.

This option displays the graph for the CPU utilization metric, which should be approximately 0 because nothing has been done yet.

In the left navigation pane, choose the Alarms dropdown list, and then choose All alarms.
You now create a metric alarm. A metric alarm watches a single CloudWatch metric or the result of a math expression based on CloudWatch metrics. The alarm performs one or more actions based on the value of the metric or expression relative to a threshold over a number of time periods. The action then sends a notification to the SNS topic that you created earlier.

Choose Create alarm.

Choose Select metric, choose EC2, and then choose Per-Instance Metrics.

Select the check box with CPUUtilization as the Metric name for the Stress Test instance name.

Choose Select metric.

On the Specify metric and conditions page, configure the following options:

METRIC
Metric name: Enter 

CPUUtilization

InstanceId: Leave the default option selected.

Statistic: Enter 

Average

Period: From the dropdown list, choose 1 minute.

CONDITIONS
Threshold type: Choose Static.

Whenever CPUUtilization is…: Choose Greater threshold.

than… Define the threshold value: Enter 

60

The following image illustrates an example of an alarm configuration.

![Alt text](image-3.png)

CloudWatch metrics that should be selected.

CloudWatch Alarm configuration example.

The following CloudWatch Alarm configurations are shown: Metric name: CPUUtilization; InstanceId: i-09xxxxxxxx; Instance name: Stress Test; Statistic: Average; Period: 1 minute; and Threshold type: Static. The threshold value is set to 60 for the Whenever CPUUtilization is greater than setting, and exceeding this threshold will set off the alarm.

Choose Next.

On the Configure actions page, configure the following options:

NOTIFICATION
Alarm state trigger: Choose In alarm.
Select an SNS topic: Choose Select an existing SNS topic.
Send a notification to…: Choose the text box, and then choose MyCwAlarm.
Choose Next, and then configure the following options:

NAME AND DESCRIPTION
Alarm name: Enter 

LabCPUUtilizationAlarm
Alarm description - optional: Enter 

CloudWatch alarm for Stress Test EC2 instance CPUUtilization
Choose Next

Review the Preview and create page, and then choose Create alarm.

SUMMARY OF TASK 2
In this task, you viewed some Amazon EC2 metrics within CloudWatch. You then created a CloudWatch alarm that initiates an In alarm state when the CPU utilization threshold exceeds 60 percent.

Task 3: Test the Cloudwatch alarm
In this task, you log in to the Stress Test EC2 instance and run a command that stresses the CPU load to 100 percent. This increase in CPU utilization activates the CloudWatch alarm, which causes Amazon SNS to send an email notification to the email address associated with the SNS topic.

On the panel to the left of these instructions, next to EC2InstanceURL, there is a link. Copy and paste this link into a new browser tab.
This link connects you to the Stress Test EC2 instance.

To manually increase the CPU load of the EC2 instance, run the following command:

sudo stress --cpu 10 -v --timeout 400s
The output from the command should look similar to the following image.

![Alt text](image-2.png)

An example of the sudo stress --cpu 10 -v --timeout 400s command in the AWS CLI.

When running the sudo stress --cpu 10 -v --timeout 400s command within the AWS CLI, the output shows that 10 CPUs were at 100 percent load over the period of 400 seconds. Once 400 seconds passes, it will drop down to 0 percent.

This command runs for 400 seconds, loads the CPU to 100 percent, and then decreases the CPU to 0 percent after the allotted time.

On the panel to the left of these instructions, next to EC2InstanceURL, there is a link. Copy and paste this link into a new browser tab to open a second terminal for the Stress Test instance.

In the new terminal, run the following command:


top
This command shows the live CPU usage.

Navigate back to the AWS console where you have the CloudWatch Alarms page open.

Choose LabCPUUtilizationAlarm.

Monitor the graph while selecting the refresh button every 1 minute until the alarm status is In alarm.

 It takes a few minutes for the alarm status to change to In alarm and for an email to send.

On the graph, you can see where CPUUtilization has increased above the 60 percent threshold.

Navigate to your email inbox for the email address that you used to configure the Amazon SNS subscription. You should see a new email notification from AWS Notifications.
SUMMARY OF TASK 3
In this task, you ran a command to load the EC2 instance to 100 percent for 400 seconds. This increase in CPU utilization activated the alarm to go into the In alarm state, and you confirmed the spike in the CPU utilization by viewing the CloudWatch graph. You also received a email notification alerting you of the In alarm state.

Task 4: Create a CloudWatch dashboard
In this task, you create a CloudWatch dashboard using the same CPUUtilization metrics that you have used throughout this lab.

CloudWatch dashboards are customizable home pages in the CloudWatch console that you can use to monitor your resources in a single view. With CloudWatch dashboards, you can even monitor resources that are spread across different Regions. You can use CloudWatch dashboards to create customized views of the metrics and alarms for your AWS resources.

Go to the CloudWatch section in the AWS console. In the left navigation pane, choose Dashboards.

Choose Create dashboard.

For Dashboard name, enter 

LabEC2Dashboard
 and then choose Create dashboard.

Choose Line.

Choose Metrics.

Choose EC2, and then choose Per-Instance Metrics.

Select the check box with Stress Test for the Instance name and CPUUtilization for the Metric name.

Choose Create widget.

Choose Save dashboard.

Now you have created a quick access shortcut to view the CPUUtilization metric for the Stress Test instance.

LAB SUMMARY
In this lab, you created a CloudWatch alarm that activated when the Stress Test instance exceeded a specific CPU utilization threshold. You created a subscription using Amazon SNS that sent an email to you if this alarm goes off. You logged in to the EC2 instance and ran a stress test command that spiked the EC2 instance to 100 percent CPU utilization.

This test simulated what could happen if a malicious actor were to gain control of an EC2 instance and spike CPU utilization. CPU spiking has various possible causes, one of which is malware.

Conclusion 
 Congratulations! You now have successfully:

Created an Amazon SNS notification
Configured a Cloudwatch alarm
Stress tested an EC2 instance
Confirmed that an Amazon SNS email was sent
Created a CloudWatch dashboard
https://awsrestart.labs.awsevents.com/lab/arn%3Aaws%3Alearningcontent%3Aus-east-1%3A470679935125%3Ablueprintversion%2FCUR-TF-100-RSSECY-3%2F281-lab-SF-MonitorEC2Instance%3A3.0.0-12270342/en-US/46a424b2-440d-424c-b01f-97df9bc7474a::eGviHkWWJEdd9cFUbwXHCR
