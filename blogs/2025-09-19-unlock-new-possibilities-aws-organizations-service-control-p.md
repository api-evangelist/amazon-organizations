---
title: "Unlock new possibilities: AWS Organizations service control policy now supports full IAM language"
url: "https://aws.amazon.com/blogs/security/unlock-new-possibilities-aws-organizations-service-control-policy-now-supports-full-iam-language/"
date: "Fri, 19 Sep 2025 17:50:10 +0000"
author: "Swara Gandhi"
feed_url: "https://aws.amazon.com/blogs/security/tag/aws-organizations/feed/"
---
<p><a href="https://aws.amazon.com" rel="noopener noreferrer" target="_blank">Amazon Web Service (AWS)</a> recently announced that <a href="https://aws.amazon.com/organizations/" rel="noopener noreferrer" target="_blank">AWS Organizations</a> now offers full <a href="https://aws.amazon.com/iam/" rel="noopener noreferrer" target="_blank">AWS Identity and Access Management (IAM)</a>&nbsp;policy language support for service control policies (SCPs). With this feature, you can&nbsp;use conditions, individual resource Amazon Resource Names (ARNs), and the <code>NotAction</code> element&nbsp;with <code>Allow</code> statements. Additionally, you can now use wildcards at the beginning or middle of the <code>Action</code> element strings and implement the <code>NotResource</code> element&nbsp;in both <code>Allow</code> and <code>Deny</code> statements in SCPs.&nbsp;This feature is now available across <a href="https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/govcloud-differences.html" rel="noopener noreferrer" target="_blank">AWS commercial and AWS GovCloud (US) </a>Regions.</p> 
<p>In this blog post, we walk through a set of newly supported SCP language capabilities that simplify permission management cases. These enhancements enable more intuitive and concise policy designs. We explore how these capabilities address past limitations to reduce operational overhead and improve policy readability. We also show what the previous implementation looked like and provide an example of how the new capability makes the intent clearer and implementation simpler.</p> 
<h2>Overview of the&nbsp;newly supported elements</h2> 
<p>The following table lists the supported SCP language elements along with their purpose and applicable effects. Elements and effects shown in <strong>bold</strong> indicate newly supported capabilities.</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Element</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Purpose</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Supported effects</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_syntax.html#scp-syntax-version" rel="noopener noreferrer" target="_blank">Version</a></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Specifies the language syntax rules to use for processing the policy.</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code>Allow</code>, <code>Deny</code></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_syntax.html#scp-syntax-statement" rel="noopener noreferrer" target="_blank">Statement</a></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Serves as the container for policy elements. You can have multiple statements in an SCP.</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code>Allow</code>, <code>Deny</code></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_syntax.html#scp-syntax-sid" rel="noopener noreferrer" target="_blank">Statement ID (Sid)</a></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">(Optional) Provides a friendly name for the statement.</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code>Allow</code>, <code>Deny</code></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_syntax.html#scp-syntax-effect" rel="noopener noreferrer" target="_blank">Effect</a></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Defines whether the SCP statement <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_getting-started_concepts.html#allowlist" rel="noopener noreferrer" target="_blank">allows</a> or <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_getting-started_concepts.html#denylist" rel="noopener noreferrer" target="_blank">denies</a> access to the IAM users and roles in an account.</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code>Allow</code>, <code>Deny</code></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_syntax.html#scp-syntax-action" rel="noopener noreferrer" target="_blank">Action</a></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Specifies the AWS service and actions that the SCP allows or denies.</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code>Allow</code>, <code>Deny</code></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_syntax.html#scp-syntax-action" rel="noopener noreferrer" target="_blank">NotAction</a></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Specifies the AWS service and actions that are exempt from the SCP. Used instead of the <code>Action</code> element.</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code><strong>Allow</strong></code>, <code>Deny</code></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_syntax.html#scp-syntax-resource" rel="noopener noreferrer" target="_blank">Resource</a></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Specifies the AWS resources that the SCP applies to.</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code><strong>Allow</strong></code>, <code>Deny</code></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_notresource.html" rel="noopener noreferrer" target="_blank">NotResource</a></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Specifies the AWS resources that are exempt from the SCP. Used instead of the <code>Resource</code>&nbsp;element.</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code><strong>Allow</strong></code>, <code><strong>Deny</strong></code></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_syntax.html#scp-syntax-condition" rel="noopener noreferrer" target="_blank">Condition</a></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Specifies the conditions for when the statement is in effect.</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code><strong>Allow</strong></code>, <code>Deny</code></td> 
  </tr> 
 </tbody> 
</table> 
<p>Additionally, you can now use the wildcard characters * and ? anywhere in the <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_syntax.html#scp-syntax-action" rel="noopener noreferrer" target="_blank">Action or NotAction</a> element. Previously, these wildcards were only allowed by themselves or at the end of an element. For example, all of the following are now valid:</p> 
<ul> 
 <li><code>"servicename:action*"</code></li> 
 <li><code>"servicename:*action"</code></li> 
 <li><code>"servicename:some*action"</code></li> 
 <li><code>"servicename:*"</code></li> 
</ul> 
<h2>Navigating new SCP language capabilities</h2> 
<p>Let’s explore recommended policy strategies and best practices by walking through some examples.</p> 
<h3>Using <code>Deny</code> with NotResource</h3> 
<p>You can use the <code>NotResource</code> element to apply a policy across resources except those explicitly listed. This is especially useful for implementing broad deny-by-default policies with scoped exceptions, simplifying policy structure while enforcing strong boundaries.</p> 
<p><strong>Example 1:</strong></p> 
<p>The goal of this example is to enforce a resource perimeter that blocks access to resources outside the organization, except for a defined set of <a href="https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_owned_resources.md" rel="noopener noreferrer" target="_blank">service-owned resources</a>.</p> 
<ul> 
 <li><strong>Previous implementation:</strong> The policy used a tag-based approach to manage exceptions. It required tagging IAM principals with <code>dp:exclude:resource:s3=true</code> to grant access to external resources. This created operational overhead in tag management and introduced potential security risks if tags were incorrectly applied.</li> 
 <li><strong>Improved implementation:</strong> With support for <code>NotResource</code> in <code>Deny</code> statements, the updated SCP uses a single, consolidated <code>Deny</code> statement denying the action except for a defined set of AWS-owned resources.</li> 
</ul> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Policy structure before NotResource support</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Policy structure after NotResource support</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"> 
    <div class="hide-language"> 
     <pre><code class="lang-css">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EnforceResourcePerimeterAWSResourcesS3",
      "Effect": "Deny",
      "Action": "s3:GetObject",
      "Resource": "*",
      "Condition": {
        "StringNotEqualsIfExists": {
          "aws:ResourceOrgID": "&lt;my-org-id&gt;",
          "aws:PrincipalTag/dp:exclude:resource:s3": "true"
        }
      }
    }
  ]
}</code></pre> 
    </div> </td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"> 
    <div class="hide-language"> 
     <pre><code class="lang-css">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EnforceResourcePerimeterAWSResources",
      "Effect": "Deny",
      "Action": "s3:GetObject",
      "NotResource": [
        "arn:aws:s3:::service-owned-bucket/*",
        "arn:aws:s3:::service-owned-bucket2/*"
      ],
      "Condition": {
        "StringNotEquals": {
          "aws:ResourceOrgID": "&lt;org-id&gt;"
        }      
       }
    }
  ]
}</code></pre> 
    </div> </td> 
  </tr> 
 </tbody> 
</table> 
<p><strong>Example 2:</strong></p> 
<p>This example denies access to <a href="https://aws.amazon.com/bedrock" rel="noopener noreferrer" target="_blank">Amazon Bedrock</a> models except for one specific model.</p> 
<ul> 
 <li><strong>Before this change:</strong>&nbsp;SCP relied on a broad permission baseline for AWS accounts within the organization by allowing access to Amazon Bedrock actions by default, while explicitly denying invocation of three specific models (examples: <code>Deepseek</code>, <code>Anthropic</code>, and <code>meta</code>). However, this approach requires continuous operational overhead to make sure policies are updated to deny access to newly added models to avoid exposure to potentially unwanted models.</li> 
 <li><strong>Improved implementation:</strong> With support for <code>NotResource</code> in <code>Deny</code> statements, the updated SCP uses a single, consolidated <code>Deny</code> statement that denies actions except Amazon models.</li> 
</ul> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Policy structure before NotResource support</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Policy structure after NotResource support</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"> 
    <div class="hide-language"> 
     <pre><code class="lang-css">{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": "bedrock:*",
			"Resource": "*"
		},
		{
			"Effect": "Deny",
			"Action": [
				"bedrock:InvokeModel",
				"bedrock:InvokeModelWithResponseStream",
				"bedrock:PutFoundationModelEntitlement"
			],
			"Resource": [
				"arn:aws:bedrock:*::foundation-model/deepseek.*",
				"arn:aws:bedrock:*::foundation-model/anthropic.*",
				"arn:aws:bedrock:*::foundation-model/meta.*"
			]
		}
	]
}</code></pre> 
    </div> </td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"> 
    <div class="hide-language"> 
     <pre><code class="lang-css">{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": "bedrock:*",
			"Resource": "*"
		},
		{
			"Sid": "Statement1",
			"Effect": "Deny",
			"Action": [
				"bedrock:InvokeModel",
				"bedrock:InvokeModelWithResponseStream",
				"bedrock:PutFoundationModelEntitlement"
			],
			"NotResource": [
				"arn:aws:bedrock:*::foundation-model/amazon.*"
			]
		}
	]
}</code></pre> 
    </div> </td> 
  </tr> 
 </tbody> 
</table> 
<h3>Using <code>Allow</code>&nbsp;with conditions</h3> 
<p>By using the <code>Condition</code> element, you can specify the circumstances under which a policy statement is in effect. While optional, this element is now supported in <code>Allow</code> statements within SCPs, enabling more precise and scalable access control.</p> 
<p><strong>Note:</strong> We recommend using explicit <code>Deny</code> statements when authoring SCPs in most cases. Using <code>Deny</code> statements help make sure that each control works independently and remains enforceable. Relying solely on allow statements and the implicit deny-by-default model can lead to unintended access, because broader or overlapping <code>Allow</code> statements can override more restrictive ones.</p> 
<p>The following example allows access to specific AWS services in certain AWS Regions.</p> 
<ul> 
 <li><strong>Before this change: </strong>The policy uses a single <code>Allow</code> statement under the Sid: <code>AllowSpecificServices</code>. It lists broad service-level actions (for example, <code>"ec2:"</code>, <code>"s3:"</code>, and so on) in the <code>Action</code> element and applies them across resources (<code>"Resource": "*"</code>). Because AWS SCPs operate under a deny-by-default model, this setup effectively permits actions across the listed services while implicitly denying access to other services not included. For example, an explicit <code>Deny</code> restricts actions outside <code>us-east-1</code>, <code>us-west-2</code>, and <code>eu-central-1</code> using a Region condition.</li> 
 <li><strong>Improved implementation:&nbsp;</strong>In the updated example, the policy allows the same services, but only when they are requested in specific Regions (for example, <code>"us-east-1"</code>, <code>"us-west-2"</code>, and <code>"eu-central-1"</code>). This is achieved using the <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-requestedregion" rel="noopener noreferrer" target="_blank">aws:RequestedRegion</a> condition key in the <code>Allow</code> statement. This enhancement allows organizations to retain basic Allow logic while introducing contextual boundaries—such as limiting access by Region, account, or resource tag—previously only possible with <code>Deny</code> conditions.</li> 
</ul> 
<p><strong>Note:</strong> We recommend using one broad <code>Allow</code> statement and multiple targeted <code>Deny</code> statements in your policies. Avoid writing additional <code>Allow</code> statements that might overlap, because doing so could lead to unintended access. The recommended approach is to start with a broad <code>Allow</code> statement and then use <code>Deny</code> statements to refine and restrict access as needed.</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Policy structure before support for Allow with conditions</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Policy structure after support for Allow with conditions</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"> 
    <div class="hide-language"> 
     <pre><code class="lang-css">{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Sid":"AllowSpecificServices",
         "Effect":"Allow",
         "Action":[
            "ec2:*",
            "s3:*",
            "rds:*",
            "lambda:*",
            "cloudformation:*",
            "iam:*",
            "cloudwatch:*"
         ],
         "Resource":"*"
      },
      {
         "Sid":"AllowAccessOnlyTo3Regions",
         "Effect":"Deny",
         "Action":"*",
         "Resource":"*",
         "Condition":{
            "StringNotEquals":{
               "aws:RequestedRegion":[
                  "us-east-1",
                  "us-west-2",
                  "eu-central-1"
               ]
            }
         }
      }
   ]
}</code></pre> 
    </div> </td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"> 
    <div class="hide-language"> 
     <pre><code class="lang-css">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowServicesBasedOnRegion",
      "Effect": "Allow",
      "Action": [
        "ec2:*",
        "s3:*",
        "rds:*",
        "lambda:*",
        "cloudformation:*",
        "iam:*",
        "cloudwatch:*"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": [
            "us-east-1",
            "us-west-2",
            "eu-central-1"
          ]
        }
      }
    }
  ]
}</code></pre> 
    </div> </td> 
  </tr> 
 </tbody> 
</table> 
<h3>Other newly supported elements</h3> 
<p>To bring SCPs to full IAM policy language support, additional elements are now supported. While technically valid, some of these constructs require additional considerations and testing in practice because of their potential for unintended access if not carefully managed.</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Newly supported feature</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Important considerations</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code>Action</code> with wildcards (<code>*</code>, <code>?</code>)</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Can help shorten policies but use with caution—new actions added by AWS will match existing wildcard patterns as designed, potentially granting unintended permissions.</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code>NotAction</code> with wildcards (<code>*</code>, <code>?</code>)</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">We recommend using <code>NotAction</code> with a <code>Deny</code> statement if you want to deny all actions&nbsp;<em>except</em>&nbsp;those listed, which helps future-proof your controls (for example, denying everything in Amazon EC2 except actions that don’t match&nbsp;<code>“*vpn*”</code>.</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code>Allow</code> with <code>NotResource</code></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Limited use cases. While supported, <code>Allow</code> with <code>NotResource</code> can default to including all resources—potentially allowing access to new resources added later. Use with caution and prefer explicit <code>Deny</code> statements when possible.</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code>Allow</code> with <code>NotAction</code></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Limited use cases. While supported, <code>Allow</code> with <code>NotAction</code> can unintentionally permit access to new actions added by AWS. Use with caution and prefer explicit <code>Deny</code> statements to maintain control as services evolve.</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code>Allow</code> with <code>Resource</code> other than&nbsp;wildcard <code>“*”</code>.</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">When using Allow with specific resources (not <code>"*"</code>), make sure your policy design avoids conflicting or overlapping <code>Allow</code> statements. Start with a broad <code>Allow</code>, then use targeted <code>Deny</code> statements to restrict access—this helps prevent unintended access from overlapping <code>Allow</code> statements.</td> 
  </tr> 
 </tbody> 
</table> 
<h2>Validate your policies with&nbsp;IAM Access Analyzer</h2> 
<p>You can use <a href="https://aws.amazon.com/iam/access-analyzer" rel="noopener noreferrer" target="_blank">AWS IAM Access Analyzer</a> to validate your SCPs before applying them, using both policy validation and custom policy checks.</p> 
<p>IAM Access Analyzer <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-policy-validation.html" rel="noopener noreferrer" target="_blank">validates your policy against IAM policy grammar and best practices</a>. You can view policy validation check findings that include security warnings, errors, general warnings, and suggestions. These findings provide actionable recommendations to help you author policies that are both functional and aligned with security best practices.</p> 
<p><a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html" rel="noopener noreferrer" target="_blank">Custom policy checks</a> are an IAM Access Analyzer capability that security teams can use to help them accurately and proactively identify critical permissions in their policies. Custom policy checks can determine whether a new version of a policy is more permissive than the previous version. They use automated reasoning—a form of static analysis—to provide a higher level of security assurance in the cloud.</p> 
<p>Custom policy checks can be embedded into continuous integration and continuous delivery (CI/CD) pipelines, so that policies can be checked without being deployed. Developers can also run custom policy checks from their local development environments and receive fast feedback on whether the policies they are authoring comply with your organization’s security standards. For more information refer to <a href="https://aws.amazon.com/blogs/security/introducing-iam-access-analyzer-custom-policy-checks/" rel="noopener noreferrer" target="_blank">introducing IAM Access Analyzer custom policy checks</a>.</p> 
<h2>Conclusion</h2> 
<p>The latest enhancements to AWS service control policies improve policy expressiveness and precision while reducing operational effort. By enabling constructs like <code>Allow</code> with conditions and specific resource ARNs, supporting <code>NotResource</code> in <code>Deny</code> statements, and expanding wildcard flexibility, you can simplify your policies and avoid layered or complex policies to achieve your goals. These updates bring SCPs in parity with IAM policy capabilities and empower organizations to implement cleaner, more intuitive access controls. As a best practice, it’s important to use these capabilities carefully—especially wildcard use—to avoid unintended permissions as AWS services evolve. We also encourage the implementation of explicit <code>Deny</code> statements as a best practice and using <code>Allow</code> statements when needed.</p> 
<hr /> 
<p>If you have feedback about this post, submit comments in the&nbsp;Comments&nbsp;section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.</p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Swara Gandhi" class="aligncenter size-full wp-image-39143" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/09/12/swaragandhi.ganswara.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Swara Gandhi</h3> 
  <p>Swara is a Senior Solutions Architect on the AWS Identity Solutions team. She works on building secure and scalable end-to-end identity solutions. She is passionate about everything identity, security, and cloud.</p> 
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Niti Prasad" class="aligncenter size-full wp-image-39143" height="160" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2025/09/12/nitiprasad.awsniti.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Niti Prasad</h3> 
  <p>Niti is a Senior Security Solutions Architect supporting Strategic Accounts. She supports customers as they look to secure and govern their AWS environment.&nbsp;Her enthusiasm for security drives her to continuously explore innovative ways to help customers protect their cloud workloads.</p> 
 </div> 
</footer>
