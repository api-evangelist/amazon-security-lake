---
title: "Create security observability using generative AI with Security Lake and Amazon Q in QuickSight"
url: "https://aws.amazon.com/blogs/security/create-security-observability-using-generative-ai-with-security-lake-and-amazon-q-in-quicksight/"
date: "Mon, 16 Sep 2024 16:31:47 +0000"
author: "Priyank Ghedia"
feed_url: "https://aws.amazon.com/blogs/security/tag/amazon-security-lake/feed/"
---
<p>Generative artificial intelligence (AI) is now a household topic and popular across various public applications. Users enter prompts to get answers to questions, write code, create images, improve their writing, and synthesize information. As people become familiar with generative AI, businesses are looking for ways to apply these concepts to their enterprise use cases in a simple, scalable, and cost-effective way. These same needs are shared by a variety of security stakeholders. For example, if security directors want to summarize their security posture in natural language, a security architect will need to triage alerts or findings and investigate <a href="https://aws.amazon.com/cloudtrail/" rel="noopener" target="_blank">AWS CloudTrail</a> logs to identify high priority remediation actions or detect potential threat actors by identifying potentially malicious activity. There are many ways to deploy solutions for these use cases.</p> 
<p>In this blog post, we review a fully serverless solution for querying data stored in <a href="https://docs.aws.amazon.com/security-lake/latest/userguide/what-is-security-lake.html" rel="noopener" target="_blank">Amazon Security Lake</a> using <a href="https://aws.amazon.com/what-is/nlp/" rel="noopener" target="_blank">natural language</a> (human language) with Amazon Q in QuickSight. This solution has multiple use cases, such as generating visualizations and querying vulnerability information for vulnerability management using tools such as <a href="https://aws.amazon.com/inspector" rel="noopener" target="_blank">Amazon Inspector</a> that feed into <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html" rel="noopener" target="_blank">AWS Security Hub</a>. The solution helps reduce the time from detection to investigation by using natural language to query CloudTrail logs and <a href="https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html" rel="noopener" target="_blank">Amazon Virtual Private Cloud (VPC) Flow Logs</a>, resulting in quicker response to threats in your environment.</p> 
<p>Amazon Security Lake is a fully managed security data lake service that automatically centralizes security data from AWS environments, software as a service (SaaS) providers, and on-premises and cloud sources into a purpose-built data lake that’s stored in your AWS account. The data lake is backed by <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> buckets, and you retain ownership over your data. Security Lake converts ingested data into Apache Parquet format and a standard open source schema called the <a href="https://docs.aws.amazon.com/security-lake/latest/userguide/open-cybersecurity-schema-framework.html" rel="noopener" target="_blank">Open Cybersecurity Schema Framework (OCSF).</a> With OCSF support, Security Lake normalizes and combines security data from AWS and a broad range of enterprise <a href="https://docs.aws.amazon.com/security-lake/latest/userguide/internal-sources.html" rel="noopener" target="_blank">security data sources.</a></p> 
<p><a href="https://aws.amazon.com/quicksight/?amazon-quicksight-whats-new.sort-by=item.additionalFields.postDateTime&amp;amazon-quicksight-whats-new.sort-order=desc" rel="noopener" target="_blank">Amazon QuickSight</a> is a cloud-scale business intelligence (BI) service that delivers insights to stakeholders, wherever they are. QuickSight connects to your data in the cloud and combines data from a variety of different sources. With QuickSight, users can meet varying analytic needs from the same source of truth through interactive dashboards, reports, natural language queries, and embedded analytics. With Amazon Q in QuickSight, business analysts and users can use natural language to build, discover, and share meaningful insights.</p> 
<p>The recent announcements for Amazon Q in QuickSight, Security Lake, and the <a href="https://docs.aws.amazon.com/security-lake/latest/userguide/open-cybersecurity-schema-framework.html" rel="noopener" target="_blank">OCSF</a> present a unique opportunity to apply generative AI to fully managed hybrid multi-cloud security related logs and findings from over 100 independent software vendors and partners.</p> 
<h2>Solution overview</h2> 
<p>The solution uses Security Lake as the data lake which has native ingestion for CloudTrail, VPC Flow Logs, and Security Hub findings as shown in Figure 1. Logs from these sources are sent to S3 buckets in your AWS account and are maintained by Security Lake. We then create <a href="https://docs.aws.amazon.com/athena/latest/ug/views.html" rel="noopener" target="_blank">Amazon Athena views</a> from tables created by Security Lake for Security Hub findings, CloudTrail logs, and VPC Flow Logs to define the interesting fields from each of the log sources. Each of these views are ingested into a QuickSight dataset. From these datasets, we generate analyses and dashboards. We use <a href="https://docs.aws.amazon.com/quicksight/latest/user/quicksight-q-topics.html" rel="noopener" target="_blank">Amazon Q topics</a> to label columns in the dataset that are human-readable and create a <a href="https://docs.aws.amazon.com/quicksight/latest/user/gen-bi-author-q-and-a.html" rel="noopener" target="_blank">named entity</a> to present contextual and multi-visual answers in response to questions. After the topics are created, users can perform their analysis using Q topics, QuickSight analyses, or QuickSight dashboards.</p> 
<div class="wp-caption aligncenter" id="attachment_35621" style="width: 790px;">
 <img alt="Figure 1: Solution architecture" class="size-full wp-image-35621" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img1.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-35621">Figure 1: Solution architecture</p>
</div> 
<p>You can use the <a href="https://docs.aws.amazon.com/security-lake/latest/userguide/manage-regions.html" rel="noopener" target="_blank">rollup AWS Region</a> feature in Security Lake to aggregate logs from multiple Regions into a single Region. Specifying a rollup Region can help you adhere to regional compliance requirements. If you use rollup Regions, you must set up the solution described in this post for datasets only in rollup Regions. If you don’t use a rollup Region, you must deploy this solution for each Region you that want to collect data from.</p> 
<h2>Prerequisites</h2> 
<p>To implement the solution described in this post, you must meet the following requirements:</p> 
<ol> 
 <li>Basic understanding of Security Lake, Athena, and QuickSight.</li> 
 <li>Security Lake is already deployed and accepting CloudTrail management events, VPC Flow Logs, and Security Hub findings as sources. If you haven’t deployed Security Lake yet, we recommend following the best practices established in the <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/log-archive.html#log-security-lake" rel="noopener" target="_blank">security reference architecture</a>.</li> 
 <li>This solution uses Security Lake data source version 2 to create the dashboards and visualizations. If you aren’t already using data source version 2, you will see a banner in your Security Lake console with instructions to update.</li> 
 <li>An existing QuickSight deployment that will be used to visualize Security Lake data or an account that is able to sign up for QuickSight to create visualizations.</li> 
 <li>QuickSight <a href="https://aws.amazon.com/quicksight/pricing/" rel="noopener" target="_blank">Author Pro and Reader Pro licenses</a> are needed for using Amazon Q features in QuickSight. Non-pro Authors and Readers can still access Q topics if an Author Pro or Admin Pro user shares the topic with them. Non-pro Authors and Readers can also access data stories if a Reader Pro, Author Pro, or Admin Pro shares one with them. Review <a href="https://docs.aws.amazon.com/quicksight/latest/user/generative-bi-get-started.html" rel="noopener" target="_blank">Generative AI features supported by each QuickSight licensing tiers</a>.</li> 
 <li><a href="https://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Manager (IAM)</a> permissions for QuickSight, Athena, Lake Formation, Security Lake, and <a href="https://aws.amazon.com/ram" rel="noopener" target="_blank">AWS Resource Access Manager</a>.</li> 
</ol> 
<p>In the following section, we walk through the steps to ingest Security Lake data into QuickSight using Athena views and then using Amazon Q in QuickSight to create visualizations and query data using natural language. </p> 
<h2>Provide cross-account query access</h2> 
<p>In alignment with our security reference architecture, it’s a best practice to isolate the Security Lake account from the accounts that are running the visualization and querying workloads. It’s recommended that QuickSight for security use cases be deployed in the <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/security-tooling.html" rel="noopener" target="_blank">security tooling account.</a> See <a href="https://aws.amazon.com/blogs/security/how-to-visualize-amazon-security-lake-findings-with-amazon-quicksight/" rel="noopener" target="_blank">How to visualize Amazon Security Lake findings with Amazon QuickSight</a> for information on how to set up cross-account query access. Follow the steps in the <em>Configure a Security Lake subscriber</em> section and <em>configure Athena to visualize your data</em> section.</p> 
<p>When you get to the create resource link steps, create a resource link for data source version 2 for Security Hub, CloudTrail, and VPC flow log tables for a total of three resource links. The way to identify data source version 2 tables is by their name; it ends in <code style="color: #000000;">_2_0</code>. For example:</p> 
<ul> 
 <li><code style="color: #000000;">amazon_security_lake_table_us_east_1_sh_findings_2_0</code></li> 
 <li><code style="color: #000000;">amazon_security_lake_table_us_east_1_cloud_trail_mgmt_2_0</code></li> 
 <li><code style="color: #000000;">amazon_security_lake_table_us_east_1_vpc_flow_2_0</code></li> 
</ul> 
<p>For the remainder of this post, we will be referencing the database name <code style="color: #000000;">security_lake_visualization</code> and the resource link names for Security Hub findings, CloudTrail logs, and VPC Flow Logs respectively, as shown in Figure 2:</p> 
<ul> 
 <li><code style="color: #000000;">securitylake_shared_resourcelink_securityhub_2_0_us_east_1</code></li> 
 <li><code style="color: #000000;">securitylake_shared_resourcelink_cloudtrail_2_0_us_east_1</code></li> 
 <li><code style="color: #000000;">securitylake_shared_resourcelink_vpcflow_2_0_us_east_1</code></li> 
</ul> 
<p style="line-height: 1.25em;"></p>
<div class="wp-caption aligncenter" id="attachment_35622" style="width: 790px;">
 <img alt="Figure 2: Lake Formation table snapshot" class="size-full wp-image-35622" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img2-1.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-35622">Figure 2: Lake Formation table snapshot</p>
</div>
<p></p> 
<p>We will call the QuickSight account the <em>visualization account</em>. If you plan to use same account as the Security Lake delegated administrator and QuickSight, then skip this step and go to the next section where you will create views in Athena.</p> 
<h2>Create views in Athena</h2> 
<p>A <a href="https://docs.aws.amazon.com/athena/latest/ug/views.html" rel="noopener" target="_blank">view</a> in Athena is a logical table that helps simplify your queries by working with only a subset of the relevant data. Follow these steps to create three views in Athena, one each for Security Hub findings, CloudTrail logs, and the VPC Flow Logs in the visualization account.</p> 
<p>These queries default to the previous week’s data starting from the previous day, but you can change the time frame by modifying the last line in the query from 8 to the number of days you prefer. Keep in mind that there is a limitation on the size of each SPICE table of 1 TB. If you want to limit the volume of data, you can delete the rows that you find unnecessary. We included the fields customers have identified as relevant to reduce the burden of writing the parsing details yourself.</p> 
<p><strong>To create views:</strong></p> 
<ol> 
 <li>Sign in to the AWS Management Console in the visualization account and navigate to the Athena console.</li> 
 <li>If a Security Lake rollup Region is used, select the rollup Region.</li> 
 <li>Choose <strong>Launch Query Editor</strong>.</li> 
 <li>If this is the first time you’re using Athena, you will need to choose a bucket to store your query results. 
  <ol> 
   <li>Choose <strong>Edit Settings</strong>.</li> 
   <li>Choose <strong>Browse S3</strong>.</li> 
   <li>Search for your bucket name.</li> 
   <li>Select the radio button next to the name of your bucket.</li> 
   <li>Select <strong>Choose</strong>.</li> 
  </ol> </li> 
 <li>For <strong>Data Source</strong>, select <strong>AWSDataCatalog</strong>.</li> 
 <li>Select <strong>Database</strong> as <strong>security_lake_visualization</strong>. If you used a different name for the database for cross account query access, then select that database. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35623" style="width: 541px;">
   <img alt="Figure 3: Athena database selection" class="size-full wp-image-35623" height="318" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img3-1.png" style="border: 1px solid #bebebe;" width="531" />
   <p class="wp-caption-text" id="caption-attachment-35623">Figure 3: Athena database selection</p>
  </div><p></p> </li> 
 <li>Copy the query for the <code style="color: #000000;">security_hub_view</code> from the <a href="https://github.com/aws-samples/create-security-observability-using-generative-ai-with-security-lake-and-amazon-q-for-quicksight" rel="noopener" target="_blank">GitHub repo</a> for this post. If you’re using a different name for the database and table resource link than the one specified in this post, edit the FROM statement at the bottom of the query to reflect the correct names.</li> 
 <li>Paste the query in the query editor and then choose <strong>Run</strong>. The name of the view is set in the first line of the query which is <code style="color: #000000;">security_insights_security_hub_vw2</code>.</li> 
 <li>To confirm this view was created correctly, choose the three dots next to the view that was created and select <strong>Preview View</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35624" style="width: 750px;">
   <img alt="Figure 4: Previewing the view" class="size-full wp-image-35624" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img4-1.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35624">Figure 4: Previewing the view</p>
  </div><p></p> </li> 
 <li>Repeat steps 5–9 to create the CloudTrail and VPC Flow Logs views. The queries for each can be found in the <a href="https://github.com/aws-samples/create-security-observability-using-generative-ai-with-security-lake-and-amazon-q-for-quicksight" rel="noopener" target="_blank">GitHub repo</a>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35625" style="width: 510px;">
   <img alt="Figure 5: Athena views" class="size-full wp-image-35625" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img5.png" style="border: 1px solid #bebebe;" width="500" />
   <p class="wp-caption-text" id="caption-attachment-35625">Figure 5: Athena views</p>
  </div><p></p> </li> 
</ol> 
<h2>Create QuickSight dataset</h2> 
<p>Now that you’ve created the views, use Athena as the data source to <a href="https://docs.aws.amazon.com/quicksight/latest/user/creating-data-sets.html" rel="noopener" target="_blank">create a dataset in QuickSight</a>. Repeat these steps for the Security Hub findings, CloudTrail logs, and VPC Flow Logs. Start by creating a dataset for the Security Hub findings.</p> 
<p><strong>To configure permissions on tables:</strong></p> 
<ol> 
 <li>Sign in to the QuickSight console in the visualization account. If a Security Lake rollup Region is used, select the rollup Region.</li> 
 <li>If this is the first time you’re using QuickSight, you must <a href="https://docs.aws.amazon.com/quicksight/latest/user/signing-up.html" rel="noopener" target="_blank">sign up for a QuickSight subscription</a>.</li> 
 <li>Although there are multiple ways to sign in to QuickSight, we used <a href="https://docs.aws.amazon.com/quicksight/latest/user/security_iam_service-with-iam.html" rel="noopener" target="_blank">IAM</a> based access to build the dashboards. To use QuickSight with Athena and Lake Formation, you first need to <a href="https://docs.aws.amazon.com/quicksight/latest/user/lake-formation.html" rel="noopener" target="_blank">authorize connections through Lake Formation</a>.</li> 
 <li>When using a cross-account configuration with <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/serverless-etl-aws-glue/aws-glue-data-catalog.html" rel="noopener" target="_blank">AWS Glue Data Catalog</a>, you need to configure permissions on tables that are shared through Lake Formation. For the use case in this post, use the following steps to grant access on the cross-account tables in the Glue Catalog. You must perform these steps for each of the Security Hub, CloudTrail, and VPC Flow Logs tables that you created in the preceding <strong>cross-account query access</strong> section. Because granting permissions on a resource link doesn’t grant permissions on the target (linked) database or table, you will grant permission twice, once to the target (linked table) and then to the resource link. 
  <ol> 
   <li>In the Lake Formation console, navigate to the <strong>Tables</strong> section and select the resource link for the Security Hub table. For example: <p><code style="color: #000000;">securitylake_shared_resourcelink_securityhub_2_0_us_east_1</code></p> </li> 
   <li>Select <strong>Actions</strong>. Under <strong>Permissions</strong>, select <strong>Grant on target</strong>.</li> 
   <li>For the next step, you need the Amazon Resource Name (ARN) of the QuickSight users or groups that need access to the table. To obtain the ARN through the <a href="https://aws.amazon.com/cli">AWS Command Line Interface (AWS CLI)</a>, run following commands (replacing account ID and Region with that of the visualization account.) You can use <a href="https://docs.aws.amazon.com/cloudshell/latest/userguide/working-with-aws-cloudshell.html">AWS CloudShell</a> for this purpose. 
    <ol> 
     <li>For users <p><code style="color: #000000;">aws quicksight list-users --aws-account-id 111122223333 --namespace default --region us-east-1</code></p> </li> 
     <li>For groups <p><code style="color: #000000;">aws quicksight list-groups --aws-account-id 111122223333 --namespace default --region us-east-1</code></p> </li> 
    </ol> </li> 
   <li>After you have the ARN of the user or group, copy it and go back to the LakeFormation console <strong>Grant on Target</strong> page. For <strong>Principals</strong>, select <strong>SAML users and groups</strong>, and then add the QuickSight user’s ARN. <p style="line-height: 1.25em;"></p>
    <div class="wp-caption aligncenter" style="width: 560px;">
     <img alt="Figure 6: Selecting principals" class="wp-image-35629" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img5b.png" style="border: 1px solid #bebebe;" width="500" />
     <p class="wp-caption-text">Figure 6: Selecting principals</p>
    </div><p></p> </li> 
   <li>For <strong>LF-Tags or catalog resources</strong>, keep the default settings. <p style="line-height: 1.25em;"></p>
    <div class="wp-caption aligncenter" id="attachment_35630" style="width: 560px;">
     <img alt="Figure 7: Table grant on target permissions" class="aligncenter size-full wp-image-35668" height="471" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/10/img6-1.png" style="border: 1px solid #bebebe;" width="531" />
     <p class="wp-caption-text" id="caption-attachment-35630">Figure 7: Table grant on target permissions</p>
    </div><p></p> </li> 
   <li>For <strong>Table permissions</strong>, select <strong>Select</strong> for both <strong>Table Permissions</strong> and <strong>Grantable Permissions</strong>, and then choose <strong>Grant</strong>. <p style="line-height: 1.25em;"></p>
    <div class="wp-caption aligncenter" id="attachment_35631" style="width: 610px;">
     <img alt="Figure 8: Selecting table permissions" class="aligncenter size-full wp-image-35631" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img6b.png" style="border: 1px solid #bebebe;" width="600" />
     <p class="wp-caption-text" id="caption-attachment-35631">Figure 8: Selecting table permissions</p>
    </div><p></p> </li> 
   <li>Navigate back to the <strong>Tables</strong> section and select the resource link for the Security Hub table. For example: <p><code style="color: #000000;">securitylake_shared_resourcelink_securityhub_2_0_us_east_1</code></p> </li> 
   <li>Select <strong>Actions</strong>. This time under <strong>Permissions</strong>, and then choose <strong>Grant</strong>.</li> 
   <li>For <strong>Principals</strong>, select SAML users and groups, and then add the QuickSight user’s ARN captured earlier.</li> 
   <li>For the <strong>LF-Tags or catalog resources</strong> section, use the default settings.</li> 
   <li>For <strong>Resource link permissions</strong> choose <strong>Describe</strong> for both <strong>Table Permissions</strong> and <strong>Grantable Permissions</strong>.</li> 
   <li>Repeat steps a–k for the CloudTrail and VPC Flow Logs resource links.</li> 
  </ol> </li> 
</ol> 
<p><strong>To create datasets from views:</strong></p> 
<ol> 
 <li>After permissions are in place, you create three datasets from the views created earlier. Because both Quicksight and Lake Formation are Regional services, verify that you’re using QuickSight in the same Region where Lake Formation is sharing the data. The simplest way to determine your Region is to check the QuickSight URL in your web browser. The Region will be at the beginning of the URL, such as <em>us-east-1</em>. To change the Region, select the settings icon in the top right of the QuickSight screen and select the correct Region from the list of available Regions in the drop-down menu.</li> 
 <li>Navigate back to the <strong>QuickSight</strong> console.</li> 
 <li>Select <strong>Datasets</strong>, and then choose <strong>New dataset</strong>.</li> 
 <li>Select <strong>Athena</strong> from the list of available data sources.</li> 
 <li>Enter a <strong>Data source name</strong>, for example <code style="color: #000000;">security_lake_securityhub_dataset</code> and leave the Athena workgroup as <code style="color: #000000;">[primary]</code>. Choose <strong>Create</strong><strong> data source</strong>.</li> 
 <li>At the <strong>Choose your table</strong> prompt, for <strong>Catalog</strong>, select <strong>AwsDataCatalog</strong>. For <strong>Database</strong>, select <code style="color: #000000;">security_lake_visualization</code>. If you used a different name for the database for cross-account query access, then select that database. For <strong>Tables</strong>, select the view name <code style="color: #000000;">security_insights_security_hub_vw2</code> to build your dashboards for Security Hub findings. Then choose <strong>Select</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35633" style="width: 510px;">
   <img alt="Figure 9: Choose a table during QuickSight dataset creation" class="size-full wp-image-35633" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img7.png" style="border: 1px solid #bebebe;" width="500" />
   <p class="wp-caption-text" id="caption-attachment-35633">Figure 9: Choose a table during QuickSight dataset creation</p>
  </div><p></p> </li> 
 <li>At the <strong>Finish dataset creation</strong> prompt, select <strong>Import to SPICE for quicker analytics</strong>. Choose <strong>Visualize</strong>. This will create a new dataset in QuickSight using the name of the Athena view, which is <code style="color: #000000;">security_insights_security_hub_vw2</code>. You will be taken to the <strong>Analysis</strong> page, exit out of it.</li> 
 <li>Go back to the QuickSight console and repeat steps 3–8 for the CloudTrail and VPC Flow Log datasets.</li> 
</ol> 
<h2>Create a topic</h2> 
<p>Now that you have created a dataset, you can create a topic. <a href="https://docs.aws.amazon.com/quicksight/latest/user/quicksight-q-topics.html" rel="noopener" target="_blank">Q topics</a> are collections of one or more datasets that represent a subject area for your business users to ask questions. Topics allow users to ask questions in natural language and to build visualizations using natural language.</p> 
<p><strong>To create a Q topic:</strong></p> 
<ol> 
 <li>Navigate to the <strong>QuickSight</strong> console.</li> 
 <li>Choose <strong>Topics</strong> in the left navigation pane. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35637" style="width: 238px;">
   <img alt="Figure 10: QuickSight navigation pane" class="size-full wp-image-35637" height="206" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img8.png" style="border: 1px solid #bebebe;" width="228" />
   <p class="wp-caption-text" id="caption-attachment-35637">Figure 10: QuickSight navigation pane</p>
  </div><p></p> </li> 
 <li>Choose <strong>New topic</strong>. Create one topic each for the Security Hub findings, CloudTrail logs, and VPC Flow Logs <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35638" style="width: 750px;">
   <img alt="Figure 11: QuickSight topic creation" class="size-full wp-image-35638" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img9.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35638">Figure 11: QuickSight topic creation</p>
  </div><p></p> </li> 
 <li>On the <strong>New topic</strong> page, do the following: 
  <ol> 
   <li>For <strong>Topic name</strong>, enter a descriptive name for the topic. Name the first one <code style="color: #000000;">SecurityHubTopic</code>. Your business users will identify the topic by this name and use it to ask questions.</li> 
   <li>For <strong>Description</strong>, enter a description for the topic. Your users can use this description to get more details about the topic.</li> 
   <li>Choose <strong>Continue</strong>.</li> 
  </ol> </li> 
 <li>On the <strong>Add data to topic</strong> page, choose the dataset you created in the <strong>Create </strong><strong>a QuickSight dataset</strong> section. Start with the Security Hub dataset <code style="color: #000000;">security_insights_security_hub_vw2</code>.</li> 
 <li>Choose <strong>Continue</strong>. It will take a few minutes to create the topic.</li> 
 <li>Now that your topic has been created, navigate to the <strong>Data</strong> tab of the topic.</li> 
 <li>Your <strong>Data Fields</strong> sub-tab should be selected already. If not, choose <strong>Data Fields</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35639" style="width: 475px;">
   <img alt="Figure 12: Topics data fields" class="size-full wp-image-35639" height="175" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img10.png" style="border: 1px solid #bebebe;" width="465" />
   <p class="wp-caption-text" id="caption-attachment-35639">Figure 12: Topics data fields</p>
  </div><p></p> </li> 
 <li>For each of the fields in the list, turn on <strong>Include </strong>to make sure that all fields are included. For this example, we selected all fields, but you can adjust the included columns as needed for your use case. Note, you might see a banner at the top of the page indicating that the indexing is in progress. Depending on the size of your data, it might take some time for Q to make those fields available for querying. Most of the time, indexing is complete in less than 15 minutes.</li> 
 <li>Review the <strong>Synonyms</strong> column. These alternate representations of your column name are automatically generated by Amazon Q. You can add and remove synonyms as needed for your use case.</li> 
 <li>At this point, you’re ready to ask questions about your data using Amazon Q in QuickSight. Choose <strong>Ask a question about SecurityHubTopic</strong> at the top of the page. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35640" style="width: 750px;">
   <img alt="Figure 13: Ask questions using Q" class="size-full wp-image-35640" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img11.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35640">Figure 13: Ask questions using Q</p>
  </div><p></p> </li> 
 <li>You can now ask questions about Security Hub findings in the prompt. Enter <code style="color: #000000;">Show me findings with compliance status failed along with control id</code>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35641" style="width: 750px;">
   <img alt="Figure 14: Q answers" class="size-full wp-image-35641" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img12.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35641">Figure 14: Q answers</p>
  </div><p></p> </li> 
 <li>Under the question, you will see how it was interpreted by QuickSight.</li> 
 <li>Repeat steps 1–13 to create CloudTrail and VPC Flow Log QuickSight topics.</li> 
</ol> 
<h2>Create named entities for your topics</h2> 
<p>Now that you’ve created your topics, you will now add named entities. Named entities are optional, but we’re using them in the solution to help make queries more effective. The information contained in named entities, the ordering of fields, and their ranking make it possible to present contextual, multi-visual answers in response to even vague questions.</p> 
<p><strong>To create a named entity:</strong></p> 
<ol> 
 <li>In the QuickSight console, navigate to <strong>Topics</strong>.</li> 
 <li>Select the Security Hub topic that you created in the previous section.</li> 
 <li>Under the <strong>Data</strong> tab, select the <strong>Named Entity</strong> subtab, and choose <strong>Add Named Entity</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35642" style="width: 434px;">
   <img alt="Figure 15: Named entity subtab" class="size-full wp-image-35642" height="171" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img13.png" style="border: 1px solid #bebebe;" width="424" />
   <p class="wp-caption-text" id="caption-attachment-35642">Figure 15: Named entity subtab</p>
  </div><p></p> </li> 
 <li>Enter <code style="color: #000000;">Security Findings</code> as the entity name.</li> 
 <li>Select the following datafields: <strong>Status</strong>, <strong>Metadata Product Name</strong>, <strong>Finding Info Title</strong>, <strong>Region</strong>, <strong>Severity</strong>, <strong>Cloud Account Uid</strong>, <strong>Time Dt</strong>, <strong>Compliance Status</strong>, and <strong>AccountId</strong>. The order of the fields helps Q to prioritize the data, so rearrange your data fields as needed. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35643" style="width: 610px;">
   <img alt="Figure 16: Security hub finding names entity creation" class="size-full wp-image-35643" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img14.png" style="border: 1px solid #bebebe;" width="600" />
   <p class="wp-caption-text" id="caption-attachment-35643">Figure 16: Security hub finding names entity creation</p>
  </div><p></p> </li> 
 <li>Choose <strong>Save</strong> in the top right corner to save your results.</li> 
 <li>Repeat steps 1–6 with the CloudTrail dataset using the following datafields: <em>API operation</em>, <strong>Time Dt</strong>, <strong>Region</strong>, <strong>Status</strong>, <strong>AccountId</strong>, <strong>API Response Error</strong>, <strong>Actor User Credential Uid</strong>, <strong>Actor User Name</strong>, <strong>Actor User Type</strong>, <strong>Api Service Name</strong>, <strong>Actor Idp Name</strong>, <strong>Cloud Provider</strong>, <strong>Session Issuer</strong>, and <strong>Unmapped</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35644" style="width: 610px;">
   <img alt="Figure 17: CloudTrail named entity creation" class="size-full wp-image-35644" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img15.png" style="border: 1px solid #bebebe;" width="600" />
   <p class="wp-caption-text" id="caption-attachment-35644">Figure 17: CloudTrail named entity creation</p>
  </div><p></p> </li> 
 <li>Repeat steps 1–6 with the VPC Flow Log dataset using the following datafields: <strong>Src Endpoint IP</strong>, <strong>Src Endpoint Port</strong>, <strong>Dst Endpoint IP</strong>, <strong>Dst Endpoint Port</strong>, <strong>Connection Info Direction</strong>, <strong>Traffic Bytes</strong>, <strong>Action</strong>, <strong>Accountid</strong>, <strong>Time Dt</strong>, and <strong>Region</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35645" style="width: 606px;">
   <img alt="Figure 18: VPC Flow log named entity creation" class="size-full wp-image-35645" height="588" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img16.png" style="border: 1px solid #bebebe;" width="596" />
   <p class="wp-caption-text" id="caption-attachment-35645">Figure 18: VPC Flow log named entity creation</p>
  </div><p></p> </li> 
</ol> 
<h2>Create visualizations using natural language</h2> 
<p>After your topic is done indexing, you can start creating visualizations using natural language. In QuickSight, an analysis is the same thing as a dashboard, but is only accessible by the authors. You can keep it private and make it as robust and detailed as you want. When you decide to publish it, the shared version is called a dashboard.</p> 
<p><strong>To create visualizations:</strong></p> 
<ol> 
 <li>Open the QuickSight console and navigate to the <strong>Analysis</strong> tab.</li> 
 <li>In the top right, select <strong>New analysis</strong>.</li> 
 <li>Select the dataset you created previously, it will have the same naming convention as the Athena view. For reference, the Athena view query created a Security Hub dataset called <code style="color: #000000;">security_insights_security_hub_vw2</code>.</li> 
 <li>Validate the information about the data set you’re going to use in the analysis and choose <strong>USE IN ANALYSIS</strong>.</li> 
 <li>On the pop up, select the <strong>interactive sheet</strong> option and choose <strong>Create</strong>.</li> 
 <li>For datasets that have a corresponding Q topic, which you created in a previous step, choose <strong>Build visual</strong> at the top of the screen. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35646" style="width: 750px;">
   <img alt="Figure 19: Build visual using natural language" class="size-full wp-image-35646" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img17.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35646">Figure 19: Build visual using natural language</p>
  </div><p></p> </li> 
 <li>Enter your prompt and choose <strong>BUILD</strong>. For example, enter <code style="color: #000000;">findings with product security hub group by control id include count</code>. Q automatically generates a visualization. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35647" style="width: 610px;">
   <img alt="Figure 20: Q response" class="size-full wp-image-35647" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img18.png" style="border: 1px solid #bebebe;" width="600" />
   <p class="wp-caption-text" id="caption-attachment-35647">Figure 20: Q response</p>
  </div><p></p> </li> 
 <li>To add to your dashboard, choose <strong>ADD TO ANALYSIS</strong> to see your new visualization module in your current analysis.</li> 
 <li>The supplied questions are targeted towards a Security Hub findings topic, where you can ask questions about your security hub findings data. For example, <code style="color: #000000;">show all Security Hub findings for critical severity for a specific resource or ARN</code>.</li> 
 <li>If you use Amazon Inspector for software vulnerability management and you want to monitor top common vulnerabilities and exposures (CVEs) affecting your organization, choose <strong>Build visual</strong> and enter <code style="color: #000000;">show all ACTIVE findings with product inspector group by Title add count</code> in the prompt. We used the keyword ACTIVE because ACTIVE is a finding state in Security Hub that indicates the finding is still active as per the finding source and Amazon Inspector has not closed the finding yet. If Amazon Inspector has closed the finding, the finding will have a state of ARCHIVED. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35648" style="width: 750px;">
   <img alt="Figure 21: Q Response for an Amazon Inspector findings question" class="size-full wp-image-35648" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img19.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35648">Figure 21: Q Response for an Amazon Inspector findings question</p>
  </div><p></p> </li> 
 <li>After you add visualization to the analysis, you can customize it further using various <a href="https://docs.aws.amazon.com/quicksight/latest/user/working-with-visual-types.html" rel="noopener" target="_blank">QuickSight visualization options</a>.</li> 
 <li>To add the remaining datasets, which allows you to visualize data from multiple datasets in a single view, select the dropdown in the left navigation under <strong>Dataset</strong>. 
  <ol> 
   <li>Select <strong>Add a new dataset</strong>.</li> 
   <li>Search the name of the remaining datasets you created previously.</li> 
   <li>Select anywhere on the name of the dataset to make the radial button blue for the single dataset you want to add. Choose <strong>Select</strong>.</li> 
  </ol> </li> 
 <li>Repeat steps 7–12 in this section to add all the corresponding datasets you created previously.</li> 
</ol> 
<blockquote>
 <p><strong>Note:</strong> When you add additional datasets to the same Analysis and use <strong>Build visual</strong> to generate visualizations using natural language, the corresponding datasets with Q Topics are populated in the drop down under the prompt. Be sure to choose the correct dataset when asking questions.</p>
</blockquote> 
<div class="wp-caption aligncenter" id="attachment_35649" style="width: 360px;">
 <img alt="Figure 22: Choosing a QuickSight dataset" class="size-full wp-image-35649" height="248" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img20.png" style="border: 1px solid #bebebe;" width="350" />
 <p class="wp-caption-text" id="caption-attachment-35649">Figure 22: Choosing a QuickSight dataset</p>
</div> 
<p><strong>To create dashboards:</strong></p> 
<ol> 
 <li>After you’ve created the visual and are ready to publish the analysis as a dashboard, select <strong>PUBLISH</strong> in the top right corner. 
  <ol> 
   <li>Enter a name for your dashboard.</li> 
   <li>Choose Publish Dashboard.</li> 
  </ol> </li> 
 <li>After your dashboard is published, your users can ask questions about the data through the dashboard as well. This dashboard can be shared with other users. Users with QuickSight Reader Pro licenses can ask questions using Amazon Q.</li> 
</ol> 
<p><strong>To ask questions using the dashboard:</strong></p> 
<ol> 
 <li>Navigate to the <strong>Dashboards</strong> section on the left navigation.</li> 
 <li>Select the dashboard you previously published.</li> 
 <li>Select <strong>Ask a question about [Topic Name]</strong> at the top of the screen. A module will open from the side of your screen. Questions can only be addressed to a single topic. To change the topic, select the name of the topic and a drop-down will appear. Select the name of the current topic to see other options and select the topic you want to ask a question about. For this example, select <strong>CloudTrailTopic</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35650" style="width: 750px;">
   <img alt="Figure 23: Selecting a topic" class="size-full wp-image-35650" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img21.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35650">Figure 23: Selecting a topic</p>
  </div><p></p> </li> 
 <li>Enter a question in the prompt. For this example, enter <code style="color: #000000;">show top API operations in the last 24 hours with accessdenied</code>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35651" style="width: 750px;">
   <img alt="Figure 24: CloudTrail question 1" class="aligncenter size-full wp-image-35669" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/10/img22-1.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35651">Figure 24: CloudTrail question 1</p>
  </div><p></p> </li> 
 <li>Enter <code style="color: #000000;">show all activity by user johndoe in the last 3 days</code>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35652" style="width: 750px;">
   <img alt="Figure 25: CloudTrail question 2" class="aligncenter size-full wp-image-35670" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/10/img23-1.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35652">Figure 25: CloudTrail question 2</p>
  </div><p></p> </li> 
 <li>Q will automatically build a small dashboard based on the questions provided.</li> 
 <li>Now change the topic to <strong>VPCFlowTopic</strong> as described in step 3.</li> 
 <li>Enter <code style="color: #000000;">show me the top 5 dst ip by bytes for outbound traffic with dst port 443</code>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_35653" style="width: 750px;">
   <img alt="Figure 26: VPC Flow Log question" class="size-full wp-image-35653" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/img24.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35653">Figure 26: VPC Flow Log question</p>
  </div><p></p> </li> 
</ol> 
<p>You can build executive summaries using <a href="https://docs.aws.amazon.com/quicksight/latest/user/working-with-stories.html" rel="noopener" target="_blank">QuickSight data stories</a>, which also use generative AI. Data stories use Amazon Q prompts and visuals to produce a draft that incorporates the details that you provide. For example, you can create a data story about how a specific CVE affects your organization by asking Q questions, then add visuals from analyses you already created.</p> 
<h2>Conclusion</h2> 
<p>In this blog post, you learned how to use generative AI for your security use cases. We showed you how to use cross-account query access to allow a QuickSight visualization account to subscribe to Security Lake data for Security Hub findings, CloudTrail logs, and VPC Flow Logs. We then provided instructions for creating, Athena views, QuickSight datasets, Q topics, named entities, and for using natural language to build dashboards and query your data. You can customize the Athena views to create, update, or delete columns and column names as needed for your use case. You can also customize the Q topics and named entities to use naming conventions and structure responses based on your organization’s needs.</p> 
<p>&nbsp;<br />If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.</p> 
<footer> 
 <div class="blog-author-box">
  <img alt="Priyank Ghedia" class="alignleft size-full wp-image-33350" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2021/12/10/Priyank-Ghedia.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Priyank Ghedia</span>
  <br />Priyank is a Senior Security Specialist Solutions Architect focused on threat detection and incident response. Priyank helps customers meet their security visibility and response objectives by building architectures using AWS security services and tools. Before AWS, he spent eight years advising customers on global networking and security operations.
 </div> 
 <div class="blog-author-box">
  <img alt="Matt Meck" class="alignleft size-full wp-image-33350" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/09/mattmeck.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Matt Meck</span>
  <br />Matt is a Sr. Worldwide Security Specialist in New York, covering the AWS Detection and Response domain and advises customers on how they can enhance their security posture and shares feedback to service teams about how AWS can enhance its services. Hiking, competitive soccer, skiing, and being with friends and family are his favorite pass times.
 </div> 
 <div class="blog-author-box">
  <img alt="Anthony Harvey" class="alignleft size-full wp-image-33350" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/06/14/aharveyr.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Anthony Harvey</span>
  <br />Anthony is a Senior Security Specialist Solutions Architect for AWS in the worldwide public sector group. Prior to joining AWS, he was a chief information security officer in local government for half a decade. He has a passion for figuring out how to do more with less and using that mindset to enable customers in their security journey.
 </div> 
</footer>
