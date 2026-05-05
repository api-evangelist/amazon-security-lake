---
title: "Patterns for consuming custom log sources in Amazon Security Lake"
url: "https://aws.amazon.com/blogs/security/patterns-for-consuming-custom-log-sources-in-amazon-security-lake/"
date: "Mon, 15 Jul 2024 15:57:50 +0000"
author: "Pratima Singh"
feed_url: "https://aws.amazon.com/blogs/security/tag/amazon-security-lake/feed/"
---
<p>As security best practices have evolved over the years, so has the range of security telemetry options. Customers face the challenge of navigating through security-relevant telemetry and log data produced by multiple tools, technologies, and vendors while trying to monitor, detect, respond to, and mitigate new and existing security issues. In this post, we provide you with three patterns to centralize the ingestion of log data into <a href="https://aws.amazon.com/security-lake/" rel="noopener" target="_blank">Amazon Security Lake</a>, regardless of the source. You can use the patterns in this post to help streamline the extract, transform and load (ETL) of security log data so you can focus on analyzing threats, detecting anomalies, and improving your overall security posture. We also provide the corresponding code and mapping for the patterns in the <a href="https://github.com/aws-samples/amazon-security-lake-transformation-library" rel="noopener" target="_blank">amazon-security-lake-transformation-library</a>.</p> 
<p>Security Lake automatically centralizes security data into a purpose-built data lake in your organization in <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a>. You can use Security Lake to collect logs from multiple sources, including natively supported AWS services, Software-as-a-Service (SaaS) providers, on-premises systems, and cloud sources.</p> 
<p>Centralized log collection in a distributed and hybrid IT environment can help streamline the process, but log sources generate logs in disparate formats. This leads to security teams spending time building custom queries based on the schemas of the logs and events before the logs can be correlated for effective incident response and investigation. You can use the patterns presented in this post to help build a scalable and flexible data pipeline to transform log data using <a href="https://github.com/ocsf" rel="noopener" target="_blank">Open Cybersecurity Schema Framework (OCSF)</a> and stream the transformed data into Security Lake.</p> 
<h2>Security Lake custom sources</h2> 
<p>You can configure <a href="https://docs.aws.amazon.com/security-lake/latest/userguide/custom-sources.html" rel="noopener" target="_blank">custom sources</a> to bring your security data into Security Lake. Enterprise security teams spend a significant amount of time discovering log sources in various formats and correlating them for security analytics. Custom source configuration helps security teams centralize distributed and disparate log sources in the same format. Security data in Security Lake is centralized and normalized into OCSF and compressed in open source, columnar <a href="https://parquet.apache.org/" rel="noopener" target="_blank">Apache Parquet</a> format for storage optimization and query efficiency. Having log sources in a centralized location and in a single format can significantly improve your security team’s timelines when performing security analytics. With Security Lake, you retain full ownership of the security data stored in your account and have complete freedom of choice for analytics. Before discussing creating custom sources in detail, it’s important to understand the OCSF core schema, which will help you map attributes and build out the transformation functions for the custom sources of your choice.</p> 
<h2>Understanding the OCSF</h2> 
<p>OCSF is a vendor-agnostic and open source standard that you can use to address the complex and heterogeneous nature of security log collection and analysis. You can extend and adapt the OCSF <a href="https://schema.ocsf.io/" rel="noopener" target="_blank">core security schema</a> for a range of use cases in your IT environment, application, or solution while complementing your existing security standards and processes. As of this writing, the most recent major version release of the schema is v1.2.0, which contains six <a href="https://schema.ocsf.io/1.0.0/categories?extensions=" rel="noopener" target="_blank">categories</a>: System Activity, Findings, Identity and Access Management, Network Activity, Discovery, and Application Activity. Each category consists of different <a href="https://schema.ocsf.io/1.0.0/classes?extensions=" rel="noopener" target="_blank">classes</a> based on the type of activity, and each class has a unique class UID. For example, File System Activity has a class UID of 1001.</p> 
<p>As of this writing, Security Lake (version 1) supports <a href="https://schema.ocsf.io/1.1.0" rel="noopener" target="_blank">OCSF v1.1.0</a>. As Security Lake continues to support newer releases of OCSF, you can continue to use the patterns from this post. However, you should revisit the mappings in case there’s a change in the classes you’re using.</p> 
<h2>Prerequisites</h2> 
<p>You must have the following prerequisites for log ingestion into Amazon Security Lake. Each pattern has a sub-section of prerequisites that are relevant to the data pipeline for the custom log source.</p> 
<ol> 
 <li><strong>AWS Organizations is configured your AWS environment</strong>. <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_tutorials_basic.html" rel="noopener" target="_blank">AWS Organizations</a> is an AWS account management service that provides account management and consolidated billing capabilities that you can use to consolidate multiple AWS accounts and manage them centrally.</li> 
 <li><strong>Security Lake is activated and a </strong><a href="https://docs.aws.amazon.com/security-lake/latest/userguide/multi-account-management.html" rel="noopener" target="_blank">delegated administrator is configured</a>. 
  <ol> 
   <li>Open the AWS Management Console and navigate to AWS Organizations. Set up an organization with a <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/log-archive.html" rel="noopener" target="_blank">Log Archive account</a>. The Log Archive account should be used as the delegated Security Lake administrator account where you will configure Security Lake. For more information on deploying the full complement of AWS security services in a multi-account environment, see <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/welcome.html" rel="noopener" target="_blank">AWS Security Reference Architecture</a>.</li> 
   <li>Configure permissions for the Security Lake administrator access by using an <a href="https://docs.aws.amazon.com/security-lake/latest/userguide/multi-account-management.html" rel="noopener" target="_blank">AWS Identity and Access Management (IAM) role</a>. This role should be used by your security teams to administer Security Lake configuration, including managing custom sources.</li> 
   <li>Enable Security Lake in the AWS Region of your choice in the Log Archive account. When you configure Security Lake, you can define your collection objectives, including log sources, the Regions that you want to collect the log sources from, and the lifecycle policy you want to assign to the log sources. Security Lake uses <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> as the underlying storage for the log data. Amazon S3 is an object storage service offering industry-leading scalability, data availability, security, and performance. S3 is built to store and retrieve data from practically anywhere. Security Lake creates and configures individual S3 buckets in each Region identified in the collection objectives in the Log Archive account.</li> 
  </ol> </li> 
</ol> 
<h2>Transformation library</h2> 
<p>With this post, we’re publishing the <a href="https://github.com/aws-samples/amazon-security-lake-transformation-library" rel="noopener" target="_blank">amazon-security-lake-transformation-library</a> project to assist with mapping custom log sources. The transformation code is deployed as an <a href="https://aws.amazon.com/lambda" rel="noopener" target="_blank">AWS Lambda</a> function. You will find the deployment automation using AWS CloudFormation in the <a href="https://github.com/aws-samples/amazon-security-lake-transformation-library/blob/main/template.yaml" rel="noopener" target="_blank">solution repository</a>.</p> 
<p>To use the transformation library, you should understand how to build the mapping configuration file. The mapping configuration file holds mapping information from raw events to OCSF formatted logs. The transformation function builds the OCSF formatted logs based on the attributes mapped in the file and streams them to the Security Lake S3 buckets.</p> 
<p>The solution deployment is a four-step process:</p> 
<ol> 
 <li>Update mapping configuration</li> 
 <li>Add a custom source in Security Lake</li> 
 <li>Deploy the log transformation infrastructure</li> 
 <li>Update the default <a href="https://aws.amazon.com/glue" rel="noopener" target="_blank">AWS Glue</a> crawler</li> 
</ol> 
<p>The mapping configuration file is a JSON-formatted file that’s used by the transformation function to evaluate the attributes of the raw logs and map them to the relevant OCSF class attributes. The configuration is based on the mapping identified in Table 3 (<em>File System Activity class mapping) </em>and extended to the <em>Process Activity</em> class. The file uses the $. notation to identify attributes that the transformation function should evaluate from the event.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{  "custom_source_events": {
        "source_name": "windows-sysmon",
        "matched_field": "$.EventId",
        "ocsf_mapping": {
            "1": {
                "schema": "process_activity",
                "schema_mapping": {   
                    "metadata": {
                        "profiles": "host",
                        "version": "v1.1.0",
                        "product": {
                            "name": "System Monitor (Sysmon)",
                            "vendor_name": "Microsoft Sysinternals",
                            "version": "v15.0"
                        }
                    },
                    "severity": "Informational",
                    "severity_id": 1,
                    "category_uid": 1,
                    "category_name": "System Activity",
                    "class_uid": 1007,
                    "class_name": "Process Activity",
                    "type_uid": 100701,
                    "time": "$.Description.UtcTime",
                    "activity_id": {
                        "enum": {
                            "evaluate": "$.EventId",
                            "values": {
                                "1": 1,
                                "5": 2,
                                "7": 3,
                                "10": 3,
                                "19": 3,
                                "20": 3,
                                "21": 3,
                                "25": 4
                            },
                            "other": 99
                        }
                    },
                    "actor": {
                        "process": "$.Description.Image"
                    },
                    "device": {
                        "type_id": 6,
                        "instance_uid": "$.UserDefined.source_instance_id"
                    },
                    "process": {
                        "pid": "$.Description.ProcessId",
                        "uid": "$.Description.ProcessGuid",
                        "name": "$.Description.Image",
                        "user": "$.Description.User",
                        "loaded_modules": "$.Description.ImageLoaded"
                    },
                    "unmapped": {
                        "rulename": "$.Description.RuleName"
                    }
                    
                }
            },
…
…
…
        }
    }
}</code></pre> 
</div> 
<p>Configuration in the mapping file is stored under the <span style="font-family: courier;">custom_source_events</span> key. You must keep the value for the key <span style="font-family: courier;">source_name</span> the same as the name of the custom source you add for Security Lake. The <span style="font-family: courier;">matched_field</span> is the key that the transformation function uses to iterate over the log events. The iterator (<span style="font-family: courier;">1</span>), in the preceding snippet, is the Sysmon event ID and the data structure that follows is the OCSF attribute mapping.</p> 
<p>Some OCSF attributes are of an <span style="font-family: courier;">Object</span> data type with a map of pre-defined values based on the event signature such as <span style="font-family: courier;">activity_id</span>. You represent such attributes in the mapping configuration as shown in the following example:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">'activity_id': {
    'enum': {
        'evaluate': '$.EventId',
        'values': {
              2: 6,
              11: 1,
              15: 1,
              24: 3,
              23: 4
         },
         'other': 99
      }
 }</code></pre> 
</div> 
<p>In the preceding snippet, you can see the words <span style="font-family: courier;">enum</span> and <span style="font-family: courier;">evaluate</span>. These keywords tell the underlying mapping function that the result will be the value from the map defined in <span style="font-family: courier;">values</span> and the key to evaluate is the <span style="font-family: courier;">EventId</span>, which is listed as the value of the <span style="font-family: courier;">evaluate</span> key. You can build your own transformation function based on your custom sources and mapping or you can extend the function provided in this post.</p> 
<h2>Pattern 1: Log collection in a hybrid environment using Kinesis Data Streams</h2> 
<p>The first pattern we discuss in this post is the collection of log data from hybrid sources such as operating system logs collected from Microsoft Windows operating systems using <a href="https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon" rel="noopener" target="_blank">System Monitor (Sysmon)</a>. Sysmon is a service that monitors and logs system activity to the Windows event log. It’s one of the log collection tools used by customers in a Windows Operating System environment because it provides detailed information about process creations, network connections, and file modifications This host-level information can prove crucial during threat hunting scenarios and security analytics.</p> 
<h3>Solution overview</h3> 
<p>The solution for this pattern uses<a href="https://aws.amazon.com/kinesis/data-streams/" rel="noopener" target="_blank"> Amazon Kinesis Data Streams</a> and Lambda to implement the schema transformation. Kinesis Data Streams is a serverless streaming service that makes it convenient to capture and process data at any scale. You can configure stream consumers—such as Lambda functions—to operate on the events in the stream and convert them into required formats—such as OCSF—for analysis without maintaining processing infrastructure. Lambda is a serverless, event-driven compute service that you can use to run code for a range of applications or backend services without provisioning or managing servers. This solution integrates Lambda with Kinesis Data Streams to launch transformation tasks on events in the stream.</p> 
<p>To stream Sysmon logs from the host, you use <a href="https://docs.aws.amazon.com/kinesis-agent-windows/latest/userguide/what-is-kinesis-agent-windows.html" rel="noopener" target="_blank">Amazon Kinesis Agent for Microsoft Windows</a>. You can run this agent on fleets of Windows servers hosted on-premises or in your cloud environment.</p> 
<div class="wp-caption aligncenter" id="attachment_34876" style="width: 790px;">
 <img alt="Figure 1: Architecture diagram for Sysmon event logs custom source" class="size-full wp-image-34876" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/07/10/img1-4.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-34876">Figure 1: Architecture diagram for Sysmon event logs custom source</p>
</div> 
<p>Figure 1 shows the interaction of services involved in building the custom source ingestion. The servers and instances generating logs run the Kinesis Agent for Windows to stream log data to the Kinesis Data Stream which invokes a consumer Lambda function. The Lambda function transforms the log data into OCSF based on the mapping provided in the configuration file and puts the transformed log data into Security Lake S3 buckets. We cover the solution implementation later in this post, but first let’s review how you can map Sysmon event streaming through Kinesis Data Streams into the relevant OCSF classes. You can deploy the infrastructure using the <a href="https://aws.amazon.com/serverless/sam/" rel="noopener" target="_blank">AWS Serverless Application Model (AWS SAM)</a> <a href="https://github.com/aws-samples/amazon-security-lake-transformation-library/blob/main/template.yaml" rel="noopener" target="_blank">template</a> provided in the solution code. AWS SAM is an extension of the <a href="https://aws.amazon.com/cli" rel="noopener" target="_blank">AWS Command Line Interface (AWS CLI)</a>, which adds functionality for building and testing applications using Lambda functions.</p> 
<h3>Mapping</h3> 
<p>Windows Sysmon events map to various OCSF classes. To build the transformation of the Sysmon events, work through the mapping of events with relevant OCSF classes. The latest version of Sysmon (v15.14) defines 30 events including a catch-all error event.</p> 
<table style="border-collapse: collapse; border: 1px solid #808080;" width="100%"> 
 <tbody>
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%"><strong>Sysmon eventID</strong></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%"><strong>Event detail</strong></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%"><strong>Mapped OCSF class </strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">1</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">Process creation</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Process Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">2</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">A process changed a file creation time</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">File System Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">3</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">Network connection</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Network Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">4</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">Sysmon service state changed</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Process Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">5</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">Process terminated</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Process Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">6</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">Driver loaded</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Kernel Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">7</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">Image loaded</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Process Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">8</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">CreateRemoteThread</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Network Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">9</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">RawAccessRead</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Memory Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">10</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">ProcessAccess</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Process Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">11</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">FileCreate</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">File System Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">12</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">RegistryEvent (Object create and delete)</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">File System Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">13</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">RegistryEvent (Value set)</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">File System Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">14</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">RegistryEvent (Key and value rename)</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">File System Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">15</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">FileCreateStreamHash</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">File System Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">16</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">ServiceConfigurationChange</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Process Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">17</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">PipeEvent (Pipe created)</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">File System Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">18</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">PipeEvent (Pipe connected)</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">File System Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">19</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">WmiEvent (WmiEventFilter activity detected)</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Process Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">20</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">WmiEvent (WmiEventConsumer activity detected)</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Process Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">21</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">WmiEvent (WmiEventConsumerToFilter activity detected)</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Process Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">22</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">DNSEvent (DNS query)</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">DNS Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">23</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">FileDelete (File delete archived)</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">File System Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">24</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">ClipboardChange (New content in the clipboard)</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">File System Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">25</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">ProcessTampering (Process image change)</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Process Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">26</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">FileDeleteDetected (File delete logged)</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">File System Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">27</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">FileBlockExecutable</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">File System Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">28</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">FileBlockShredding</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">File System Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">29</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">FileExecutableDetected</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">File System Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="20%">255</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">Sysmon error</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Process Activity</td> 
  </tr> 
 </tbody>
</table> 
<p align="center"><em>Table 1: Sysmon event mapping with OCSF (v1.1.0) classes</em></p> 
<p>Start by mapping the Sysmon events to the relevant OCSF classes in plain text as shown in Table 1 before adding them to the mapping configuration file for the transformation library. This mapping is flexible; you can choose to map an event to a different event class depending on the standard defined within the security engineering function. Based on our mapping, Table 1 indicates that a majority of the events reported by Sysmon align with the <a href="https://schema.ocsf.io/1.1.0/classes/file_activity?extensions=" rel="noopener" target="_blank">File System Activity</a> or the <a href="https://schema.ocsf.io/1.1.0/classes/process_activity?extensions=" rel="noopener" target="_blank">Process Activity</a> class. Registry events map better with the <em>Registry Key Activity</em> and <em>Registry Value Activity</em> classes, but these classes are deprecated in OCSF v1.0.0, so we recommend using <em>File System Activity</em> instead of registry events for compatibility with future versions of OCSF. You can be selective about the events captured and reported by Sysmon by altering the Sysmon configuration file. For this post, we’re using the <a href="https://github.com/olafhartong/sysmon-modular/blob/master/sysmonconfig.xml" rel="noopener" target="_blank">sysmonconfig.xml</a> published in the <a href="https://github.com/olafhartong/sysmon-modular" rel="noopener" target="_blank">sysmon-modular</a> project. The project provides a modular configuration along with publishing tactics, techniques, and procedures (TTPs) with Sysmon events to help in <a href="https://www.mitre.org/news-insights/publication/ttp-based-hunting" rel="noopener" target="_blank">TTP-based threat hunting</a> use cases. If you have your own curated Sysmon configuration, you can use that. While this solution offers mapping advice, if you’re using your own Sysmon configuration, you should make sure that you’re mapping the relevant attributes using this solution as a guide. As a best practice, mapping should be non-destructive to keep your information after the OCSF transformation. If there are attributes in the log data that you cannot map to an available attribute in the OCSF class, then you should use the <span style="font-family: courier;">unmapped</span> attribute to collect all such information. In this pattern, <span style="font-family: courier;">RuleName</span> captures the TTPs associated with the Sysmon event, because TTPs don’t map to a specific attribute within OCSF.</p> 
<p>Across all classes in OCSF, there are some common attributes that are mandatory. The common mandatory attributes are mapped shown in Table 2. You need to set these attributes regardless of the OCSF class you’re transforming the log data to.</p> 
<table align="center" style="border-collapse: collapse; border: 1px solid #808080;" width="75%"> 
 <tbody>
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%"><strong>OCSF</strong></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%"><strong>Raw</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">metadata.profiles</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">[host]</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">metadata.version</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">v1.1.0</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">metadata.product.name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">System Monitor (Sysmon)</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">metadata.product.vendor_name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Microsoft Sysinternals</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">metadata.product.version</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">v15.14</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">severity</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Informational</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">severity_id</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">1</td> 
  </tr> 
 </tbody>
</table> 
<p align="center"><em>Table 2: Mapping mandatory attributes</em></p> 
<p>Each OCSF <a href="https://schema.ocsf.io/1.1.0/classes?extensions=" rel="noopener" target="_blank">class has its own schema</a>, which is extendable. After mapping the common attributes, you can map the attributes in the File System Activity class relevant to the log information. Some of the attribute values can be derived from a map of options standardised by the OCSF schema. One such attribute is <span style="font-family: courier;">Activity ID</span>. Depending on the type of activity performed on the file, you can assign a value from the pre-defined set of values in the schema such as 0 if the event activity is unknown, 1 if a file was created, 2 if a file was read, and so on. You can find more information on standard attribute maps in <a href="https://schema.ocsf.io/1.1.0/classes/file_activity?extensions=" rel="noopener" target="_blank">File System Activity, System Activity Category</a>.</p> 
<h4>File system activity mapping example</h4> 
<p>The following is a sample file creation event reported by Sysmon:</p> 
<table style="border-collapse: collapse; border: 0px; font-style: italic;" width="75%"> 
 <tbody>
  <tr> 
   <td style="border: 0px;" width="25%">File created:</td> 
   <td style="border: 0px;" width="75%"></td> 
  </tr> 
  <tr> 
   <td style="border: 0px;" width="25%">RuleName:</td> 
   <td style="border: 0px;" width="75%">technique_id=T1574.010,technique_name=Services File Permissions Weakness</td> 
  </tr> 
  <tr> 
   <td style="border: 0px;" width="25%">UtcTime:</td> 
   <td style="border: 0px;" width="75%">2023-10-03 23:50:22.438</td> 
  </tr> 
  <tr> 
   <td style="border: 0px;" width="25%">ProcessGuid:</td> 
   <td style="border: 0px;" width="75%">{78c8aea6-5a34-651b-1900-000000005f01}</td> 
  </tr> 
  <tr> 
   <td style="border: 0px;" width="25%">ProcessId:</td> 
   <td style="border: 0px;" width="75%">1128</td> 
  </tr> 
  <tr> 
   <td style="border: 0px;" width="25%">Image:</td> 
   <td style="border: 0px;" width="75%">C:\Windows\System32\svchost.exe</td> 
  </tr> 
  <tr> 
   <td style="border: 0px;" width="25%">TargetFilename:</td> 
   <td style="border: 0px;" width="75%">C:\Windows\ServiceState\EventLog\Data\lastalive1.dat</td> 
  </tr> 
  <tr> 
   <td style="border: 0px;" width="25%">CreationUtcTime:</td> 
   <td style="border: 0px;" width="75%">2023-10-03 00:04:00.984</td> 
  </tr> 
  <tr> 
   <td style="border: 0px;" width="25%">User:</td> 
   <td style="border: 0px;" width="75%">NT AUTHORITY\LOCAL SERVICE</td> 
  </tr> 
 </tbody>
</table> 
<p>When the event is streamed to the Kinesis Data Streams stream, the Kinesis Agent can be used to enrich the event. We’re enriching the event with <span style="font-family: courier;">source_instance_id</span> using <a href="https://github.com/aws-samples/amazon-security-lake-transformation-library/blob/63aac3c731935b0efca3c6797e58224480c1caa9/windows-sysmon/kinesis_agent_configuration.json&quot; \l &quot;L14" rel="noopener" target="_blank">ObjectDecoration</a> configured in the <a href="https://github.com/aws-samples/amazon-security-lake-transformation-library/blob/main/windows-sysmon/kinesis_agent_configuration.json" rel="noopener" target="_blank">agent configuration file</a>.</p> 
<p>Because the transformation Lambda function reads from a Kinesis Data Stream, we use the event information from the stream to map the attributes of the File System Activity class. The following mapping table has attributes mapped to the values based on OCSF requirements, the values enclosed in brackets (&lt;&gt;) will come from the event. In the solution implementation section for this pattern, you learn about the transformation Lambda function and mapping implementation for a sample set of events.</p> 
<table style="border-collapse: collapse; border: 1px solid #808080;" width="100%"> 
 <tbody>
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="25%"><strong>OCSF</strong></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="75%"><strong>Raw</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="25%">category_uid</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="75%">1</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="25%">category_name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="75%">System Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="25%">class_uid</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="75%">1001</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="25%">class_name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="75%">File System Activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="25%">time</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="75%">&lt;UtcTime&gt;</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="25%">activity_id</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="75%">1</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="25%">actor</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="75%">{process: {name: &lt;Image&gt;}}</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="25%">device</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="75%">{type_id: 6}</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="25%">unmapped</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="75%">{pid: &lt;ProcessId&gt;, uid: &lt;ProcessGuid&gt;, name: &lt;Image&gt;, user: &lt;User&gt;, rulename: &lt;RuleName&gt;}</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="25%">file</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="75%">{name: &lt;TargetFilename&gt;, type_id: ‘1’}</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="25%">type_uid</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="75%">100101</td> 
  </tr> 
 </tbody>
</table> 
<p align="center"><em>Table 3: File System Activity class mapping with raw log data</em></p> 
<h3>Solution implementation</h3> 
<p>The solution implementation is published in the AWS Samples GitHub repository titled <a href="https://github.com/aws-samples/amazon-security-lake-transformation-library" rel="noopener" target="_blank">amazon-security-lake-transformation-library</a> in <a href="https://github.com/aws-samples/amazon-security-lake-transformation-library/blob/main/CS1-windows-sysmon.md" rel="noopener" target="_blank">the windows-sysmon instructions</a>. You will use the repository to deploy the solution in your AWS account.</p> 
<p>First update the mapping configuration, then add the custom source in Security Lake and deploy and configure the log streaming and transformation infrastructure, which includes the Kinesis Data Stream, transformation Lambda function and associated IAM roles.</p> 
<h4>Step 1: Update mapping configuration</h4> 
<p>Each supported custom source documentation contains the mapping configuration. Update the mapping configuration for the windows-sysmon custom source for the transformation function.</p> 
<p>You can find the mapping configuration in the custom source instructions in the <a href="https://github.com/aws-samples/amazon-security-lake-transformation-library/blob/main/CS1-windows-sysmon.md#mapping" rel="noopener" target="_blank">amazon-security-lake-transformation-library repository</a>.</p> 
<h4>Step 2: Add a custom source in Security Lake</h4> 
<p>As of this writing, Security Lake natively supports <a href="https://docs.aws.amazon.com/security-lake/latest/userguide/internal-sources.html" rel="noopener" target="_blank">AWS CloudTrail, Amazon Route 53 DNS logs, AWS Security Hub findings, Amazon Elastic Kubernetes Service (Amazon EKS) Audit Logs, Amazon Virtual Private Cloud (Amazon VPC) Flow Logs, and AWS Web Application Firewall (AWS WAF)</a>. For other log sources that you want to bring into Security Lake, you must configure the custom sources. For the Sysmon logs, you will create a custom source using the Security Lake API. We recommend using dashes in custom source names as opposed to underscores to be able to configure granular access control for S3 objects.</p> 
<ol> 
 <li>To add the custom source for Sysmon events, configure an IAM role for the <a href="https://aws.amazon.com/glue" rel="noopener" target="_blank">AWS Glue</a> crawler that will be associated with the custom source to update the schema in the Security Lake AWS Glue database. You can deploy the <a href="https://github.com/aws-samples/amazon-security-lake-transformation-library/blob/main/ASLCustomSourceGlueRole.yaml" rel="noopener" style="font-family: courier;" target="_blank">ASLCustomSourceGlueRole.yaml</a> CloudFormation template to automate the creation of the IAM role associated with the custom source AWS Glue crawler.</li> 
 <li>Capture the Amazon Resource Name (ARN) for the IAM role, which is configured as an output of the infrastructure deployed in the previous step.</li> 
 <li>Add a custom source using the following AWS CLI command. Make sure you replace the <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWS_ACCOUNT_ID&gt;</span>, <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;SECURITY_LAKE_REGION&gt;</span> and the <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;GLUE_IAM_ROLE_ARN&gt;</span> placeholders with the AWS account ID you’re deploying into, the Security Lake deployment Region and the ARN of the IAM role created above, respectively. External ID is a unique identifier that is used to establish trust with the <a href="https://docs.aws.amazon.com/security-lake/latest/APIReference/API_AwsIdentity.html" rel="noopener" target="_blank">AWS identity</a>. You can use External ID to add <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user_externalid.html" rel="noopener" target="_blank">conditional access</a> from third-party sources and to subscribers. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">aws securitylake create-custom-log-source \
   --source-name windows-sysmon \
   --configuration crawlerConfiguration={"roleArn=<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;GLUE_IAM_ROLE_ARN&gt;</span>"},providerIdentity={"externalId=CustomSourceExternalId123,principal=<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;AWS_ACCOUNT_ID&gt;</span>"} \
   --event-classes FILE_ACTIVITY PROCESS_ACTIVITY \
   --region <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;SECURITY_LAKE_REGION&gt;</span></code></pre> 
  </div> 
  <blockquote>
   <p><strong>Note:</strong> When creating the custom log source, you only need to specify FILE_ACTIVITY and PROCESS_ACTIVITY event classes as these are the only classes mapped in the example configuration deployed in Step 1. If you extend your mapping configuration to handle additional classes, you would add them here.</p>
  </blockquote> </li> 
</ol> 
<h4>Step 3: Deploy the transformation infrastructure</h4> 
<p>The solution uses the AWS SAM framework—an open source framework for building serverless applications—to deploy the OCSF transformation infrastructure. The infrastructure includes a transformation Lambda function, Kinesis data stream, IAM roles for the Lambda function and the hosts running the Kinesis Agent, and encryption keys for the Kinesis data stream. The Lambda function is configured to read events streamed into the Kinesis Data Stream and transform the data into OCSF based on the mapping configuration file. The transformed events are then written to an S3 bucket managed by Security Lake. A sample of the configuration file is provided in the solution repository capturing a subset of the events. You can extend the same for the remaining Sysmon events.</p> 
<p style="font-weight: 900; font-style: italic;">To deploy the infrastructure:</p> 
<ol> 
 <li>Clone the solution codebase into your choice of integrated development environment (IDE). You can also use <a href="https://aws.amazon.com/cloudshell" rel="noopener" target="_blank">AWS CloudShell</a> or <a href="https://aws.amazon.com/cloud9" rel="noopener" target="_blank">AWS Cloud9</a>.</li> 
 <li>Sign in to the Security Lake delegated administrator account.</li> 
 <li>Review the prerequisites and detailed deployment steps in the project’s <a href="https://github.com/aws-samples/amazon-security-lake-transformation-library/blob/main/CS1-windows-sysmon.md#add-custom-source-in-security-lake" rel="noopener" target="_blank">README file</a>. Use the SAM CLI to build and deploy the streaming infrastructure by running the following commands: 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">sam build

sam deploy –guided</code></pre> 
  </div> </li> 
</ol> 
<h4>Step 4: Update the default AWS Glue crawler</h4> 
<p>Sysmon logs are a complex use case because a single source of logs contains events mapped to multiple schemas. The transformation library handles this by writing each schema to different prefixes (folders) within the target Security Lake bucket. The AWS Glue crawler deployed by Security Lake for the custom log source must be updated to handle prefixes that contain differing schemas.</p> 
<p style="font-weight: 900; font-style: italic;">To update the default AWS Glue crawler:</p> 
<ol> 
 <li>In the Security Lake delegated administrator account, navigate to the <a href="https://console.aws.amazon.com/glue/home" rel="noopener" target="_blank">AWS Glue console</a>.</li> 
 <li>Navigate to <strong>Crawlers</strong> in the <strong>Data Catalog</strong> section. Search for the crawler associated with the custom source. It will have the same name as the custom source name. For example, windows-sysmon. Select the check box next to the crawler name, then choose <strong>Action</strong> and select <strong>Edit Crawler</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_34803" style="width: 750px;">
   <img alt="Figure 2: Select and edit an AWS Glue crawler" class="size-full wp-image-34803" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/07/05/img2-3.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-34803">Figure 2: Select and edit an AWS Glue crawler</p>
  </div><p></p> </li> 
 <li>Select <strong>Edit</strong> for the <strong>Step 2: Choose data sources and classifiers</strong> section on the <strong>Review and update</strong> page.</li> 
 <li>In the <strong>Choose data sources and classifiers</strong> section, make the following changes: 
  <ul> 
   <li>For <strong>Is your data already mapped to Glue tables?</strong>, change the selection to <strong>Not yet</strong>.</li> 
   <li>For <strong>Data sources</strong>, select <strong>Add a data source</strong>. In the selection prompt, select the Security Lake S3 bucket location as presented in the output of the create-custom-source command above. For example, <span style="font-family: courier;">s3://aws-security-data-lake-</span><span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;region&gt;</span><span style="font-family: courier;">–</span><span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;exampleid&gt;</span><span style="font-family: courier;">/ext/windows-sysmon/</span>. Make sure you include the path all the way to the custom source name and replace the <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;region&gt;</span> and <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;exampleid&gt;</span> placeholders with the actual values. Then choose <strong>Add S3 data source</strong>.</li> 
   <li>Choose <strong>Next</strong>.</li> 
   <li>On the <strong>Configure security settings</strong> page, leave everything as is and choose <strong>Next</strong>.</li> 
   <li>On the <strong>Set output and scheduling</strong> page, select the <strong>Target database</strong> as the Security Lake Glue database.</li> 
   <li>In a separate tab, navigate to AWS Glue &gt; Tables. Copy the name of the custom source table created by Security Lake.</li> 
   <li>Navigate back to the AWS Glue crawler configuration tab, update the <strong>Table name prefix</strong> with the copied table name and add an underscore (_) at the end. For example, <span style="font-family: courier;">amazon_security_lake_table_ap_southeast_2_ext_windows_sysmon_</span>.</li> 
   <li>Under <strong>Advanced options</strong>, select the checkbox for <strong>Create a single schema for each S3 path</strong> and for <strong>Table level</strong> enter <span style="font-family: courier;">4</span>.</li> 
   <li>Make sure you allow the crawler to enforce table schema to the partitions by selecting the <strong>Update all new and existing partitions with metadata from the table</strong> checkbox.</li> 
   <li>For the <strong>Crawler schedule</strong> section, select <strong>Monthly</strong> from the <strong>Frequency</strong> dropdown. For <strong>Minute</strong>, enter <span style="font-family: courier;">0</span>. This configuration will run the crawler every month.</li> 
   <li>Choose <strong>Next</strong>, then <strong>Update</strong>.</li> 
  </ul> </li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_34804" style="width: 790px;">
 <img alt="Figure 3: Set AWS Glue crawler output and scheduling" class="size-full wp-image-34804" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/07/05/img3-3.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-34804">Figure 3: Set AWS Glue crawler output and scheduling</p>
</div> 
<p style="font-weight: 900; font-style: italic;">To configure hosts to stream log information:</p> 
<p>As discussed in the <strong>Solution overview </strong>section, you use Kinesis Data Streams with a Lambda function to stream Sysmon logs and transform the information into OCSF.</p> 
<ol> 
 <li><a href="https://docs.aws.amazon.com/kinesis-agent-windows/latest/userguide/getting-started.html#getting-started-installation" rel="noopener" target="_blank">Install Kinesis Agent for Microsoft Windows</a>. There are three ways to install Kinesis Agent on Windows Operating Systems. Using <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/" rel="noopener" target="_blank">AWS Systems Manager</a> helps automate the deployment and upgrade process. You can also install Kinesis Agent by using a Windows installer package or PowerShell scripts.</li> 
 <li>After installation you must configure Kinesis Agent to stream log data to Kinesis Data Streams (you can use the following code for this). Kinesis Agent for Windows helps capture important metadata of the host system and enrich information streamed to the Kinesis Data Stream. The Kinesis Agent configuration file is located at <span style="font-family: courier;">%PROGRAMFILES%\Amazon\AWSKinesisTap\appsettings.json</span> and includes three parts—sources, pipes, and sinks: 
  <ul> 
   <li><a href="https://docs.aws.amazon.com/kinesis-agent-windows/latest/userguide/source-object-declarations.html" rel="noopener" target="_blank">Sources</a> are plugins that gather telemetry.</li> 
   <li><a href="https://docs.aws.amazon.com/kinesis-agent-windows/latest/userguide/sink-object-declarations.html" rel="noopener" target="_blank">Sinks</a> stream telemetry information to different AWS services, including but not limited to Amazon Kinesis.</li> 
   <li><a href="https://docs.aws.amazon.com/kinesis-agent-windows/latest/userguide/pipe-object-declarations.html" rel="noopener" target="_blank">Pipes</a> connect a source to a sink.</li> 
  </ul> 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">{
  "Sources": [
    {
      "Id": "Sysmon",
      "SourceType": "WindowsEventLogSource",
      "LogName": "Microsoft-Windows-Sysmon/Operational"
    }
  ],
  "Sinks": [
    {
      "Id": "SysmonLogStream",
      "SinkType": "KinesisStream",
      "StreamName": "<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;LogCollectionStreamName&gt;</span>",
      "ObjectDecoration": "source_instance_id={ec2:instance-id};",
      "Format": "json",
      "RoleARN": "<span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;KinesisAgentIAMRoleARN&gt;</span>"
    }
  ],
  "Pipes": [
    {
       "Id": "JsonLogSourceToKinesisLogStream",
       "SourceRef": "Sysmon",
       "SinkRef": "SysmonLogStream"
    }
  ],
  "SelfUpdate": 0,
  "Telemetrics": { "off": "true" }
}</code></pre> 
  </div> <p>The preceding configuration shows the information flow through sources, pipes, and sinks using the Kinesis Agent for Windows. Use the sample configuration file provided in the solution repository. Observe the <span style="font-family: courier;">ObjectDecoration</span> key in the <span style="font-family: courier;">Sink</span> configuration; you can use this key to add key information to identify the generating system. For example, to identify whether the event is being generated by an <a href="https://aws.amazon.com/ec2/" rel="noopener" target="_blank">Amazon Elastic Compute Cloud (Amazon EC2)</a> instance or a hybrid server. This information can be used to map the <span style="font-family: courier;">Device</span> attribute in the various OCSF classes such as File System Activity and Process Activity. The <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;KinesisAgentIAMRoleARN&gt;</span> is configured by the transformation library deployment unless you create your own IAM role and provide it as a parameter to the deployment.</p> <p>Update the Kinesis agent configuration file <span style="font-family: courier;">%PROGRAMFILES%\Amazon\AWSKinesisTap\appsettings.json</span> with the contents of the <span style="font-family: courier;">kinesis_agent_configuration.json</span> file from this repository. Make sure you replace the <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;LogCollectionStreamName&gt;</span> and <span style="font-family: courier; color: #ff0000; font-style: italic;">&lt;KinesisAgentIAMRoleARN&gt;</span> placeholders with the value of the CloudFormation outputs, <span style="font-family: courier;">LogCollectionStreamName</span> and <span style="font-family: courier;">KinesisAgentIAMRoleARN</span>, that you captured in the <strong>Deploy transformation infrastructure</strong> step.</p> </li> 
 <li><a href="https://docs.aws.amazon.com/kinesis-agent-windows/latest/userguide/getting-started.html#getting-started-start-service" rel="noopener" target="_blank">Start Kinesis Agent</a> on the hosts to start streaming the logs to Security Lake buckets. Open an elevated PowerShell command prompt window, and start Kinesis Agent for Windows using the following PowerShell command: 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">Start-Service -Name AWSKinesisTap</code></pre> 
  </div> </li> 
</ol> 
<h2>Pattern 2: Log collection from services and products using AWS Glue</h2> 
<p>You can use <a href="https://aws.amazon.com/vpc" rel="noopener" target="_blank">Amazon VPC</a> to launch resources in an isolated network. <a href="https://aws.amazon.com/network-firewall/" rel="noopener" target="_blank">AWS Network Firewall</a> provides the capability to filter network traffic at the perimeter of your VPCs and define stateful rules to configure fine-grained control over network flow. Common Network Firewall use cases include intrusion detection and protection, Transport Layer Security (TLS) inspection, and egress filtering. Network Firewall supports <a href="https://docs.aws.amazon.com/network-firewall/latest/developerguide/firewall-logging-destinations.html" rel="noopener" target="_blank">multiple destinations for log delivery</a>, including Amazon S3. </p> 
<p>In this pattern, you focus on adding a custom source in Security Lake where the product in use delivers raw logs to an S3 bucket.</p> 
<h3>Solution overview</h3> 
<p>This solution uses an S3 bucket (the <em>staging bucket</em>) for raw log storage using the prerequisites defined earlier in this post. Use AWS Glue to configure the ETL and load the OCSF transformed logs into the Security Lake S3 bucket.</p> 
<div class="wp-caption aligncenter" id="attachment_34805" style="width: 790px;">
 <img alt="Figure 4: Architecture using AWS Glue for ETL" class="size-full wp-image-34805" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/07/05/img4-2.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-34805">Figure 4: Architecture using AWS Glue for ETL</p>
</div> 
<p>Figure 4 shows the architecture for this pattern. This pattern applies to AWS services or partner services that natively support log storage in S3 buckets. The solution starts by defining the OCSF mapping.</p> 
<h3>Mapping</h3> 
<p>Network firewall records two types of log information—alert logs and netflow logs. Alert logs report traffic that matches the stateful rules configured in your environment. Flow logs are network traffic flow logs that capture network traffic information for standard stateless rule groups. You can use stateful rules for use cases such as egress filtering to restrict the external domains that the resources deployed in a VPC in your AWS account have access to. In the Network Firewall use case, events can be mapped to various attributes in the <em>Network Activity</em> class in the <em>Network Activity</em> category.</p> 
<p style="font-weight: 900; font-style: italic;">Network firewall sample event: netflow log</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
    "firewall_name":"firewall",
    "availability_zone":"us-east-1b",
    "event_timestamp":"1601587565",
    "event":{
        "timestamp":"2020-10-01T21:26:05.007515+0000",
        "flow_id":1770453319291727,
        "event_type":"netflow",
        "src_ip":"45.129.33.153",
        "src_port":47047,
        "dest_ip":"172.31.16.139",
        "dest_port":16463,
        "proto":"TCP",
        "netflow":{
            "pkts":1,
            "bytes":60,
            "start":"2020-10-01T21:25:04.070479+0000",
            "end":"2020-10-01T21:25:04.070479+0000",
            "age":0,
            "min_ttl":241,
            "max_ttl":241
        },
        "tcp":{
            "tcp_flags":"02",
            "syn":true
        }
    }
}</code></pre> 
</div> 
<p style="font-weight: 900; font-style: italic;">Network firewall sample event: alert log</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
    "firewall_name":"firewall",
    "availability_zone":"zone",
    "event_timestamp":"1601074865",
    "event":{
        "timestamp":"2020-09-25T23:01:05.598481+0000",
        "flow_id":1111111111111111,
        "event_type":"alert",
        "src_ip":"10.16.197.56",
        "src_port":49157,
        "dest_ip":"10.16.197.55",
        "dest_port":8883,
        "proto":"TCP",
        "alert":{
            "action":"allowed",
            "signature_id":2,
            "rev":0,
            "signature":"",
            "category":"",
            "severity":3
        }
    }
}</code></pre> 
</div> 
<p>The target mapping for the preceding alert logs is as follows:</p> 
<p style="font-weight: 900; font-style: italic;">Mapping for alert logs</p> 
<table style="border-collapse: collapse; border: 1px solid #808080;" width="100%"> 
 <tbody>
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%"><strong>OCSF</strong></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%"><strong>Raw</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">app_name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">&lt;firewall_name&gt;</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">activity_id</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">6</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">activity_name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">Traffic</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">category_uid</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">4</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">category_name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">Network activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">class_uid</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">4001</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">type_uid</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">400106</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">class_name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">Network activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">dst_endpoint</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">{ip: &lt;event.dest_ip&gt;, port: &lt;event.dest_port&gt;}</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">src_endpoint</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">{ip: &lt;event.src_ip&gt;, port: &lt;event.src_port&gt;}</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">time</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">&lt;event.timestamp&gt;</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">severity_id</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">&lt;event.alert.severity&gt;</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">connection_info</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">{uid: &lt;event.flow_id&gt;, protocol_name: &lt;event.proto&gt;}</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">cloud.provider</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">AWS</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">metadata.profiles</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">[cloud, firewall]</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">metadata.product.name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">AWS Network Firewall</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">metadata.product.feature.name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">Firewall</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">metadata.product.vendor_name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">AWS</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">severity</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">High</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">unmapped</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">{alert: {action: &lt;event.alert.action&gt;, signature_id: &lt;event.alert.signature_id&gt;, rev: &lt;event.alert.rev&gt;, signature: &lt;event.alert.signature&gt;, category: &lt;event.alert.category&gt;, tls_inspected: &lt;event.alert.tls_inspected&gt;}}</td> 
  </tr> 
 </tbody>
</table> 
<p style="font-weight: 900; font-style: italic;">Mapping for netflow logs</p> 
<table style="border-collapse: collapse; border: 1px solid #808080;" width="100%"> 
 <tbody>
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%"><strong>OCSF</strong></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%"><strong>Raw</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">app_name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">&lt;firewall_name&gt;</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">activity_id</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">6</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">activity_name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">Traffic</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">category_uid</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">4</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">category_name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">Network activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">class_uid</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">4001</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">type_uid</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">400106</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">class_name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">Network activity</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">dst_endpoint</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">{ip: &lt;event.dest_ip&gt;, port: &lt;event.dest_port&gt;}</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">src_endpoint</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">{ip: &lt;event.src_ip&gt;, port: &lt;event.src_port&gt;}</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">Time</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">&lt;event.timestamp&gt;</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">connection_info</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">{uid: &lt;event.flow_id&gt;, protocol_name: &lt;event.proto&gt;, tcp_flags: &lt;event.tcp.tcp_flags&gt;}</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">cloud.provider</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">AWS</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">metadata.profiles</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">[cloud, firewall]</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">metadata.product.name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">AWS Network Firewall</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">metadata.product.feature.name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">Firewall</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">metadata.product.vendor_name</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">AWS</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">severity</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">Informational</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">severity_id</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">1</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">start_time</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">&lt;event.netflow.start&gt;</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">end_time</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">&lt;event.netflow.end&gt;</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">Traffic</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">{bytes: &lt;event.netflow.bytes&gt;, packets: &lt;event.netflow.packets&gt;}</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="30%">Unmapped</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="70%">{availability_zone: &lt;availability_zone&gt;, event_type: &lt;event.event_type&gt;, netflow: {age: &lt;event.netflow.age&gt;, min_ttl: &lt;event.netflow.min_ttl&gt;, max_ttl: &lt;event.netflow.max_ttl&gt;}, tcp: {syn: &lt;event.tcp.syn&gt;, fin: &lt;event.tcp.fin&gt;, ack: &lt;event.tcp.ack&gt;, psh: &lt;event.tcp.psh&gt;}}</td> 
  </tr> 
 </tbody>
</table> 
<h3>Solution implementation</h3> 
<p>The solution implementation is published in the AWS Samples GitHub repository titled <a href="https://github.com/aws-samples/amazon-security-lake-transformation-library" rel="noopener" target="_blank">amazon-security-lake-transformation-library</a> in the <a href="https://github.com/aws-samples/amazon-security-lake-transformation-library/blob/main/CS2-aws-network-firewall.md" rel="noopener" target="_blank">Network Firewall instructions</a>. Use the repository to deploy this pattern in your AWS account. The solution deployment is a four-step process:</p> 
<ol> 
 <li>Update the mapping configuration</li> 
 <li>Configure the log source to use Amazon S3 for log delivery</li> 
 <li>Add a custom source in Security Lake</li> 
 <li>Deploy the log staging and transformation infrastructure</li> 
</ol> 
<p>Because Network Firewall logs can be mapped to a single OCSF class, you don’t need to update the AWS Glue crawler as in the previous pattern. However, you must update the AWS Glue crawler if you want to add a custom source with multiple OCSF classes.</p> 
<h4>Step 1: Update the mapping configuration</h4> 
<p>Each supported custom source documentation contains the mapping configuration. Update the mapping configuration for the Network Firewall custom source for the transformation function.</p> 
<p>The mapping configuration can be found in the custom source instructions in the <a href="https://github.com/aws-samples/amazon-security-lake-transformation-library/blob/main/CS2-aws-network-firewall.md#mapping" rel="noopener" target="_blank">amazon-security-lake-transformation-library repository</a></p> 
<h4>Step 2: Configure the log source to use S3 for log delivery</h4> 
<p><a href="https://docs.aws.amazon.com/network-firewall/latest/developerguide/logging-s3.html" rel="noopener" target="_blank">Configure Network Firewall to log to Amazon S3</a>. The transformation function infrastructure deploys a staging S3 bucket for raw log storage. If you already have an S3 bucket configured for raw log delivery, you can update the value of the parameter <span style="font-family: courier;">RawLogS3BucketName</span> during deployment. The deployment configures event notifications with <a href="https://aws.amazon.com/sqs" rel="noopener" target="_blank">Amazon Simple Queue Service (Amazon SQS)</a>. The transformation Lambda function is invoked by SQS event notifications when Network Firewall delivers log files in the staging S3 bucket.</p> 
<h4>Step 3: Add a custom source in Security Lake</h4> 
<p>As with the previous pattern, add a custom source for Network Firewall in Security Lake. In the previous pattern you used the AWS CLI to create and configure the custom source. In this pattern, we take you through the steps to do the same using the AWS console.</p> 
<p style="font-weight: 900; font-style: italic;">To add a custom source:</p> 
<ol> 
 <li>Open the <a href="https://console.aws.amazon.com/securitylake/" rel="noopener" target="_blank">Security Lake</a> console</li> 
 <li>In the navigation pane, select <strong>Custom sources</strong>.</li> 
 <li>Then select <strong>Create custom source</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_34809" style="width: 750px;">
   <img alt="Figure 5: Create a custom source" class="size-full wp-image-34809" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/07/05/img5-2.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-34809">Figure 5: Create a custom source</p>
  </div><p></p> </li> 
 <li>Under <strong>Custom source details</strong> enter a name for the custom log source such as <span style="font-family: courier;">network_firewall</span> and choose <strong>Network Activity</strong> as the <strong>OCSF Event class</strong> <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_34810" style="width: 590px;">
   <img alt="Figure 6: Data source name and OCSF Event class" class="size-full wp-image-34810" height="325" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/07/05/img6-1.png" style="border: 1px solid #bebebe;" width="580" />
   <p class="wp-caption-text" id="caption-attachment-34810">Figure 6: Data source name and OCSF Event class</p>
  </div><p></p> </li> 
 <li>Under <strong>Account details</strong>, enter your AWS account ID for the <strong>AWS account ID</strong> and <strong>External ID</strong> fields. Leave <strong>Create and use a new service role</strong> selected and choose <strong>Create</strong>. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_34811" style="width: 750px;">
   <img alt="Figure 7: Account details and service access" class="size-full wp-image-34811" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/07/05/img7.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-34811">Figure 7: Account details and service access</p>
  </div><p></p> </li> 
 <li>The custom log source will now be available.</li> 
</ol> 
<h4>Step 4: Deploy transformation infrastructure</h4> 
<p>As with the previous pattern, use AWS SAM CLI to deploy the transformation infrastructure.</p> 
<p style="font-weight: 900; font-style: italic;">To deploy the transformation infrastructure:</p> 
<ol> 
 <li>Clone the solution codebase into your choice of IDE.</li> 
 <li>Sign in to the Security Lake delegated administrator account.</li> 
 <li>The infrastructure is deployed using the <a href="https://aws.amazon.com/serverless/sam/" rel="noopener" target="_blank">AWS SAM</a>, which is an open source framework for building serverless applications. Review the prerequisites and detailed deployment steps in the <a href="https://github.com/aws-samples/amazon-security-lake-transformation-library/blob/main/CS2-aws-network-firewall.md#deployment" rel="noopener" target="_blank">project’s README file</a>. Use the SAM CLI to build and deploy the streaming infrastructure by running the following commands: 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">sam build

sam deploy --guided</code></pre> 
  </div> </li> 
</ol> 
<h2>Clean up</h2> 
<p>The resources created in the previous patterns can be cleaned up by running the following command:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">sam delete</code></pre> 
</div> 
<p>You also need to manually delete the custom source by following the instructions from the <a href="https://docs.aws.amazon.com/security-lake/latest/userguide/custom-sources.html#delete-custom-source" rel="noopener" target="_blank">Security Lake User Guide</a>.</p> 
<h2>Pattern 3: Log collection using integration with supported AWS services.</h2> 
<p>In a threat hunting and response use case, customers often use multiple sources of logs to correlate information to find more information on unauthorized third-party interactions originating from trusted software vendors. These interactions can be due to vulnerable components in the product or exposed credentials such as integration API keys. An operationally effective way to source logs from partner software and external vendors is to use the supported AWS services that natively integrate with Security Lake.</p> 
<h3>AWS Security Hub</h3> 
<p><a href="https://aws.amazon.com/security-hub/" rel="noopener" target="_blank">AWS Security Hub</a> is a cloud security posture measurement service that provides a comprehensive view of the security posture of your AWS environment. Security Hub supports integration with several AWS services including AWS Systems Manager Patch Manager, Amazon Macie, Amazon GuardDuty, and Amazon Inspector. For the full list, see <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-internal-providers.html" rel="noopener" target="_blank">AWS service integrations with AWS Security Hub</a>. Security Hub also integrates with multiple <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-partner-providers.html" rel="noopener" target="_blank">third-party partner products</a> that you can use. These products support sending findings to Security Hub seamlessly.</p> 
<p>Security Lake natively supports ingestion of Security Hub findings, which centralizes the findings from the source integrations into Security Lake. Before you start building a custom source, we recommend you review whether the product is supported by Security Hub, which could remove the need for building manual mapping and transformation solutions.</p> 
<h3>AWS AppFabric</h3> 
<p><a href="https://aws.amazon.com/appfabric/" rel="noopener" target="_blank">AWS AppFabric</a> is a fully managed software as a service (SaaS) interoperability solution. Security Lake supports AppFabric output schema and format—OCSF and JSON respectively. Security Lake supports AppFabric as a custom source using Amazon Kinesis Data Firehose delivery stream. You can find step-by-step instructions in the <a href="https://docs.aws.amazon.com/appfabric/latest/adminguide/security-lake.html" rel="noopener" target="_blank">AppFabric user guide</a>.</p> 
<h2>Conclusion</h2> 
<p>Security Lake offers customers the capability to centralize disparate log sources in a single format, OCSF. Using OCSF improves correlation and enrichment activities because security teams no longer have to build queries based on the individual log source schema. Log data is normalized such that customers can use the same schema across the log data collected. Using the patterns and solution identified in this post, you can significantly reduce the effort involved in building custom sources to bring your own data into Security Lake.</p> 
<p>You can extend the concepts and mapping function code provided in the amazon-security-lake-transformation-library to build out a log ingestion and ETL solution. You can use the flexibility offered by Security Lake and the custom source feature to ingest log data generated by all sources including third-party tools, log forwarding software, AWS services, and hybrid solutions.</p> 
<p>In this post, we provided you with three patterns that you can use across multiple log sources. The most flexible being Pattern 1, where you can choose the OCSF mapped class and attributes that are in-line with your organizational mappings and custom source configuration with Security Lake. You can continue to use the mapping function code from the amazon-security-lake-transformation-library demonstrated through this post and update the mapping variable for the OCSF class you’re mapping to. This solution can be scaled to build a range of custom sources to enhance your threat detection and investigation workflow.</p> 
<p>If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.</p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Pratima Singh" class="aligncenter size-full wp-image-28989" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/04/03/pxs_headshot.jpg" width="120" /> 
  </div> 
  <p> <span class="lb-h4">Pratima Singh</span><br /> Pratima is a Security Specialist Solutions Architect with Amazon Web Services based out of Sydney, Australia. She is a security enthusiast who enjoys helping customers find innovative solutions to complex business challenges. Outside of work, Pratima enjoys going on long drives and spending time with her family at the beach. </p>
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Chris Lamont-Smith" class="aligncenter size-full wp-image-34794" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2023/11/01/chris_lamont-smith2.jpg" width="120" /> 
  </div> 
  <p> <span class="lb-h4">Chris Lamont-Smith</span><br /> Chris is a Senior Security Consultant working in the Security, Risk, and Compliance team for AWS ProServe based out of Perth, Australia. He enjoys working in the area where security and data analytics intersect and is passionate about helping customers gain actionable insights from their security data. When Chris isn’t working, he is out camping or off-roading with his family in the Australian bush. </p>
 </div> 
</footer>
