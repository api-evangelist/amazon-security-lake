---
title: "File integrity monitoring with AWS Systems Manager and Amazon Security Lake"
url: "https://aws.amazon.com/blogs/security/file-integrity-monitoring-with-aws-systems-manager-and-amazon-security-lake/"
date: "Tue, 27 Jan 2026 18:21:23 +0000"
author: "Adam Nemeth"
feed_url: "https://aws.amazon.com/blogs/security/tag/amazon-security-lake/feed/"
---
<div class="Page-articleBody"> 
 <div class="RichTextArticleBody RichTextBody"> 
  <p>Customers need solutions to track inventory data such as files and software across <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/ec2" rel="noopener" target="_blank">Amazon Elastic Compute Cloud (Amazon EC2)</a></span> instances, detect unauthorized changes, and integrate alerts into their existing security workflows.</p> 
  <p>In this blog post, I walk you through a highly scalable serverless file integrity monitoring solution. It uses <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-inventory.html" rel="noopener" target="_blank">AWS Systems Manager Inventory</a></span> to collect file metadata from <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/ec2/" rel="noopener" target="_blank">Amazon EC2</a></span> instances. The metadata is sent through the <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/systems-manager/latest/userguide/Explorer-resource-data-sync-configuring-multi.html" rel="noopener" target="_blank">Systems Manager Resource Data Sync</a></span> feature to a versioned <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a></span> bucket, storing one inventory object for each EC2 instance. Each time a new object is created in Amazon S3, an <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/NotificationHowTo.html" rel="noopener" target="_blank">Amazon S3 Event Notification</a></span> triggers a custom <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/lambda/" rel="noopener" target="_blank">AWS Lambda</a></span> function. This Lambda function compares the latest inventory version with the previous one to detect file changes. If a file that isn’t expected to change has been created, modified, or deleted, the function creates an actionable finding in <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/security-hub/" rel="noopener" target="_blank">AWS Security Hub</a></span>. Findings are then ingested by <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/security-lake/" rel="noopener" target="_blank">Amazon Security Lake</a></span> in a standard <span class="LinkEnhancement"><a class="Link" href="https://ocsf.io/" rel="noopener" target="_blank">OCSF</a></span> format, which centralizes and normalizes the data. Finally, the data can be analyzed using <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/athena/" rel="noopener" target="_blank">Amazon Athena</a></span> for one-time queries, or by building visual dashboards with <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/quicksight/" rel="noopener" target="_blank">Amazon QuickSight</a></span> and <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/opensearch-service/" rel="noopener" target="_blank">Amazon OpenSearch Service</a></span>. Figure 1 summarizes this flow:</p> 
  <div class="wp-caption alignnone" id="attachment_41246" style="width: 760px;">
   <img alt="Figure 1: File integrity monitoring workflow" class="size-full wp-image-41246" height="983" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2026/01/22/2877.1.png" style="border: 1px solid #bebebe;" width="750" />
   <p class="wp-caption-text" id="caption-attachment-41246">Figure 1: File integrity monitoring workflow</p>
  </div> 
  <p>This integration offers an alternative to the default <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/config/" rel="noopener" target="_blank">AWS Config</a></span> and <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-setup-prereqs.html" rel="noopener" target="_blank">Security Hub integration,</a></span> which relies on limited data (for example, no file modification timestamps). The solution presented in this post provides control and flexibility to implement custom logic tailored to your operational needs and support security-related efforts.</p> 
  <p>This flexible solution can also be used with other <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/systems-manager/latest/userguide/inventory-schema.html" rel="noopener" target="_blank">Systems Manager Inventory metadata</a></span>, such as installed applications, network configurations, or Windows registry entries, enabling custom detection logic across a wide range of operational and security use cases.</p> 
  <p>Now let’s build the file integrity monitoring solution.</p> 
  <div class="RichTextHeading"> 
   <h2>Prerequisites</h2> 
  </div> 
  <p>Before you get started, you need an AWS account with permissions to create and manage AWS resources such as Amazon EC2, <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/systems-manager" rel="noopener" target="_blank">AWS Systems Manager</a></span>, Amazon S3, and Lambda.</p> 
  <div class="RichTextHeading"> 
   <h2>Step 1: Start an EC2 instance</h2> 
  </div> 
  <p>Start by launching an EC2 instance and creating a file that you will later modify to simulate an unauthorized change.</p> 
  <p>Create an <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a></span> role to allow the EC2 instance to communicate with Systems Manager:</p> 
  <ol class="rte2-style-ol" id="rte-d023c395-f7c8-11f0-85b0-f765fcf3faad" start="1"> 
   <li>Open the AWS Management Console and go to <b>IAM</b>, choose <b>Roles </b>from the navigation pane, and then choose <b>Create role</b>.</li> 
   <li>Under <b>Trusted entity type</b>, select <b>AWS service</b>, select <b>EC2</b> as the use case, and choose <b>Next</b>.</li> 
   <li>On the <b>Add permissions</b> page, search for and select the <b>AmazonSSMManagedInstanceCore</b> IAM policy, then choose <b>Next</b>.</li> 
   <li>Enter <b>SSMAccessRole</b> as the role name and choose <b>Create role</b>.</li> 
   <li>The new <b>SSMAccessRole</b> should now appear in your list of IAM roles:</li> 
  </ol> 
  <div class="wp-caption alignnone" id="attachment_41247" style="width: 946px;">
   <img alt="Figure 2: Create an IAM role for communication with Systems Manager" class="size-full wp-image-41247" height="251" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2026/01/22/2877.2.png" style="border: 1px solid #bebebe;" width="936" />
   <p class="wp-caption-text" id="caption-attachment-41247">Figure 2: Create an IAM role for communication with Systems Manager</p>
  </div> 
  <p>Start an EC2 instance:</p> 
  <ol class="rte2-style-ol" id="rte-669001d0-f7cf-11f0-96f6-6bbbd6ebe640" start="1"> 
   <li>Open the Amazon EC2 console and choose <b>Launch Instance</b>.</li> 
   <li>Enter a <b>Name</b>, keep the default Linux Amazon Machine Image (AMI), and select an <b>Instance type</b> (for example, <b>t3.micro</b>).</li> 
   <li>Under <b>Advanced details</b>: 
    <ol class="rte2-style-ol" id="rte-669001d1-f7cf-11f0-96f6-6bbbd6ebe640" start="1" type="a"> 
     <li><b>IAM instance profile</b>, select the previously created <b>SSMAccessRole</b></li> 
     <li>Create a fictitious payment application configuration file in the <code class="CodeInline" style="color: #000;">/etc/paymentapp/</code> folder on the EC2 instance. Later, you will modify it to demonstrate a file-change event for integrity monitoring. To create this file during EC2 startup, copy and paste the following script into <b>User data.</b></li> 
    </ol> </li> 
  </ol> 
  <div class="Enhancement"> 
   <div class="Enhancement-item"> 
    <div class="CodeBlockWP hide-language"> 
     <div class="code-toolbar"> 
      <pre class="unlimited-height-code language-text"><code class="language-text">#!/bin/bash
mkdir -p /etc/paymentapp
echo "db_password=initial123" &gt; /etc/paymentapp/config.yaml</code></pre> 
     </div> 
    </div> 
   </div> 
  </div> 
  <div class="wp-caption alignnone" id="attachment_41248" style="width: 890px;">
   <img alt="Figure 3: Adding the application configuration file" class="size-full wp-image-41248" height="382" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2026/01/22/2877.3.png" style="border: 1px solid #bebebe;" width="880" />
   <p class="wp-caption-text" id="caption-attachment-41248">Figure 3: Adding the application configuration file</p>
  </div> 
  <ol class="rte2-style-ol" id="rte-669028e0-f7cf-11f0-96f6-6bbbd6ebe640" start="4"> 
   <li>Leave the remaining settings as default, choose <b>Proceed without key pair</b>, and then select <b>Launch Instance</b>. A key pair isn’t required for this demo because you use Session Manager for access.</li> 
  </ol> 
  <div class="RichTextHeading"> 
   <h2>Step 2: Enable Security Hub and Security Lake</h2> 
  </div> 
  <p>If Security Hub and Security Lake are already enabled, you can skip to <b>Step 3</b>.<br /> To start, enable Security Hub, which collects and aggregates security findings. <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html" rel="noopener" target="_blank">AWS Security Hub CSPM</a></span> adds continuous monitoring and automated checks against best practices.</p> 
  <ol class="rte2-style-ol" id="rte-669028e1-f7cf-11f0-96f6-6bbbd6ebe640" start="1"> 
   <li>Open the Security Hub console.</li> 
   <li>Choose <b>Security Hub CSPM</b> from the navigation pane and then select <b>Enable AWS Security Hub CSPM</b> and choose <b>Enable Security Hub CSPM </b>at the bottom of the page.</li> 
  </ol> 
  <p><b>Note: </b>For this demo, you don’t need the <b>Security standards</b> options and can clear them.</p> 
  <div class="wp-caption alignnone" id="attachment_41249" style="width: 967px;">
   <img alt="Figure 4: Enable Security Hub CSP" class="size-full wp-image-41249" height="666" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2026/01/22/2877.4.png" style="border: 1px solid #bebebe;" width="957" />
   <p class="wp-caption-text" id="caption-attachment-41249">Figure 4: Enable Security Hub CSP</p>
  </div> 
  <p>Next, activate Security Lake to start collecting actionable findings from Security Hub:</p> 
  <ol class="rte2-style-ol" id="rte-d023c397-f7c8-11f0-85b0-f765fcf3faad" start="1"> 
   <li>Open the <b>Amazon Security Lake</b> console and choose <b>Get Started</b>.</li> 
   <li>Under <b>Data sources</b>, select <b>Ingest specific AWS sources</b>.</li> 
   <li>Under <b>Log and event </b>sources, select <b>Security Hub </b>(you will use this only for this demo):</li> 
  </ol> 
  <div class="wp-caption alignnone" id="attachment_41250" style="width: 1298px;">
   <img alt="Figure 5: Select log and event sources" class="size-full wp-image-41250" height="838" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2026/01/22/2877.5.jpg" style="border: 1px solid #bebebe;" width="1288" />
   <p class="wp-caption-text" id="caption-attachment-41250">Figure 5: Select log and event sources</p>
  </div> 
  <ol class="rte2-style-ol" id="rte-d023c398-f7c8-11f0-85b0-f765fcf3faad" start="1"> 
   <li>Under <b>Select Regions</b>, choose <b>Specific Regions</b> and make sure you select the AWS Region that you’re using.</li> 
   <li>Use the default option to <b>Create and use a new service role</b>.</li> 
   <li>Choose <b>Next</b> and <b>Next </b>again, then choose <b>Create</b>.</li> 
  </ol> 
  <div class="RichTextHeading"> 
   <h2>Step 3: Configure Systems Manager Inventory and sync to Amazon S3</h2> 
  </div> 
  <p>With Security Hub and Security Lake enabled, the next step is to enable Systems Manager Inventory to collect file metadata and configure a Resource Data Sync to export this data to S3 for analysis.</p> 
  <ol class="rte2-style-ol" id="rte-d023eaa0-f7c8-11f0-85b0-f765fcf3faad" start="1"> 
   <li>Create an S3 bucket by carefully following the instructions in the section <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/systems-manager/latest/userguide/inventory-create-resource-data-sync.html#datasync-before-you-begin" rel="noopener" target="_blank">To create and configure an Amazon S3 bucket for resource data sync</a></span>.</li> 
   <li>After you created the bucket, enable versioning in the Amazon S3 console by opening the bucket’s <b>Properties</b> tab, choosing <b>Edit</b> under <b>Bucket Versioning</b>, selecting <b>Enable</b>, and saving your changes. Versioning causes each new inventory snapshot to be saved as a separate version, so that you can track file changes over time.</li> 
  </ol> 
  <p><b>Note:</b> In production, enable <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/ServerLogs.html" rel="noopener" target="_blank">S3 server access logging</a></span> on the inventory bucket to keep an audit trail of access requests, <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/example-bucket-policies.html?utm_source=chatgpt.com#example-bucket-policies-HTTP-HTTPS" rel="noopener" target="_blank">enforce HTTPS-only access</a></span>, and enable <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html" rel="noopener" target="_blank">CloudTrail data events for S3</a></span> to record who accessed or modified inventory files.</p> 
  <p>The next step is to enable Systems Manager Inventory and set up the resource data sync:</p> 
  <ol class="rte2-style-ol" id="rte-d023eaa5-f7c8-11f0-85b0-f765fcf3faad" start="1"> 
   <li>In the <b>Systems Manager</b> console, go to <b>Fleet Manager</b>, choose <b>Account management</b>, and select <b>Set up inventory</b>.</li> 
   <li>Keep the default values but deselect every inventory type except <b>File</b>. Set a <b>Path</b> to limit collection to the files relevant for this demo and your security requirements. Under <b>File</b>, set the <b>Path</b> to: <code class="CodeInline" style="color: #000;">/etc/paymentapp/</code>.</li> 
  </ol> 
  <div class="wp-caption alignnone" id="attachment_41251" style="width: 946px;">
   <img alt="Figure 6: Set the parameters and path" class="size-full wp-image-41251" height="668" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2026/01/22/2877.6.png" style="border: 1px solid #bebebe;" width="936" />
   <p class="wp-caption-text" id="caption-attachment-41251">Figure 6: Set the parameters and path</p>
  </div> 
  <ol class="rte2-style-ol" id="rte-d023eaa6-f7c8-11f0-85b0-f765fcf3faad" start="1"> 
   <li>Choose <b>Setup Inventory</b>.</li> 
   <li>In Fleet Manager, choose <b>Account management</b> and select <b>Resource Data Syncs</b>.</li> 
   <li>Choose <b>Create resource data sync</b>, enter a <b>Sync name</b>, and enter the name of the versioned S3 bucket you created earlier.</li> 
   <li>Select <b>This Region</b> and then choose <b>Create</b>.</li> 
  </ol> 
  <div class="RichTextHeading"> 
   <h2>Step 4: Implement the Lambda function</h2> 
  </div> 
  <p>Next, complete the setup to detect changes and create findings. Each time Systems Manager Inventory writes a new object to Amazon S3, an <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/NotificationHowTo.html" rel="noopener" target="_blank">S3 Event Notification</a></span> triggers a Lambda function that compares the latest and previous object versions. If it finds created, modified, or deleted files, it creates a security finding. To accomplish this, you will create the Lambda function, set its environment variables, add the helper layer, and attach the required permissions.</p> 
  <p>The following is an example finding generated in <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-findings-format.html" rel="noopener" target="_blank">AWS Security Finding Format</a></span> (ASFF) and sent to Security Hub. In this example, you see a notification about a file change on the EC2 instance listed under the <code class="CodeInline" style="color: #000;">Resources</code> section.</p> 
  <div class="Enhancement"> 
   <div class="Enhancement-item"> 
    <div class="CodeBlockWP hide-language"> 
     <div class="code-toolbar"> 
      <pre class="unlimited-height-code language-text"><code class="language-text">{
	...
"Id": "fim-i-0b8f40f4de065deba-2025-07-12T13:48:31.741Z",
	"AwsAccountId": "XXXXXXXXXXXX",
	"Types": [
		"Software and Configuration Checks/File Integrity Monitoring"
	],
	"Severity": {
		"Label": "MEDIUM"
	},
	"Title": "File changes detected via SSM Inventory",
	"Description": "0 created, 1 modified, 0 deleted file(s) on instance i-0b8f40f4de065deba",
	"Resources": [
		{
			"Type": "AwsEc2Instance",
			"Id": "i-0b8f40f4de065deba"
		}
	],
	...
}</code></pre> 
     </div> 
    </div> 
   </div> 
  </div> 
  <div class="RichTextHeading"> 
   <h3>Create the Lambda function</h3> 
  </div> 
  <p>This function detects file changes, reports findings, and removes unused Amazon S3 object versions to reduce costs.</p> 
  <ol class="rte2-style-ol" id="rte-66904ff0-f7cf-11f0-96f6-6bbbd6ebe640" start="1"> 
   <li>Open the <b>Lambda console </b>and choose <b>Create function </b>in the navigation pane.</li> 
   <li>For <b>Function Name</b> enter <code class="CodeInline" style="color: #000;">fim-change-detector</code>.</li> 
   <li>Select <b>Author from scratch</b>, enter a function name, select the latest <b>Python</b> runtime, and choose <b>Create function</b>.</li> 
   <li>On the <b>Code</b> tab, paste the following main function and choose <b>Deploy</b>.</li> 
  </ol> 
  <div class="Enhancement"> 
   <div class="Enhancement-item"> 
    <div class="CodeBlockWP hide-language"> 
     <div class="code-toolbar"> 
      <pre class="unlimited-height-code language-text"><code class="language-text">import boto3, os, json, re
from datetime import datetime, UTC
from urllib.parse import unquote_plus
from helpers import is_critical, load_file_metadata, is_modified, extract_instance_id

s3 = boto3.client('s3')
securityhub = boto3.client('securityhub')

CRITICAL_FILE_PATTERNS = os.environ["CRITICAL_FILE_PATTERNS"].split(",")
SEVERITY_LABEL = os.environ["SEVERITY_LABEL"]
	
def lambda_handler(event, context):
	# Safe event handling
	if "Records" not in event or not event["Records"]:
		return

	# Extract S3 event
	record = event['Records'][0]
	bucket = record['s3']['bucket']['name']
	key = unquote_plus(record['s3']['object']['key'])
	current_version = record['s3']['object'].get('versionId')
	if not current_version:
		return

	# Fetching the region name
	account_id = context.invoked_function_arn.split(":")[4]
	region = boto3.session.Session().region_name

	# Get object versions (latest first)
	versions = s3.list_object_versions(Bucket=bucket, Prefix=key).get('Versions', [])
	versions = sorted(versions, key=lambda v: v['LastModified'], reverse=True)

	# Find previous version
	idx = next((i for i,v in enumerate(versions) if v["VersionId"] == current_version), None)
	if idx is None or idx + 1 &gt;= len(versions):
		return
	prev_version = versions[idx+1]["VersionId"]

	# Load both versions
	current = load_file_metadata(bucket, key, current_version)
	previous = load_file_metadata(bucket, key, prev_version)

	# Compare
	created = {p for p in set(current) - set(previous) if is_critical(p)}
	deleted = {p for p in set(previous) - set(current) if is_critical(p)}
	modified = {p for p in set(current) &amp; set(previous) if is_critical(p) and is_modified(p, current, previous)}

	# Report if changes were found
	if created or deleted or modified:
		instance_id = extract_instance_id(bucket, key, current_version)
		now = datetime.now(UTC).isoformat(timespec='milliseconds').replace('+00:00', 'Z')
		finding = {
			"SchemaVersion": "2018-10-08",
			"Id": f"fim-{instance_id}-{now}",
			"ProductArn": f"arn:aws:securityhub:{region}:{account_id}:product/{account_id}/default",
			"AwsAccountId": account_id,
			"GeneratorId": "ssm-inventory-fim",
			"CreatedAt": now,
			"UpdatedAt": now,
			"Types": ["Software and Configuration Checks/File Integrity Monitoring"],
			"Severity": {"Label": SEVERITY_LABEL},
			"Title": "File changes detected via SSM Inventory",
			"Description": (
				f"{len(created)} created, {len(modified)} modified, "
				f"{len(deleted)} deleted file(s) on instance {instance_id}"
			),
			"Resources": [{"Type": "AwsEc2Instance", "Id": instance_id}]
		}
		securityhub.batch_import_findings(Findings=[finding])

	# No change – delete older S3 version
	else:
		if prev_version != current_version:
			try:
				s3.delete_object(Bucket=bucket, Key=key, VersionId=prev_version)
			except Exception as e:
				print(f"Delete previous S3 object version failed: {e}")</code></pre> 
     </div> 
    </div> 
   </div> 
  </div> 
  <p><b>Note</b>: In production, set <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html" rel="noopener" target="_blank">Lambda reserved concurrency</a></span> to prevent unbounded scaling, configure a <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-property-function-deadletterqueue.html" rel="noopener" target="_blank">dead letter queue (DLQ)</a></span> to capture failed invocations, and optionally attach the function to an <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc.html" rel="noopener" target="_blank">Amazon VPC for network isolation</a></span>.</p> 
  <div class="RichTextHeading"> 
   <h3>Configure environment variables</h3> 
  </div> 
  <p>Configure the two required environment variables in the Lambda console. These two variables (one for critical paths to monitor and one for security finding severity) must be set or the function will fail.</p> 
  <ol class="rte2-style-ol" id="rte-66904ff1-f7cf-11f0-96f6-6bbbd6ebe640" start="1"> 
   <li>Open the Lambda console and choose <b>Configuration </b>and then select <b>Environment variables</b>.</li> 
   <li>Choose <b>Edit </b>and then choose<b> Add environment variable</b>.</li> 
   <li>Under <b>Key</b>, choose <b>CRITICAL_FILE_PATTERNS</b> 
    <ol class="rte2-style-ol" id="rte-66904ff2-f7cf-11f0-96f6-6bbbd6ebe640" start="1" type="a"> 
     <li>Enter <code class="CodeInline" style="color: #000;">^/etc/paymentapp/config.*$</code> as the value.</li> 
     <li>Set the <b>SEVERITY_LABEL</b> to <b>MEDIUM</b>.</li> 
    </ol> </li> 
  </ol> 
  <div class="wp-caption alignnone" id="attachment_41292" style="width: 852px;">
   <img alt="Figure 7: CRITICAL_FILE_PATTERNS and SEVERITY_LABEL configuration" class="wp-image-41292 size-full" height="361" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2026/01/23/2877.new_-1.png" style="border: 1px solid #bebebe;" width="842" />
   <p class="wp-caption-text" id="caption-attachment-41292">Figure 7: CRITICAL_FILE_PATTERNS and SEVERITY_LABEL configuration</p>
  </div> 
  <div class="RichTextHeading"> 
   <h3>Set up permissions</h3> 
  </div> 
  <p>The next step is to attach permissions to the Lambda function</p> 
  <ol class="rte2-style-ol" id="rte-d02411b3-f7c8-11f0-85b0-f765fcf3faad" start="1"> 
   <li>In your Lambda function, choose <b>Configuration</b> and then select <b>Permissions</b>.</li> 
   <li>Under <b>Execution role</b>, select the role name that will lead to the role in IAM.</li> 
   <li>Choose <b>Add permissions</b> and select <b>Create inline policy</b>. Select <b>JSON view</b>.</li> 
   <li>Paste the <b>following policy</b>, and make sure to replace <code class="CodeInline" style="color: #000;">&lt;bucket-name&gt;</code> with the name of your S3 bucket, and you also update <code class="CodeInline" style="color: #000;">&lt;region&gt;</code> and <code class="CodeInline" style="color: #000;">&lt;account-id&gt;</code> with your AWS Region and Account ID:</li> 
  </ol> 
  <div class="Enhancement"> 
   <div class="Enhancement-item"> 
    <div class="CodeBlockWP hide-language"> 
     <div class="code-toolbar"> 
      <pre class="unlimited-height-code language-text"><code class="language-text">{
"Version": "2012-10-17",
"Statement": [
	{
		"Effect": "Allow",
		"Action": "securityhub:BatchImportFindings",
		"Resource": "arn:aws:securityhub:&lt;region&gt;:&lt;account-id&gt;:product/&lt;account-id&gt;/default"
	},
	{
		"Effect": "Allow",
		"Action": [
			"s3:GetObject",
			"s3:GetObjectVersion",
			"s3:ListBucketVersions",
			"s3:DeleteObjectVersion"
		],
		"Resource": [
			"arn:aws:s3:::&lt;bucket-name&gt;",
			"arn:aws:s3:::&lt;bucket-name&gt;/*"
			]
		}
	]
}</code></pre> 
     </div> 
    </div> 
   </div> 
  </div> 
  <ol class="rte2-style-ol" id="rte-d02438c0-f7c8-11f0-85b0-f765fcf3faad" start="5"> 
   <li>To finalize, enter a <b>Policy name</b> and choose <b>Create policy</b>.</li> 
  </ol> 
  <div class="RichTextHeading"> 
   <h3>Add functions to the Lambda layer</h3> 
  </div> 
  <p>For better modularity, add some helper functions to a <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/lambda/latest/dg/python-layers.html" rel="noopener" target="_blank">Lambda layer</a></span>. These functions are already referenced in the import section of the preceding Lambda function’s Python code. The helper functions check critical paths, load file metadata, compare modification times, and extract the EC2 instance ID.</p> 
  <p>Open <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/cloudshell/" rel="noopener" target="_blank">AWS CloudShell</a></span> from the top-right corner of the AWS console header, then copy and paste the following script and press Enter. It creates the helper layer and attaches it to your Lambda function.</p> 
  <div class="Enhancement"> 
   <div class="Enhancement-item"> 
    <div class="CodeBlockWP hide-language"> 
     <div class="code-toolbar"> 
      <pre class="unlimited-height-code language-text"><code class="language-text">#!/bin/bash
set -e
FUNCTION_NAME="fim-change-detector"
LAYER_NAME="fim-change-detector-layer"

mkdir -p python
cat &gt; python/helpers.py &lt;&lt; 'EOF'
import json, re, os
from dateutil.parser import parse as parse_dt
import boto3
s3 = boto3.client('s3')
CRITICAL_FILE_PATTERNS = os.environ.get("CRITICAL_FILE_PATTERNS", "").split(",")

def is_critical(path):
	return any(re.match(p.strip(), path) for p in CRITICAL_FILE_PATTERNS if p.strip())

def load_file_metadata(bucket, key, version_id):
	obj = s3.get_object(Bucket=bucket, Key=key, VersionId=version_id)
	data = {}
	for line in obj['Body'].read().decode().splitlines():
		if line.strip():
			i = json.loads(line)
			n, d, m = i.get("Name","").strip(), i.get("InstalledDir","").strip(), i.get("ModificationTime","").strip()
			if n and d and m: data[f"{d.rstrip('/')}/{n}"] = m
	return data

def is_modified(path, current, previous):
	try: return parse_dt(current[path]) != parse_dt(previous[path])
	except: return current[path] != previous[path]

def extract_instance_id(bucket, key, version_id):
	obj = s3.get_object(Bucket=bucket, Key=key, VersionId=version_id)
	for line in obj['Body'].read().decode().splitlines():
		if line.strip():
			r = json.loads(line)
			if "resourceId" in r: return r["resourceId"]
	return None
EOF

zip -r helpers_layer.zip python &gt;/dev/null
LAYER_VERSION_ARN=$(aws lambda publish-layer-version \
	--layer-name "$LAYER_NAME" \
	--description "Helper functions for File Integrity Monitoring" \
	--zip-file fileb://helpers_layer.zip \
	--compatible-runtimes python3.13 \
	--query 'LayerVersionArn' \
	--output text)

aws lambda update-function-configuration \
	--function-name "$FUNCTION_NAME" \
	--layers "$LAYER_VERSION_ARN" &gt;/dev/null
echo "Layer created and attached to the Lambda function."</code></pre> 
     </div> 
    </div> 
   </div> 
  </div> 
  <div class="RichTextHeading"> 
   <h2>Step 5: Set up S3 Event Notifications</h2> 
  </div> 
  <p>Finally, set up <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/EventNotifications.html" rel="noopener" target="_blank">S3 Event Notifications</a></span> to trigger the Lambda function when new inventory data arrives.</p> 
  <ol class="rte2-style-ol" id="rte-d02438c4-f7c8-11f0-85b0-f765fcf3faad" start="1"> 
   <li>Open the S3 console and select the Systems Manager Inventory bucket that you created.</li> 
   <li>Choose <b>Properties</b> and select <b>Event notifications</b>.</li> 
   <li>Choose <b>Create event notification</b>. 
    <ol class="rte2-style-ol" id="rte-66907700-f7cf-11f0-96f6-6bbbd6ebe640" start="1" type="a"> 
     <li>Enter an <b>Event name</b>.</li> 
     <li>In the <b>Prefix</b> field, enter <code class="CodeInline" style="color: #000;">AWS%3AFile/</code> to limit Lambda triggers to file inventory objects only.<br /> <b>Note: </b>The prefix contains a : character, which must be URL-encoded as <code class="CodeInline" style="color: #000;">%3A</code>.</li> 
     <li>Under <b>Event types</b>, select <b>Put</b>.</li> 
     <li>At the bottom, select your newly created <b>Lambda function</b>, and choose <b>Save changes</b>.</li> 
    </ol> </li> 
  </ol> 
  <p>In this example, inventory collection runs every 30 minutes (48 times each day) but can be adjusted based on security requirements to optimize costs. The Lambda function is triggered once for each instance whenever a new inventory object is created. You can further reduce event volume by filtering EC2 instances through <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/notification-how-to-filtering.html" rel="noopener" target="_blank">S3 Event Notification prefixes</a></span>, enabling focused monitoring of high-value instances.</p> 
  <div class="RichTextHeading"> 
   <h2>Step 6: Test the file change detection flow</h2> 
  </div> 
  <p>Now that the EC2 instance is running and the sample configuration file <code class="CodeInline" style="color: #000;">/etc/paymentapp/config.yaml</code> has been initialized, you’re ready to simulate an unauthorized change to test the file integrity monitoring setup.</p> 
  <ol class="rte2-style-ol" id="rte-66907701-f7cf-11f0-96f6-6bbbd6ebe640" start="1"> 
   <li>Open the <b>Systems Manager</b> console.</li> 
   <li>Go to <b>Session Manager</b> and choose <b>Start session</b>.</li> 
   <li>Select your <b>EC2 instance</b> and choose <b>Start Session</b>.</li> 
   <li>Run the following command to modify the file:</li> 
  </ol> 
  <p><code class="CodeInline" style="color: #000;">echo “db_password=hacked456" | sudo tee /etc/paymentapp/config.yaml</code></p> 
  <p>This simulates a configuration tampering event. During the next Systems Manager Inventory run, the updated metadata will be saved to Amazon S3.</p> 
  <p>To manually trigger this:</p> 
  <ol class="rte2-style-ol" id="rte-d02438c6-f7c8-11f0-85b0-f765fcf3faad" start="1"> 
   <li>Open the Systems Manager console<b> </b>and choose <b>State Manager</b>.</li> 
   <li>Select your association and choose <b>Apply association now</b> to start the inventory update.</li> 
   <li>After the association status changes to <b>Success</b>, check your <b>SSM Inventory S3 bucket</b> in the <code class="CodeInline" style="color: #000;">AWS:File</code> folder and review the inventory object and its versions.</li> 
   <li>Open the Security Hub console and choose Findings. After a short delay, you should see a new finding like the one shown in Figure 8:</li> 
  </ol> 
  <div class="wp-caption alignnone" id="attachment_41253" style="width: 824px;">
   <img alt="Figure 8: View file change findings" class="size-full wp-image-41253" height="644" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2026/01/22/2877.8.jpg" style="border: 1px solid #bebebe;" width="814" />
   <p class="wp-caption-text" id="caption-attachment-41253">Figure 8: View file change findings</p>
  </div> 
  <div class="RichTextHeading"> 
   <h2>Step 7: Query and visualize findings</h2> 
  </div> 
  <p>While Security Hub provides a centralized view of findings, you can deepen your analysis using <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/athena" rel="noopener" target="_blank">Amazon Athena</a></span> to run SQL queries directly on the normalized Security Lake data in Amazon S3. This data follows the <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/security-lake/latest/userguide/open-cybersecurity-schema-framework.html" rel="noopener" target="_blank">Open Cybersecurity Schema Framework</a></span> (OCSF), which is a vendor-neutral standard that simplifies integration and analysis of security data across different tools and services.</p> 
  <p>The following is an example Athena query:</p> 
  <div class="Enhancement"> 
   <div class="Enhancement-item"> 
    <div class="CodeBlockWP hide-language"> 
     <div class="code-toolbar"> 
      <pre class="unlimited-height-code language-text"><code class="language-text">SELECT
	finding_info.desc AS description,
	class_uid AS class_id,
	severity AS severity_label,
	type_name AS finding_type,
	time_dt AS event_time,
	region,
	accountid
FROM amazon_security_lake_table_us_east_1_sh_findings_2_0</code></pre> 
     </div> 
    </div> 
   </div> 
  </div> 
  <p><b>Note:</b> Be sure to adjust the <code class="CodeInline" style="color: #000;">FROM</code> clause for other Regions. Security Lake processes findings before they appear in Athena, so expect a short delay between ingestion and data availability.<br /> You will see a similar result for the preceding query, shown in Figure 9:</p> 
  <div class="wp-caption alignnone" id="attachment_41254" style="width: 1440px;">
   <img alt="Figure 9: Athena query result in the Amazon Athena query editor" class="size-full wp-image-41254" height="124" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2026/01/22/2877.9.png" style="border: 1px solid #bebebe;" width="1430" />
   <p class="wp-caption-text" id="caption-attachment-41254">Figure 9: Athena query result in the Amazon Athena query editor</p>
  </div> 
  <p>Security Lake classifies this finding as an OCSF <span class="LinkEnhancement"><a class="Link" href="https://schema.ocsf.io/classes/detection_finding" rel="noopener" target="_blank">2004 Class, Detection Finding</a></span>. You can explore the full schema definitions at <span class="LinkEnhancement"><a class="Link" href="https://schema.ocsf.io/" rel="noopener" target="_blank">OCSF Categories</a></span>. For more query examples, see the <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/security-lake/latest/userguide/security-hub-query-examples-sourceversion2.html" rel="noopener" target="_blank">Security Lake query examples</a></span>.<br /> For visual exploration and real-time insights, you can integrate Security Lake with OpenSearch Service and QuickSight, both of which now offer extensive generative AI support. For a guided walkthrough using QuickSight, see <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/blogs/security/how-to-visualize-amazon-security-lake-findings-with-amazon-quicksight/" rel="noopener" target="_blank">How to visualize Amazon Security Lake findings with Amazon QuickSight</a></span>.</p> 
  <div class="RichTextHeading"> 
   <h2>Clean up</h2> 
  </div> 
  <p>After testing the step-by-step guide, make sure to clean up the resources you created for this post to avoid ongoing costs.</p> 
  <ol class="rte2-style-ol" id="rte-d0245fd6-f7c8-11f0-85b0-f765fcf3faad" start="1"> 
   <li>Terminate the EC2 instance</li> 
   <li>Delete the Resource Data Sync and Inventory Association</li> 
   <li>Remove the Lambda function.</li> 
   <li>Disable Security Lake and Security Hub CSPM</li> 
   <li>Delete IAM roles created for this post</li> 
   <li>Delete the associated SSM Resource Data Sync and Security Lake S3 buckets.</li> 
  </ol> 
  <div class="RichTextHeading"> 
   <h2>Conclusion</h2> 
  </div> 
  <p>In this post, you learned how to use Systems Manager Inventory to track file integrity, report findings to Security Hub, and analyze them using Security Lake.<br /> You can access the full sample code to set up this solution in the <span class="LinkEnhancement"><a class="Link" href="https://github.com/aws-samples/sample-inventory-monitor-fim" rel="noopener" target="_blank">AWS Samples repository</a></span>.<br /> While this post uses a single-account, single-Region setup for simplicity, <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/security-lake/latest/userguide/multi-account-management.html" rel="noopener" target="_blank">Security Lake supports collecting data across multiple accounts</a></span> and Regions using <span class="LinkEnhancement"><a class="Link" href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a></span>. You can also use a <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/systems-manager/latest/userguide/inventory-create-resource-data-sync.html" rel="noopener" target="_blank">Systems Manager resource data sync</a></span> to send inventory data to a central S3 bucket.</p> 
  <p><span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/security-lake/latest/userguide/getting-started.html" rel="noopener" target="_blank">Getting Started with Amazon Security Lake</a></span> and <span class="LinkEnhancement"><a class="Link" href="https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-inventory.html" rel="noopener" target="_blank">Systems Manager Inventory</a></span> provides guidance for enabling scalable, cloud-centric monitoring with full operational context.</p> 
  <footer> 
   <div class="blog-author-box">
    <img alt="Adam Nemeth " class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2026/01/22/adamnemeth.nemethad-1.jpg" style="width: 93.750px; height: 125px; margin: 12px 18px 6px 12px;" />
    <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Adam Nemeth</span>
    <br /> Adam is a Senior Solutions Architect and generative AI enthusiast at AWS, helping financial services customers by embracing the Day 1 culture and customer obsession of Amazon. With over 24 years of IT experience, Adam previously worked at UBS as an architect and has also served as a delivery lead, consultant, and entrepreneur. He lives in Switzerland with his wife and their three children.
   </div> 
  </footer> 
 </div> 
</div>
