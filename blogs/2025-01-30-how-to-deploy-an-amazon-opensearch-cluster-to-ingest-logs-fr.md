---
title: "How to deploy an Amazon OpenSearch cluster to ingest logs from Amazon Security Lake"
url: "https://aws.amazon.com/blogs/security/how-to-deploy-an-amazon-opensearch-cluster-to-ingest-logs-from-amazon-security-lake/"
date: "Thu, 30 Jan 2025 16:17:14 +0000"
author: "Kevin Low"
feed_url: "https://aws.amazon.com/blogs/security/tag/amazon-security-lake/feed/"
---
<blockquote>
 <p><strong>April 29, 2025:</strong> We’ve updated this post to make it simpler for customers to deploy the resources.</p>
</blockquote> 
<blockquote>
 <p><strong>July 29, 2024:</strong> Original publication date of this post. The current version was updated to make the instructions clearer and compatible with OCSF 1.1.</p>
</blockquote> 
<hr /> 
<p>Customers often require multiple log sources across their AWS environment to empower their teams to respond and investigate security events. In part one of this two-part blog post, I show you how you can use <a href="https://aws.amazon.com/opensearch-service/" rel="noopener" target="_blank">Amazon OpenSearch Service</a> to ingest logs collected by <a href="https://aws.amazon.com/security-lake" rel="noopener" target="_blank">Amazon Security Lake</a> to facilitate near real-time monitoring.</p> 
<p>Many customers use&nbsp;Security Lake&nbsp;to automatically centralize security data from&nbsp;<a href="https://aws.amazon.com/aws" rel="noopener" target="_blank">Amazon Web Services (AWS)</a>&nbsp;environments, software as a service (SaaS) providers, on-premises workloads, and cloud sources into a purpose-built data lake in their AWS environment. OpenSearch Service&nbsp;is a managed service that customers can use to deploy, operate, and scale OpenSearch clusters in the AWS Cloud. It natively integrates with Security Lake to enable customers to perform interactive log analytics and searches across large datasets, create enterprise visualization and dashboards, and perform analysis across disparate applications and logs. With&nbsp;<a href="https://docs.aws.amazon.com/opensearch-service/latest/developerguide/security-analytics.html" rel="noopener" target="_blank">Amazon OpenSearch Security Analytics</a>, customers can also gain visibility into the security posture of their organization’s infrastructure, monitor for anomalous activity, detect potential security threats in near real time, and initiate alerts to pre-configured destinations.</p> 
<p>Without using Amazon OpenSearch Service, customers would need to build, deploy and manage infrastructure for an analytics solution, such as an <a href="https://aws.amazon.com/what-is/elk-stack/" rel="noopener" target="_blank">ELK stack.</a></p> 
<h2>Prerequisites</h2> 
<p>Security Lake should already be deployed. For details on how to deploy Security Lake, see&nbsp;<a href="https://docs.aws.amazon.com/security-lake/latest/userguide/getting-started.html" rel="noopener" target="_blank">Getting started with Amazon Security Lake</a>. You will need&nbsp;<a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a>&nbsp;permissions to <a href="https://aws.amazon.com/cloudshell/" rel="noopener" target="_blank">AWS Cloudshell</a> manage Security Lake, OpenSearch Service, <a href="https://aws.amazon.com/cognito" rel="noopener" target="_blank">Amazon Cognito</a>, <a href="https://aws.amazon.com/secrets-manager" rel="noopener" target="_blank">AWS Secrets Manager</a>, and <a href="https://aws.amazon.com/ec2" rel="noopener" target="_blank">Amazon Elastic Compute Cloud (Amazon EC2)</a>, and to create IAM roles to follow along with this post. The solution can be deployed in any AWS Region that has at least 3 Availability Zones, supports Security Lake, OpenSearch, and <a href="https://docs.aws.amazon.com/opensearch-service/latest/developerguide/ingestion.html" rel="noopener" target="_blank">OpenSearch Ingestion</a>.</p> 
<h2>Solution overview</h2> 
<p>The architecture diagram in Figure 1 shows the completed architecture of the solution.</p> 
<ol> 
 <li>The OpenSearch Service cluster is deployed within a virtual private cloud (VPC) across three Availability Zones for high availability.</li> 
 <li>The OpenSearch Service cluster ingests logs from Security Lake using an OpenSearch Ingestion pipeline.</li> 
 <li>The cluster is accessed by end users through a public-facing proxy hosted on an Amazon EC2 instance. 
  <ol> 
   <li>To reduce costs, the template doesn’t deploy a dead letter queue (DLQ) for the OpenSearch Ingestion pipeline. You can add one later if you want.</li> 
   <li>Instead of a public facing proxy, you can deploy a VPN to access your cluster.</li> 
  </ol> </li> 
 <li>Authentication to the cluster is managed with&nbsp;Amazon Cognito.</li> 
</ol> 
<p style="line-height: 1.25em;"></p>
<div class="wp-caption aligncenter" id="attachment_37081" style="width: 790px;">
 <img alt="Figure 1: Solution architecture" class="size-full wp-image-37081" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img1.jpg" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-37081">Figure 1: Solution architecture</p>
</div>
<p></p> 
<h2>Planning the deployment</h2> 
<p>This section will help you plan your OpenSearch service deployment, including what nodes you should choose, the amount of storage to allocate, and where to deploy the cluster.</p> 
<h3>Deciding instances for the OpenSearch Service master and data nodes</h3> 
<p>First, determine what instance type to use for the master and data nodes. If your workload generates less than 100 GB of Security Lake logs per day, we recommend using&nbsp;three m6g.large.search&nbsp;master nodes and&nbsp;three r6g.large.search&nbsp;data nodes. You can start small and scale up or scale out later. For more information about deciding the size and number of instances, see <a href="https://aws.amazon.com/blogs/big-data/get-started-with-amazon-opensearch-service-t-shirt-size-your-domain/" rel="noopener" target="_blank">Get started with Amazon OpenSearch Service</a>. Note the instance types that you have selected on a text editor because you will use this as an input for the <a href="https://aws.amazon.com/cloudformation" rel="noopener" target="_blank">AWS CloudFormation</a> template that you will deploy later.</p> 
<h3>Configuring storage</h3> 
<p>To optimize your storage costs, you need to plan your data strategy. In this architecture, Security Lake is used for long-term log storage. Because Security Lake uses <a href="https://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a>, you can optimize long-term storage costs. You can configure OpenSearch Service to ingest priority logs based on the recent data that you can use for near-real time detection and alerting. Your team can query logs in Security Lake using its <a href="https://docs.aws.amazon.com/documentdb/latest/developerguide/openSearch-zero-etl.html" rel="noopener" target="_blank">Zero-ETL integration with OpenSearch Service</a> to analyze older logs.</p> 
<p>Therefore, Security Lake should serve as your primary long-term log storage, with OpenSearch Service storing only the most recent logs.</p> 
<p>The number of days of logs in OpenSearch Service will depend on how many days’ worth of data you need to investigate at a given time. I recommend storing 15 days of data in OpenSearch Service. This allows you to react to and investigate the most immediate security events while optimizing storage costs for older logs.</p> 
<p>The next step is to determine the volume of logs generated by Security Lake.</p> 
<ol> 
 <li>Sign in to the Security Lake delegated administrator account.</li> 
 <li>Go to the AWS Management Console for Security Lake. Choose <strong>Usage</strong> in the navigation pane.</li> 
 <li>On the Usage screen, select <strong>Last 30 days</strong> as the range of usage.</li> 
 <li>Add the total <strong>Actual usage</strong> for the last 30 days for the data sources that you intend to send to OpenSearch. If you have used Security Lake for less than 30 days, you can use the&nbsp;<strong>Total predicted usage&nbsp;per month</strong>. Divide this figure by 30 to get the daily data volume.</li> 
</ol> 
<p style="line-height: 1.25em;"></p>
<div class="wp-caption aligncenter" id="attachment_37082" style="width: 790px;">
 <img alt="Figure 2: Select range of usage" class="size-full wp-image-37082" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img2.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-37082">Figure 2: Select range of usage</p>
</div>
<p></p> 
<p>To determine the total storage needed, multiply the data generated by Security Lake per day by the retention period you chose, then by 1.1 to account for the indexes, then multiply that number by 1.15 for overhead storage. For more information about calculating storage, see <a href="https://aws.amazon.com/blogs/big-data/get-started-with-amazon-opensearch-service-t-shirt-size-your-domain/" rel="noopener" target="_blank">Get started with Amazon OpenSearch Service</a>.</p> 
<p>To determine the amount of&nbsp;<a href="https://aws.amazon.com/ebs" rel="noopener" target="_blank">Amazon Elastic Block Store (Amazon EBS)</a>&nbsp;storage that you need per node, take the total amount of storage and divide it by the number of nodes that you have. Round that number up to the nearest whole number. You can increase the amount of storage after deployment when you have a better understanding of your workload. Make a note of this number in a text editor because you’ll use it as an input in the CloudFormation template later.</p> 
<p><strong>Example 1:</strong> 10 GB of Security Lake logs generated per day, stored for 30 days in OpenSearch Service in three nodes</p> 
<ul> 
 <li>10 GB of Security Lake logs stored for 30 days = 10 GB * 30 = 300 GB</li> 
 <li>Account for additional space for indexes and overhead space = 300 GB * 1.1 * 1.15 = 379.5 GB</li> 
 <li>Divide the storage required across three nodes, rounded up = 379.5/3 ≈ 127 GB per node</li> 
 <li>You would need 127 GB per node in OpenSearch Service</li> 
</ul> 
<p><strong>Example 2:</strong> 200 GB of Security Lake logs generated per day, stored for 15 days in OpenSearch Service across six nodes</p> 
<ul> 
 <li>200 GB of Security Lake logs stored for 15 days = 200 GB * 15 = 3000 GB</li> 
 <li>Account for additional space for indexes and overhead space = 3000 GB * 1.1 * 1.15 = 3795 GB</li> 
 <li>Divide the storage required across three nodes, rounded up = 3795/6 ≈ 633 GB per node</li> 
 <li>You would need 633 GB per node in OpenSearch Service</li> 
</ul> 
<h2>Where to deploy the cluster?</h2> 
<p>If you have an&nbsp;<a href="https://aws.amazon.com/controltower" rel="noopener" target="_blank">AWS Control Tower</a>&nbsp;deployment or have a deployment modelled after the&nbsp;<a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/welcome.html" rel="noopener" target="_blank">AWS Security Reference Architecture (AWS SRA)</a>, Security Lake should be deployed in the&nbsp;<em>Log Archive</em>&nbsp;account. Because security best practices recommend that the&nbsp;Log Archive&nbsp;account should not be frequently accessed, the OpenSearch Service cluster should be deployed into your&nbsp;<em>Audit</em> account or <em>Security Tooling</em> account.</p> 
<p>You need to deploy your Security Lake subscriber in the same Region as your Security Lake roll-up Region. If you have more than one roll-up Region, choose the Region that collects logs from the Regions you want to monitor.</p> 
<p>Your cluster needs to be deployed in the same Region as your Security Lake subscriber be able to access data.</p> 
<h2>Setting up the Security Lake subscriber</h2> 
<p>Before deploying the solution, create a Security Lake subscriber in your Security Lake roll-up Region so that OpenSearch Service can access data from Amazon Security Lake.</p> 
<ol> 
 <li>Access the Security Lake console in your Log Archive account.</li> 
 <li>Choose <strong>Subscribers </strong>in the navigation pane.</li> 
 <li>Choose <strong>Create subscriber</strong>.</li> 
 <li>On the <strong>Create subscriber </strong>page, enter a name, such as <code style="color: #000000;">OpenSearch-subscriber</code>.</li> 
 <li>Under <strong>Data Access, </strong>select Under <strong>S3 notification type, </strong>select <strong>SQS queue</strong>.</li> 
 <li>Under <strong>Subscriber credentials, </strong>enter the AWS account ID for the account you plan to deploy the OpenSearch cluster to, which should be your Security Tooling</li> 
 <li>Enter <code style="color: #000000;">OpenSearchIngestion-<span style="color: #ff0000; font-style: italic;">&lt;AWS account ID&gt;</span></code> under <strong>External ID</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37083" style="width: 750px;">
   <img alt="Figure 3: Configuring the Security Lake subscriber" class="size-full wp-image-37083" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img3.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-37083">Figure 3: Configuring the Security Lake subscriber</p>
  </div><p></p> </li> 
 <li>Leave <strong>All log and event sources </strong>selected and choose <strong>Create.</strong></li> 
</ol> 
<p>After the subscriber has been created, you will need to collect information to facilitate the deployment.</p> 
<p><strong>To gather necessary information:</strong></p> 
<ol> 
 <li>Select the subscriber that you just created.</li> 
 <li>Derive the S3 bucket name from the <strong>S3 bucket ARN </strong>and store it in a text editor. The Amazon Resource Name (ARN) is formatted as <code style="color: #000000;">arn:aws:s3:::<span style="color: #ff0000; font-style: italic;">&lt;bucket name&gt;</span></code>. The bucket name should look like <code style="color: #000000;">aws-security-data-lake-<span style="color: #ff0000; font-style: italic;">&lt;region&gt;</span>-xxxxx</code>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37084" style="width: 850px;">
   <img alt="Figure 4: Derive the S3 bucket name from the Subscriber details page" class="size-full wp-image-37084" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img4.png" style="border: 1px solid #bebebe;" width="840" />
   <p class="wp-caption-text" id="caption-attachment-37084">Figure 4: Derive the S3 bucket name from the Subscriber details page</p>
  </div><p></p> </li> 
 <li>Go to the <a href="https://aws.amazon.com/sqs" rel="noopener" target="_blank">Amazon Simple Queue Service (Amazon SQS)</a> console and select the SQS queue created as part of the Security Lake subscriber. It should look like <code style="color: #000000;">AmazonSecurityLake-xxxxxxxxx-Main-Queue</code>. Note the queue’s ARN and URL in your text editor. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37085" style="width: 750px;">
   <img alt="Figure 5: Relevant details from the SQS queue" class="size-full wp-image-37085" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img5.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-37085">Figure 5: Relevant details from the SQS queue</p>
  </div><p></p> </li> 
</ol> 
<h2>Deploy the solution</h2> 
<p>To deploy the solution in your Security Tooling account, use a CloudFormation template. This template deploys the OpenSearch Service cluster, OpenSearch Ingestion pipeline, and an <a href="https://aws.amazon.com/lambda" rel="noopener" target="_blank">AWS Lambda</a> function to initialize the cluster. You can find the repository containing all the resources <a href="https://github.com/aws-samples/ocsf-for-opensearch" rel="noopener" target="_blank">here</a>.</p> 
<p><strong>To download the necessary resources for deployment:</strong></p> 
<ol> 
 <li>In your Security Tooling account, make sure you are in the same Region as your Security Lake subscriber. Open AWS CloudShell by selecting the CloudShell icon next to the search bar.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38104" style="width: 612px;">
   <img alt="Figure 6: How to access the CloudShell terminal" class="size-full wp-image-38104" height="214" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/29/Figure6r.png" style="border: 1px solid #bebebe;" width="602" />
   <p class="wp-caption-text" id="caption-attachment-38104">Figure 6: How to access the CloudShell terminal</p>
  </div> </li> 
 <li>In the Cloudshell terminal, clone the <a href="https://github.com/aws-samples/ocsf-for-opensearch" rel="noopener" target="_blank">repository</a> with <code style="color: #000000;">git clone https://github.com/aws-samples/ocsf-for-opensearch.git</code></li> 
 <li>Change the working directory to the newly created repository by running <code style="color: #000000;">cd ocsf-for-opensearch</code></li> 
 <li>Run this command <code style="color: #000000;">sh deploy-script.sh</code>. The script will create the service linked roles, create an S3 bucket, and copy the deployment files to it.</li> 
 <li>Wait for the script to complete. The final line should say <code style="color: #000000;">Setup complete. Asset bucket name: os-stack-deploy-assets-xxxxxxx</code>. Record the bucket name in a text editor.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38105" style="width: 612px;">
   <img alt="Figure 7: Deriving the Asset Bucket Name after the script completes" class="size-full wp-image-38105" height="79" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/29/Figure7r.png" style="border: 1px solid #bebebe;" width="602" />
   <p class="wp-caption-text" id="caption-attachment-38105">Figure 7: Deriving the Asset Bucket Name after the script completes</p>
  </div> </li> 
</ol> 
<p><strong>To deploy the OpenSearch cluster:</strong></p> 
<ol> 
 <li>To download the CloudFormation template that builds the OpenSearch service cluster, right click on this <a href="https://raw.githubusercontent.com/aws-samples/ocsf-for-opensearch/main/deployment/cfn/quickstart-kickoff.json" rel="noopener" target="_blank">link</a> and select save link as.</li> 
 <li>Access the CloudFormation console. In the CloudFormation console, upload the CloudFormation template you just downloaded and select <strong>Next</strong>.</li> 
 <li>Enter a name for your stack. A name like <code style="color: #000000;">os-stack-&lt;day&gt;-&lt;month&gt;</code> can help you keep track of deployments.</li> 
 <li>Enter the name of the Asset Bucket that you saved earlier.</li> 
 <li>Enter the instance types and Amazon EBS volume size that you noted earlier.</li> 
 <li>Enter the IP address range that you want to allow to access the proxy’s security group. You should limit this to your corporate IP range. You can set it as <code style="color: #000000;">0.0.0/0</code> if you want to expose it to the public internet.</li> 
 <li>Fill in the details of the Security Lake bucket and the subscriber Amazon SQS queue ARN, URL, and Region. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_38106" style="width: 612px;">
   <img alt="Figure 8: Add stack parameters" class="size-full wp-image-38106" height="764" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/29/Figure8r.png" style="border: 1px solid #bebebe;" width="602" />
   <p class="wp-caption-text" id="caption-attachment-38106">Figure 8: Add stack parameters</p>
  </div><p></p> </li> 
 <li>Under <strong>Stack failure options</strong>, select <strong>Preserve successfully provisioned resources</strong>. This will allow you to troubleshoot the stack deployment in the event of a failure. </li> 
 <li>Check the acknowledgements in the <strong>Capabilities</strong> section.</li> 
 <li>Choose <strong>Create stack</strong> to begin deploying the resources.</li> 
 <li>It will take 20–30 minutes to deploy the multiple nested templates. Wait for the main stack (not the nested ones) to achieve the <code style="color: #000000;">CREATE_COMPLETE</code> status before proceeding to the next step.</li> 
 <li>Go to the&nbsp;<strong>Outputs</strong> pane of the main CloudFormation stack. Save the <strong>DashboardsProxyURL</strong>, <strong>OpenSearchInitRoleARN</strong>, and <strong>PipelineRole</strong> values in a text editor to refer to later. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37087" style="width: 750px;">
   <img alt="Figure 9: The stacks in the CREATE_COMPLETE state with the outputs panel shown" class="size-full wp-image-37087" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img7.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-37087">Figure 9: The stacks in the CREATE_COMPLETE state with the outputs panel shown</p>
  </div><p></p> </li> 
 <li>Open the&nbsp;<strong>DashboardsProxyURL</strong> value in a new tab.<br /> 
  <blockquote>
   <p><strong>Note</strong>:&nbsp;Because the proxy relies on a self-signed certificate, you will get an insecure certificate warning. You can safely ignore this warning and proceed. For a production workload, you should issue a trusted private certificate from your internal public key infrastructure or use&nbsp;<a href="https://aws.amazon.com/private-ca/" rel="noopener" target="_blank">AWS Private Certificate Authority</a>.</p>
  </blockquote> </li> 
 <li>You will be presented with the Amazon Cognito sign-in page. Use <code style="color: #000000;">administrator</code> as the username.</li> 
 <li>Access Secrets Manager to find the password. Select the secret that was created as part of the stack. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37088" style="width: 750px;">
   <img alt="Figure 10: Retrieve the secret value" class="size-full wp-image-37088" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img8.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-37088">Figure 10: The Cognito password in Secrets Manager</p>
  </div><p></p> </li> 
 <li>Choose <strong>Retrieve secret value</strong> to get the password. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37089" style="width: 750px;">
   <img alt="Figure 11: Retrieve the secret value" class="size-full wp-image-37089" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img9.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-37089">Figure 11: Retrieve the secret value</p>
  </div><p></p> </li> 
 <li>After signing in, you will be prompted to change your password and will be redirected to the OpenSearch dashboard.</li> 
 <li>If you see a pop-up that states <strong>Start by adding your own data</strong>, select <strong>Explore on my own</strong>. On the next page, <strong>Introducing new OpenSearch Dashboards look &amp; feel</strong>, choose <strong>Dismiss</strong>.</li> 
 <li>If you see a pop-up that states <strong>Select your tenant</strong>, select <strong>Global</strong>, and then choose <strong>Confirm</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37090" style="width: 750px;">
   <img alt="Figure 12: Select and confirm your tenant" class="size-full wp-image-37090" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img10.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-37090">Figure 12: Select and confirm your tenant</p>
  </div><p></p> </li> 
</ol> 
<p><strong>To initialize the OpenSearch cluster:</strong></p> 
<ol> 
 <li>Choose the menu icon (three stacked horizontal lines) on the top left and select <strong>Security</strong> under the <strong>Management</strong> section. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37091" style="width: 513px;">
   <img alt="Figure 13: Navigating to the Security page in the OpenSearch console" class="size-full wp-image-37091" height="1199" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img11.png" style="border: 1px solid #bebebe;" width="503" />
   <p class="wp-caption-text" id="caption-attachment-37091">Figure 13: Navigating to the Security page in the OpenSearch console</p>
  </div><p></p> </li> 
 <li>Select <strong>Roles</strong>. On the <strong>Roles</strong> page, search for the <code style="color: #000000;">all_access</code> role and select it.</li> 
 <li>Select <strong>Mapped users</strong>, and then select <strong>Manage mapping</strong>.</li> 
 <li>On the <strong>Map user</strong> screen, choose <strong>Add another backend role</strong>. Paste the value for the <strong>OpenSearchInitRoleARN</strong> from the list of CloudFormation outputs. Choose <strong>Map</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37092" style="width: 750px;">
   <img alt="Figure 14: Mapping the role on the Security page in the OpenSearch console" class="size-full wp-image-37092" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img12.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-37092">Figure 14: Mapping the role on the Security page in the OpenSearch console</p>
  </div><p></p> </li> 
 <li>Leave this tab open and return to the AWS Management console. Go to the <strong>AWS Lambda</strong> console and select the function named <strong>xxxxxx-OS_INIT.</strong></li> 
 <li>In the function screen, choose <strong>Test</strong>, and then <strong>Create new test event</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37093" style="width: 750px;">
   <img alt="Figure 15: Creating the test event in the Lambda console" class="size-full wp-image-37093" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img13.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-37093">Figure 15: Creating the test event in the Lambda console</p>
  </div><p></p> </li> 
 <li>Choose <strong>Invoke</strong>. The function should run for about 30 seconds. The execution results should show the component templates that have been created. This Lambda function creates the component and index templates to ingest <a href="https://github.com/ocsf/" rel="noopener" target="_blank">Open Cybersecurity Framework</a> (OCSF) formatted data, a set of indices and aliases that correspond with the OCSF classes generated by Security Lake, and a rollover policy that will rollover the index daily or if it becomes larger than 40 GB. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37094" style="width: 750px;">
   <img alt="Figure 16: Invoking the Lambda function in the Lambda console" class="size-full wp-image-37094" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img14.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-37094">Figure 16: Invoking the Lambda function in the Lambda console</p>
  </div><p></p> </li> 
</ol> 
<p><strong>To set up the pipeline</strong></p> 
<ol> 
 <li>Return to the <strong>Map user</strong> page on the OpenSearch console.</li> 
 <li>Choose <strong>Add another backend role</strong>. Paste the value of the <strong>PipelineRole </strong>from the CloudFormation template output. This will allow the OpenSearch Ingestion to write to the cluster. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37095" style="width: 750px;">
   <img alt="Figure 17: Mapping the OpenSearch Ingestion role" class="size-full wp-image-37095" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img15.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-37095">Figure 17: Mapping the OpenSearch Ingestion role</p>
  </div><p></p> </li> 
 <li>Access the Amazon S3 console in the Log Archive account where Security Lake is hosted.</li> 
 <li>Select the Security Lake bucket in your roll-up Region. It should look like <code style="color: #000000;">aws-security-data-lake-region-xxxxxxxxxx</code>.</li> 
 <li>Choose <strong>Permissions</strong>, then <strong>Edit</strong> under <strong>Bucket policy</strong>.</li> 
 <li>Add this policy to the end of the existing bucket policy. Replace the <code style="color: #000000;">Principal</code> with the ARN of the <code style="color: #000000;">PipelineRole</code> and the name of your Security Lake bucket in the <code style="color: #000000;">Resource</code> section. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">{
            "Sid": "Cross Account Permissions",
            "Effect": "Allow",
            "Principal": {
                "AWS": "<span style="color: #ff0000; font-style: italic;">&lt;Pipeline role ARN&gt;</span>"
            },
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::<span style="color: #ff0000; font-style: italic;">&lt;Security Lake bucket name&gt;</span>/*",
                "arn:aws:s3:::<span style="color: #ff0000; font-style: italic;">&lt;Security Lake bucket name&gt;</span>"
            ]
        }</code></pre> 
  </div> <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37096" style="width: 750px;">
   <img alt="Figure 18: The modified S3 bucket access policy" class="size-full wp-image-37096" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img16.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-37096">Figure 18: The modified S3 bucket access policy</p>
  </div><p></p> </li> 
 <li>Choose <strong>Save changes.</strong></li> 
</ol> 
<p><strong>To upload the index patterns and dashboards</strong></p> 
<ol> 
 <li>Download the <code style="color: #000000;">Security-lake-objects.ndjson</code> file by right-clicking on this <a href="https://raw.githubusercontent.com/aws-samples/ocsf-for-opensearch/main/assets/OCSF_objects.ndjson" rel="noopener" target="_blank">link</a> and selecting <strong>Save link as</strong>.</li> 
 <li>Access the <strong>Dashboards Management</strong> page through the navigation menu.</li> 
 <li>Choose <strong>Saved objects</strong> in the navigation pane.</li> 
 <li>On the <strong>Saved Objects page</strong>, choose <strong>Import</strong> on the right side of the screen. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37097" style="width: 750px;">
   <img alt="Figure 19: Import saved objects" class="size-full wp-image-37097" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img17.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-37097">Figure 19: Import saved objects</p>
  </div><p></p> </li> 
 <li>Choose <strong>Import</strong> and select the <code style="color: #000000;">Security-lake-objects.ndjson</code> file that you downloaded previously.</li> 
 <li>Leave <strong>Create new objects with unique IDs</strong> selected and choose <strong>Import</strong>.</li> 
 <li>You can now view the ingested logs on the <strong>Discover</strong> page and visualizations on the <strong>Dashboards</strong> page, which you can find on the navigation bar. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_37098" style="width: 750px;">
   <img alt="Figure 20: The Discover page displaying ingested logs" class="size-full wp-image-37098" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img18.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-37098">Figure 20: The Discover page displaying ingested logs</p>
  </div><p></p> </li> 
</ol> 
<h2>Clean up</h2> 
<p>To avoid unwanted charges, delete the main CloudFormation template, named <code style="color: #000000;">os-stack-<span style="color: #ff0000; font-style: italic;">&lt;day&gt;</span>-<span style="color: #ff0000; font-style: italic;">&lt;month&gt;</span></code> (not the nested stacks).</p> 
<div class="wp-caption aligncenter" id="attachment_37099" style="width: 790px;">
 <img alt="Figure 21: Select the main stack in the CloudFormation console" class="size-full wp-image-37099" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img19.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-37099">Figure 21: Select the main stack in the CloudFormation console</p>
</div> 
<p>Delete the S3 bucket that contains the deployment assets. </p> 
<p>Modify the Security Lake bucket policy in the logging account to remove the section you added that trusted the <code style="color: #000000;">PipelineRole</code>. Be careful not to modify the rest of the policy because it could impact the functioning of Security Lake and other subscribers.</p> 
<div class="wp-caption aligncenter" id="attachment_37100" style="width: 790px;">
 <img alt="Figure 22: The S3 bucket policy with the relevant sections that needed to be deleted" class="size-full wp-image-37100" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/01/14/img20.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-37100">Figure 22: The S3 bucket policy with the relevant sections that needed to be deleted</p>
</div> 
<h2>Conclusion</h2> 
<p>In this post, you learned how to plan an OpenSearch deployment with Amazon OpenSearch Service to ingest logs from Amazon Security Lake. With this solution, you’re able to aggregate and manage logs with Security Lake and visualize and monitor those logs with OpenSearch Service. After deployment, monitor the OpenSearch Service metrics to determine if you need to scale this up or out for improved performance. In part 2, I will show you how to set up the Security Analytics detector to generate alerts to security findings in near-real time.</p> 
<p>If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.<br />&nbsp;</p> 
<footer> 
 <div class="blog-author-box">
  <img alt="Kevin Low" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/10/25/Kevin_Low.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Kevin Low</span>
  <br />Kevin is a Security Solutions Architect at AWS who helps the largest customers across ASEAN build securely. He specializes in threat detection and incident response and is passionate about integrating resilience and security. Outside of work, he loves spending time with his wife and dog, a poodle called Noodle.
 </div> 
</footer>
