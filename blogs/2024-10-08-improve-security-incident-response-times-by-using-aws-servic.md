---
title: "Improve security incident response times by using AWS Service Catalog to decentralize security notifications"
url: "https://aws.amazon.com/blogs/security/improve-security-incident-response-times-by-using-aws-service-catalog-to-decentralize-security-notifications/"
date: "Tue, 08 Oct 2024 13:08:09 +0000"
author: "Cheng Wang"
feed_url: "https://aws.amazon.com/blogs/security/tag/aws-organizations/feed/"
---
<p>Many organizations continuously receive security-related findings that highlight resources that aren’t configured according to the organization’s security policies. The findings can come from threat detection services like <a href="https://aws.amazon.com/guardduty/" rel="noopener" target="_blank">Amazon GuardDuty</a>, or from cloud security posture management (CSPM) services like <a href="https://aws.amazon.com/security-hub/" rel="noopener" target="_blank">AWS Security Hub</a>, or other sources. An important question to ask is: How, and how soon, are your teams notified of these findings?</p> 
<p>Often, security-related findings are streamed to a single centralized security team or Security Operations Center (SOC). Although it’s a best practice to<a href="https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/sec_detect_investigate_events_logs.html" rel="noopener" target="_blank"> capture logs, findings, and metrics in standardized locations</a>, the centralized team might not be the best equipped to make configuration changes in response to an incident. <a href="https://docs.aws.amazon.com/whitepapers/latest/aws-security-incident-response-guide/define-roles-and-responsibilities.html" rel="noopener" target="_blank">Involving the owners or developers of the impacted applications and resources</a> is key because they have the context required to respond appropriately. Security teams often have manual processes for locating and contacting workload owners, but they might not be up to date on the current owners of a workload. Delays in notifying workload owners can increase the time to resolve a security incident or a resource misconfiguration.</p> 
<p>This post outlines a decentralized approach to security notifications, using a self-service mechanism powered by <a href="https://aws.amazon.com/servicecatalog/" rel="noopener" target="_blank">AWS Service Catalog</a> to enhance response times. With this mechanism, workload owners can subscribe to receive near real-time Security Hub notifications for their AWS accounts or workloads through email. The notifications include those from <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-findings-providers.html" rel="noopener" target="_blank">Security Hub product integrations</a> like GuardDuty, <a href="https://docs.aws.amazon.com/health/latest/ug/what-is-aws-health.html" rel="noopener" target="_blank">AWS Health</a>, <a href="https://aws.amazon.com/inspector/" rel="noopener" target="_blank">Amazon Inspector</a>, and <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-partner-providers.html" rel="noopener" target="_blank">third-party products</a>, as well as notifications of non-compliance with security standards. These notifications can better equip your teams to configure AWS resources properly and reduce the exposure time of unsecured resources.</p> 
<h2>End-user experience</h2> 
<p>After you deploy the solution in this post, users in assigned groups can access a least-privilege <a href="https://aws.amazon.com/iam/identity-center/" rel="noopener" target="_blank">AWS IAM Identity Center</a> permission set, called <strong>SubscribeToSecurityNotifications</strong>, for their AWS accounts (Figure 1). The solution can also work with existing permission sets or federated IAM roles without IAM Identity Center.</p> 
<div class="wp-caption aligncenter" id="attachment_35984" style="width: 608px;">
 <img alt="Figure 1: IAM Identity Center portal with the permission set to subscribe to security notifications" class="size-full wp-image-35984" height="604" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/10/07/img1-1.png" style="border: 1px solid #bebebe;" width="598" />
 <p class="wp-caption-text" id="caption-attachment-35984">Figure 1: IAM Identity Center portal with the permission set to subscribe to security notifications</p>
</div> 
<p>After the user chooses <strong>SubscribeToSecurityNotifications</strong>, they are redirected to an AWS Service Catalog product for subscribing to security notifications and can see instructions on how to proceed (Figure 2).</p> 
<div class="wp-caption aligncenter" id="attachment_35985" style="width: 790px;">
 <img alt="Figure 2: AWS Service Catalog product view" class="size-full wp-image-35985" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/10/07/img2.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-35985">Figure 2: AWS Service Catalog product view</p>
</div> 
<p>The user can then choose the <strong>Launch product</strong> button and enter one or more email addresses and the minimum severity level for notifications (Critical, High, Medium, or Low). If the AWS account has multiple workloads, they can choose to receive only the notifications related to the applications they own by specifying the resource tags. They can also choose to restrict security notifications to include or exclude specific security products (Figure 3).</p> 
<div class="wp-caption aligncenter" id="attachment_35986" style="width: 790px;">
 <img alt="Figure 3: Service Catalog security notifications product parameters" class="size-full wp-image-35986" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/10/07/img3.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-35986">Figure 3: Service Catalog security notifications product parameters</p>
</div> 
<p>You can update the Service Catalog product configurations after provisioning by doing the following:</p> 
<ol> 
 <li>In the Service Catalog console, in the left navigation menu, choose <strong>Provisioned products</strong>.</li> 
 <li>Select the provisioned product, choose <strong>Actions</strong>, and then choose <strong>Update</strong>.</li> 
 <li>Update the parameters you want to change.</li> 
</ol> 
<p>For accounts that have multiple applications, each application owner can set up their own notifications by provisioning an additional Service Catalog product. You can use the <strong>Filter findings by tag</strong> parameters to receive notifications only for a specific application. The example shown in Figure 3 specifies that the user will receive notifications only from resources with the tag key app and the tag value <code style="color: #000000;">BigApp1</code> or <code style="color: #000000;">AnotherApp</code>.</p> 
<p>After confirming the subscription, the user starts to receive email notifications for new Security Hub findings in near real-time. Each email contains a summary of the finding in the subject line, the account details, the finding details, recommendations (if any), the list of resources affected with their tags, and an IAM Identity Center <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/createshortcutlink.html" rel="noopener" target="_blank">shortcut link</a> to the Security Hub finding (Figure 4). The email ends with the raw JSON of the finding.</p> 
<div class="wp-caption aligncenter" id="attachment_35987" style="width: 711px;">
 <img alt="Figure 4: Sample email showing details of the security notification" class="size-full wp-image-35987" height="589" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/10/07/img4.jpg" style="border: 1px solid #bebebe;" width="701" />
 <p class="wp-caption-text" id="caption-attachment-35987">Figure 4: Sample email showing details of the security notification</p>
</div> 
<p>Choosing the link in the email takes the user directly to the AWS account and the finding in Security Hub, where they can see more details and search for related findings (Figure 5).</p> 
<div class="wp-caption aligncenter" id="attachment_35988" style="width: 790px;">
 <img alt="Figure 5: Security Hub finding detail page, linked from the notification email" class="size-full wp-image-35988" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/10/07/img5.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-35988">Figure 5: Security Hub finding detail page, linked from the notification email</p>
</div> 
<h2>Solution overview</h2> 
<p>We’ve provided two deployment options for this solution; a simpler option and one that is more advanced.</p> 
<p>Figure 6 shows the simpler deployment option of using the requesting user’s IAM permissions to create the resources required for notifications.</p> 
<div class="wp-caption aligncenter" id="attachment_35989" style="width: 790px;">
 <img alt="Figure 6: Architecture diagram of the simpler configuration of the solution" class="size-full wp-image-35989" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/10/07/img6.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-35989">Figure 6: Architecture diagram of the simpler configuration of the solution</p>
</div> 
<p>The solution involves the following steps:</p> 
<ol> 
 <li>Create a central <strong>Subscribe to AWS Security Hub notifications</strong> Service Catalog product in an AWS account which is shared with the entire organization in <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a> or with specific <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_ous.html" rel="noopener" target="_blank">organizational units (OUs)</a>. Configure the product with the names of IAM roles or IAM Identity Center permission sets that can launch the product.</li> 
 <li>Users who sign in through the designated IAM roles or permission sets can access the shared Service Catalog product from the AWS Management Console and enter the required parameters such as their email address and the minimum severity level for notifications.</li> 
 <li>The Service Catalog product creates an <a href="https://aws.amazon.com/cloudformation/" rel="noopener" target="_blank">AWS CloudFormation</a> stack, which creates an <a href="https://aws.amazon.com/sns/" rel="noopener" target="_blank">Amazon Simple Notification Service (Amazon SNS)</a> topic and an <a href="https://aws.amazon.com/eventbridge/" rel="noopener" target="_blank">Amazon EventBridge</a> rule that filters new Security Hub finding events that match the user’s parameters, such as minimum severity level. The rule then formats the Security Hub JSON event message to make it human-readable by using native EventBridge <a href="https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-transform-target-input.html" rel="noopener" target="_blank">input transformers</a>. The formatted message is then sent to SNS, which emails the user.</li> 
</ol> 
<p>We also provide a more advanced and recommended deployment option, shown in Figure 7. This option involves using an <a href="https://aws.amazon.com/lambda/" rel="noopener" target="_blank">AWS Lambda</a> function to enhance messages by doing conversions from UTC to your selected time zone, setting the email subject to the finding summary, and including an IAM Identity Center <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/createshortcutlink.html" rel="noopener" target="_blank">shortcut link</a> to the finding. To not require your users to have permissions for creating Lambda functions and IAM roles, a Service Catalog <a href="https://docs.aws.amazon.com/servicecatalog/latest/adminguide/constraints-launch.html" rel="noopener" target="_blank">launch role</a> is used to create resources on behalf of the user, and this role is restricted by using IAM <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html" rel="noopener" target="_blank">permissions boundaries</a>.</p> 
<div class="wp-caption aligncenter" id="attachment_35990" style="width: 790px;">
 <img alt="Figure 7: Architecture diagram of the solution when using the calling user’s permissions" class="size-full wp-image-35990" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/10/07/img7.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-35990">Figure 7: Architecture diagram of the solution when using the calling user’s permissions</p>
</div> 
<p>The architecture is similar to the previous option, but with the following changes:</p> 
<ol> 
 <li>Create a CloudFormation <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html" rel="noopener" target="_blank">StackSet</a> in advance to pre-create an IAM role and an IAM permissions boundary policy in every AWS account. The IAM role is used by the Service Catalog product as a launch role. It has permissions to create CloudFormation resources such as SNS topics, as well as to create IAM roles that are restricted by the IAM permissions boundary policy that allows only publishing SNS messages and writing to <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html" rel="noopener" target="_blank">Amazon CloudWatch Logs</a>.</li> 
 <li>Users who want to subscribe to security notifications require only minimal permissions; just enough to access Service Catalog and to <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_passrole.html" rel="noopener" target="_blank">pass</a> the pre-created role (from the preceding step) to Service Catalog. This solution provides a sample AWS Identity Center permission set with these minimal permissions.</li> 
 <li>The Service Catalog product uses a Lambda function to format the message to make it human-readable. The stack creates an IAM role, limited by the permissions boundary, and the role is assumed by the Lambda function to publish the SNS message.</li> 
</ol> 
<h2>Prerequisites</h2> 
<p>The solution installation requires the following:</p> 
<ol> 
 <li>Administrator-level access to <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a>. AWS Organizations must have <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_org_support-all-features.html" rel="noopener" target="_blank">all features</a></li> 
 <li><a href="https://aws.amazon.com/security-hub/" rel="noopener" target="_blank">Security Hub</a> enabled in the accounts you are monitoring.</li> 
 <li>An AWS account to host this solution, for example the Security Hub administrator account or a shared services account. This cannot be the management account.</li> 
 <li>One or more AWS accounts to consume the Service Catalog product.</li> 
 <li>Authentication that uses <a href="https://aws.amazon.com/iam/identity-center/" rel="noopener" target="_blank">AWS IAM Identity Center</a> or federated IAM role names in every AWS account for users accessing the Service Catalog product.</li> 
 <li>(Optional, only required when you opt to use Service Catalog launch roles) CloudFormation StackSet creation access to either the management account or a CloudFormation <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-orgs-delegated-admin.html" rel="noopener" target="_blank">delegated administrator account</a>.</li> 
 <li>This solution supports notifications coming from multiple AWS Regions. If you are operating Security Hub in multiple Regions, for a simplified deployment evaluate the Security Hub <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/finding-aggregation.html" rel="noopener" target="_blank">cross-Region aggregation</a> feature and <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/finding-aggregation-enable.html" rel="noopener" target="_blank">enable</a> it for the applicable Regions.</li> 
</ol> 
<h2>Walkthrough</h2> 
<p>There are four steps to deploy this solution:</p> 
<ol> 
 <li>Configure AWS Organizations to allow Service Catalog product sharing.</li> 
 <li>(Optional, recommended) Use CloudFormation StackSets to deploy the Service Catalog launch IAM role across accounts.</li> 
 <li>Service Catalog product creation to allow users to subscribe to Security Hub notifications. This needs to be deployed in the specific Region you want to monitor your Security Hub findings in, or where you enabled <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/finding-aggregation.html" rel="noopener" target="_blank">cross-Region aggregation</a>.</li> 
 <li>(Optional, recommended) Provision least-privileged IAM Identity Center permission sets.</li> 
</ol> 
<h3>Step 1: Configure AWS Organizations</h3> 
<p>Service Catalog organizations sharing in AWS Organizations must be enabled, and the account that is hosting the solution must be one of the delegated administrators for Service Catalog. This allows the Service Catalog product to be shared to other AWS accounts in the organization.</p> 
<p>To enable this configuration, sign in to the AWS Management Console in the management AWS account, launch the <a href="https://aws.amazon.com/cloudshell/" rel="noopener" target="_blank">AWS CloudShell</a> service, and enter the following commands. Replace the <code style="color: #ff0000; font-style: italic;">&lt;Account ID&gt;</code> variable with the ID of the account that will host the Service Catalog product.</p> 
<pre><code class="lang-bash"># Enable AWS Organizations integration in Service Catalog
aws servicecatalog enable-aws-organizations-access

# Nominate the account to be one of the delegated administrators for Service Catalog
aws organizations register-delegated-administrator --account-id <span style="color: #ff0000; font-style: italic;">&lt;Account ID&gt;</span> --service-principal servicecatalog.amazonaws.com</code></pre> 
<h3>Step 2: (Optional, recommended) Deploy IAM roles across accounts with CloudFormation StackSets</h3> 
<p>The following steps create a CloudFormation StackSet to deploy a Service Catalog launch role and permissions boundary across your accounts. This is highly recommended if you plan to enable Lambda formatting, because if you skip this step, only users who have permissions to create IAM roles will be able to subscribe to security notifications.</p> 
<p><strong>To deploy IAM roles with StackSets</strong></p> 
<ol> 
 <li>Sign in to the AWS Management Console from the management AWS account, or from a CloudFormation <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-orgs-delegated-admin.html" rel="noopener" target="_blank">delegated administrator</a></li> 
 <li>Download the <a href="https://github.com/aws-samples/improving-security-incident-response-times-by-decentralizing-notifications/blob/main/01_IAM/SecurityHub_notifications_IAM_role_stackset.yaml" rel="noopener" target="_blank">CloudFormation template</a> for creating the StackSet.</li> 
 <li>Navigate to the AWS CloudFormation page.</li> 
 <li>Choose <strong>Create stack</strong>, and then choose <strong>With new resources (standard)</strong>.</li> 
 <li>Choose <strong>Upload a template file</strong> and upload the CloudFormation template that you downloaded earlier:<code style="color: #000000;">SecurityHub_notifications_IAM_role_stackset.yaml</code>. Then choose <strong>Next</strong>.</li> 
 <li>Enter the stack name <strong>SecurityNotifications-IAM-roles-StackSet</strong>.</li> 
 <li>Enter the following values for the parameters: 
  <ol> 
   <li><strong>AWS Organization ID</strong>: Start <a href="https://aws.amazon.com/cloudshell/" rel="noopener" target="_blank">AWS CloudShell</a> and enter the command provided in the parameter description to get the organization ID.</li> 
   <li><strong>Organization root ID or OU ID(s)</strong>: To deploy the IAM role and permissions boundary to every account, enter the organization root ID using CloudShell and the command in the parameter description. To deploy to specific OUs, enter a comma-separated list of OU IDs. Make sure that you include the OU of the account that is hosting the solution.</li> 
   <li><strong>Current Account Type</strong>: Choose either <strong>Management account</strong> or <strong>Delegated administrator account</strong>, as needed.</li> 
   <li><strong>Formatting method</strong>: Indicate whether you plan to use the Lambda formatter for Security Hub notifications, or native EventBridge formatting with no Lambda functions. If you’re unsure, choose <strong>Lambda</strong>.</li> 
  </ol> </li> 
 <li>Choose <strong>Next</strong>, and then optionally enter tags and choose <strong>Submit</strong>. Wait for the stack creation to finish.</li> 
</ol> 
<h3>Step 3: Create Service Catalog product</h3> 
<p>Next, run the included installation script that creates the CloudFormation templates that are required to deploy the Service Catalog product and <a href="https://docs.aws.amazon.com/servicecatalog/latest/adminguide/catalogs_portfolios.html" rel="noopener" target="_blank">portfolio</a>.</p> 
<p><strong>To run the installation script</strong></p> 
<ol> 
 <li>Sign in to the console of the AWS account and Region that will host the solution, and start the AWS CloudShell service.</li> 
 <li>In the terminal, enter the following commands: <pre><code class="lang-bash">git clone https://github.com/aws-samples/improving-security-incident-response-times-by-decentralizing-notifications.git

cd improving-security-incident-response-times-by-decentralizing-notifications

./install.sh</code></pre> </li> 
</ol> 
<p>The script will ask for the following information:</p> 
<ul> 
 <li>Whether you will be using the <strong>Lambda formatter</strong> (as opposed to the native EventBridge formatter).</li> 
 <li>The <strong>timezone</strong> to use for displaying dates and times in the email notifications, for example <strong>Australia/Melbourne</strong>. The default is UTC.</li> 
 <li>The Service Catalog <strong>provider display</strong> <strong>name</strong>, which can be your company, organization, or team name.</li> 
 <li>The Service Catalog <strong>product version</strong>, which defaults to <strong>v1</strong>. Increment this value if you make a change in the product CloudFormation template file.</li> 
 <li>Whether you deployed the IAM role StackSet in Step 2, earlier.</li> 
 <li>The <strong>principal type</strong> that will use the Service Catalog product. If you are using IAM Identity Center, enter <code style="color: #000000;">IAM_Identity_Center_Permission_Set</code>. If you have federated IAM roles configured, enter <code style="color: #000000;">IAM role name</code>.</li> 
 <li>If you entered <code style="color: #000000;">IAM_Identity_Center_Permission_Set</code> in the previous step, enter the IAM Identity Center URL subdomain. This is used for creating a shortcut URL link to Security Hub in the email. For example, if your URL looks like this: <code style="color: #000000;">https://d-abcd1234.awsapps.com/start/#/</code>, then enter <code style="color: #000000;">d-abcd1234</code>.</li> 
 <li>The principals that will have access to the Service Catalog product across the AWS accounts. If you’re using IAM Identity Center, this will be a permission set name. If you plan to deploy the provided permission set in the next step (Step 4), press enter to accept the default value <strong>SubscribeToSecurityNotifications</strong>. Otherwise, enter an appropriate permission set name (for example <code style="color: #000000;">AWSPowerUserAccess</code>) or IAM role name that users use.</li> 
</ul> 
<p>The script creates the following CloudFormation stacks:</p> 
<ul> 
 <li><a href="https://github.com/aws-samples/improving-security-incident-response-times-by-decentralizing-notifications/blob/main/02_Service_Catalog/01_Bucket/SecurityHub_notifications_SC-Bucket.yaml" rel="noopener" target="_blank"><strong>SecurityHub_notifications_SC-Bucket.yaml</strong></a>: This stack creates an <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage (Amazon S3)</a> bucket that contains the file <a href="https://github.com/aws-samples/improving-security-incident-response-times-by-decentralizing-notifications/blob/main/02_Service_Catalog/02_ServiceCatalog_Product/SecurityHub-Notifications.yaml" rel="noopener" target="_blank"><strong>SecurityHub-Notifications.yaml</strong></a>, which is the CloudFormation template file associated with the Service Catalog product. The script modifies the Mappings section of the template file that has the configuration details depending on the answers to the installation script questions, and then uploads the file to the bucket.</li> 
 <li><a href="https://github.com/aws-samples/improving-security-incident-response-times-by-decentralizing-notifications/blob/main/02_Service_Catalog/03_ServiceCatalog_Portfolio/SecurityHub_notifications_ServiceCatalog_Portfolio.yaml" rel="noopener" target="_blank"><strong>SecurityHub_notifications_ServiceCatalog_Portfolio.yaml</strong></a>: This stack creates a Service Catalog portfolio and product using the Amazon S3 bucket from the previous step and gives permissions to the required principals to launch the product.</li> 
</ul> 
<p>After the script finishes the installation, it outputs the Service Catalog Product ID, which you will need in the next step. The script then asks whether it should automatically share this Service Catalog portfolio with the entire organization or a specific account, or whether you will configure sharing to specific OUs manually.</p> 
<p><strong>(Optional) To manually configure sharing with an OU</strong></p> 
<ol> 
 <li>In the Service Catalog console, choose <strong>Portfolios</strong>.</li> 
 <li>Choose <strong>Subscribe to AWS Security Hub notifications</strong>.</li> 
 <li>On the <strong>Share</strong> tab, choose <strong>Add a share</strong>.</li> 
 <li>Choose <strong>AWS&nbsp;Organization</strong>, and then select the OU. The product will be shared to the accounts and child OUs within the selected OU.</li> 
 <li>Select <strong>Principal sharing</strong>, and then choose <strong>Share</strong>.</li> 
</ol> 
<p>To expand this solution across Regions, enable Security Hub <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/finding-aggregation.html" rel="noopener" target="_blank">cross-Region aggregation</a>. This results in the email notifications coming from the linked Regions that are configured in Security Hub, even though the Service Catalog product is instantiated in a single Region. If cross-Region aggregation isn’t enabled and you want to monitor multiple Regions, you must repeat the preceding steps in all the Regions you are monitoring.</p> 
<h3>Step 4: (Optional, recommended) Provision IAM Identity Center permission sets</h3> 
<p>This step requires you to have completed Step 2 (Deploy IAM roles across accounts with CloudFormation StackSets).</p> 
<p>If you’re using IAM Identity Center, the following steps create a custom permission set, <strong>SubscribeToSecurityNotifications</strong>, that provides least-privileged access for users to subscribe to security notifications. The permission set redirects to the Service Catalog page to launch the product.</p> 
<p><strong>To provision Identity Center permission sets</strong></p> 
<ol> 
 <li>Sign in to the AWS Management Console from the management AWS account, or from an IAM Identity Center <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/delegated-admin.html" rel="noopener" target="_blank">delegated administrator</a></li> 
 <li>Download the <a href="https://github.com/aws-samples/improving-security-incident-response-times-by-decentralizing-notifications/blob/main/03_IAM_Identity_Center_PermissionSets/SecurityHub_notifications_PermissionSets.yaml" rel="noopener" target="_blank">CloudFormation template</a> for creating the permission set.</li> 
 <li>Navigate to the AWS CloudFormation page.</li> 
 <li>Choose <strong>Create stack</strong>, and then choose <strong>With new resources (standard)</strong>.</li> 
 <li>Choose <strong>Upload a template file</strong> and upload the CloudFormation template you downloaded earlier: <code style="color: #000000;">SecurityHub_notifications_PermissionSets.yaml</code>. Then choose <strong>Next</strong>.</li> 
 <li>Enter the stack name <strong>SecurityNotifications-PermissionSet</strong>.</li> 
 <li>Enter the following values for the parameters: 
  <ol> 
   <li><strong>AWS IAM Identity Center Instance ARN</strong>: Use the AWS CloudShell command in the parameter description to get the IAM Identity Center ARN.</li> 
   <li><strong>Permission set name</strong>: Use the default value <strong>SubscribeToSecurityNotifications</strong>.</li> 
   <li><strong>Service Catalog product ID</strong>: Use the last output line of the install.sh script in Step 3, or alternatively get the product ID from the Service Catalog console for the product account.</li> 
  </ol> </li> 
 <li>Choose <strong>Next</strong>. Then optionally enter tags and choose <strong>Next</strong> Wait for the stack creation to finish.</li> 
</ol> 
<p>Next, go to the IAM Identity Center console, select your AWS accounts, and <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/howtoviewandchangepermissionset.html#howtochangepermissionset" rel="noopener" target="_blank">assign access</a> to the<strong> SubscribeToSecurityNotifications</strong> permission set for your users or groups.</p> 
<h3>Testing</h3> 
<p>To test the solution, sign in to an AWS account, making sure to sign in with the designated IAM Identity Center permission set or IAM role. Launch the product in Service Catalog to subscribe to Security Hub security notifications.</p> 
<p>Wait for a Security Hub notification. For example, if you have the <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/fsbp-standard.html" rel="noopener" target="_blank">AWS Foundational Security Best Practices (FSBP)</a> standard enabled, creating an S3 bucket with no server access logging enabled should generate a notification within a few minutes.</p> 
<h2>Additional considerations</h2> 
<p>Keep in mind the following:</p> 
<ul> 
 <li>There is a <a href="https://aws.amazon.com/sns/pricing/" rel="noopener" target="_blank">cost for each SNS email notification</a> sent out, as well as for <a href="https://aws.amazon.com/servicecatalog/pricing/" rel="noopener" target="_blank">Service Catalog API</a> calls and execution of <a href="https://aws.amazon.com/lambda/pricing/" rel="noopener" target="_blank">Lambda functions</a> (if enabled).</li> 
 <li>Consider enabling Security Hub <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/controls-findings-create-update.html#consolidated-control-findings" rel="noopener" target="_blank">consolidated control findings</a> so you don’t receive multiple email notifications for a control that applies to multiple standards.</li> 
 <li>The blog post <a href="https://aws.amazon.com/blogs/security/considerations-for-security-operations-in-the-cloud/" rel="noopener" target="_blank">Considerations for security operations in the cloud</a> compares and contrasts the centralized, decentralized, and hybrid models for security operations.</li> 
 <li>The <a href="https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/sec_detect_investigate_events_noncompliant_resources.html" rel="noopener" target="_blank">Initiate remediation for non-compliant resources</a> and <a href="https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/incident-response.html" rel="noopener" target="_blank">Incident response</a> sections of the <a href="https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html" rel="noopener" target="_blank">Security Pillar</a> of the <a href="https://aws.amazon.com/architecture/well-architected/" rel="noopener" target="_blank">AWS Well-Architected Framework</a> walk through best practices for remediation and incident response.</li> 
</ul> 
<h2>Cleanup</h2> 
<p>To remove unneeded resources after testing the solution, follow these steps:</p> 
<ol> 
 <li>In the workload account or accounts where the product was launched: 
  <ol> 
   <li>Go to the Service Catalog provisioned products page and terminate each associated provisioned product. This stops security notifications from being sent to the email address associated with the product.</li> 
  </ol> </li> 
 <li>In the AWS account that is hosting the directory: 
  <ol> 
   <li>In the Service Catalog console, choose <strong>Portfolios</strong>, and then choose <strong>Subscribe to AWS Security Hub notifications</strong>. On the <strong>Share</strong> tab, select the items in the list and choose <strong>Actions</strong>, then choose<strong> Unshare</strong>.</li> 
   <li>In the CloudFormation console, delete the <strong>SecurityNotifications-Service-Catalog</strong> stack.</li> 
   <li>In the Amazon S3 console, for the two buckets starting with <strong>securitynotifications-sc-bucket</strong>, select the bucket and choose <strong>Empty</strong> to empty the bucket.</li> 
   <li>In the CloudFormation console, delete the <strong>SecurityNotifications-SC-Bucket</strong> stack.</li> 
  </ol> </li> 
 <li>If applicable, go to the management account or the CloudFormation delegated administrator account and delete the <strong>SecurityNotifications-IAM-roles-StackSet</strong> stack.</li> 
 <li>If applicable, go to the management account or the IAM Identity Center delegated administrator account and delete the <strong>SecurityNotifications-PermissionSet</strong> stack.</li> 
</ol> 
<h2>Conclusion</h2> 
<p>This solution described in this blog post enables you to set up a self-service standardized mechanism that application or workload owners can use to get security notifications within minutes through email, as opposed to being contacted by a security team later. This can help to improve your security posture by reducing the incident resolution time, which reduces the time that a security issue remains active.</p> 
<p>&nbsp;<br />If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.<br />&nbsp;</p> 
<footer> 
 <div class="blog-author-box">
  <img alt="Cheng Wang" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/10/07/chengwww.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Cheng Wang</span>
  <br />Cheng is a Solutions Architect at AWS in Melbourne, Australia. With a strong consulting background, he leverages his cloud infrastructure expertise to support enterprise customers in designing and deploying cloud architectures that drive efficiency and innovation.
 </div> 
 <div class="blog-author-box">
  <img alt="Karthikeyan KM" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/10/07/mohankkm.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Karthikeyan KM</span>
  <br />KM is a Senior Technical Account Manager who supports enterprise users at AWS. He has over 18 years of IT experience, and he enjoys building reliable, scalable, and efficient solutions.
 </div> 
 <div class="blog-author-box">
  <img alt="Randy Patrick" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/10/07/randyap.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Randy Patrick</span>
  <br />Randy is a Senior Technical Account Manager who supports enterprise customers at AWS. He has 20 years of IT experience, which he uses to build secure, resilient, and optimized solutions. In his spare time, Randy enjoys tinkering in home theater, playing guitar, and hiking trails at the park.
 </div> 
 <div class="blog-author-box"> 
  <p><span class="lb-h4">Contributor</span></p> 
  <p>Special thanks to Rizvi Rahim, who made significant contributions to this post.</p> 
 </div> 
</footer>
