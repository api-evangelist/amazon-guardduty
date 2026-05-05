---
title: "Testing and evaluating GuardDuty detections"
url: "https://aws.amazon.com/blogs/security/testing-and-evaluating-guardduty-detections/"
date: "Tue, 28 Jan 2025 19:47:55 +0000"
author: "Marshall Jones"
feed_url: "https://aws.amazon.com/blogs/security/tag/amazon-guardduty/feed/"
---
<p><a href="https://aws.amazon.com/guardduty/" rel="noopener" target="_blank">Amazon GuardDuty</a> is a threat detection service that continuously monitors, analyzes, and processes <a href="https://aws.amazon.com/" rel="noopener" target="_blank">Amazon Web Services (AWS)</a> data sources and logs in your AWS environment. GuardDuty uses threat intelligence feeds, such as lists of malicious IP addresses and domains, file hashes, and machine learning (ML) models to identify suspicious and potentially malicious activity in your AWS environment. When GuardDuty identifies a potential security issue, it creates a <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_findings.html" rel="noopener" target="_blank">GuardDuty finding</a> that gives you information about what the potential security issue is, the resources involved, and contextualized information that’s key to remediating the issue. GuardDuty helps you monitor for the latest threats by continually expanding threat detection to emerging and common threats.</p> 
<p>Whether you’re new to GuardDuty or are a long-time user, it’s recommended that you understand the different GuardDuty <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html" rel="noopener" target="_blank">finding types</a> and <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_findings-summary.html" rel="noopener" target="_blank">finding details</a> and practice responding to them as suggested in the <a href="https://docs.aws.amazon.com/wellarchitected/latest/framework/security.html" rel="noopener" target="_blank">security pillar</a> of <a href="https://aws.amazon.com/architecture/well-architected/?wa-lens-whitepapers.sort-by=item.additionalFields.sortDate&amp;wa-lens-whitepapers.sort-order=desc&amp;wa-guidance-whitepapers.sort-by=item.additionalFields.sortDate&amp;wa-guidance-whitepapers.sort-order=desc" rel="noopener" target="_blank">AWS Well-Architected</a>.</p> 
<p>In this blog post, I dive deep into an open source tool for testing GuardDuty findings and then walk through three examples of how you can use this tool to test and improve your response to GuardDuty findings.</p> 
<h2>Overview</h2> 
<p>If you want to learn more about GuardDuty, you can read about the finding types in this<a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html" rel="noopener" target="_blank"> AWS documentation</a>. However, customers often want realistic findings in their environment to understand what a finding looks like and to practice responding hands on. While you can use GuardDuty to create <a href="https://docs.aws.amazon.com/guardduty/latest/ug/sample_findings.html" rel="noopener" target="_blank">sample findings</a> in your environment, these findings are approximations populated with placeholder values and look different from real findings. Additionally, you cannot practice remediation with these findings because they’re not tied to actual resources in your account. This can be helpful if you only want to see what details are in a finding, but if you want to practice a real-world scenario, these sample findings might not be adequate.</p> 
<p>To address this use case and provide customers with a secure and reliable way to test the threat detection capabilities of GuardDuty, the GuardDuty service team launched an open source project called <a href="https://github.com/awslabs/amazon-guardduty-tester" rel="noopener" target="_blank">GuardDuty Tester</a>. The GuardDuty Tester creates infrastructure in your environment to simulate different security issues so that you can test GuardDuty findings that mirror actual security issues that you might encounter, such as crypto mining or a reverse shell being created on an <a href="https://aws.amazon.com/ec2/" rel="noopener" target="_blank">Amazon Elastic Compute Cloud (Amazon EC2)</a> instance. The GuardDuty Tester was originally released in 2018 as an <a href="https://aws.amazon.com/cloudformation" rel="noopener" target="_blank">AWS CloudFormation</a> template and was focused more on testing investigation workflows than on a wide range of finding types. AWS has since released an updated version that uses the <a href="https://aws.amazon.com/cdk/" rel="noopener" target="_blank">AWS Cloud Development Kit (AWS CDK)</a> to make the infrastructure code easier to read and expanded the test coverage to over 100 unique finding types and resource combinations.</p> 
<p>The ability to create findings across different resource types such as Amazon EC2, <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a>, and <a href="https://aws.amazon.com/eks/" rel="noopener" target="_blank">Amazon Elastic Kubernetes Service (Amazon EKS)</a> is a valuable resource for your security team, allowing them to simulate various types of threats with isolated infrastructure so that you don’t need to compromise your deployed workloads to improve response actions and techniques. Remember that the GuardDuty Tester doesn’t cover every possible scenario, but is instead focused on threat intelligence and rules-based findings. Anomaly-based findings, which require learning about how you operate your environment, aren’t included in the GuardDuty Tester.</p> 
<h2>Getting started with the GuardDuty Tester</h2> 
<p>The GuardDuty Tester is deployed by using the <a href="https://aws.amazon.com/cdk" rel="noopener" target="_blank">AWS CDK</a> to create the required infrastructure and scripts to generate the GuardDuty findings. For safety, AWS recommends that you deploy the GuardDuty Tester in a nonproduction environment in an account that’s used specifically for this purpose. This way, your security team can differentiate between test GuardDuty findings and findings for other workloads that they’re monitoring.</p> 
<p>In this post, I won’t walk through configuring the GuardDuty Tester because this is already documented in the <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_findings-scripts.html" rel="noopener" target="_blank">GuardDuty documentation</a>. Instead, I will go over what you need to know about the GuardDuty Tester and some of the benefits.</p> 
<p>Figure 1 shows the GuardDuty Tester architecture, which includes the resources necessary to create GuardDuty findings for various protection plans such as Amazon S3 buckets, Amazon EC2 instances, and an Amazon EKS cluster. The tester also deploys a dedicated GuardDuty Tester instance where you will run the scripts needed to create the GuardDuty findings.</p> 
<div class="wp-caption aligncenter" id="attachment_37204" style="width: 790px;">
 <img alt="Figure 1: GuardDuty Tester architecture" class="size-full wp-image-37204" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/23/img1.jpg" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-37204">Figure 1: GuardDuty Tester architecture</p>
</div> 
<p>The GuardDuty Tester provides key features including:</p> 
<ul> 
 <li><strong>A wide range of threat scenario simulations</strong>: Resources that the GuardDuty Tester can create findings for include Amazon S3, <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a>, <a href="https://aws.amazon.com/ecs/" rel="noopener" target="_blank">Amazon Elastic Container Service (Amazon ECS)</a> for both Amazon EC2 and <a href="https://aws.amazon.com/fargate" rel="noopener" target="_blank">AWS Fargate</a> hosted workloads, Amazon EKS, and <a href="https://aws.amazon.com/lambda/" rel="noopener" target="_blank">AWS Lambda</a> and covers over 105 threat scenarios. This includes GuardDuty runtime monitoring as well as other GuardDuty protection plans.</li> 
 <li><strong>Access through AWS Systems Manager</strong>: The GuardDuty Tester provides secure access by using <a href="https://aws.amazon.com/systems-manager" rel="noopener" target="_blank">Systems Manager</a> to minimize open ports to the internet and allowing access only through Systems Manager.</li> 
 <li><strong>Modular scripts</strong>: With an expanded library of tests available, the GuardDuty Tester accepts user parameters to set the scope of the tests to run, which gives you greater flexibility for different testing scenarios.</li> 
</ul> 
<p>Setting up the GuardDuty Tester environment is straightforward and requires only a few commands. As outlined in the&nbsp;<a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_findings-scripts.html#prerequisites-gdu-tester-script" rel="noopener" target="_blank">documentation</a> and the <a href="https://github.com/awslabs/amazon-guardduty-tester/blob/master/README.md" rel="noopener" target="_blank">README</a> file in the repository, there are a number of prerequisites to set up the stack. These prerequisites include Python 3+, git, the <a href="https://aws.amazon.com/cli" rel="noopener" target="_blank">AWS Command Line Interface (AWS CLI)</a>, <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html" rel="noopener" target="_blank">AWS Systems Manager Session Manager plugin</a>, <a href="https://www.npmjs.com/" rel="noopener" target="_blank">npm</a>, <a href="https://www.docker.com/" rel="noopener" target="_blank">Docker</a>, and a subscription to <a href="https://aws.amazon.com/marketplace/pp/prodview-fznsw3f7mq7to" rel="noopener" target="_blank">Kali Linux image for Amazon EC2</a>. You will have to subscribe to the Kali Linux instance in <a href="https://aws.amazon.com/marketplace" rel="noopener" target="_blank">AWS Marketplace</a><u>,</u> but will be charged for the instance only while the GuardDuty Tester is deployed. After these prerequisites are met, you can clone the repository, install the packages, and deploy the GuardDuty Tester to your AWS account.</p> 
<p>Deploying the GuardDuty Tester can take 20–30 minutes, but if you’re following along with this post, I assume that you have deployed the GuardDuty Tester into your environment and have started your Systems Manager session as stated on Part A of <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_findings-scripts.html#run-gdu-tester-script" rel="noopener" target="_blank">Step 3 – Run tester scripts</a> in the GuardDuty documentation. Now, I will dive into the first testing example.</p> 
<h2>Manual investigation</h2> 
<p>The first test use case is about getting familiar with what GuardDuty findings look like and the details that a finding gives you. This might be one of your first steps after turning on GuardDuty, or this might be an activity that you perform to help new team members understand GuardDuty findings.</p> 
<h4>To start a manual investigation:</h4> 
<ol> 
 <li>Run the following command in your Systems Manager session to view the GuardDuty Tester options. <pre><code class="lang-powershell">Python3 guardduty_tester.py --help</code></pre> </li> 
 <li>Run the following command in your Systems Manager session to create the first test finding. <pre><code class="lang-powershell">Python3 guardduty_tester.py - -ec2 - -runtime only - -tactics impact</code></pre> </li> 
 <li>Before creating the findings, the GuardDuty Tester prompts you to confirm that it’s allowed to change GuardDuty settings in the environment. For example, if you’ve chosen to create findings related to the <a href="https://docs.aws.amazon.com/guardduty/latest/ug/runtime-monitoring.html" rel="noopener" target="_blank">GuardDuty runtime monitoring</a> feature but don’t have this feature enabled, the GuardDuty Tester will enable it for the tests and then disable it after testing is complete.<br /> 
  <blockquote>
   <p><strong>Note</strong>: This will start the 30-day trial of the enabled features in this account, in this AWS Region, even if the feature is disabled after testing is complete. More information about GuardDuty pricing and free trials can be found on the <a href="https://aws.amazon.com/guardduty/pricing/" rel="noopener" target="_blank">GuardDuty pricing page</a>.</p>
  </blockquote> </li> 
 <li>After choosing y which indicates “yes”, the GuardDuty Tester reports the number of domain reputation findings it’s expecting. Figure 2 shows an example of the expected findings. You can learn more about domain reputation findings in the <a href="https://docs.aws.amazon.com/guardduty/latest/ug/findings-runtime-monitoring.html#impact-runtime-abuseddomainrequestreputation" rel="noopener" target="_blank">GuardDuty finding documentation</a>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37205" style="width: 750px;">
   <img alt="Figure 2: Generated GuardDuty findings in the console" class="size-full wp-image-37205" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/23/img2-2.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-37205">Figure 2: Generated GuardDuty findings in the console</p>
  </div><p></p> </li> 
 <li>After the GuardDuty Tester is finished, wait a few minutes and then go to the <a href="https://console.aws.amazon.com/guardduty" rel="noopener" target="_blank">AWS Management Console for GuardDuty</a> to see the findings. In this example, there are four new GuardDuty findings as expected from step 4 and shown in Figure 3. With the findings generated, you can start your manual investigation. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37206" style="width: 750px;">
   <img alt="Figure 3: GuardDuty finding details" class="size-full wp-image-37206" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/23/img3-2.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-37206">Figure 3: GuardDuty finding details</p>
  </div><p></p> </li> 
</ol> 
<p>In the preceding figure, you can see some of the finding details presented—such as the action type and the process information—that can help you quickly identify what trigger started the suspicious communication. From here, I encourage you to use this finding to practice your runbooks for investigation and response. For example, you might start with validating and triaging the finding before moving into evidence collection and remediation. If you don’t have incident response runbooks already built, you can use this finding as an example to get started. There are multiple open source examples such as <a href="https://github.com/aws-samples/aws-incident-response-playbooks" rel="noopener" target="_blank">AWS incident response playbooks</a> and <a href="https://github.com/aws-samples/aws-customer-playbook-framework?tab=readme-ov-file" rel="noopener" target="_blank">AWS customer response playbook</a>. A runbook will help your team evaluate the information provided in the GuardDuty finding and understand what else they need to know about your specific environment to properly respond to the finding. For example, in the finding, you will have resource and actor information but not things such as who is the account owner or point of contact for security for that account.</p> 
<h2>Creating alerts</h2> 
<p>The next use case highlights how to create alerts based on GuardDuty findings. When setting up alerting automation with tools such as <a href="https://aws.amazon.com/sns/" rel="noopener" target="_blank">Amazon Simple Notification Service (Amazon SNS)</a> and Slack, you should create a finding using the GuardDuty Tester to test that you’ve configured your alert correctly. See <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_findings_cloudwatch.html" rel="noopener" target="_blank">Creating custom responses to GuardDuty findings</a> for information about creating alerts with either of these tools. Figure 4 shows a sample EventBridge rule that will send GuardDuty findings to SNS.</p> 
<div class="wp-caption aligncenter" id="attachment_37207" style="width: 790px;">
 <img alt="Figure 4: EventBridge rule to send GuardDuty findings to SNS" class="size-full wp-image-37207" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/23/img4-2.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-37207">Figure 4: EventBridge rule to send GuardDuty findings to SNS</p>
</div> 
<p>For this post, I assume that you’ve already configured an <a href="https://aws.amazon.com/eventbridge/" rel="noopener" target="_blank">Amazon EventBridge</a> rule and Amazon SNS alert.</p> 
<h4>To test alerts:</h4> 
<ol> 
 <li>Run the following command in your Systems Manager session to create a privileged container finding. <pre><code class="lang-powershell">Python3 guardduty_tester.py - -finding ‘PrivilegedEscalation:Kubernetes/PrivilegedContainer’</code></pre> </li> 
 <li>Shortly after creating this finding, you should see an SNS alert based on the finding type.</li> 
</ol> 
<p style="line-height: 1.25em;"></p>
<div class="wp-caption aligncenter" id="attachment_37208" style="width: 790px;">
 <img alt="Figure 5: SNS notification from a GuardDuty finding" class="size-full wp-image-37208" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/23/img5-1.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-37208">Figure 5: SNS notification from a GuardDuty finding</p>
</div>
<p></p> 
<p>If you’ve configured the alert correctly, you will see an email similar to Figure 5. The email demonstrates that SNS notifications were successfully configured and tested using the GuardDuty Tester. If this is a new finding, you will receive this SNS notification shortly after the GuardDuty Tester generates the finding, but if this is an updated finding, then the timing will be based on the <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_findings_cloudwatch.html#guardduty_findings_cloudwatch_notification_frequency" rel="noopener" target="_blank">notification frequency configured in the account.</a></p> 
<p>There are many ways that customers consume GuardDuty findings in their environments. Whether you’re using Amazon SNS or another mechanism such as a chat application, ticketing system, or a security information and event management (SIEM) solution, you can use this example of an EventBridge rule and the GuardDuty Tester to test out your notification pipeline.</p> 
<h2>Automated response</h2> 
<p>For the third use case, I show you how to create an automated action based on a GuardDuty finding. In this example, I create a finding based on an EC2 instance connecting to a Bitcoin mining domain, then based on this finding, I use Lambda to tag the instance to assist with identification during that investigation steps that follow. Although this is a simple example, it shows you what you can do by combining EventBridge rules and Lambda functions. If you want to create an automated response for GuardDuty runtime monitoring findings that requires making a host-level modification, you can use EventBridge rules with <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/run-command.html" rel="noopener" target="_blank">AWS Systems Manager Run Command</a> to run commands locally on a host to remediate a security issue.</p> 
<p>Start by creating a Lambda function that will take a GuardDuty event delivered by EventBridge, pull out the instance ID information, and then use that as a parameter in the <code style="color: #000000;">create_tags</code> API call. See the following example code.</p> 
<pre><code class="lang-python">import json
import boto3
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    try:
        # Extract the necessary information from the GuardDuty finding
        instance_id = event['detail']['resource']['instanceDetails']['instanceId']
        account_id = event['detail']['accountId']
        region = event['detail']['region']

        # Create an EC2 client
        ec2 = boto3.client('ec2', region_name=region)

        # Add the "infected" and "cryptomining" tag value pair to the instance
        ec2.create_tags(
            Resources=[instance_id],
            Tags=[
                {
                    'Key': 'infected',
                    'Value': 'cryptomining'
                }
            ]
        )

        logger.info(f"Tagged instance {instance_id} with 'infected=cryptomining' in account {account_id} and region {region}")
        return {
            'statusCode': 200,
            'body': 'Instance tagged successfully'
        }
    except Exception as e:
        logger.error(f"Error tagging instance {instance_id}: {str(e)}")
        return {
            'statusCode': 500,
            'body': f"Error tagging instance: {str(e)}"
        }</code></pre> 
<p>Next, I create an EventBridge rule specific to the Bitcoin mining finding that I want to test, shown in Figure 6. The target is the Lambda function that I just created.</p> 
<div class="wp-caption aligncenter" id="attachment_37209" style="width: 790px;">
 <img alt="Figure 6: EventBridge rule for crypto mining GuardDuty finding" class="size-full wp-image-37209" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/23/img6-2.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-37209">Figure 6: EventBridge rule for crypto mining GuardDuty finding</p>
</div> 
<p>Now that the EventBridge rule is in place with the Lambda function as the target, I can use the GuardDuty Tester to trigger a Bitcoin mining finding and test my solution with the following command.</p> 
<pre><code class="lang-powershell">Python3 guardduty_tester.py - - finding ‘CryptoCurrency:EC2/BitcoinTool.B!DNS’</code></pre> 
<p>After the finding is generated, I go to my EC2 instance, where there’s a new instance tag with a key of <code style="color: #000000;">infected</code> and a value of <code style="color: #000000;">cryptomining</code>, shown in Figure 7.</p> 
<div class="wp-caption aligncenter" id="attachment_37210" style="width: 790px;">
 <img alt="Figure 7: Updated tags after automated response" class="size-full wp-image-37210" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/23/img7-1.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-37210">Figure 7: Updated tags after automated response</p>
</div> 
<p>Although this is a general example, you can use the same approach across various actions that you might take in response to a GuardDuty finding and then test them using the GuardDuty Tester. Examples include using Lambda to add logic in <a href="https://aws.amazon.com/waf/" rel="noopener" target="_blank">AWS WAF</a>, a network access control list (network ACL), or <a href="https://aws.amazon.com/network-firewall/" rel="noopener" target="_blank">AWS Network Firewall</a> to block suspicious traffic, or use Systems Manager Run Command to end a malicious process that’s running on a host.</p> 
<h2>Conclusion</h2> 
<p>The updated GuardDuty Tester represents a significant advancement in helping organizations validate and gain confidence in GuardDuty threat detection. The GuardDuty Tester now provides more comprehensive coverage of GuardDuty runtime monitoring and protection plans across various AWS services.</p> 
<p>By using the GuardDuty Tester and following the use cases in this post, you can proactively assess your threat detection readiness, identify potential gaps, and implement necessary measures to help you fortify your AWS environments against evolving cyber threats.</p> 
<p>&nbsp;<br />If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.<br />&nbsp;</p> 
<footer> 
 <div class="blog-author-box">
  <img alt="Marshall Jones" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2021/12/15/Marshall-Jones-Author.jpeg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Marshall Jones</span>
  <br />Marshall is a Worldwide Security Specialist Solutions Architect at AWS. His background is in AWS consulting and security architecture and focused on a variety of security domains including edge, threat detection, and compliance. Today, he’s focused on helping enterprise AWS customers adopt and operationalize AWS security services to increase security effectiveness and reduce risk.
 </div> 
</footer>
