---
title: "Accelerate incident response with Amazon Security Lake"
url: "https://aws.amazon.com/blogs/security/accelerate-incident-response-with-amazon-security-lake/"
date: "Tue, 28 May 2024 15:54:05 +0000"
author: "Jerry Chen"
feed_url: "https://aws.amazon.com/blogs/security/tag/amazon-security-lake/feed/"
---
<blockquote>
 <p><strong>September 20, 2024:</strong> Updated the incident response life cycle related wording in the first blog of this series, so to better align with the NIST defined terms.</p>
</blockquote> 
<hr /> 
<p>This blog post is the first of a two-part series that will demonstrate the value of&nbsp;<a href="https://aws.amazon.com/security-lake/" rel="noopener" target="_blank">Amazon Security Lake</a>&nbsp;and how you can use it and other resources to accelerate your incident response (IR) capabilities. Security Lake is a purpose-built data lake that centrally stores your security logs in a common, industry-standard format. In part one, we will first demonstrate the value Security Lake can bring at each phase of the&nbsp;<a href="https://www.nist.gov/privacy-framework/nist-sp-800-61" rel="noopener" target="_blank">National Institute of Standards and Technology (NIST) SP 800-61 Computer Security Incident Handling Guide</a>. We will then demonstrate how you can configure Security Lake in a multi-account deployment by using the&nbsp;<a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/welcome.html" rel="noopener" target="_blank">AWS Security Reference Architecture (AWS SRA)</a>.</p> 
<p>In <a href="https://aws.amazon.com/blogs/security/accelerate-incident-response-with-amazon-security-lake-part-2/" rel="noopener" target="_blank">part two of this series</a>, we’ll walk through an example to show you how to use Security Lake and other AWS services and tools to drive an incident to resolution.</p> 
<p>At&nbsp;<a href="https://aws.amazon.com/" rel="noopener" target="_blank">Amazon Web Services (AWS)</a>, security is our top priority. When security incidents occur, customers need the right capabilities to quickly investigate and resolve them. Security Lake enhances your capabilities, especially during the detection and analysis phases, which can reduce time to resolution and business impact. We also cover&nbsp;<a href="https://docs.aws.amazon.com/wellarchitected/latest/framework/a-incident-response.html" rel="noopener" target="_blank">incident response</a>&nbsp;specifically in the security pillar of the&nbsp;<a href="https://aws.amazon.com/architecture/well-architected/" rel="noopener" target="_blank">AWS Well-Architected Framework</a>, provide&nbsp;<a href="https://docs.aws.amazon.com/whitepapers/latest/aws-security-incident-response-guide/aws-security-incident-response-guide.html" rel="noopener" target="_blank">prescriptive guidance on preparing for and handling incidents</a>, and publish&nbsp;<a href="https://github.com/aws-samples/aws-incident-response-playbooks" rel="noopener" target="_blank">incident response playbooks</a>.</p> 
<h2>Incident response life cycle</h2> 
<p><a href="https://www.nist.gov/privacy-framework/nist-sp-800-61" rel="noopener" target="_blank">NIST SP 800-61</a>&nbsp;describes a set of steps you use to resolve an incident. These include preparation (Phase 1), detection and analysis (Phase 2), containment, eradication and recovery (Phase 3), and finally post-incident activities (Phase 4).</p> 
<p>Figure 1 shows the workflow of incident response defined by NIST SP 800-61. The response flows from Phase 1 through Phase 4, with Phases 2 and 3 often being an iterative process. We will discuss the value of&nbsp;<a href="https://aws.amazon.com/security-lake/" rel="noopener" target="_blank">Security Lake</a>&nbsp;at each phase of the NIST incident response handling process, with a focus on preparation, detection, and analysis.</p> 
<div class="wp-caption aligncenter" id="attachment_35691" style="width: 790px;">
 <img alt="Figure 1: NIST 800-61 incident response life cycle. Source: NIST 800-61" class="size-full wp-image-35691" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/11/img1.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-35691">Figure 1: NIST 800-61 incident response life cycle. Source: NIST 800-61</p>
</div> 
<h3>Phase 1: Preparation</h3> 
<p>Preparation helps you ensure that tools, processes, and people are prepared for incident response. In some cases, preparation can also help you identify systems, networks, and applications that might not be sufficiently secure. For example, you might determine you need certain system logs for incident response, but discover during preparation that those logs are not enabled.</p> 
<p>Figure 2 shows how Security Lake can accelerate the preparation phase during the incident response process. Through native integration with various security data sources from both AWS services and third-party tools, Security Lake simplifies the integration and concentration of security data, which also facilitates training and rehearsal for incident response.</p> 
<div class="wp-caption aligncenter" id="attachment_35692" style="width: 790px;">
 <img alt="Figure 2: Amazon Security Lake data consolidation for IR preparation" class="size-full wp-image-35692" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/11/img2.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-35692">Figure 2: Amazon Security Lake data consolidation for IR preparation</p>
</div> 
<p>Some challenges in the preparation phase include the following:</p> 
<ul> 
 <li>Insufficient incident response planning, training, and rehearsal –Time constraints or insufficient resources can slow down preparation.</li> 
 <li>Complexity of system integration and data sources – An increasing number of security data sources and integration points require additional integration effort, or increase risk that some log sources are not integrated.</li> 
 <li>Centralized log repository for mixed environments – Customers with both on-premises and cloud infrastructure told us that consolidating logs for those mixed environments was a challenge.</li> 
</ul> 
<p>Security Lake can help you deal with these challenges in the following ways:</p> 
<ul> 
 <li>Simplify system integration with security data normalization 
  <ul> 
   <li>Security Lake provides a central repository to store your security log data from various&nbsp;<a href="https://docs.aws.amazon.com/security-lake/latest/userguide/source-management.html" rel="noopener" target="_blank">data sources</a> with less integration effort.</li> 
   <li>Security Lake uses the&nbsp;<a href="https://github.com/ocsf" rel="noopener" target="_blank">Open Cybersecurity Schema Framework (OCSF)</a> standard to ingest log data in a common format, regardless of the source. This common format eases integration with custom&nbsp;<a href="https://docs.aws.amazon.com/security-lake/latest/userguide/custom-sources.html" rel="noopener" target="_blank">data sources</a>&nbsp;and simplifies integration with&nbsp;<a href="https://docs.aws.amazon.com/security-lake/latest/userguide/subscriber-management.html" rel="noopener" target="_blank">data subscribers</a>.</li> 
  </ul> </li> 
 <li>Streamline data consolidation across mixed environments 
  <ul> 
   <li>Security Lake supports multiple log sources, including&nbsp;<a href="https://docs.aws.amazon.com/security-lake/latest/userguide/internal-sources.html" rel="noopener" target="_blank">AWS native services</a> and&nbsp;<a href="https://docs.aws.amazon.com/security-lake/latest/userguide/custom-sources.html" rel="noopener" target="_blank">custom sources</a>, which include third-party partner solutions, other cloud platforms and your on-premises log sources. For example, see&nbsp;<a href="https://aws.amazon.com/blogs/security/get-custom-data-into-amazon-security-lake-through-ingesting-azure-activity-logs/" rel="noopener" target="_blank">this blog post</a>&nbsp;to learn how to ingest Microsoft Azure activity logs into Security Lake.</li> 
  </ul> </li> 
 <li>Facilitate IR planning and testing 
  <ul> 
   <li>Security Lake reduces the undifferentiated heavy lifting needed to get security data into tooling so teams spend less time on configuration and data extract, transform, and load (ETL) work and more time on preparedness.</li> 
   <li>With a purpose-built security data lake and data retention policies that you define, security teams can integrate data-driven decision making into their planning and testing, answering questions such as “which incident handling capabilities do we prioritize?” and running&nbsp;<a href="https://wa.aws.amazon.com/wellarchitected/2020-07-02T19-33-23/wat.concept.gameday.en.html" rel="noopener" target="_blank">Well-Architected game days</a>.</li> 
  </ul> </li> 
</ul> 
<h3>Phases 2 and 3: Detection and Analysis, Containment, Eradication and Recovery</h3> 
<p>The Detection and Analysis phase (Phase 2) should lead you to understand the immediate cause of the incident and what steps need to be taken to contain it. Once contained, it’s critical to fully eradicate the issue. These steps form Phase 3 of the incident response cycle. You want to ensure that those malicious artifacts or exploits are removed from systems and verify that the impacted service has recovered from the incident.</p> 
<p>Figure 3 shows how Security Lake can enable effective detection and analysis. Doing so enables teams to quickly contain, eradicate, and recover from the incident. Security Lake natively integrates with other AWS analytics services, such as&nbsp;<a href="https://aws.amazon.com/athena/" rel="noopener" target="_blank">Amazon Athena</a>,&nbsp;<a href="https://aws.amazon.com/quicksight/" rel="noopener" target="_blank">Amazon QuickSight</a>, and&nbsp;<a href="https://aws.amazon.com/opensearch-service/" rel="noopener" target="_blank">Amazon OpenSearch Service</a>, which makes it easier for your security team to generate insights on the nature of the incident and to take relevant remediation steps.</p> 
<div class="wp-caption aligncenter" id="attachment_35693" style="width: 790px;">
 <img alt="Figure 3: Amazon Security Lake accelerates IR Detection and Analysis, Containment, Eradication, and Recovery" class="size-full wp-image-35693" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/11/img3-1.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-35693">Figure 3: Amazon Security Lake accelerates IR Detection and Analysis, Containment, Eradication, and Recovery</p>
</div> 
<p>Common challenges present in phases 2 and 3 include the following:</p> 
<ul> 
 <li>Challenges generating insights from disparate data sources 
  <ul> 
   <li>Inability to generate insights from security data means teams are less likely to discover an incident, as opposed to having the breach revealed to them by a third party (such as a threat actor).</li> 
   <li>Breaches disclosed by a threat actor might involve higher costs than incidents discovered by the impacted organizations themselves, because typically the unintended access has progressed for longer and impacted more resources and data than if the impacted organization discovered it sooner.</li> 
  </ul> </li> 
 <li>Inconsistency of data visibility and data siloing 
  <ul> 
   <li>Security log data silos may slow IR data analysis because it’s challenging to gather and correlate the necessary information to understand the full scope and impact of an incident. This can lead to delays in identifying the root cause, assessing the damage, and taking remediation steps.</li> 
   <li>Data silos might also mean additional permissions management overhead for administrators.</li> 
  </ul> </li> 
 <li>Disparate data sources add barriers to adopting new technology, such as AI-driven security analytics tools 
  <ul> 
   <li>AI-driven security analysis requires a large amount of security data from various data sources, which might be in disparate formats. Without a centralized security data repository, you might need to make additional effort to ingest and normalize data for model training.</li> 
  </ul> </li> 
</ul> 
<p>Security Lake offers native support for log ingestion for a range of AWS security services, including&nbsp;<a href="https://aws.amazon.com/cloudtrail/" rel="noopener" target="_blank">AWS CloudTrail</a>,&nbsp;<a href="https://aws.amazon.com/security-hub/" rel="noopener" target="_blank">AWS Security Hub</a>, and&nbsp;<a href="https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html" rel="noopener" target="_blank">VPC Flow Logs</a>. Additionally, you can configure Security Lake to ingest external sources. This helps enrich findings and alerts.</p> 
<p>Security Lake addresses the preceding challenges as follows:</p> 
<ul> 
 <li>Unleash security detection capability by centralizing detection data 
  <ul> 
   <li>With a purpose-built security data lake with a standard object schema, organizations can centrally access their security data—AWS and third-party—using the same set of IR tools. This can help you investigate incidents that involve multiple resources and complex timelines, which could require access logs, network logs, and other security findings. For example, use&nbsp;<a href="https://aws.amazon.com/athena/" rel="noopener" target="_blank">Amazon Athena</a> to query all your security data. You can also build a&nbsp;<a href="https://aws.amazon.com/blogs/security/how-to-visualize-amazon-security-lake-findings-with-amazon-quicksight/" rel="noopener" target="_blank">centralized security finding dashboard with Amazon QuickSight</a>.</li> 
  </ul> </li> 
 <li>Reduce management burden 
  <ul> 
   <li>With Security Lake, permissions complexity is reduced. You use the same access controls in&nbsp;<a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> to make sure that only the right people and systems have access to sensitive security data.</li> 
  </ul> </li> 
</ul> 
<p>See&nbsp;<a href="https://aws.amazon.com/blogs/security/generate-machine-learning-insights-for-amazon-security-lake-data-using-amazon-sagemaker/" rel="noopener" target="_blank">this blog post</a>&nbsp;for more details on generating machine learning insights for Security Lake data by using&nbsp;<a href="https://aws.amazon.com/pm/sagemaker" rel="noopener" target="_blank">Amazon SageMaker</a>.</p> 
<h3>Phase 4: Post-Incident Activity</h3> 
<p>Continuous improvement helps customers to further develop their IR capabilities. Teams should integrate lessons learned into their tools, policies, and processes. You decide on&nbsp;<a href="https://docs.aws.amazon.com/security-lake/latest/userguide/lifecycle-management.html" rel="noopener" target="_blank">lifecycle policies for your security data</a>. You can then retroactively review event data for insight and to support lessons learned. You can also&nbsp;<a href="https://aws.amazon.com/blogs/security/how-to-share-security-telemetry-per-ou-using-amazon-security-lake-and-aws-lake-formation/" rel="noopener" target="_blank">share security telemetry</a>&nbsp;at levels of granularity you define. Your organization can then establish distributed data views for forensic purposes and other purposes, while enforcing least privilege for data governance. </p> 
<p>Figure 4 shows how Security Lake can accelerate the post-incident activity phase during the incident response process. Security Lake natively integrates with&nbsp;<a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a>&nbsp;to enable data sharing across various OUs within the organization, which further unleashes the power of machine learning to automatically create insights for incident response.</p> 
<div class="wp-caption aligncenter" id="attachment_35694" style="width: 790px;">
 <img alt="Figure 4: Security Lake accelerates post-incident activity" class="size-full wp-image-35694" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/11/img4.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-35694">Figure 4: Security Lake accelerates post-incident activity</p>
</div> 
<p>Having covered some advantages of working with your data in Security Lake, we will now demonstrate best practices for getting Security Lake set up.</p> 
<h2>Setting up for success with Security Lake</h2> 
<p>Most of the customers we work with run multiple AWS accounts, usually with&nbsp;<a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a>. With that in mind, we’re going to show you how to set up Security Lake and related tooling in line with guidance in the&nbsp;<a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/welcome.html" rel="noopener" target="_blank">AWS Security Reference Architecture (AWS SRA)</a>. The AWS SRA provides guidance on how to deploy AWS security services in a multi-account environment. You will have one AWS account for security tooling and a different account to centralize log storage. You’ll run Security Lake in this log storage account.</p> 
<p>If you just want to use Security Lake in a standalone account, follow&nbsp;<a href="https://docs.aws.amazon.com/security-lake/latest/userguide/getting-started.html" rel="noopener" target="_blank">these instructions</a>.</p> 
<h3>Set up Security Lake in your logging account</h3> 
<p>Most of the instructions we link to in this section describe the process using either the console or AWS CLI tools. Where necessary, we’ve described the console experience for illustrative purposes.</p> 
<p>The&nbsp;<a href="https://docs.aws.amazon.com/security-lake/latest/userguide/security-iam-awsmanpol.html#security-iam-awsmanpol-AmazonSecurityLakeAdministrator" rel="noopener" target="_blank">AmazonSecurityLakeAdministrator</a>&nbsp;AWS managed IAM policy grants the permissions needed to set up Security Lake and related services. Note that you may want to further refine permissions, or remove that managed policy after Security Lake and the related services are set up and running.</p> 
<p><strong>To set up Security Lake in your logging account</strong></p> 
<ol> 
 <li>Note down the AWS account number that will be your&nbsp;<a href="https://docs.aws.amazon.com/security-lake/latest/userguide/multi-account-management.html#designated-admin" rel="noopener" target="_blank">delegated administrator account</a>. This will be your centralized archive logs account. In the AWS Management Console, sign in to your Organizations management account and set up delegated administration for Security Lake.</li> 
 <li>Sign in to the delegated administrator account, go to the Security Lake console, and choose&nbsp;Get started. Then follow&nbsp;<a href="https://docs.aws.amazon.com/security-lake/latest/userguide/getting-started.html#get-started-console" rel="noopener" target="_blank">these instructions from the Security Lake User Guide</a>. While you’re setting this up, note the following specific guidance (this will make it easier to follow <a href="https://aws.amazon.com/blogs/security/accelerate-incident-response-with-amazon-security-lake-part-2/" rel="noopener" target="_blank">the second blog post in this series</a>): <p>Define source objective:&nbsp;For&nbsp;Sources to ingest, we recommend that you select&nbsp;Ingest the AWS default sources. However, if you want to include S3 data events, you’ll need to select&nbsp;Ingest specific AWS sources&nbsp;and then select&nbsp;CloudTrail – S3 data events. Note that we use these events for responding to the incident in <a href="https://aws.amazon.com/blogs/security/accelerate-incident-response-with-amazon-security-lake-part-2/" rel="noopener" target="_blank">blog post part 2</a>, when we really drill down into user activity.</p> <p>Figure 5 shows the configuration of sources to ingest in Security Lake.</p> 
  <div class="wp-caption aligncenter" id="attachment_35695" style="width: 750px;">
   <img alt="Figure 5: Sources to ingest in Security Lake" class="size-full wp-image-35695" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/11/img5.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35695">Figure 5: Sources to ingest in Security Lake</p>
  </div> <p>We recommend leaving the other settings on this page as they are.</p> <p>Define target objective: We recommend that you choose&nbsp;Add rollup Region&nbsp;and add multiple AWS Regions to a designated rollup Region. The rollup Region is the one to which you will consolidate logs. The contributing Region is the one that will contribute logs to the rollup Region.</p> <p>Figure 6 shows how to select the rollup regions.</p> 
  <div class="wp-caption aligncenter" id="attachment_35696" style="width: 750px;">
   <img alt="Figure 6: Select rollup Regions" class="size-full wp-image-35696" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/11/img6.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35696">Figure 6: Select rollup Regions</p>
  </div> </li> 
</ol> 
<p>You now have Security Lake enabled, and in the background, additional services such as&nbsp;<a href="https://aws.amazon.com/lake-formation/" rel="noopener" target="_blank">AWS Lake Formation</a>&nbsp;and&nbsp;<a href="https://aws.amazon.com/glue/" rel="noopener" target="_blank">AWS Glue</a>&nbsp;have been configured to organize your Security Lake data.</p> 
<p>Now you need to&nbsp;<a href="https://docs.aws.amazon.com/security-lake/latest/userguide/subscriber-query-access.html" rel="noopener" target="_blank">configure a subscriber with query access</a>&nbsp;so that you can query your Security Lake data. Here are a few recommendations:</p> 
<ol> 
 <li>Subscribers are specific to a Region, so you want to make sure that you set up your subscriber in the same Region as your rollup Region.</li> 
 <li>You will also set up an External ID. This is a value you define, and it’s used by the IAM role to prevent the&nbsp;<a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/confused-deputy.html" rel="noopener" target="_blank">confused deputy problem</a>. Note that the subscriber will be your security tooling account.</li> 
 <li>You will select&nbsp;Lake Formation for&nbsp;Data access, which will create shares in&nbsp;<a href="https://aws.amazon.com/ram/" rel="noopener" target="_blank">AWS Resource Access Manager (AWS RAM)</a>&nbsp;that will be shared with the account that you specified in&nbsp;Subscriber credentials.</li> 
 <li>If you’ve already set up Security Lake at some time in the past, you should select&nbsp;Specific log and event sources and confirm the source and version you want the subscriber to access. If it’s a new implementation, we recommend using version 2.0 or greater.</li> 
 <li>There’s a note in the console that says the subscribing account will need to accept the RAM resource shares. However, if you’re using AWS Organizations, you don’t need to do that; the resource share will already list a status of&nbsp;Active when you select the&nbsp;Shared with me &gt;&gt; Resource shares&nbsp;in the subscriber (security tooling) account RAM console.</li> 
</ol> 
<blockquote>
 <p><strong>Note:</strong>&nbsp;If you prefer a visual guide, you can refer to&nbsp;<a href="https://www.youtube.com/watch?v=fKGhscpwN-k" rel="noopener" target="_blank">this video</a>&nbsp;to set up Security Lake in AWS Organizations.</p>
</blockquote> 
<h3>Set up Amazon Athena and AWS Lake Formation in the security tooling account</h3> 
<p>If you go to Athena in your security tooling account, you won’t see your Security Lake tables yet because the tables are shared from the Security Lake account. Although&nbsp;<a href="https://docs.aws.amazon.com/lake-formation/latest/dg/resource-links-about.html" rel="noopener" target="_blank">services such as Amazon Athena can’t directly access databases or tables across accounts</a>, the use of resource links overcomes this challenge.</p> 
<p><strong>To set up Athena and Lake Formation</strong></p> 
<ol> 
 <li>Go to the Lake Formation console in the security tooling account and follow the&nbsp;<a href="https://docs.aws.amazon.com/lake-formation/latest/dg/create-resource-link-table.html" rel="noopener" target="_blank">instructions to create resource links for the shared Security Lake tables</a>. You’ll most likely use the&nbsp;Default database and will see your tables there. The table names in that database start with&nbsp;amazon_security_lake_table. You should expect to see about eight tables there. <p>Figure 7 shows the shared tables in the Lake Formation service console.</p> 
  <div class="wp-caption aligncenter" id="attachment_35697" style="width: 750px;">
   <img alt="Figure 7: Shared tables in Lake Formation" class="size-full wp-image-35697" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/11/img7.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35697">Figure 7: Shared tables in Lake Formation</p>
  </div> <p>You will need to create resource links for each table, as described in the instructions from the Lake Formation Developer Guide.</p> <p>Figure 8 shows the resource link creation process.</p> 
  <div class="wp-caption aligncenter" id="attachment_35698" style="width: 750px;">
   <img alt="Figure 8: Creating resource links" class="size-full wp-image-35698" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/09/11/img8.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-35698">Figure 8: Creating resource links</p>
  </div> </li> 
 <li>Next, go to Amazon Athena in the same Region. If Athena is not set up,&nbsp;<a href="https://docs.aws.amazon.com/athena/latest/ug/getting-started.html" rel="noopener" target="_blank">follow the instructions to get it set up for SQL queries</a>. Note that you won’t need to create a database—you’re going to use the “default” database that already exists. Select it from the&nbsp;Database drop-down menu in the&nbsp;Query editor view.</li> 
 <li>In the&nbsp;Tables section, you should see all your Security Lake tables (represented by whatever names you gave them when you created the resource links in step 1, earlier).</li> 
</ol> 
<h3>Get your incident response playbooks ready</h3> 
<p>Incident response playbooks are an important tool that enable responders to work more effectively and consistently, and enable the organization to get incidents resolved more quickly. We’ve created some&nbsp;<a href="https://github.com/aws-samples/aws-incident-response-playbooks" rel="noopener" target="_blank">ready-to-go templates</a>&nbsp;to get you started. You can further customize these templates to meet your needs. In <a href="https://aws.amazon.com/blogs/security/accelerate-incident-response-with-amazon-security-lake-part-2/" rel="noopener" target="_blank">part two of this post</a>, you’ll be using the&nbsp;<a href="https://github.com/aws-samples/aws-incident-response-playbooks/blob/master/playbooks/IRP-DataAccess.md" rel="noopener" target="_blank">Unintended Data Access to an Amazon Simple Storage Service (Amazon S3) bucket</a>&nbsp;playbook to resolve an incident. You can download that playbook so that you’re ready to follow it to get that incident resolved.</p> 
<h2>Conclusion</h2> 
<p>This is the first post in a two-part series about accelerating security incident response with Security Lake. We highlighted common challenges that decelerate customers’ incident responses across the phases outlined by NIST SP 800-61 and how Security Lake can help you address those challenges. We also showed you how to set up Security Lake and related services for incident response.</p> 
<p>In <a href="https://aws.amazon.com/blogs/security/accelerate-incident-response-with-amazon-security-lake-part-2/" rel="noopener" target="_blank">the second part of this series</a>, we’ll walk through a specific security incident—unintended data access—and share prescriptive guidance on using Security Lake to accelerate your incident response process.</p> 
<p>If you have feedback about this post, submit comments in the&nbsp;Comments&nbsp;section below. If you have questions about this post,&nbsp;<a href="https://console.aws.amazon.com/support/home" rel="noopener" target="_blank">contact AWS Support</a>.</p> 
<p><strong>Want more AWS Security news? Follow us on <a href="https://twitter.com/AWSsecurityinfo" rel="noopener noreferrer" target="_blank" title="Twitter">Twitter</a>.</strong></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Jerry Chen" class="aligncenter size-full wp-image-34349" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/05/23/jerchen.jpg" width="120" /> 
  </div> 
  <p class="lb-h4">Jerry Chen</p> 
  <p>Jerry is currently a Senior Cloud Optimization Success Solutions Architect at AWS. He focuses on cloud security and operational architecture design for AWS customers and partners. You can follow Jerry on <a href="https://www.linkedin.com/in/zeyun-jerry-chen-89002b35/" rel="noopener" target="_blank">LinkedIn</a>.</p> 
  <p></p>
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Frank Phillis" class="aligncenter size-full wp-image-34348" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/05/23/phillisf.jpg" width="120" /> 
  </div> 
  <p class="lb-h4">Frank Phillis</p> 
  <p>Frank is a Senior Solutions Architect (Security) at AWS. He enables customers to get their security architecture right. Frank specializes in cryptography, identity, and incident response. He’s the creator of the popular AWS Incident Response playbooks and regularly speaks at security events. When not thinking about tech, Frank can be found with his family, riding bikes, or making music.</p> 
  <p></p>
 </div> 
</footer>
