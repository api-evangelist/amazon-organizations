---
title: "How to import existing AWS Organizations SCPs and RCPs to CloudFormation"
url: "https://aws.amazon.com/blogs/security/how-to-import-existing-aws-organizations-scps-and-rcps-to-cloudformation/"
date: "Wed, 23 Apr 2025 20:02:03 +0000"
author: "Swara Gandhi"
feed_url: "https://aws.amazon.com/blogs/security/tag/aws-organizations/feed/"
---
<p>Many <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a> customers begin by creating and manually applying <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html" rel="noopener" target="_blank">service control policies (SCPs)</a>&nbsp;and <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_rcps.html" rel="noopener" target="_blank">resource control policies (RCPs)</a> through the AWS Management Console or <a href="https://aws.amazon.com/cli" rel="noopener" target="_blank">AWS Command Line Interface (AWS CLI)</a> when they first set up their environments.&nbsp;However, as the organization grows and the number of policies increases, this manual approach can become cumbersome. It can result in limited visibility into all implemented SCPs and RCPs, the targets they’re attached to (such as accounts, organizational units [OUs], or nested OUs), and the ability to manage updates effectively. Without clear visibility and proper access controls, it becomes challenging to track who’s making changes and how they are being made.</p> 
<p>Importing existing SCPs and RCPs into <a href="https://aws.amazon.com/cloudformation" rel="noopener" target="_blank">AWS CloudFormation</a> can help streamline the management of your policies by enabling history tracking, policy validation through <a href="https://docs.aws.amazon.com/cloudformation-cli/latest/hooks-userguide/creating-and-managing-hooks.html" rel="noopener" target="_blank">CloudFormation Hooks</a>, and rollback capabilities. You can also <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/git-sync-concepts-terms.html" rel="noopener" target="_blank">sync stacks with source code stored in a Git repository with Git sync</a>.&nbsp;With Git sync, you can use pull requests and version tracking to configure, deploy, and update your CloudFormation stacks from a centralized location. When you commit changes to the template or the deployment file, CloudFormation automatically updates the stack.</p> 
<p>In this post, I provide a solution to import existing SCPs and RCPs into <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-guide.html" rel="noopener" target="_blank">AWS CloudFormation templates</a> using the <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/generate-IaC.html" rel="noopener" target="_blank">CloudFormation infrastructure as code generator (IaC generator)</a>. By using the IaC generator, you can automate the management of your SCPs and RCPs at scale.</p> 
<blockquote>
 <p><strong>Important:</strong> Only existing policies are brought into CloudFormation; policies are not recreated.</p>
</blockquote> 
<h2 id="solution-overview">Solution overview</h2> 
<p>The solution in this post includes a command line tool for discovering SCPs and RCPs in your organization and automating policy import into CloudFormation templates.&nbsp;The following figure shows the end-to-end flow:</p> 
<div class="wp-caption aligncenter" id="attachment_38073" style="width: 1298px;">
 <img alt="Figure 1: Solution overview" class="size-full wp-image-38073" height="364" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/22/Import-Organizations-SCPs-RCPs-1.jpeg" width="1288" />
 <p class="wp-caption-text" id="caption-attachment-38073">Figure 1: Solution overview</p>
</div> 
<p>The end-to-end flow shown in the preceding figure includes:</p> 
<ol type="1"> 
 <li><strong>Run the tool:</strong> The tool automates the following steps and can be run in the management account or delegated administrator account. 
  <ol type="a"> 
   <li><strong>Identify SCPs and RCPs in the organization:&nbsp;</strong>The tool begins by making API calls to the Organizations service to retrieve the policies in your environment. It then provides a count of the total number of SCPs and RCPs present.</li> 
   <li><strong>Identify AWS Control Tower SCPs and RCPs and policies without targets:</strong> 
    <ol type="i"> 
     <li><strong>AWS Control Tower SCPs and RCPs:</strong> The tool checks for <a href="https://aws.amazon.com/controltower/" rel="noopener" target="_blank">SCPs and RCPs created by AWS Control Tower</a>&nbsp;and lists them in the output for your review. 
      <ul> 
       <li>SCPs are identified by the <code style="color: #000000;">aws-guardrails-</code> prefix in their policy names.</li> 
       <li>RCPs are identified by the&nbsp;<code style="color: #000000;">AWSControlTower-Controls-</code> prefix in their policy names.</li> 
      </ul> </li> 
     <li><strong>Policies with no targets:</strong>&nbsp;The tool also identifies SCPs and RCPs that aren’t attached to an <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_ous.html" rel="noopener" target="_blank">organizational unit (OU)</a>, <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_getting-started_concepts.html" rel="noopener" target="_blank">account</a>, or <a href="https://docs.aws.amazon.com/organizations/latest/APIReference/API_Root.html#:~:text=A%20root%20is%20a%20top,AWS%20account%20in%20the%20organization." rel="noopener" target="_blank">root</a> and lists them. These policies might be redundant or need reassignment.</li> 
    </ol> </li> 
   <li><strong>CloudFormation IaC generator scan:</strong>&nbsp;At this stage, you will be prompted to confirm whether you want to import the policies to the CloudFormation templates using&nbsp;CloudFormation resource scan. If you select <strong>yes</strong>, the tool will initiate a CloudFormation resource scan using <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/choose-iac-tool/cloudformation.html" rel="noopener" target="_blank">IaC generator</a>&nbsp;to get details about the policies including policy name, targets, policy tags, and so on.</li> 
   <li><strong>Create template from scanned policy resources:</strong> The tool&nbsp;generates CloudFormation template with the policy resources. The template will include the policies without targets (if any).</li> 
  </ol> </li> 
 <li><strong>Review process:</strong> After the template is generated, it’s recommended that you preview the template using <a href="https://console.aws.amazon.com/cloudformation/home?#iac-generator" rel="noopener" target="_blank">IaC generator</a> from&nbsp;the CloudFormation console. We recommend viewing the template resource section to verify and adjust the generated templates as needed (step 11 of the solution deployment).</li> 
 <li><strong>Create CloudFormation stacks with the generated templates:&nbsp;</strong>After reviewing the templates, import them into CloudFormation stacks for deployment. It’s important to note that only existing policies are brought into CloudFormation—policies aren’t recreated. The templates reflect the current policies and policy attributes.</li> 
</ol> 
<h2 id="consideration-before-implementing-the-solution">Consideration before implementing the solution</h2> 
<p>There are some considerations that you need to keep in mind before implementing the solution.</p> 
<ul> 
 <li>If you have enabled the <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_delegate_policies.html" rel="noopener" target="_blank">AWS Organizations policy management delegation</a>, you should run this solution from the delegated administrator account.&nbsp;Otherwise, you can run the solution using the management account.<br /> 
  <blockquote>
   <p><strong>Note:</strong> <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_delegate_policies.html" rel="noopener" target="_blank">Delegating management of organizations policies</a> to a delegated administrator member account is recommended best practice.</p>
  </blockquote> </li> 
 <li>AWS Control Tower SCPs and RCPs (with or without targets) won’t be imported to the CloudFormation templates because they should be managed using AWS Control Tower.&nbsp;<a href="https://docs.aws.amazon.com/controltower/latest/userguide/walkthrough-delete.html" rel="noopener" target="_blank">Changes made to AWS Control Tower resources</a> outside of AWS Control Tower can cause drift and&nbsp;affect AWS Control Tower functionality in unpredictable ways.</li> 
 <li><a href="https://console.aws.amazon.com/organizations/?#/policies/p-FullAWSAccess" rel="noopener" target="_blank">FullAWSAccess</a> SCP and <a href="https://677276094262-lagb2dh4.us-east-1.console.aws.amazon.com/organizations/v2/home/policies/resource-control-policy/p-RCPFullAWSAccess" rel="noopener" target="_blank">RCPFullAWSAccess</a>&nbsp;RCP are <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_reference_available-policies.html" rel="noopener" target="_blank">AWS managed policies</a> that won’t be imported to CloudFormation because CloudFormation stacks do not allow importing AWS managed resources.</li> 
 <li>You might see multiple CloudFormation templates created if you exceed the <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html" rel="noopener" target="_blank">CloudFormation template size quotas</a>. To help ensure smooth creation, the tool is designed to automatically split the content into multiple templates if necessary, allowing you to stay within the quotas while still accommodating the imported content.</li> 
 <li>Note that the generated templates have the following attributes set by default. 
  <ul> 
   <li><a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-deletionpolicy.html?icmpid=docs_cfn_console" rel="noopener" target="_blank">Deletion policy</a>: Set to <code style="color: #000000;">Retain</code>. This enables persisting&nbsp;the policies even when their related stack is deleted.</li> 
   <li><a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-updatereplacepolicy.html?icmpid=docs_cfn_console" rel="noopener" target="_blank">Update Replace policy:</a> Set to <code style="color: #000000;">Delete</code>. This enables deletion of the physical ID associated with the policy when the policy is updated.</li> 
  </ul> </li> 
</ul> 
<h2 id="solution-deployment">Solution deployment</h2> 
<p>Now that you understand the solution and know the considerations to keep in mind, you’re ready to deploy the solution.</p> 
<ol type="1"> 
 <li>Clone the solution repository. 
  <div class="hide-language"> 
   <pre><code class="lang-text">git clone https://github.com/aws-samples/sample-tool-for-importing-existing-AWS-SCPs-and-RCPs
</code></pre> 
  </div> </li> 
 <li>Navigate to the directory of the cloned repository. 
  <div class="hide-language"> 
   <pre><code class="lang-text">cd sample-tool-for-importing-existing-AWS-SCPs-and-RCPs/
</code></pre> 
  </div> </li> 
 <li>Install the solution (Python 3.10+ is supported) 
  <div class="hide-language"> 
   <pre><code class="lang-text">pip install .
</code></pre> 
  </div> </li> 
 <li>If you want to use a particular <a href="https://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> principal to run this tool, create a <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html" rel="noopener" target="_blank">profile</a> in <code style="color: #000000;">~./aws/config</code>&nbsp;using an IAM principal from your <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_getting-started_concepts.html" rel="noopener" target="_blank">AWS Organizations management account</a>. If you have delegated policy management to a member account, make sure that you use the IAM principal from the&nbsp;<a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_delegate_policies.html" rel="noopener" target="_blank">delegated administrator account</a>.&nbsp;<br /> 
  <blockquote>
   <p><strong>Note:&nbsp;</strong>The IAM principal will need to have&nbsp;the following permissions to be able to successfully run the tool.</p>
  </blockquote> 
  <div class="hide-language"> 
   <pre><code class="lang-text">{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "organizations:Describe*",
                "organizations:List*",
                "cloudformation:Describe*",
                "cloudformation:Detect*",
                "cloudformation:Estimate*",
                "cloudformation:Get*",
                "cloudformation:List*",
                "cloudformation:CreateGeneratedTemplate",
                "cloudformation:StartResourceScan",
                "cloudformation:UpdateGeneratedTemplate",
                "cloudformation:CreateStack",
                "cloudformation:DeleteStack",
                "cloudformation:UpdateStack",
                "cloudformation:DeleteGeneratedTemplate"
            ],
            "Resource": "*"
        }
    ]
}
</code></pre> 
  </div> </li> 
 <li>You can run the tool specifying a profile name as a command line argument. Use the following command, replacing <code style="color: #ff0000;"><em>&lt;profile_name&gt;</em></code> with the name of the profile you created in step 4. If you do not specify a profile, the default profile from the file <code style="color: #000000;">~./aws/config</code> will be used. 
  <div class="hide-language"> 
   <pre><code class="lang-text">policy-importer --profile <span style="color: #ff0000;"><em>&lt;profile_name&gt;</em></span>
</code></pre> 
  </div> </li> 
 <li>After the preceding command is executed, you will see an output displaying the total number of SCPs and RCPs found in the organization. The output will also list AWS Control Tower managed policies as INFO, in addition to policies without targets as a WARNING. At this point, you can enter <code style="color: #000000;">Yes</code> to proceed with scanning to import the policies, or enter <code style="color: #000000;">No</code> if you want to exit.<br /> 
  <blockquote>
   <p> <strong>Note:</strong> If policies without targets are detected, we recommend stopping at this point. Either delete the policies without targets or assign appropriate targets to them. You can then rerun the tool from step 5. If you proceed without addressing the policies without targets, be aware that these policies will also be included in the CloudFormation template. </p>
  </blockquote> <p> </p>
  <div class="wp-caption aligncenter" id="attachment_38075" style="width: 1298px;">
   <img alt="Figure 2: Terminal view with policies identified in the organization" class="size-full wp-image-38075" height="771" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/22/Import-Organizations-SCPs-RCPs-2.jpeg" style="border: 1px solid #bebebe;" width="1288" />
   <p class="wp-caption-text" id="caption-attachment-38075">Figure 2: Terminal view with policies identified in the organization</p>
  </div> </li> 
 <li>If you enter <code style="color: #000000;">Yes</code>, the CloudFormation IaC resource scan will begin immediately. You will see the resource scan ID Amazon Resource Name (ARN) displayed.<br /> 
  <blockquote>
   <p><strong>Note:&nbsp;</strong>A scan can take up to 10 minutes for every 1,000 resources. </p>
  </blockquote> <p></p>
  <div class="wp-caption aligncenter" id="attachment_38076" style="width: 1298px;">
   <img alt="Figure 3: Terminal view with AWS CloudFormation resource scan details" class="size-full wp-image-38076" height="58" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/22/Import-Organizations-SCPs-RCPs-3.jpeg" style="border: 1px solid #bebebe;" width="1288" />
   <p class="wp-caption-text" id="caption-attachment-38076">Figure 3: Terminal view with AWS CloudFormation resource scan details</p>
  </div> </li> 
 <li>You can also review the scan progress from the&nbsp;<a href="https://console.aws.amazon.com/cloudformation/home?#iac-generator" rel="noopener" target="_blank">IaC generator page</a> of the CloudFormation console as shown in the following figure. To get to the IaC generator page, go to the CloudFormation console and choose <strong>IaC generator</strong> from the navigation pane.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38077" style="width: 1298px;">
   <img alt="Figure 4:&nbsp;View the scan summary in the CloudFormation console to track progress" class="size-full wp-image-38077" height="557" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/22/Import-Organizations-SCPs-RCPs-4.jpeg" style="border: 1px solid #bebebe;" width="1288" />
   <p class="wp-caption-text" id="caption-attachment-38077">Figure 4:&nbsp;View the scan summary in the CloudFormation console to track progress</p>
  </div> </li> 
 <li>Upon completion of the scan, the template generation process will be initiated.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38078" style="width: 1298px;">
   <img alt="Figure 5: Terminal view showing CloudFormation template being created" class="size-full wp-image-38078" height="175" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/22/Import-Organizations-SCPs-RCPs-5.jpeg" style="border: 1px solid #bebebe;" width="1288" />
   <p class="wp-caption-text" id="caption-attachment-38078">Figure 5: Terminal view showing CloudFormation template being created</p>
  </div> </li> 
 <li>After the template creation is finished, sign in to the <a href="https://console.aws.amazon.com/cloudformation/home?#iac-generator" rel="noopener" target="_blank">AWS CloudFormation IaC console</a>. Choose the <strong>Templates</strong> tab to review the generated templates and verify that they align with your requirements.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38079" style="width: 1299px;">
   <img alt="Figure 6: View CloudFormation templates in the console" class="size-full wp-image-38079" height="504" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/22/Import-Organizations-SCPs-RCPs-6.jpeg" style="border: 1px solid #bebebe;" width="1289" />
   <p class="wp-caption-text" id="caption-attachment-38079">Figure 6: View CloudFormation templates in the console</p>
  </div> </li> 
 <li>You can review the policies added to a template by selecting a template name.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38080" style="width: 1297px;">
   <img alt="Figure 7: Review policies included in the template" class="size-full wp-image-38080" height="571" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/22/Import-Organizations-SCPs-RCPs-7.jpeg" style="border: 1px solid #bebebe;" width="1287" />
   <p class="wp-caption-text" id="caption-attachment-38080">Figure 7: Review policies included in the template</p>
  </div> </li> 
 <li>When satisfied, you can proceed to import the templates into CloudFormation stacks for deployment by selecting <strong>Import to stack</strong>.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38081" style="width: 1298px;">
   <img alt="Figure 8: Import a template into the CloudFormation stack" class="size-full wp-image-38081" height="585" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/22/Import-Organizations-SCPs-RCPs-8.jpeg" style="border: 1px solid #bebebe;" width="1288" />
   <p class="wp-caption-text" id="caption-attachment-38081">Figure 8: Import a template into the CloudFormation stack</p>
  </div> </li> 
 <li>Follow the prompts to create a stack.<br /> 
  <div class="wp-caption aligncenter" id="attachment_38082" style="width: 1297px;">
   <img alt="Figure 9: AWS CloudFormation stack example" class="size-full wp-image-38082" height="565" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/22/Import-Organizations-SCPs-RCPs-9.jpeg" style="border: 1px solid #bebebe;" width="1287" />
   <p class="wp-caption-text" id="caption-attachment-38082">Figure 9: AWS CloudFormation stack example</p>
  </div> </li> 
 <li>The&nbsp;tool automatically creates a folder named <strong>Policies</strong> in your current directory and downloads the generated templates.<br /> 
  <blockquote>
   <p><strong>Note:&nbsp;</strong>If you encounter errors, check the <a href="https://github.com/aws-samples/sample-tool-for-importing-existing-AWS-SCPs-and-RCPs/tree/main?tab=readme-ov-file#error-handling" rel="noopener" target="_blank">recommended solutions</a> for common issues.</p>
  </blockquote> </li> 
</ol> 
<h2 id="recommended-next-steps">Recommended next steps</h2> 
<p>As shown in the following figure, there are two recommended next steps.</p> 
<div class="wp-caption aligncenter" id="attachment_38083" style="width: 1298px;">
 <img alt="Figure 10: Solution overview including recommended next steps" class="size-full wp-image-38083" height="417" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/22/Import-Organizations-SCPs-RCPs-10.jpeg" width="1288" />
 <p class="wp-caption-text" id="caption-attachment-38083">Figure 10: Solution overview including recommended next steps</p>
</div> 
<p>After the existing policies are imported into a CloudFormation stack, we recommend storing your CloudFormation templates in a private Git repository. You can use the <code style="color: #000000;">Policies</code> folder that was automatically created by the tool in the current local directory with the generated templates downloaded and set up a continuous integration and delivery (CI/CD) pipeline to efficiently manage the imported policies.</p> 
<p>By using a Git repository, you can use version control features like pull requests, branch management, and history tracking. This approach allows your team to efficiently review, update, and deploy policies with better collaboration and control. You can also create a CI/CD pipeline to automate the deployment of changes to your CloudFormation stacks, helping to ensure that updates are consistent and reliable.</p> 
<p>We also recommend incorporating&nbsp;<a href="https://docs.aws.amazon.com/cloudformation-cli/latest/hooks-userguide/creating-and-managing-hooks.html" rel="noopener" target="_blank">CloudFormation Hooks</a> in your environment. CloudFormation Hooks can help validate policies (and other resources) against best practices, to help ensure that they adhere to the correct syntax, follow security best practices, and minimize potential vulnerabilities.</p> 
<p>Related resources:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/git-sync.html" rel="noopener" target="_blank">Syncing stacks with Git source code</a></li> 
 <li><a href="https://aws.amazon.com/blogs/security/validate-iam-policies-in-cloudformation-templates-using-iam-access-analyzer/" rel="noopener" target="_blank">Blog:&nbsp;Validate IAM policies in CloudFormation templates using IAM Access Analyzer</a></li> 
 <li><a href="https://www.youtube.com/watch?v=Td2Gu2DRD7A&amp;ab_channel=AWSEvents" rel="noopener" target="_blank">Demonstration:&nbsp;Validate your IAM policies with AWS CloudFormation hooks</a></li> 
</ul> 
<h2 id="conclusion">Conclusion</h2> 
<p>Importing existing AWS Organizations service control policies (SCPs) and resource control policies (RCPs)&nbsp;into CloudFormation provides an efficient and scalable approach to managing and automating your AWS governance.&nbsp;After they’ve been imported, you can manage and update policies directly in CloudFormation, to help ensure consistency and version control across your organization.&nbsp;The&nbsp;tool also creates a <code style="color: #000000;">Policies</code> folder in your current directory, storing downloaded templates for use as a central repository and integration with a CI/CD pipeline.</p> 
<p>By using <a href="https://docs.aws.amazon.com/cloudformation-cli/latest/hooks-userguide/what-is-cloudformation-hooks.html" rel="noopener" target="_blank">CloudFormation Hooks</a>, you can further improve your policy management by validating SCPs and RCPs against best practices and policy grammar. This approach centralizes your policy updates, making governance more automated and efficient while reducing the risk of misconfiguration.</p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Swara Gandhi" class="aligncenter size-full wp-image-38089" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/04/23/Swara-Gandhi.jpeg" width="120" /> 
  </div> 
  <h3 class="lb-h4">Swara Gandhi</h3> 
  <p>Swara is a Senior Solutions Architect on the AWS Identity Solutions team. She works on building secure and scalable end-to-end identity solutions. She is passionate about everything identity, security, and cloud.</p> 
  <p></p>
 </div> 
</footer>
