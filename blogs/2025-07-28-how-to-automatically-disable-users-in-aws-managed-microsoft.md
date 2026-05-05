---
title: "How to automatically disable users in AWS Managed Microsoft AD based on GuardDuty findings"
url: "https://aws.amazon.com/blogs/security/how-to-automatically-disable-users-in-aws-managed-microsoft-ad-based-on-guardduty-findings/"
date: "Mon, 28 Jul 2025 15:45:16 +0000"
author: "Tim Kingdon"
feed_url: "https://aws.amazon.com/blogs/security/tag/amazon-guardduty/feed/"
---
<p>Organizations are facing an increasing number of security threats, especially in the form of compromised user accounts. Manually monitoring and acting on suspicious activities is not only time-consuming but also prone to human error. The lack of automated responses to security incidents can lead to disastrous consequences, such as data breaches and financial loss.</p> 
<p>In this blog post, I show you how to detect suspicious events using <a href="https://aws.amazon.com/guardduty" rel="noopener noreferrer" target="_blank">Amazon GuardDuty</a> and create an automation from those findings to disable user accounts in <a href="https://aws.amazon.com/directoryservice/" rel="noopener noreferrer" target="_blank">AWS Directory Service for Microsoft Active Directory</a>.</p> 
<p>This post addresses scenarios where, for example, you have a web server that uses a Microsoft Active Directory user account (service account) to access an application or database resources on other servers, and you want to automate disabling the user account if suspicious activity is detected.</p> 
<p>I walk you through how to deploy Microsoft Active Directory in AWS Directory Services, set up GuardDuty to monitor <a href="https://aws.amazon.com/ec2" rel="noopener noreferrer" target="_blank">Amazon Elastic Compute Cloud (Amazon EC2)</a> instances, and configure <a href="https://aws.amazon.com/eventbridge" rel="noopener noreferrer" target="_blank">Amazon EventBridge</a> with <a href="https://aws.amazon.com/step-functions" rel="noopener noreferrer" target="_blank">AWS Step Functions</a> to trigger <a href="https://aws.amazon.com/systems-manager" rel="noopener noreferrer" target="_blank">AWS Systems Manager</a> Run Command to obtain the username and disable the user in Active Directory.</p> 
<h2>Solution overview</h2> 
<p>In this example, shown in Figure 1, you deploy a test EC2 instance and enable GuardDuty runtime monitoring. Findings will trigger an EventBridge rule that executes a Step Functions state machine, which runs two Systems Manager Run Command documents that discover the username and disable that user using the directory administration EC2 instance.</p> 
<div class="wp-caption aligncenter" id="attachment_39117" style="width: 1001px;">
 <img alt="Figure 1: Solution architecture" class="size-full wp-image-39117" height="241" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/07/03/image-1-4.png" width="991" />
 <p class="wp-caption-text" id="caption-attachment-39117">Figure 1: Solution architecture</p>
</div> 
<h2>GuardDuty</h2> 
<p>GuardDuty is an automated threat detection service that continuously monitors for suspicious activity and unauthorized behavior to protect your AWS accounts, workloads, and data stored in <a href="https://aws.amazon.com/s3" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service (Amazon S3)</a>.</p> 
<p>To activate GuardDuty:</p> 
<ol> 
 <li>Go to GuardDuty on the AWS Management Console. 
  <ol type="a"> 
   <li>If you’re activating GuardDuty for the first time, under <strong>Try threat detection with GuardDuty</strong>, select <strong>All Features</strong> and then choose <strong>Get Started</strong>.</li> 
   <li>If you’ve used GuardDuty before, select <strong>Runtime Monitoring </strong>and then choose <strong>Enable</strong> under <strong>Runtime Monitoring</strong>.</li> 
  </ol> <p></p>
  <div class="wp-caption aligncenter" id="attachment_39130" style="width: 574px;">
   <img alt="Figure 2: GuardDuty Runtime Monitoring enabled with EC2 monitoring" class="size-full wp-image-39130" height="469" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/07/03/Figure-2-1.png" style="border: 1px solid #bebebe;" width="564" />
   <p class="wp-caption-text" id="caption-attachment-39130">Figure 2: GuardDuty Runtime Monitoring enabled with EC2 monitoring</p>
  </div> </li> 
</ol> 
<h2>AWS Managed Microsoft AD</h2> 
<p>AWS Managed Microsoft AD provides a fully managed service for Microsoft Active Directory (AD) in the AWS Cloud. When you create your directory, AWS deploys two domain controllers in separate Availability Zones that are exclusively yours for high availability. For use cases that require even higher resilience and performance in a specific AWS Region or during specific hours, you can scale AWS Managed Microsoft AD by deploying additional domain controllers to meet your needs. These domain controllers can help load balance, increase overall performance, or provide additional nodes to protect against temporary availability issues. Using AWS Managed Microsoft AD, you can define the correct number of domain controllers for your directory based on your use case.</p> 
<p><strong>To deploy a new AWS Managed Microsoft AD:</strong></p> 
<ol> 
 <li>Go to the Directory Service console.</li> 
 <li>Choose <strong>Set up directory</strong> and select <strong>AWS Managed Microsoft AD</strong>.</li> 
 <li>Select <strong>Standard Edition</strong> and enter a Directory DNS name and password.</li> 
 <li>Select a virtual private cloud (VPC), for this example use the <strong>Default VPC</strong>.</li> 
 <li>Choose <strong>Create directory</strong>.</li> 
</ol> 
<h2>Directory administration EC2 instance</h2> 
<p>This directory administration EC2 instance will be used to control the Microsoft Active Directory using AWS Systems Manager.</p> 
<p><strong>To deploy the directory administration EC2 instance:</strong></p> 
<ol> 
 <li>If you have deployed a new directory, you might need to wait 20–45 minutes until the directory status is <strong>Active</strong>.</li> 
 <li>Select the <strong>Directory ID</strong>.</li> 
 <li>Choose <strong>Actions</strong> and select <strong>Launch directory Administration EC2 Instance</strong>, using the default options.</li> 
</ol> 
<p>Alternatively, you can build your own Windows EC2 instances with a role that has the <code style="color: #000000;">AmazonSSMManagedInstanceCore</code> policy, join it to the Active Directory domain, and install Active Directory management tools.</p> 
<p><strong>To remotely connect to the directory administration EC2 instance:</strong></p> 
<ol> 
 <li>Go to the <strong>Systems Manager </strong>console.</li> 
 <li>Open <strong>Fleet Manager</strong> from the navigation pane.</li> 
 <li>Select the <strong>Node ID</strong> for the instance with the name ending <strong>managementInstance</strong>.</li> 
 <li>Choose <strong>Node Actions</strong> (top right), select <strong>Connect</strong>, and then choose <strong>Connect with Remote Desktop</strong>.</li> 
 <li>Enter the username admin and the directory password that you set earlier.</li> 
</ol> 
<h3>Create a test Active Directory user</h3> 
<p>You will use this test user account to sign in to an EC2 instance and initiate a command that simulates suspicious activity that results in this account being disabled.</p> 
<p><strong>To use the directory administration EC2 instance to create a test user on the Active Directory:</strong></p> 
<ol> 
 <li>From the <strong>management EC2 instance</strong>, open the start menu, select <strong>Windows Administrative Tools</strong> and then open <strong>Active Directory Users and Computers</strong>.</li> 
 <li>Browse to your <strong>Domain</strong>, the <strong>Domain OU</strong>, and then the <strong>Users OU</strong>, right-click and choose <strong>New</strong> and then select <strong>User</strong>.</li> 
 <li>Create a <strong>TestUser</strong> user, making sure that you don’t select <strong>Account is disabled</strong>.</li> 
</ol> 
<h3>Create a privileged domain service account</h3> 
<p>You will create this domain user account with delegated permissions to be used by Systems Manager Windows Service.</p> 
<p><strong>To use the directory administration EC2 instance to create a service account in AD:</strong></p> 
<ol> 
 <li>From the <strong>management EC2 instance</strong>, open the start menu, select <strong>Windows Administrative Tools</strong>, and then open <strong>Active Directory Users and Computers</strong>.</li> 
 <li>Browse to your <strong>Domain</strong>, the <strong>Domain OU</strong>, and then the <strong>Users OU</strong>. Right-click and select <strong>New</strong>, and then select <strong>User</strong></li> 
 <li>Create an <strong>SSMService</strong> user, making sure that you don’t select <strong>Account is disabled</strong>.</li> 
</ol> 
<p><strong>To delegate permission to the service account in AD:</strong></p> 
<ol> 
 <li>Right-click on the <strong>Users OU</strong> and select <strong>Delegate Control</strong>.</li> 
 <li>Choose <strong>Next</strong> on the <strong>Delegation of Control Wizard</strong>.</li> 
 <li><strong>Add</strong> the new service user you created earlier and choose <strong>Next</strong>.</li> 
 <li>Select <strong>Create a custom task to delegate</strong> and choose <strong>Next</strong>.</li> 
 <li>Select <strong>Only the following objects in the folder</strong> and select <strong>User Objects</strong>, then choose <strong>Next</strong>.</li> 
 <li>Select <strong>General</strong> and <strong>Property-specific</strong> to show the permissions, select <strong>Read userAccountControl</strong> and <strong>Write userAccountControl</strong> (near the end of the list), then choose <strong>Next</strong> and <strong>Finish</strong>.</li> 
</ol> 
<p><strong>To add a service account to the local administrators group:</strong></p> 
<ol> 
 <li>From the <strong>management EC2 instance</strong>, open the start menu, select <strong>Windows Administrative Tools</strong>, and then open <strong>Computer Management</strong>.</li> 
 <li>Browse to <strong>Local Users and Groups</strong>, then to <strong>Groups</strong>.</li> 
 <li>Right-click on <strong>Administrators</strong> and select <strong>Properties</strong>.</li> 
 <li>Choose <strong>Add </strong>to add the new service user you created earlier and choose<strong> OK</strong>.</li> 
</ol> 
<h3>Configure Systems Manager</h3> 
<p>Configure Systems Manager on the directory administration EC2 instance with permission to manage the Active Directory.</p> 
<p><strong>To configure Systems Manager:</strong></p> 
<ol> 
 <li>From the <strong>management EC2 instance</strong> from the <strong>Start Menu</strong>, select <strong>Windows Administrative Tools</strong>, and then open <strong>Services</strong>.</li> 
 <li>Locate the <strong>Amazon SSM Agent</strong>, right-click, and select <strong>Properties</strong>.</li> 
 <li>Select the <strong>Log On</strong> tab and select <strong>This account</strong>.</li> 
 <li>Within <strong>This account</strong> enter the <strong>privileged domain username</strong> you created earlier followed by <code style="color: #000000;">@</code> and then the domain name, for example <code style="color: #000000;">SSMService@corp.example.com</code>. Enter your password and choose <strong>OK</strong>.<br /> 
  <div class="wp-caption aligncenter" id="attachment_39131" style="width: 612px;">
   <img alt="Figure 3: Microsoft Windows Services showing Systems Manager Agent settings" class="size-full wp-image-39131" height="329" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/07/03/Figure-3-1.png" style="border: 1px solid #bebebe;" width="602" />
   <p class="wp-caption-text" id="caption-attachment-39131">Figure 3: Microsoft Windows Services showing Systems Manager Agent settings</p>
  </div> </li> 
 <li>Choose <strong>OK</strong> on the <strong>This account has been granted Log On As A Service right</strong> and <strong>The new logon name will not take effect until you stop and restart the service</strong> popups.</li> 
 <li>Right-click <strong>Amazon SSM Agent</strong> and select <strong>Restart</strong>.</li> 
</ol> 
<h2>Systems Manager Run Command</h2> 
<p>Run Command is a feature of Systems Manager that can remotely and securely manage the configuration of your managed nodes. You can use Run Command to automate common administrative tasks and perform one-time configuration changes at scale. You can use Run Command from the console, the <a href="https://aws.amazon.com/cli" rel="noopener noreferrer" target="_blank">AWS Command Line Interface (AWS CLI)</a>, <a href="https://aws.amazon.com/powershell/" rel="noopener noreferrer" target="_blank">AWS Tools for PowerShell</a>, or the AWS SDKs. Run Command is offered at no additional cost.</p> 
<p><strong>To create a Run Command document with a PowerShell command to disable domain user accounts:</strong></p> 
<ol> 
 <li>Go to the AWS Systems Manager console.</li> 
 <li>Select <strong>Documents</strong> under <strong>Change Management Tools</strong>.</li> 
 <li>Choose <strong>Create document</strong> and select <strong>Command or Session</strong>.</li> 
 <li>Enter a name, for example <code style="color: #000000;">DisableADUser</code>.</li> 
 <li>Select document type <strong>Command</strong>.</li> 
 <li>Select <strong>YAML</strong> and then enter the following code: 
  <div class="hide-language"> 
   <pre><code class="lang-text">---
schemaVersion: "2.2"
description: "Disable AD Users"
parameters:
  UserName:
    type: String
    description: "(Required) The username to disable."
mainSteps:
- action: "aws:runPowerShellScript"
  name: "DisableUser"
  inputs:
    runCommand:
    - "import-module activedirectory"
    - "$disableuser = get-aduser {{ UserName }} | select-object -ExpandProperty DistinguishedName"
    - "dsmod user $disableuser -disabled yes"
</code></pre> 
  </div> </li> 
 <li>Choose <strong>Create document</strong>.</li> 
</ol> 
<p><strong>To create a Run Command document with a bash command to find a username from a UserID:</strong></p> 
<ol> 
 <li>Follow steps 1–3 from the previous procedure.</li> 
 <li>Enter a name, for example <code style="color: #000000;">GetUsernameFromID</code>.</li> 
 <li>Select document type <strong>Command</strong>.</li> 
 <li>Select <strong>YAML</strong> and then enter the following code: 
  <div class="hide-language"> 
   <pre><code class="lang-text">---
description: "Get Username from Linux"
schemaVersion: "2.2"
parameters:
  UserId:
    type: String
    description: "(Required) The User ID to find."
    default: "1000"
mainSteps:
- action: aws:runShellScript
  name: GetLinuxUsername
  precondition:
    StringEquals:
    - platformType
    - Linux
  inputs:
    timeoutSeconds: 7200
    runCommand:
      - "#!/bin/bash"
      - "#"
      - "UserName=$(id -nu {{ UserId }})"
      - "if [[ $UserName == *'@'* ]]; then"
      - "echo ${UserName%@*} "
      - "else if [[ $UserName == *'\\'* ]]; then"
      - "echo $UserName | sed 's/.*\\\\//g'"
      - "fi"
      - "fi"
  outputs:
    - Name: output
      Selector: $.Payload.output
      Type: String
</code></pre> 
  </div> </li> 
 <li>Choose <strong>Create document</strong>.</li> 
</ol> 
<h2>Step Functions</h2> 
<p>Step Functions&nbsp;is a serverless orchestration service that you can use to coordinate multiple AWS services, microservices, and third-party integrations into business-critical applications. Step Functions is widely used for orchestrating complex workflows, such as loan processing, fraud detection, risk management, and compliance processes. By breaking down these processes into a series of steps, Step Functions provides a clear overview and control of the entire workflow. This helps make sure that it executes each stage correctly and in the right order. One of the critical aspects of using Step Functions in regulated industries is the importance of security and data protection.</p> 
<p>By the end of this section, your state machine should have a sequential flow that starts with a choice that defaults to <em>No UserID found</em> and with the UserID present, includes the steps <code style="color: #000000;">Find Username</code>, <code style="color: #000000;">Wait</code>, <code style="color: #000000;">Get Username</code>, and <code style="color: #000000;">Disable AD User</code>. If it doesn’t, you can drag the actions into the correct order or change the next state associated with each action. Alternatively, copy this <a href="https://aws-security-blog-content.s3.us-east-1.amazonaws.com/public/sample/2686-how-to-automatically-disable-users-in-aws-managed-microsoft-ad-based-on-guardduty-findings/Step-function-definition.json" rel="noopener" target="_blank">state machine definition JSON</a> and import it directly into Step Functions.</p> 
<p><strong>To create a Step Functions state machine to execute the Systems Manager Run Commands:</strong></p> 
<ol> 
 <li>Go to the Step Functions console.</li> 
 <li>Choose <strong>Get Started</strong>.</li> 
 <li>Choose <strong>Create your own</strong>.</li> 
 <li>Enter a name for the state machine, select <strong>Standard</strong>, and choose <strong>Continue</strong>.</li> 
 <li>Select <strong>JSONPath</strong> as the state machine query language.</li> 
 <li>From the navigation pane, search for and add the <strong>Pass</strong> action by <strong>dragging the action</strong> to the center window.</li> 
 <li>Add the <strong>Systems Manager: SendCommand</strong> Action for Finding the Username using Run Command.</li> 
 <li>Select the<strong> SendCommand</strong>, change the state name to <code style="color: #000000;">Find Username</code>, and then enter the following code into <strong>API Parameters</strong> on the right side of the screen. 
  <div class="hide-language"> 
   <pre><code class="lang-text">{
  "DocumentName": "<span style="color: #ff0000;"><em>GetUsernameFromID</em></span>",
  "Parameters": {
    "UserId.$": "States.Array(States.JsonToString($.detail.service.runtimeDetails.process.euid))"
  },
  "Targets": [
    {
      "Key": "InstanceIds",
      "Values.$": "States.Array($.detail.resource.instanceDetails.instanceId)"
    }
  ]
}
</code></pre> 
  </div> </li> 
 <li>With <strong>SendCommand </strong>selected, select the <strong>Input/Output tab</strong>, select <strong>Add original input to output using ResultPath</strong>, select <strong>Combine original input with result</strong>, and enter the following: 
  <div class="hide-language"> 
   <pre><code class="lang-text">$.RunCommand.State
</code></pre> 
  </div> </li> 
 <li>Add a <strong>Wait</strong> Action and set the <strong>number of seconds </strong>to wait before resuming the execution to <strong>5</strong> seconds.</li> 
 <li>Add a <strong>Systems Manager: GetCommandInvocation</strong> action, which will get the Username value from Run Command and change the state name to <strong>Get Username</strong>, then enter the following API Parameters. 
  <div class="hide-language"> 
   <pre><code class="lang-text">{
  "CommandId.$": "$.RunCommand.State.Command.CommandId",
  "InstanceId.$": "$.detail.resource.instanceDetails.instanceId"
}
</code></pre> 
  </div> </li> 
 <li>On the <strong>Input/Output</strong> tab, select <strong>Transform result with ResultSelector</strong> and enter the following: 
  <div class="hide-language"> 
   <pre><code class="lang-text">{
  "StandardOutputContent.$": "States.StringSplit($.StandardOutputContent,'\n')"
}
</code></pre> 
  </div> </li> 
 <li>Add a <strong>Systems Manager: SendCommand</strong> action which will disable the Active Directory user using Run Command. Change the state name to <strong>Disable AD User</strong> then enter the following API Parameters, changing the<strong> InstanceIds </strong>value to the ID of your Active Directory Management server. 
  <div class="hide-language"> 
   <pre><code class="lang-text">{
  "DocumentName": "<span style="color: #ff0000;"><em>DisableADUser</em></span>",
  "Parameters": {
    "UserName.$": "$.StandardOutputContent"
  },
  "Targets": [
    {
      "Key": "InstanceIds",
      "Values": [
        "<span style="color: #ff0000;"><em>i-0b22a22eec53b9321</em></span>"
      ]
    }
  ]
}
</code></pre> 
  </div> </li> 
 <li>Add a <strong>Choice</strong> action, choose the <strong>pencil icon</strong> next to Rule #1, choose <strong>Edit conditions</strong>, enter the variable <code style="color: #000000;">$.detail.service.runtimeDetails.process.euid</code>, select operator <strong>is present, </strong>value <strong>true</strong>, leave <strong>Not</strong> as blank, and choose <strong>Save Conditions</strong>.</li> 
 <li>Re-arrange the state machine layout to the same structure as displayed in Figure 4, with a sequential flow that starts with a choice that defaults to <em>No UserID found</em> and with the UserID present includes the steps <strong>Find Username</strong>, <strong>Wait</strong>, <strong>Get Username</strong>, and <strong>Disable AD User</strong>.<br /> 
  <div class="wp-caption aligncenter" id="attachment_39136" style="width: 606px;">
   <img alt="Figure 4: Step Functions state machine structure" class="size-full wp-image-39136" height="584" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/07/03/image-4r2.png" width="596" />
   <p class="wp-caption-text" id="caption-attachment-39136">Figure 4: Step Functions state machine structure</p>
  </div> </li> 
 <li>Choose <strong>Create</strong> (top right) and then <strong>Confirm</strong> to create the step function state machine.</li> 
</ol> 
<p><strong>To add permissions to enable the State Machine to run System Manager commands:</strong></p> 
<ol> 
 <li>Within the newly created state machine, choose <strong>Config</strong> (top center).</li> 
 <li>Choose <strong>View in IAM</strong>, under <strong>Permissions</strong>, <strong>Execution role</strong>.</li> 
 <li>Choose <strong>Add permissions</strong>, <strong>Attach Polices</strong> (center right).</li> 
 <li>Search for and select <strong>AmazonSSMAutomationRole</strong> and choose <strong>Add permission</strong>.</li> 
</ol> 
<h2>EventBridge</h2> 
<p>EventBridge helps developers build event-driven architectures (EDA) by connecting loosely coupled publishers and consumers using event routing, filtering, and transformation. To create an EventBridge rule that triggers the Systems Manger Run Command document you created earlier:</p> 
<ol> 
 <li>Go to the Amazon EventBridge console and select <strong>Create rule</strong> with <strong>EventBridge Rule</strong>.</li> 
 <li>Enter a name, for example <code style="color: #000000;">GuardDutyDisableADuser</code>.</li> 
 <li>Select <strong>Rule with an event pattern</strong> and choose <strong>Next</strong>.</li> 
 <li>Under the Event pattern JSON window, choose <strong>Edit pattern </strong>and enter the following: 
  <div class="hide-language"> 
   <pre><code class="lang-text">{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"]
}
</code></pre> 
  </div> </li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>Select <strong>AWS Service</strong>.</li> 
 <li>Select <strong>Step Functions state machine</strong> as the target.</li> 
 <li>Select the state machine you created earlier, for example <strong>MyStateMachine-A123456789</strong>.</li> 
 <li>Choose <strong>Next</strong> twice and choose <strong>Create rule</strong></li> 
</ol> 
<h2>Create a test EC2 instance</h2> 
<p>To generate alerts on GuardDuty, you create a domain joined Linux EC2 instance. For this example, you’ll use two separate EC2 instances so you can monitor for activity from each instance within GuardDuty and use EventBridge to create automations.</p> 
<p><strong>To create an </strong><a href="https://aws.amazon.com/iam/" rel="noopener noreferrer" target="_blank">AWS Identity and Access Management (IAM)</a> <strong>role to permit the EC2 instance to join the AD:</strong></p> 
<ol> 
 <li>Go to the IAM console.</li> 
 <li>Select<strong> Policies</strong> from the navigation pane.</li> 
 <li>Choose <strong>Create policy</strong> (top right).</li> 
 <li>Select Policy editor <strong>JSON</strong>, enter the following code and choose <strong>Next</strong>. 
  <div class="hide-language"> 
   <pre><code class="lang-text">{
"Version": "2012-10-17",
"Statement": [
	{
		"Effect": "Allow",
		"Action": [
			"secretsmanager:GetSecretValue",
			"secretsmanager:DescribeSecret"
			],
		"Resource": "*"
	}
	]
}
</code></pre> 
  </div> </li> 
 <li>Enter the <strong>Policy name</strong>, for example <code style="color: #000000;">SecretsManagerGetSecrets</code>, and choose <strong>Create policy</strong>.</li> 
 <li>Select <strong>Roles</strong> from navigation pane.</li> 
 <li>Choose <strong>Create role</strong> (top right).</li> 
 <li>Select <strong>AWS service</strong> and choose <strong>EC2</strong> from the service or use case selection, then choose <strong>Next</strong>.</li> 
 <li>Search for and select the following policies and choose <strong>Next</strong> 
  <ul> 
   <li>AmazonSSMDirectoryServiceAccess</li> 
   <li>AmazonSSMManagedInstanceCore</li> 
   <li>SecretsManagerGetSecrets (created earlier)</li> 
  </ul> </li> 
 <li>Enter the <strong>role name</strong>, for example <code style="color: #000000;">EC2DomainJoin</code>, and choose <strong>Create role</strong>.</li> 
</ol> 
<p><strong>To create a secret that will be used to store privileged credentials used to join EC2 instances to the domain:</strong></p> 
<ol> 
 <li>Go to the Secrets Manager console.</li> 
 <li>Select <strong>Store a new secret</strong>.</li> 
 <li>Select <strong>Other type of secret</strong>.</li> 
 <li>Add the following keys with the corresponding value of a domain username and password that have permissions to join computers to the domain: 
  <ol type="a"> 
   <li>awsSeamlessDomainUsername</li> 
   <li>awsSeamlessDomainPassword</li> 
  </ol> </li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li><strong>Enter the following</strong> secret name, replacing <code style="color: #ff0000;"><em>&lt;d-1234567890&gt;</em></code> with your directory ID. 
  <div class="hide-language"> 
   <pre><code class="lang-text">aws/directory-services/<span style="color: #ff0000;"><em>&lt;d-1234567890&gt;</em></span>/seamless-domain-join
</code></pre> 
  </div> </li> 
 <li>Choose <strong>Next </strong>twice, then <strong>Store</strong>.</li> 
</ol> 
<p>For more information more, see <a href="https://docs.aws.amazon.com/directoryservice/latest/admin-guide/ad_connector_seamlessly_join_linux_instance.html" rel="noopener noreferrer" target="_blank">Seamlessly joining an Amazon EC2 Linux instance to your AWS Managed Microsoft AD Active Directory</a>.</p> 
<p><strong>To create a domain joined EC2 instance for testing this GuardDuty automation:</strong></p> 
<ol> 
 <li>Go to the Amazon EC2 console.</li> 
 <li>Select <strong>Instances</strong> from navigation pane.</li> 
 <li>Choose <strong>Launch Instances</strong>.</li> 
 <li>Select <strong>Amazon Linux AMI</strong>.</li> 
 <li>Select an existing <strong>Key Pair</strong> or create a new key pair.</li> 
 <li>Scroll to the bottom and select <strong>Advanced details</strong>.</li> 
 <li>Within <strong>Domain join directory</strong>, select the domain</li> 
 <li>Within <strong>IAM instance profile</strong>, select the <strong>EC2DomainJoin</strong> role that you created earlier.</li> 
 <li>Choose <strong>Launch Instance</strong>.</li> 
</ol> 
<h2>Testing</h2> 
<p>To simulate a threat, use a GuardDuty test domain that GuardDuty will recognize as a command and control server.</p> 
<ol> 
 <li>Go to the Amazon EC2 console.</li> 
 <li>Choose <strong>Instances</strong> from the navigation pane.</li> 
 <li>Select the test EC2 instance that you created earlier.</li> 
 <li>Choose <strong>Connect</strong>, select the <strong>Session Manager </strong>tab, and choose <strong>Connect</strong></li> 
 <li>Authenticate with your test user by entering <code style="color: #000000;">su</code> followed by the test user with the domain name that you created earlier. For example <code style="color: #000000;">su TestUser@example.com</code>, then enter the password.</li> 
 <li>Enter the command <code style="color: #000000;">curl guarddutyc2activityb.com</code>. 
  <ul> 
   <li>You will receive an error because the page won’t resolve, but GuardDuty will have detected suspicious events.</li> 
  </ul> </li> 
 <li>Go to the GuardDuty console and select <strong>Findings</strong> from the navigation pane.</li> 
 <li>Within 3–5 minutes, you should see a high severity finding for <strong>Backdoor:EC2/C&amp;CActivity.B!DNS</strong>.</li> 
</ol> 
<blockquote>
 <p><strong>Note</strong>: You must archive the GuardDuty finding before re-running this test, because the EventBridge rule only runs once against a GuardDuty finding with the same details. To archive the finding, select the check box next to the <strong>Backdoor:EC2/C&amp;CActivity.B!DNS</strong> finding, choose <strong>Actions</strong> (top right), and select <strong>Archive</strong>.</p>
</blockquote> 
<div class="wp-caption aligncenter" id="attachment_39122" style="width: 1081px;">
 <img alt="Figure 5: GuardDuty simulated findings" class="size-full wp-image-39122" height="305" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/07/03/image-5-1.png" style="border: 1px solid #bebebe;" width="1071" />
 <p class="wp-caption-text" id="caption-attachment-39122">Figure 5: GuardDuty simulated findings</p>
</div> 
<p>If you go back to <strong>Active Directory Users and Computers</strong> on the Directory Administration EC2 instance, you should see that the Test User is now disabled. You can enable the user by right-clicking on the user and selecting <strong>Enable Account</strong>.</p> 
<div class="wp-caption aligncenter" id="attachment_39123" style="width: 654px;">
 <img alt="Figure 6: Active Directory Users and Computers showing the disabled test use" class="size-full wp-image-39123" height="337" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/07/03/image-6-2.png" style="border: 1px solid #bebebe;" width="644" />
 <p class="wp-caption-text" id="caption-attachment-39123">Figure 6: Active Directory Users and Computers showing the disabled test use</p>
</div> 
<h2>Conclusion</h2> 
<p>In this post, you learned how to deploy AWS Managed AD, Systems Manager Run Command, EventBridge, Step Functions, and GuardDuty to monitor for suspicious events and disable the associated Active Directory user account.</p> 
<p>You can expand this scenario by creating Run Command documents that reset Active Directory passwords, disable computer accounts, or Active Directory tasks supported by Microsoft PowerShell. Additionally, you can add steps within the Step Functions state machine to notify administrators through <a href="https://aws.amazon.com/sns" rel="noopener noreferrer" target="_blank">Amazon Simple Notification Service (Amazon SNS)</a> or add additional checks with <a href="https://aws.amazon.com/lambda" rel="noopener noreferrer" target="_blank">AWS Lambda</a>.</p> 
<p>Although this post uses AWS Managed Microsoft AD, the same functionality can be achieved with a manual deployment of Active Directory on Amazon EC2 or on-premises, either by using an EC2 instance joined to the Active Directory domain with the Active Directory administration tools installed or by installing Systems Manager agent onto a management server on-premises.</p> 
<p>If you have feedback about this post, submit comments in the <strong>Comments</strong> section below. If you have questions about this post, start a new thread on <a href="https://repost.aws/tags/TAkQ_AMw65SICuEGEmuUXv4g/amazon-guardduty" rel="noopener" target="_blank">AWS re:Post GuardDuty</a> or <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank" title="contact AWS Support">contact AWS Support</a>.</p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Tim Kingdon" class="aligncenter size-full wp-image-39126" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/07/03/Tim-Kingdon.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Tim Kingdon</h3> 
  <p>Tim is a Senior Partner Solutions Architect at AWS and has more than 25 years of experience in healthcare, financial, government, and defence industries. In his role, he provides strategic technical guidance to partners and helps drive their success through technical enablement initiatives.</p> 
  <p></p>
 </div> 
</footer>
