---
title: "Mapping AWS security services to MITRE frameworks for threat detection and mitigation"
url: "https://aws.amazon.com/blogs/security/mapping-aws-security-services-to-mitre-frameworks-for-threat-detection-and-mitigation/"
date: "Tue, 13 May 2025 15:49:20 +0000"
author: "Pratima Singh"
feed_url: "https://aws.amazon.com/blogs/security/tag/amazon-guardduty/feed/"
---
<p>In the cloud security landscape, organizations benefit from aligning their controls and practices with industry standard frameworks such as MITRE ATT&amp;CK<sup>®</sup>, MITRE Engage<sup>TM</sup>, and MITRE D3FEND<sup>TM</sup>. MITRE frameworks are structured, openly accessible models that document threat actor behaviors to help organizations improve threat detection and response.</p> 
<div class="wp-caption aligncenter" id="attachment_38232" style="width: 1333px;">
 <img alt="Figure 1: Interaction between the various MITRE frameworks" class="size-full wp-image-38232" height="965" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/08/image1-1.png" width="1323" />
 <p class="wp-caption-text" id="caption-attachment-38232">Figure 1: Interaction between the various MITRE frameworks</p>
</div> 
<p>Figure 1 showcases how the frameworks interact with each other to identify threatening behavior and provide actionable defensive measures. MITRE ATT&amp;CK provides insights into threat actor behavior while D3FEND translates insights from ATT&amp;CK into actionable defensive measures. MITRE Engage uses both ATT&amp;CK and D3FEND to plan proactive engagement strategies that disrupt threat actor activity. As organizations use AWS to enhance their operational capabilities, implementing comprehensive security strategies becomes an important part of cloud adoption.</p> 
<p>This blog post explores how AWS security services align with the MITRE frameworks to provide a systematic approach for threat detection and mitigation. We’ll examine how organizations can use AWS security tools such as <a href="https://aws.amazon.com/guardduty" rel="noopener" target="_blank">Amazon GuardDuty</a>, <a href="https://aws.amazon.com/security-lake" rel="noopener" target="_blank">Amazon Security Lake</a>, and <a href="https://aws.amazon.com/security-hub" rel="noopener" target="_blank">AWS Security Hub</a> in conjunction with MITRE frameworks to implement security controls across different stages of their cloud security operations.</p> 
<h2 id="understanding-mitre-frameworks">Understanding MITRE frameworks</h2> 
<p>Today’s security teams face increasingly sophisticated threats, with actors continuously evolving their tactics, techniques, and procedures (TTPs). To help organizations strengthen their security posture, industry frameworks such as MITRE ATT&amp;CK, D3FEND, and Engage provide structured methodologies for understanding and responding to these threats.</p> 
<p>Understanding these threats through a risk lifecycle approach is crucial for security teams. This structured methodology enables teams to detect anomalies early, map threats to known risk stages, and implement proactive defense mechanisms. By following a risk lifecycle approach, organizations can enhance threat intelligence, improve incident response, and minimize dwell time, ultimately strengthening their security posture against evolving cyber threats.</p> 
<p>The integration of MITRE ATT&amp;CK, D3FEND, and Engage frameworks offers organizations a comprehensive approach across the <a href="https://docs.aws.amazon.com/security-ir/latest/userguide/what-is.html" rel="noopener" target="_blank">security operations lifecycle</a>. At the foundation, MITRE ATT&amp;CK provides a common language for describing threat actor TTPs. This knowledge base is invaluable during threat modeling and risk assessment, helping teams identify potential vulnerabilities and threat vectors.</p> 
<p>Building upon ATT&amp;CK, MITRE D3FEND complements the tactical knowledge with a framework for defensive countermeasures. It suggests proactive security controls, such as implementing least privilege access or securing system configurations. This allows organizations to align their defenses directly with known exploit patterns.</p> 
<p>MITRE Engage then adds a layer of active defense capabilities. It guides security teams in planning and implementing strategies that can help in three different ways and potentially simultaneously. Defenders can expose threat actors by detecting them as they attempt to access or operate on infrastructure. Defenders can use Engage to help impose costs by causing threat actors to focus on fake infrastructure rather than legitimate assets. Finally, defenders can set up enticing fake targets to lure threat actors into exploiting them and thereby revealing tradecraft.</p> 
<p>A MITRE operation that was run in conjunction with a partner might clarify how this is valuable. MITRE worked with a partner to set up a fake network to appear as a specific type of entity. The goal was to elicit TTPs from a specific advanced persistent threat (APT) for which MITRE and the partner had a recent malware sample. MITRE ran the sample on the fake network and observed the APT’s activities. From that operation, MITRE gathered a list of specific TTPs that were executed by a script in a particular order that helped the partner develop a novel analytic. Plus, in reviewing event traces, MITRE found a flaw in a well-known security tool that missed a specific type of process-tampering event. This was disclosed to the vendor, who fixed that in later versions. Finally, every minute of operating in this environment imposed a cost on the APT by diverting resources from real victims. Full details of the exercise were <a href="https://archive.org/details/Shmoocon-2022/Shmoocon2022-Karen_Lamb%2C_Gabby_Raymond%2C_%26_Maretta_Morovitz-She_doesn%E2%80%99t_even_go_here.mp4" rel="noopener" target="_blank">presented at Shmoocon 2022</a>.</p> 
<p>As we move through the security operations lifecycle, these three MITRE frameworks continue to work in concert:</p> 
<ul> 
 <li>During detection and monitoring, ATT&amp;CK informs threat hunting and log analysis and correlation, D3FEND strengthens real-time detection and anomaly tracking, and Engage enables strategic detection through deception techniques.</li> 
 <li>When responding to incidents, ATT&amp;CK helps map incident progression, D3FEND automates response actions, and Engage provides methods to gather additional intelligence about threat activities.</li> 
 <li>In the post-incident phase, ATT&amp;CK helps map the incident chain for better detection tuning, D3FEND refines security controls, and Engage expands deception tactics based on lessons learned. By integrating these efforts, organizations can implement a systematic approach to security operations that combines tactical knowledge, defensive measures, and strategic engagement capabilities.</li> 
</ul> 
<h2 id="aligning-aws-to-mitre-frameworks">Aligning AWS to MITRE frameworks</h2> 
<p>AWS offers a broad set of cloud services with high security at global scale, and has proven experience helping businesses innovate faster. Customers use AWS services in various configurations to build solutions for their bespoke business needs. A fundamental aspect of using AWS is understanding the <a href="https://aws.amazon.com/compliance/shared-responsibility-model/" rel="noopener" target="_blank">Shared Responsibility Model</a>, shown in Figure 2 that follows.</p> 
<p><em></em></p>
<div class="wp-caption aligncenter" id="attachment_38233" style="width: 2006px;">
 <em><img alt="Figure 2: AWS Shared Responsibility Model" class="size-full wp-image-38233" height="1086" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/08/image2.png" style="border: 1px solid #bebebe;" width="1996" /><p class="wp-caption-text" id="caption-attachment-38233">Figure 2: AWS Shared Responsibility Model</p></em>
</div>
<p></p> 
<p>AWS is responsible for security <em>of</em> the cloud, while customers are responsible for security <em>in</em> the cloud. This means that AWS is responsible for protecting the infrastructure that runs the services offered in the AWS Cloud, while customer responsibility is determined by the AWS Cloud services that a customer selects. As customers embark on their cloud security journey, we help them understand two important concepts of cloud-scale environments:</p> 
<ul> 
 <li><strong>Interconnected resources and configurations:</strong> Cloud architectures consist of interconnected entities—ranging from virtual machines using <a href="https://aws.amazon.com/ec2" rel="noopener" target="_blank">Amazon Elastic Compute Cloud (Amazon EC2)</a> to serverless functions using <a href="https://aws.amazon.com/lambda" rel="noopener" target="_blank">AWS Lambda</a>. To help customers maintain visibility and control, AWS offers native tools designed for cloud-scale management.</li> 
 <li><strong>Dynamic access management and least privilege:</strong> Cloud environments require robust authentication mechanisms and fine-grained permissions. AWS provides comprehensive identity and access management tools to implement least privilege access and manage dynamic workloads effectively.</li> 
</ul> 
<p>To support our customers’ security needs, AWS offers native security services that align with industry-standard frameworks like MITRE ATT&amp;CK, D3FEND, and Engage. Here’s how these services map across the security lifecycle:</p> 
<p>For threat modeling and risk assessment, Security Lake aggregates logs for MITRE ATT&amp;CK-based analytics, while <a href="https://aws.amazon.com/inspector" rel="noopener" target="_blank">Amazon Inspector</a> scans for vulnerabilities mapped to threat actor techniques. <a href="https://aws.amazon.com/macie" rel="noopener" target="_blank">Amazon Macie</a> detects sensitive data exposure across AWS resources.</p> 
<p>When implementing preventive controls, implementing least privilege for access is fundamental. <a href="https://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> and <a href="https://aws.amazon.com/organizations" rel="noopener" target="_blank">AWS Organizations</a> provide capabilities to enforce least privilege across your AWS environment. You can use IAM permissions and service control policies (SCPs) to build an identity perimeter. <a href="https://aws.amazon.com/waf" rel="noopener" target="_blank">AWS Web Application Firewall (AWS WAF)</a> provides application-layer protections, while you can use <a href="https://aws.amazon.com/secrets-manager" rel="noopener" target="_blank">AWS Secrets Manager</a> to store <em>honey tokens</em>. Secrets Manager is an AWS service that you can use to centrally manage the lifecycle of secrets. Honey tokens act as digital decoys that simulate legitimate credentials or sensitive data, enticing threat actors to reveal their presence when they interact with them. When triggered, these tokens generate real-time alerts and detailed event logs, enabling swift investigation and deeper insights into threat actor tactics. Deploying honey tokens on AWS involves creating decoy credentials or sensitive data entries that serve no legitimate purpose yet are closely monitored for unauthorized access attempts. One common approach is to use Secrets Manager to store fake secrets that mimic real credentials. When such tokens, stored in Secrets Manager, are accessed, the service generates detailed event logs with <a href="https://aws.amazon.com/cloudtrail" rel="noopener" target="_blank">AWS CloudTrail</a> and <a href="https://aws.amazon.com/cloudwatch" rel="noopener" target="_blank">Amazon CloudWatch</a>. You can continuously monitor these logs and events and configure them to alert you if the decoys are ever accessed.</p> 
<p>During the detection and monitoring phase, GuardDuty identifies unusual activity patterns across your AWS accounts and workloads, <a href="https://aws.amazon.com/detective" rel="noopener" target="_blank">Amazon Detective</a> helps investigate these anomalies by analyzing root causes and plotting out the incident scope in an interactive way, while Security Hub centralizes security alerts and enables automated responses across your environment.</p> 
<p>For incident response, containment, and recovery, Lambda and Step Functions help automate responses when security events occur. AWS Shield and WAF work together to provide real-time threat mitigation against denial-of-service type threats like distributed denial of service (DDoS), while Security Lake and Detective provide the necessary data and tools for conducting thorough forensic analysis. In 2024, AWS announced the AWS Security Incident Response service that uses automated monitoring and investigation through the AWS Customer Incident Response Team to prepare for, respond to, and recover from security events. You can use the service to augment your cloud-based security response function aligned with <a href="https://docs.aws.amazon.com/wellarchitected/latest/framework/sec-10.html" rel="noopener" target="_blank">AWS security best practices</a>.</p> 
<p>By blocking malicious traffic, Shield and WAF provide real-time DDoS mitigation. AWS deception tactics could include redirecting threat actors to honeypots or deploying decoy <a href="https://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> files to enhance engagement strategies, like the honey token deployment and storage using Secrets Manager explained earlier in this post. Post incident, Security Lake and Detective assist in forensic analysis, while Security Hub and IAM policies refine security controls based on past exploit trends. MITRE Engage tactics can further evolve by analyzing honeypot interactions. By integrating these AWS security services, you can detect, prevent, and deceive threat actors effectively, strengthening your organization’s overall security posture. The following table maps MITRE lifecycle stages to AWS services and tools.</p> 
<table border="1" width="0"> 
 <colgroup> 
  <col style="width: 22%;" /> 
  <col style="width: 23%;" /> 
  <col style="width: 26%;" /> 
  <col style="width: 27%;" /> 
 </colgroup> 
 <tbody> 
  <tr> 
   <td><strong>Lifecycle stage</strong></td> 
   <td><strong>AWS tools for MITRE ATT&amp;CK (detect and map)</strong></td> 
   <td><strong>AWS tools for MITRE D3FEND (prevent and contain)</strong></td> 
   <td><strong>AWS tools for MITRE Engage (deceive and disrupt)</strong></td> 
  </tr> 
  <tr> 
   <td><strong>Threat modeling and risk assessment</strong></td> 
   <td>Security Lake, Amazon Inspector, Macie, and Security Hub</td> 
   <td>IAM policies and AWS WAF</td> 
   <td>Secrets Manager and honey tokens</td> 
  </tr> 
  <tr> 
   <td><strong>Detection and monitoring</strong></td> 
   <td>GuardDuty, CloudTrail, and Security Hub</td> 
   <td>Detective, auto-remediation using AWS services such as <a href="https://aws.amazon.com/eventbridge/" rel="noopener" target="_blank">Amazon EventBridge</a>, Lambda, and Step Functions.</td> 
   <td>Fake IAM users, and decoy Amazon S3 files</td> 
  </tr> 
  <tr> 
   <td><strong>Incident response and containment</strong></td> 
   <td>Step Functions, Lambda, GuardDuty, <a href="https://aws.amazon.com/security-incident-response/" rel="noopener" target="_blank">AWS Security Incident Response</a>, and Detective</td> 
   <td>Auto-block using AWS WAF, multi-factor authentication (MFA) enforcement, and AWS Security Incident Response</td> 
   <td>Redirect exploits to honeypots</td> 
  </tr> 
  <tr> 
   <td><strong>Post-incident and intelligence</strong></td> 
   <td>Analyze and correlate logs with Security Lake, <a href="https://aws.amazon.com/athena" rel="noopener" target="_blank">Amazon Athena</a>, and Detective</td> 
   <td>IAM hardening and <a href="https://aws.amazon.com/config" rel="noopener" target="_blank">AWS Config</a></td> 
   <td>Adaptive deception traps</td> 
  </tr> 
 </tbody> 
</table> 
<p>You can use Table 1 as a guide to understand how AWS services map to the various lifecycle stages in the incident response lifecycle. We will now demonstrate how GuardDuty, an AWS security service that continuously monitors your AWS accounts and workloads to provide automated threat detection, works in line with the MITRE ATT&amp;CK framework.</p> 
<h2 id="guardduty-mitre-framework-integration-in-action">GuardDuty: MITRE framework integration in action</h2> 
<p>In 2024, AWS worked extensively with MITRE to create new techniques and sub-techniques, and to update some of the existing detection objects in the <a href="https://attack.mitre.org/matrices/enterprise/cloud/" rel="noopener" target="_blank"><u>MITRE ATT&amp;CK cloud matrix</u></a>. The work that AWS did with MITRE drew from real-world threat actor techniques performed against AWS customers and helped to provide more detailed information and specific detections on how threat actors abuse AWS services. For example, AWS threat detection teams observed a new tactic in the cloud environment (<a href="https://attack.mitre.org/techniques/T1485/001/" rel="noopener" target="_blank"><u>T1485.001 | Data Destruction: Lifecycle-Triggered Deletion</u></a>) where threat actors could modify lifecycle policies for S3 buckets to delete all objects stored in the bucket. This technique, along with associated mitigations, detection, and references was submitted back to the MITRE ATT&amp;CK framework.</p> 
<p>AWS security services such as <a href="https://aws.amazon.com/security-incident-response/" rel="noopener" target="_blank"><u>AWS Security Incident Response</u></a> and GuardDuty use MITRE ATT&amp;CK to provide threat intelligence and detailed information on threats identified in an AWS account. You can examine how these AWS security services integrate with MITRE ATT&amp;CK through a specific example. <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty-extended-threat-detection.html" rel="noopener" target="_blank">GuardDuty Extended Threat Detection</a> helps customers with contextual threat detection in their AWS environment and aligns the signals with the MITRE ATT&amp;CK lifecycle. GuardDuty automatically detects and correlates individual findings with connected resources to produce an attack sequence finding. Consider an attack sequence finding generated by GuardDuty detecting data compromise in your AWS account. We will use this as an example in this post.</p> 
<p>To begin, the finding summary includes a textual description of the sequence of events and the TTPs detected, as shown in Figure 3. It also shows a summary of the observed TTP identifiers, AWS API calls, and IP addresses.</p> 
<div class="wp-caption aligncenter" id="attachment_38234" style="width: 656px;">
 <img alt="Figure 3: GuardDuty finding summary visible in the service console" class="size-full wp-image-38234" height="673" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/08/image3.png" style="border: 1px solid #bebebe;" width="646" />
 <p class="wp-caption-text" id="caption-attachment-38234">Figure 3: GuardDuty finding summary visible in the service console</p>
</div> 
<p>As seen in Figure 4, every attack sequence finding highlights the signals and the MITRE tactic associated with the activity. The finding shown in Figure 4 shows the full lifecycle of the threat from discovery to impact.</p> 
<div class="wp-caption aligncenter" id="attachment_38235" style="width: 1700px;">
 <img alt="Figure 4: Signals and MITRE tactics alignment" class="size-full wp-image-38235" height="1094" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/08/image4.png" style="border: 1px solid #bebebe;" width="1690" />
 <p class="wp-caption-text" id="caption-attachment-38235">Figure 4: Signals and MITRE tactics alignment</p>
</div> 
<p>Diving deeper into each signal reveals the specific MITRE tactic associated with the activity and the technique identifier. Another interesting feature is that you can see the correlation between the AWS API call associated with the resources involved in the attack sequence and the user agent.</p> 
<p>Figure 5 shows one of the signals associated with the attack sequence in the previous finding. A data exfiltration activity has been reported because of the nature of the AWS API call (<code style="color: #000000;">s3:GetObject</code>) and the user agent (<code style="color: #000000;">Kali Linux</code>) that was used to perform the activity. The level of detail for each signal is contextual based on the type of activity and tactic.</p> 
<div class="wp-caption aligncenter" id="attachment_38236" style="width: 560px;">
 <img alt="Figure 5: Details for a single signal within a GuardDuty attack sequence finding" class="size-full wp-image-38236" height="624" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/08/image5-1.png" style="border: 1px solid #bebebe;" width="550" />
 <p class="wp-caption-text" id="caption-attachment-38236">Figure 5: Details for a single signal within a GuardDuty attack sequence finding</p>
</div> 
<p>Figure 6 shows another signal from the same finding, but in this case the level of detail includes the malicious IP lists and suspicious network activity detected in relation to the signal and associated resources.</p> 
<div class="wp-caption aligncenter" id="attachment_38237" style="width: 610px;">
 <img alt="Figure 6: Details of TTPs associated with an indicator within a GuardDuty attack sequence finding" class="size-full wp-image-38237" height="817" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/05/08/image6.png" style="border: 1px solid #bebebe;" width="600" />
 <p class="wp-caption-text" id="caption-attachment-38237">Figure 6: Details of TTPs associated with an indicator within a GuardDuty attack sequence finding</p>
</div> 
<p>This information can be downloaded in a JSON-formatted file. The information from the JSON document can be used to automate responses and remediations for the detections.</p> 
<h2 id="conclusion">Conclusion</h2> 
<p>AWS security services work together to support the implementation of MITRE frameworks—ATT&amp;CK for threat detection, D3FEND for preventative security, and Engage for threat actor engagement across the cybersecurity lifecycle. As demonstrated through the GuardDuty Extended Threat Detection example, these integrations provide customers with practical, actionable security capabilities across their AWS environment. The alignment of AWS security services with MITRE frameworks helps you build security operations using industry-standard methodologies, implement automated detection and response capabilities, maintain visibility across your AWS environment, and continuously enhance your security controls.</p> 
<p>Through this integration of AWS security services with MITRE frameworks, you can implement comprehensive security operations that evolve with your organization’s business needs. To get started, visit the GuardDuty console to enable Extended Threat Detection, and explore our <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty-extended-threat-detection.html" rel="noopener" target="_blank">documentation</a> to learn more about implementing these security capabilities in your AWS environment. Join us at <a href="https://reinforce.awsevents.com/" rel="noopener" target="_blank">AWS re:Inforce 2025</a> to learn more about AWS security services, including deep dives into the integration of Amazon GuardDuty with MITRE frameworks and hands-on workshops with AWS security experts.</p> 
<p>If you have feedback about this post, submit comments in the <strong>Comments</strong> section below. </p> 
<footer> 
 <div class="blog-author-box">
  <img alt="Pratima Singh" class="alignleft size-full" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/04/03/pxs_headshot.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" width="120" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Pratima Singh</span>
  <br />Pratima is a Security Specialist Solutions Architect with AWS, based out of Sydney, Australia. She is a security enthusiast who enjoys helping customers find innovative solutions to complex business challenges. Outside of work, Pratima enjoys going on long drives and spending time with her family at the beach.
 </div> 
 <div class="blog-author-box"> 
  <p><span class="lb-h4">Contributors</span></p> 
  <p>Special thanks to Dr. Stanley Barr, Senior Principal Scientist at MITRE, and Jess Modini, former Advisory Solutions Architect at AWS, who made significant contributions to this post.</p> 
 </div> 
</footer>
