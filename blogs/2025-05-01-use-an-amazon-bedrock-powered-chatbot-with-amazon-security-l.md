---
title: "Use an Amazon Bedrock powered chatbot with Amazon Security Lake to help investigate incidents"
url: "https://aws.amazon.com/blogs/security/use-an-amazon-bedrock-powered-chatbot-with-amazon-security-lake-to-help-investigate-incidents/"
date: "Thu, 01 May 2025 14:46:34 +0000"
author: "Madhunika Reddy Mikkili"
feed_url: "https://aws.amazon.com/blogs/security/tag/amazon-security-lake/feed/"
---
<p>In <a href="https://aws.amazon.com/blogs/security/generate-ai-powered-insights-for-amazon-security-lake-using-amazon-sagemaker-studio-and-amazon-bedrock/" rel="noopener" target="_blank">part 2</a> of this series, we showed you how to use <a href="https://aws.amazon.com/sagemaker-ai/studio/" rel="noopener" target="_blank">Amazon SageMaker Studio</a> notebooks with natural language input to assist with threat hunting. This is done by using SageMaker Studio to automatically generate and run SQL queries on <a href="https://aws.amazon.com/athena/" rel="noopener" target="_blank">Amazon Athena</a> with <a href="https://aws.amazon.com/bedrock/" rel="noopener" target="_blank">Amazon Bedrock</a> and <a href="https://aws.amazon.com/security-lake/" rel="noopener" target="_blank">Amazon Security Lake</a>. The Security Lake service team and the <a href="https://github.com/ocsf" rel="noopener" target="_blank">Open Cybersecurity Schema Framework</a> (OCSF) community continue to add <a href="https://docs.aws.amazon.com/security-lake/latest/userguide/source-management.html" rel="noopener" target="_blank">additional log sources</a> and <a href="https://github.com/ocsf/examples" rel="noopener" target="_blank">OCSF mappings</a> to enable Security Lake to provide a consolidated source for customers to conduct security investigation.</p> 
<p>Because security logging data sources continually grow, organizations need to provide a mechanism for their security teams to understand and query those data sources. You might have existing investigation and response playbooks that your security teams need to be well-versed in and know when to use. It can take security teams an extended period of time to onboard and understand the available security data sources and playbooks and how to efficiently use them to reduce the <a href="https://docs.aws.amazon.com/whitepapers/latest/aws-security-incident-response-guide/mean-time-to-respond.html" rel="noopener" target="_blank">mean time to respond</a>.</p> 
<p>In this post, we show you how to extend the functionality from the <a href="https://aws.amazon.com/blogs/security/generate-ai-powered-insights-for-amazon-security-lake-using-amazon-sagemaker-studio-and-amazon-bedrock/" rel="noopener" target="_blank">previous post</a>. You will learn how to deploy a security chatbot with a graphical user interface (GUI) and a serverless backend powered by an Amazon Bedrock agent that incorporates existing playbooks to investigate or respond to a security event. The chatbot demonstrates purpose-built Amazon Bedrock agents that help address security concerns depending on the user’s natural language input. The solution has a single GUI that provides a direct interface with the Amazon Bedrock agent to create and invoke SQL queries or provide recommendations for internal incident response playbooks to investigate or respond to possible security events.</p> 
<h2 id="security-chatbot-sample-solution-overview">Security chatbot sample solution overview</h2> 
<div class="wp-caption aligncenter" id="attachment_38115" style="width: 946px;">
 <img alt="Figure 1: Security chatbot sample solution architecture diagram" class="size-full wp-image-38115" height="744" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/29/security_chat_arch_diagram.png" width="936" />
 <p class="wp-caption-text" id="caption-attachment-38115">Figure 1: Security chatbot sample solution architecture diagram</p>
</div> 
<p>Application flow as shown in Figure 1:</p> 
<ol type="1"> 
 <li>User submits a query through the React UI.<br /> 
  <blockquote>
   <p><strong>Note</strong>: The React UI used in this solution doesn’t have authentication built in. It’s recommended that you add authentication capabilities that follow your organization’s security requirements. You can add <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/authenticate-react-app-users-cognito-amplify-ui.html" rel="noopener" target="_blank">authentication capabilities by using Amazon Cognito and AWS Amplify UI</a>.</p>
  </blockquote> </li> 
 <li>The user’s query is sent to an <a href="https://aws.amazon.com/api-gateway/" rel="noopener" target="_blank">Amazon API Gateway</a> REST API, which invokes the <code style="color: #000000;">Invoke Agent</code> <a href="https://aws.amazon.com/lambda" rel="noopener" target="_blank">AWS Lambda</a> function.</li> 
 <li>The Lambda function invokes the Amazon Bedrock agent with the user’s query.</li> 
 <li>The Amazon Bedrock agent (using Anthropic’s Claude 3 Sonnet) processes the query and decides between retrieving information from the playbooks or by querying Security Lake using <a href="https://aws.amazon.com/athena" rel="noopener" target="_blank">Amazon Athena</a>.</li> 
</ol> 
<p>For playbook knowledge base queries:</p> 
<ol start="5" type="1"> 
 <li>The Amazon Bedrock agent queries the playbooks knowledge base and retrieves relevant results.</li> 
</ol> 
<p>For Security Lake data queries:</p> 
<ol start="6" type="1"> 
 <li>The Amazon Bedrock agent queries the schema knowledge base and retrieves the Security Lake table schemas to create an SQL query.</li> 
 <li>The Amazon Bedrock agent invokes the SQL query action from the Amazon Bedrock action group, passing the SQL query as a parameter.</li> 
 <li>The action group invokes the <code style="color: #000000;">Execute SQL on Athena</code> Lambda function, which executes the query on Athena and returns the results to the Amazon Bedrock agent.</li> 
</ol> 
<p>After retrieving results from the knowledge base or action group:</p> 
<ol start="9" type="1"> 
 <li>The Amazon Bedrock agent uses the retrieved information to formulate the final response and sends it back to the <code style="color: #000000;">Invoke Agent</code> Lambda function.</li> 
 <li>The Lambda function sends the response back to the client using an API Gateway WebSocket API.</li> 
 <li>API Gateway delivers the response to the React UI using a WebSocket connection to the client.</li> 
 <li>The agent’s response is displayed to the user in the chat interface.</li> 
</ol> 
<h3 id="prerequisites">Prerequisites</h3> 
<p>Before deploying the sample solution, complete the following prerequisites:</p> 
<ol type="1"> 
 <li><a href="https://docs.aws.amazon.com/security-lake/latest/userguide/getting-started.html" rel="noopener" target="_blank">Enable Security Lake</a> in your organization in <a href="https://aws.amazon.com/organizations" rel="noopener" target="_blank">AWS Organizations</a> and specify a <a href="https://docs.aws.amazon.com/security-lake/latest/userguide/multi-account-management.html" rel="noopener" target="_blank">delegated administrator</a> account to manage the Security Lake configuration for all member accounts in your organization. Configure Security Lake with the appropriate log sources: <a href="https://aws.amazon.com/vpc/" rel="noopener" target="_blank">Amazon Virtual Private Cloud (Amazon VPC)</a> Flow Logs, <a href="https://aws.amazon.com/security-hub/" rel="noopener" target="_blank">AWS Security Hub</a>, <a href="https://aws.amazon.com/cloudtrail/" rel="noopener" target="_blank">AWS CloudTrail</a>, and <a href="https://aws.amazon.com/route53/" rel="noopener" target="_blank">Amazon Route53</a>.</li> 
 <li><a href="https://docs.aws.amazon.com/security-lake/latest/userguide/subscriber-query-access.html" rel="noopener" target="_blank">Create subscriber query access</a> from the source Security Lake AWS account to the subscriber AWS account.</li> 
 <li>Accept a <a href="https://docs.aws.amazon.com/lake-formation/latest/dg/accepting-ram-invite.html" rel="noopener" target="_blank">resource share request</a> in the subscriber AWS account in <a href="https://aws.amazon.com/ram/" rel="noopener" target="_blank">AWS Resource Access Manager (AWS RAM)</a>.</li> 
 <li>Create a database link in <a href="https://aws.amazon.com/lake-formation/" rel="noopener" target="_blank">AWS Lake Formation</a> in the subscriber AWS account and grant access for the <a href="https://aws.amazon.com/athena" rel="noopener" target="_blank">Athena</a> tables in the Security Lake AWS account.</li> 
 <li>Grant Anthropic’s <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html#add-model-access" rel="noopener" target="_blank">Claude v3 model access</a> for Amazon Bedrock in the AWS subscriber account where you will deploy the solution. If you try to use a model before you enable it in your AWS account, you will get an error message.</li> 
</ol> 
<p>With the prerequisites in place, the sample solution architecture provisions the following resources:</p> 
<ol type="1"> 
 <li><a href="https://aws.amazon.com/cloudfront" rel="noopener" target="_blank">Amazon CloudFront</a> with an <a href="https://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> origin.</li> 
 <li>An Amazon S3 static website for the chatbot UI.</li> 
 <li>An API Gateway to call a Lambda function.</li> 
 <li>A Lambda function to invoke the Amazon Bedrock agent.</li> 
 <li>An Amazon Bedrock agent with a knowledge base. 
  <ol type="a"> 
   <li>An Amazon Bedrock agent action group to generate and invoke SQL queries on Athena. 
    <ol type="i"> 
     <li>An Amazon Bedrock knowledge base to reference example Athena table schemas in Security Lake. Although the Amazon Bedrock agent can get rows directly from the Athena table, providing example table schemas improves SQL query generation accuracy for table columns in Security Lake.</li> 
     <li>An Amazon Bedrock knowledge base to reference existing incident response playbooks. By incorporating this knowledge base, the Amazon Bedrock agent can suggest actions for investigation or response based on existing playbooks that have already been approved by your organization.</li> 
    </ol> </li> 
  </ol> </li> 
</ol> 
<h3 id="cost">Cost</h3> 
<p>Before deploying the sample solution and walking through this post, it’s important to understand the cost of the AWS services being used. The cost will largely depend on the amount of data you interact with in Amazon Bedrock and by querying Security Lake with Athena.</p> 
<ol type="1"> 
 <li>Security Lake costs are determined by the volume of log and event data ingested from AWS services. Security Lake orchestrates other AWS services on your behalf, which incur separate charges. You can find more information on pricing for the respective services: <a href="https://aws.amazon.com/s3/pricing/" rel="noopener" target="_blank">Amazon S3</a>, <a href="https://aws.amazon.com/glue/pricing/" rel="noopener" target="_blank">AWS Glue</a>,&nbsp;<a href="https://aws.amazon.com/eventbridge/pricing/" rel="noopener" target="_blank">Amazon EventBridge</a>,&nbsp;<a href="https://aws.amazon.com/lambda/pricing/" rel="noopener" target="_blank">AWS Lambda</a>,&nbsp;<a href="https://aws.amazon.com/sqs/pricing/" rel="noopener" target="_blank">Amazon Simple Query Service (Amazon SQS)</a>, and&nbsp;<a href="https://aws.amazon.com/sns/pricing/" rel="noopener" target="_blank">Amazon Simple Notification Service (Amazon SNS)</a>.</li> 
 <li>Amazon Bedrock on-demand pricing is based on the selected large language model (LLM) and the number of input and output tokens. A token is comprised of a few characters and refers to the basic unit of text that a model learns to understand the user input and prompts. For more details, see <a href="https://aws.amazon.com/bedrock/pricing/" rel="noopener" target="_blank">Amazon Bedrock pricing</a>.</li> 
 <li>The SQL queries generated by Amazon Bedrock are invoked using Athena. Athena cost is based on the amount of data scanned within Security Lake for that query. For more details, see <a href="https://aws.amazon.com/athena/pricing/" rel="noopener" target="_blank">Athena pricing</a>.</li> 
</ol> 
<h2 id="deploy-the-sample-chatbot">Deploy the sample chatbot</h2> 
<p>You can deploy the sample solution by using <a href="https://aws.amazon.com/cdk/" rel="noopener" target="_blank">AWS Cloud Development Kit (AWS CDK)</a>. For instructions and more information on using the AWS CDK, see <a href="https://aws.amazon.com/getting-started/guides/setup-cdk/" rel="noopener" target="_blank">Get Started with AWS CDK</a>.</p> 
<ol> 
 <li>Clone the <a href="https://github.com/aws-samples/sample-generative-ai-chatbot-for-amazon-security-lake" rel="noopener" target="_blank">sample-generative-ai-chatbot-for-amazon-security-lake</a> repository.</li> 
 <li>Navigate to the project’s root folder.</li> 
 <li>Install the project dependencies.</li> 
 <li>Build and deploy the app using the following commands: 
  <div class="hide-language"> 
   <pre><code class="lang-text">npm install -g aws-cdk
npm install 
cdk synth
</code></pre> 
  </div> </li> 
 <li>Run the following commands in your terminal while signed in to your subscriber AWS account. Replace <code style="color: #ff0000;"><em>&lt;INSERT_AWS_ACCOUNT&gt;</em></code> with your account number and replace <code style="color: #ff0000;"><em>&lt;INSERT_REGION&gt;</em></code> with the AWS Region that you want the solution deployed to. 
  <div class="hide-language"> 
   <pre><code class="lang-text">cdk bootstrap aws://<span style="color: #ff0000;"><em>&lt;INSERT_AWS_ACCOUNT&gt;</em></span>/<span style="color: #ff0000;"><em>&lt;INSERT_REGION&gt;</em></span>
cdk deploy –all
</code></pre> 
  </div> </li> 
</ol> 
<p>As part of the CDK deployment, there is an Output value for the React Application URL (FrontendAppStack.ReactAppUrl). You will use this value to interact with the GenAI application. Wait up to 5 mins for the URL to be live. </p> 
<h2 id="post-deployment-configuration-steps">Post-deployment configuration steps</h2> 
<p>Now that you’ve deployed the solution, you need to add permissions to allow the Lambda function’s <a href="https://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> role and Amazon Bedrock to interact with your Security Lake data.</p> 
<h3 id="grant-permission-to-the-security-lake-database">Grant permission to the Security Lake database</h3> 
<ol type="1"> 
 <li>Copy the Lambda’s role ARN from the “BedrockAppStack” CloudFormation stack. The resource in the stack is named “athenaAgentSecurityLakeActionGroupLambdaServiceRole********”. </li> 
 <li>Go to the <strong>Lake Formation</strong> console.</li> 
 <li>Select the <strong>amazon_security_lake_glue_db_<code style="color: #ff0000;"><em>&lt;YOUR-REGION&gt;</em></code></strong> database. For example, if your Security Lake is in us-east-1, the value would be <code style="color: #000000;">amazon_security_lake_glue_db_us_east_1</code></li> 
 <li>For <strong>Actions</strong>, select <strong>Grant</strong>.</li> 
 <li>In <strong>Grant Data Permissions</strong>, select <strong>SAML Users and Groups</strong>.</li> 
 <li>Paste the Lambda function IAM role ARN from Step 1.</li> 
 <li>In <strong>Database Permissions</strong>, select <strong>Describe</strong>, and then choose <strong>Grant</strong>.</li> 
</ol> 
<h3 id="grant-permission-to-security-lake-tables">Grant permission to Security Lake tables</h3> 
<p>You must repeat the following steps for each source configured within Security Lake. For example, if you have four sources configured within Security Lake, you must grant permissions for the Lambda function IAM role to each table. If you have multiple sources that are in separate Regions and you don’t have a rollup Region configured in Security Lake, you must repeat the steps for each source in each Region.</p> 
<p>The following example grants permissions to the Security Hub table within Security Lake. For more information about granting table permissions, see the <a href="https://docs.aws.amazon.com/lake-formation/latest/dg/granting-table-permissions.html#table-start-grant" rel="noopener" target="_blank">AWS Lake Formation user guide</a>.</p> 
<ol type="1"> 
 <li>Copy the Lambda’s role ARN from the “BedrockAppStack” CloudFormation stack. The resource in the stack is named as “athenaAgentSecurityLakeActionGroupLambdaServiceRole********”.</li> 
 <li>Go to the <strong>Lake Formation</strong> console.</li> 
 <li>Select the <code style="color: #000000;">amazon_security_lake_glue_db_<span style="color: #ff0000;"><em>&lt;YOUR-REGION&gt;</em></span></code> database.<br /> For example, if your Security Lake database is in us-east-1, the value would be <strong>amazon_security_lake_glue_db_us_east-1</strong></li> 
 <li>Choose <strong>View Tables</strong>.</li> 
 <li>Select the <code style="color: #000000;">amazon_security_lake_table_<span style="color: #ff0000;"><em>&lt;YOUR-REGION&gt;</em></span>_sh_findings_1_0</code> table.<br /> For example, if your Security Lake table is in us-east-1, the value would be <strong>amazon_security_lake_table_us_east_1_sh_findings_1_0</strong> <p></p> 
  <blockquote>
   <p><strong>Note:</strong> Each table must be granted access individually. Selecting <strong>All Tables</strong> won’t grant the access needed to query Security Lake.</p>
  </blockquote> </li> 
 <li>For <strong>Actions</strong>, select <strong>Grant</strong>.</li> 
 <li>In <strong>Grant Data Permissions</strong>, select <strong>SAML Users and Groups</strong>.</li> 
 <li>Paste the Lambda function IAM role ARN from Step 1.</li> 
 <li>In <strong>Table Permissions</strong>, select <strong>Describe</strong>, and then <strong>Grant</strong>.</li> 
</ol> 
<h3>Sync data sources</h3> 
<p>After you deploy the infrastructure, you need to sync the data sources in the Amazon Bedrock knowledge bases so that the data in Amazon S3 can be vectorized and made available in <a href="https://aws.amazon.com/opensearch-service/features/serverless/" rel="noopener" target="_blank">Amazon OpenSearch Serverless</a>, which is the service used as a vector source by the knowledge bases in this solution.</p> 
<ul> 
 <li>In the <a href="https://console.aws.amazon.com/bedrock/" rel="noopener" target="_blank">Amazon Bedrock console</a>, select <strong>Knowledge base</strong> and find the two Amazon Bedrock knowledge bases deployed in this solution: <code style="color: #000000;">gen-ai-sec-lake-table-schema</code> and <code style="color: #000000;">gen-ai-sec-lake-runbooks</code>. Navigate to each knowledge base and its data source. Then choose <strong>Sync</strong> for each data source.</li> 
</ul> 
<h3 id="get-the-cloudfront-distribution-url">Get the CloudFront distribution URL</h3> 
<p>As part of the sample solution, the chatbot uses an externally available CloudFront distribution URL. It’s recommended that you add appropriate security controls that align to your organization’s security requirements to the sample solution. For example, you might want to <a href="https://github.com/awslabs/cognito-at-edge" rel="noopener" target="_blank">add authentication to CloudFront using Amazon Cognito and Lambda@Edge</a> to help prevent unauthorized users from accessing this chatbot. You can also <a href="https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/SecurityAndPrivateContent.html" rel="noopener" target="_blank">configure secure access and restrict access to the content</a>.</p> 
<ol type="1"> 
 <li>Navigate to <a href="https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/" rel="noopener" target="_blank">CloudFormation</a> in the console.</li> 
 <li>In the <strong>Stacks</strong> section, select the <strong>FrontendAppStack</strong>.</li> 
 <li>Select the <strong>Outputs</strong> tab.</li> 
 <li>Copy the value ReactAppUrl. </li> 
</ol> 
<h2 id="investigate-with-your-security-chatbot">Investigate with your security chatbot</h2> 
<p>Now that you’ve deployed the sample solution and configured the appropriate permissions, you’re ready to use natural language input to generate and invoke SQL queries and to recommend internal incident response playbooks.</p> 
<h3 id="generate-and-invoke-sql-queries">Generate and invoke SQL queries</h3> 
<p>Although most security professionals already know how to create and invoke SQL queries, time is extremely important during a possible security event. Adding natural-language-to-SQL capability to the security agent can help reduce your mean time to respond, which can reduce the overall impact of a security event. Furthermore, you can use these same techniques for threat hunting in your AWS environment.</p> 
<p><strong>Question 1</strong>: Which AWS accounts have the most security findings for EC2 instances in the last 7 days?</p> 
<p><strong>Answer 1</strong>: The query results show the top two AWS accounts with the most security findings related to EC2 instances in the last 7 days. The account with ID XXXXXX10 had the highest number of findings at 252, followed by account XXXXXX28 with 197 findings.</p> 
<p>The query covered the key requirements, so I provided the final answer summarizing the top accounts and their finding counts.</p> 
<p>The question and response are shown in Figure 2.</p> 
<div class="wp-caption aligncenter" id="attachment_38149" style="width: 889px;">
 <img alt="Figure 2: Response for most EC2 security findings" class="size-full wp-image-38149" height="181" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/30/Figure_2.jpg" style="border: 1px solid #bebebe;" width="879" />
 <p class="wp-caption-text" id="caption-attachment-38149">Figure 2: Response for most EC2 security findings</p>
</div> 
<p>The more context you provide to the chatbot, the better the query that will be generated. Providing a time window also gives a smaller data set to review and give quicker results.</p> 
<p><strong>Question 2</strong>: In those AWS accounts, what were the top EC2 security finding types?</p> 
<p><strong>Answer 2</strong>: Based on the query results, the top EC2 security finding types in the AWS accounts with the most EC2 instance findings in the last 7 days were:</p> 
<p>For account XXXXXX10:</p> 
<ol type="1"> 
 <li>“Compliance Finding: Update” with 252 findings</li> 
</ol> 
<blockquote>
 <p>For account XXXXXX28:</p>
</blockquote> 
<ol type="1"> 
 <li>“Compliance Finding Update” with 166 findings</li> 
 <li>“Compliance Finding Create” with 24 findings</li> 
 <li>“Vulnerability Finding: Update” with 2 findings</li> 
 <li>“Vulnerability Finding: Create” with 2 findings.</li> 
</ol> 
<p>The question and response are shown in Figure 3.</p> 
<div class="wp-caption aligncenter" id="attachment_38150" style="width: 850px;">
 <img alt="Figure 3: Response for top EC2 security finding types" class="size-full wp-image-38150" height="293" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/30/Figure_3.jpg" style="border: 1px solid #bebebe;" width="840" />
 <p class="wp-caption-text" id="caption-attachment-38150">Figure 3: Response for top EC2 security finding types</p>
</div> 
<p>The results show that the issues are primarily around Security Hub controls mapped to the <code style="color: #000000;">type_name</code> OCSF column. If you want to look at a different column, you can provide additional instructions or update the question.</p> 
<p><strong>Question 3</strong>: Which EC2 instances had vulnerability security findings?</p> 
<p><strong>Answer 3</strong>: Based on the query results, the following EC2 instances had vulnerability security findings in the AWS account with the most EC2 instance findings in the last 7 days:</p> 
<p>In account XXXXXX28:</p> 
<ol type="1"> 
 <li>arn:aws:ec2:us-east-1: XXXXXX28:instance/&lt;instance-id&gt;</li> 
 <li>arn:aws:ec2:us-east-1: XXXXXX28:instance/&lt;instance-id&gt;</li> 
</ol> 
<blockquote>
 <p>There were no vulnerability findings for EC2 instances in account XXXXXX10 during this time period.</p>
</blockquote> 
<p>The question and response are shown in Figure 4.</p> 
<div class="wp-caption aligncenter" id="attachment_38151" style="width: 857px;">
 <img alt="Figure 4: Response for EC2 instances with vulnerability security findings" class="size-full wp-image-38151" height="184" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/30/Figure_4.jpg" style="border: 1px solid #bebebe;" width="847" />
 <p class="wp-caption-text" id="caption-attachment-38151">Figure 4: Response for EC2 instances with vulnerability security findings</p>
</div> 
<p>In this question, the search is narrowed down to two <a href="https://aws.amazon.com/ec2" rel="noopener" target="_blank">Amazon Elastic Compute Cloud (Amazon EC2)</a> instances within a specific AWS account to take a deeper look into.</p> 
<h3 id="recommend-incident-response-playbooks">Recommend incident response playbooks</h3> 
<p>As part of this sample solution, we used the <a href="https://github.com/aws-samples/aws-incident-response-playbooks" rel="noopener" target="_blank">AWS Incident Response Playbook Samples</a> GitHub repository and uploaded them to an S3 bucket. Alternatively, you can create or reference your own playbooks to investigate and respond to various security events to refer to. After each question, the chatbot provides an answer along with an Amazon S3 URL to the playbook. The security analyst must have the appropriate permissions to download the playbook from Amazon S3.</p> 
<p><strong>Question 1</strong>: Do I have any playbooks that would help investigate a compromised EC2 instance for crypto mining or malware?</p> 
<p><strong>Answer 1</strong>: Yes, the search results include a relevant playbook for investigating a potentially compromised EC2 instance that could be used for crypto mining, malware, or ransomware. The key steps outlined in the playbook are:</p> 
<ol type="1"> 
 <li>Acquire, preserve, and document evidence – Identify indicators of compromise, determine attack vector, document impacted applications/users</li> 
 <li>Contain the incident – Isolate the compromised instance if possible without service disruption, remove from auto-scaling group if applicable</li> 
 <li>Eradicate the incident – Follow the steps to remove the malware/crypto miner, restore from known good backups/AMIs</li> 
 <li>Recovery – Spin up new instances from trusted AMIs, restore the data, and monitor for persistence of the threat</li> 
 <li>Lessons learned – Analyze the incident, identify root causes, and implement preventative measures</li> 
</ol> 
<p>The playbook provides detailed guidance on each of these phases in the context of an EC2 ransomware incident, but the overall process can be adapted for investigating crypto mining, malware, or other threats on EC2 as well.</p> 
<p>The question and response are shown in Figure 5.</p> 
<div class="wp-caption aligncenter" id="attachment_38152" style="width: 959px;">
 <img alt="Figure 5: Response for EC2 IR playbooks" class="size-full wp-image-38152" height="186" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/30/Figure_5.jpg" style="border: 1px solid #bebebe;" width="949" />
 <p class="wp-caption-text" id="caption-attachment-38152">Figure 5: Response for EC2 IR playbooks</p>
</div> 
<p>The previous threat-hunting exercise identified two EC2 instances that had vulnerabilities present. After further investigation, if a security analyst believes the EC2 instance is compromised, they can ask for an IR playbook to respond to the security event. Once they determine which playbook makes the most sense, they can either click on the S3 URL or look at other playbooks to use.</p> 
<p><strong>Question 2</strong>: Do I have any playbooks that would help respond to ransomware in my S3 bucket?</p> 
<p><strong>Answer 2</strong>: Yes, the search results include a playbook that can help respond to a ransomware incident involving an S3 bucket. The “Incident Response Playbook Template for Unintended Access to an Amazon S3 Bucket” outlines steps for:</p> 
<ol type="1"> 
 <li>Gathering evidence of the unintended access</li> 
 <li>Containing the incident by restricting access to the affected bucket(s)</li> 
 <li>Eradicating the issue by removing any malicious data/objects and restoring from backup</li> 
 <li>Recovering by re-enablement access to the bucket for legitimate users</li> 
 <li>Conducting post-incident activities like reviewing logs, determining root cause, and implementing preventative measures</li> 
</ol> 
<p>While focused on generic unintended access rather than ransomware specifically, many of these steps could be adapted and applied to a ransomware scenario targeting data in S3. The playbook provides a solid framework for incident response on S3.</p> 
<p>Additionally, the general “Ransomware Incident Response Playbook” template covers common steps for any ransomware incident, such as:</p> 
<ol type="1"> 
 <li>Acquiring and preserving evidence of the ransomware infection</li> 
 <li>Containing the incident by isolating affected systems</li> 
 <li>Identifying and remove the ransomware</li> 
 <li>Recovering data from backups or known good sources</li> 
 <li>Conducting post-incident review</li> 
</ol> 
<p>While not S3-specific, this playbook provides complementary guidance that could be used in conjunction with the S3 access playbook for an S3 ransomware incident.</p> 
<p>The question and response are shown in Figure 6.<br /> </p>
<div class="wp-caption aligncenter" id="attachment_38153" style="width: 969px;">
 <img alt="Figure 6: Response for S3 ransomware IR playbooks" class="size-full wp-image-38153" height="309" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/30/Figure_6.jpg" style="border: 1px solid #bebebe;" width="959" />
 <p class="wp-caption-text" id="caption-attachment-38153">Figure 6: Response for S3 ransomware IR playbooks</p>
</div>
<p></p> 
<p>The response provides a high-level overview of the specific playbook that’s being referenced to provide the analyst with context before diving deeper into any single playbook.</p> 
<h3 id="adding-functionality-to-the-security-chatbot">Adding functionality to the security chatbot</h3> 
<p>This sample solution was developed to show the art of the possible. Each customer uses AWS resources to address their business needs in their own way, and security teams must be appropriately equipped to help secure their respective environments. Here are some possible enhancements that you can incorporate into the sample solution to align to your security use-cases and needs.</p> 
<ol type="1"> 
 <li>Incorporate an <a href="https://aws.amazon.com/dynamodb/" rel="noopener" target="_blank">Amazon DynamoDB</a> table to use as part of reporting interactions tied to a specific event or finding GUID. By incorporating an audit trail, you can tie actions taken by the agent and associated resources to a security event and validate the outcome of the investigation before taking action.</li> 
 <li>Tuning the backend chatbot agent to query <a href="https://explore.alas.aws.amazon.com/" rel="noopener" target="_blank">Amazon Linux Security Center’s Common Vulnerabilities and Exposures (CVE) list</a> or <a href="https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=aws" rel="noopener" target="_blank">MITRE’s CVE list</a> to see which AWS resources might be in scope and send out consolidated messages to resource owners with recommended actions.</li> 
 <li>Tuning the backend chatbot agent to take natural language requests and respond with <a href="https://docs.aws.amazon.com/opensearch-service/latest/developerguide/security-analytics.html" rel="noopener" target="_blank">detectors or correlation rules for Amazon OpenSearch</a> or query language for custom detections in your security information and event management (SIEM) tool.</li> 
 <li>Adding a new data source to Athena, <a href="https://aws.amazon.com/blogs/mt/how-to-query-your-aws-resource-configuration-states-using-aws-config-and-amazon-athena/" rel="noopener" target="_blank">such as AWS Config</a>, to provide the analyst with additional capabilities to query AWS resource configuration across the AWS environment that might have been impacted by a security event. For example, if a security finding shows that an S3 bucket has been made public, querying what and when other configuration changes were made to the S3 bucket.</li> 
 <li>Incorporating <a href="https://github.com/awslabs/multi-agent-orchestrator" rel="noopener" target="_blank">multi-agent-orchestration</a> to scale the use of multiple Amazon Bedrock agents that can be tuned towards niche security use cases by respective teams. The chatbot can speak directly to a classifier or controller, which then addresses the user’s natural language request and orchestrates across one or more agents to generate a response. For example, if a user asks which EC2 instances might have been impacted by a security event and which playbook to use to respond, the classifier agent could direct the initial query to the agent in this sample solution. In the same chat window, the analyst could ask if there are any open CVEs for the EC2 instances in scope to get a list of CVEs to address within the AWS account.</li> 
 <li>For long running Athena queries, you can incorporate an <a href="https://docs.aws.amazon.com/step-functions/" rel="noopener" target="_blank">AWS Step Function</a> to the workflow and <a href="https://docs.aws.amazon.com/step-functions/latest/dg/connect-to-resource.html#connect-wait-token" rel="noopener" target="_blank">incorporate a task token</a> to wait for the Athena results to return.</li> 
</ol> 
<h2 id="clean-up">Clean up</h2> 
<p>If you deployed the security chatbot sample solution by using the <strong>Launch Stack</strong> button and the console with the CloudFormation template <code style="color: #000000;">security_genai_chatbot_cfn</code>, do the following to clean up:</p> 
<ol type="1"> 
 <li>In the <a href="https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1" rel="noopener" target="_blank">CloudFormation console</a> for the account and Region where you deployed the solution, choose the <strong>SecurityGenAIChatbot</strong> stack.</li> 
 <li>Choose the option to <strong>Delete</strong> the stack.</li> 
</ol> 
<p>If you deployed the solution by using the AWS CDK, run the command <code style="color: #000000;">cdk destroy --all</code>.</p> 
<h2 id="conclusion">Conclusion</h2> 
<p>The sample solution demonstrates how you can use task-oriented Amazon Bedrock agents and natural language input to help accelerate investigation and analysis and increase your overall security posture. We provided an example of a sample solution with a user interface that is powered by an Amazon Bedrock agent, which you can extend to add additional task-oriented agents, each with their own instructions, knowledge bases, and models. By extending the use of AI-powered agents, you can help your security team operate more efficiently across multiple security domains within your AWS environment.</p> 
<p>The backend for the chatbot to investigate security events uses Security Lake, which normalizes data into <a href="https://docs.aws.amazon.com/security-lake/latest/userguide/open-cybersecurity-schema-framework.html" rel="noopener" target="_blank">Open Cybersecurity Schema Framework (OCSF)</a>; as long as the data schema is normalized, the solution can be applied to other data lakes within your AWS environment.</p> 
<p>To learn more, see the other posts in this series:</p> 
<ul> 
 <li><a href="https://aws.amazon.com/blogs/security/generate-machine-learning-insights-for-amazon-security-lake-data-using-amazon-sagemaker/" rel="noopener" target="_blank">Part 1: Generate machine learning insights for Amazon Security Lake data using Amazon SageMaker</a></li> 
 <li><a href="https://aws.amazon.com/blogs/security/generate-ai-powered-insights-for-amazon-security-lake-using-amazon-sagemaker-studio-and-amazon-bedrock/" rel="noopener" target="_blank">Part 2: Generate AI powered insights for Amazon Security Lake using Amazon SageMaker Studio and Amazon Bedrock</a></li> 
</ul> 
<p>Use the comments section to provide feedback. If you have questions about this post, start a new thread on the <a href="https://repost.aws/selections/KPEKXMuRyuSyyxnsuruRrtlg/generative-ai-on-aws" rel="noopener" target="_blank">Generative AI on AWS re:Post</a> or <a href="https://console.aws.amazon.com/support/home" rel="noopener" target="_blank">contact AWS Support</a>.</p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Madhunika-Reddy-Mikkili" class="aligncenter size-full wp-image-33069" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/01/12/Madhunika-Reddy-Mikkili.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Madhunika Reddy Mikkili</h3> 
  <p>Madhunika is a Data and Machine Learning Engineer at AWS. She is passionate about helping customers achieve their goals using data analytics and machine learning.</p> 
  <p></p>
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Author" class="aligncenter size-full wp-image-22628" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2021/10/20/Jonathan-Nguyen-Author.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Jonathan Nguyen</h3> 
  <p>Jonathan is a Principal Security Solution Architect at AWS. He helps large financial services customers develop a comprehensive security strategy and solutions to meet their security and compliance requirements in AWS.</p> 
  <p></p>
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Harsh Asnani" class="aligncenter size-full wp-image-33070" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/01/12/Harsh-Asnani.png" width="120" /> 
  </div> 
  <h3 class="lb-h4">Harsh Asnani</h3> 
  <p>Harsh is a Machine Learning Engineer at AWS specializing in ML theory, MLOPs, and production generative AI frameworks. His background is in applied data science with a focus on operationalizing AI workloads in the cloud at scale.</p> 
  <p></p>
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Michael Massey" class="aligncenter size-full wp-image-38043" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/21/Michael-Massey.jpg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Michael Massey</h3> 
  <p>Michael is a Cloud Application Architect at AWS, where he specializes in building frontend and backend cloud-centered applications. He designs and implements scalable and highly-available solutions and architectures that help customers achieve their business goals.t</p> 
  <p></p>
 </div> 
</footer>
