---
title: "Enforce resource configuration to control access to new features with AWS"
url: "https://aws.amazon.com/blogs/security/enforce-resource-configuration-to-control-access-to-new-features-with-aws/"
date: "Mon, 16 Dec 2024 20:18:32 +0000"
author: "Yossi Cohen"
feed_url: "https://aws.amazon.com/blogs/security/tag/aws-organizations/feed/"
---
<p>Establishing and maintaining an effective security and governance posture has never been more important for enterprises. This post explains how you, as a security administrator, can use <a href="https://aws.amazon.com/" rel="noopener" target="_blank">Amazon Web Services (AWS)</a> to enforce resource configurations in a manner that is designed to be secure, scalable, and primarily focused on feature gating.</p> 
<p>In this context, <em>feature gatin</em>g means that newly supported AWS features and configurations can’t be used unless you explicitly approve them. With feature gating, you maintain control over your AWS environment when new services and capabilities are introduced.</p> 
<p>This blog post demonstrates a unique approach to giving users, such as DevOps teams, controlled flexibility within safe boundaries by allowing resource provisioning that uses only approved configurations. This approach also accommodates configurations that will be supported in future versions of the resource, keeping them restricted until explicitly approved, as shown in Figure 1.</p> 
<div class="wp-caption aligncenter" id="attachment_36850" style="width: 790px;">
 <img alt="Figure 1: Restrict resource provisioning to approved configurations only" class="size-large wp-image-36850" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/12/img1-2-1024x376.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-36850">Figure 1: Restrict resource provisioning to approved configurations only</p>
</div> 
<h2>Apply your resource configuration enforcement</h2> 
<p>As shown in Figure 2, our solution for resource configuration enforcement (RCFGE) uses <a href="https://docs.aws.amazon.com/cloudformation-cli/latest/hooks-userguide/what-is-cloudformation-hooks.html" rel="noopener" target="_blank">AWS CloudFormation Hooks</a>. By using Hooks, you can run custom logic during the provisioning of resources. These are proactive controls because you inspect and enforce resource configurations <em>before</em> the resource is created, updated, or deleted.</p> 
<p>Your Hook will only be effective if CloudFormation supports the AWS resources that you are using and if you implement a <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html" rel="noopener" target="_blank">service control policy (SCP)</a> that helps prevent users from provisioning resources outside of CloudFormation.</p> 
<div class="wp-caption aligncenter" id="attachment_36851" style="width: 790px;">
 <img alt="Figure 2: How CloudFormation Hooks work" class="size-large wp-image-36851" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/12/img2-1-1024x522.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-36851">Figure 2: How CloudFormation Hooks work</p>
</div> 
<p>The flow shown in Figure 2 consists of the following five steps:</p> 
<ol> 
 <li>DevSecOps registers and configures a CloudFormation Hook in the account.</li> 
 <li>DevOps specifies a CloudFormation template that defines the required resources and configurations.</li> 
 <li>CloudFormation creates a new stack resource, starting the provisioning process based on the template.</li> 
 <li>The Hook is triggered before provisioning for each resource that’s defined in the template, and runs custom validation logic.</li> 
 <li>If the validation checks pass, CloudFormation proceeds with provisioning; if not, the process is terminated.</li> 
</ol> 
<h2>Make your solution scalable</h2> 
<p>To achieve scalable operations, you should implement a reusable and generic Hook that targets all supported CloudFormation resource types. This Hook enforces resource configuration by loading resource specification files from an external object storage, such as an&nbsp;<a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a>&nbsp;bucket.</p> 
<p>These specification files define validation rules in a declarative language. Using this approach, you can add and remove resource configuration validation rules by editing the declarative files. When you externalize custom logic as decoupled validation rules from the Hook, DevSecOps personnel can manage these rules at scale without affecting your infrastructure.</p> 
<div class="wp-caption aligncenter" id="attachment_36852" style="width: 790px;">
 <img alt="Figure 3: Externalize custom logic as validation rule files in an S3 bucket" class="size-large wp-image-36852" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/12/img3-1024x522.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-36852">Figure 3: Externalize custom logic as validation rule files in an S3 bucket</p>
</div> 
<p>Figure 3 shows how the solution has been revised to support this approach. Steps 1–3 are the same as in the flow shown in Figure 2:</p> 
<ol> 
 <li>DevSecOps registers and configures a CloudFormation Hook in the account.</li> 
 <li>DevOps specifies a CloudFormation template that defines the required resources and configurations.</li> 
 <li>CloudFormation creates a new stack resource, starting the provisioning process based on the template.</li> 
 <li>The Hook is triggered before provisioning for each resource that’s defined in the template.</li> 
 <li>The Hook loads the relevant resource specification file from the S3 bucket and executes the validation rules against the current resource in the CloudFormation template.</li> 
 <li>If the validation checks pass, CloudFormation proceeds with provisioning; if not, the process is terminated.</li> 
</ol> 
<p>You need to configure the <a href="https://docs.aws.amazon.com/cloudformation-cli/latest/hooks-userguide/hooks-type-schema-syntax.html" rel="noopener" target="_blank">Hook schema</a> and the <a href="https://docs.aws.amazon.com/cloudformation-cli/latest/hooks-userguide/hook-configuration-schema.html" rel="noopener" target="_blank">Hook configuration schema</a> to evaluate the configurations of all supported resources across your AWS accounts before changes are provisioned. This setup should cover create, update, and delete operations so that the Hook can help prevent non-approved configurations across stacks.</p> 
<p>By using&nbsp;<a href="https://docs.aws.amazon.com/cfn-guard/latest/ug/what-is-guard.html" rel="noopener" target="_blank">AWS CloudFormation Guard</a>, you can externalize validation rules from the Hook, as described in <a href="https://aws.amazon.com/blogs/security/extend-your-pre-commit-hooks-with-aws-cloudformation-guard" rel="noopener" target="_blank">Extend your pre-commit hooks with AWS CloudFormation Guard</a>. Guard is an open source, general purpose, policy-as-code (PaC) evaluation tool that validates CloudFormation templates against custom rules to help you stay aligned with your organizational policies. For example, the&nbsp;<a href="https://docs.aws.amazon.com/controltower/latest/controlreference/s3-rules.html#ct-s3-pr-1-rule" rel="noopener" target="_blank">CT.S3.PR.1 rule specification</a>&nbsp;demonstrates a Guard rule that requires an S3 bucket to have its settings configured to block public access. These validation rules apply to currently supported AWS resource configurations and features, but they don’t restrict potential future properties.</p> 
<h2>Boost your solution with feature gating</h2> 
<p>Your risk model might lead you to look for mechanisms that further restrict the AWS resource configurations that you allow in your environments. As you will see, the proposed solution restricts authorized workforce users so that they can use new configurations only if you enable them. The proposed approach uses feature gating because it continues to enforce your configurations even when AWS adds new options for your resources.</p> 
<p>Guard aims to validate required constraints; but to meet the feature gating objective, you should implement validation rules that check whether resource configurations fulfill structural constraints described by the restricted version of CloudFormation resource schemas. These schemas help you confine the possible resource configurations that can be provisioned in your environment no matter what new configurations AWS introduces.</p> 
<div class="wp-caption aligncenter" id="attachment_36853" style="width: 790px;">
 <img alt="Figure 4: Enforce resource configuration with restricted resource schema templates" class="size-large wp-image-36853" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/12/img4-1024x560.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-36853">Figure 4: Enforce resource configuration with restricted resource schema templates</p>
</div> 
<p>Figure 4 shows an updated version of the same flow where validation rules are implemented by using restricted resource schema templates, which are stored in an S3 bucket. These templates are based on the original&nbsp;<a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-resource-specification.html" rel="noopener" target="_blank">CloudFormation resource schemas</a>, representing a snapshot of these schemas at a specific point in time. Steps 1–4 are the same as in the flow shown in Figure 3:</p> 
<ol> 
 <li>DevSecOps registers and configures a CloudFormation Hook in the account.</li> 
 <li>DevOps specifies a CloudFormation template that defines the required resources and configurations.</li> 
 <li>CloudFormation creates a new stack resource, starting the provisioning process based on the template.</li> 
 <li>The Hook is triggered before provisioning for each resource that’s defined in the template.</li> 
 <li>The Hook loads the relevant restricted resource schema template file from the S3 bucket and uses it to execute schema validation against the current resource in the CloudFormation template.</li> 
 <li>If the validation checks pass, CloudFormation proceeds with provisioning; if not, the process is terminated.</li> 
</ol> 
<p>A restricted resource schema template is a subset of its corresponding original CloudFormation resource schema. It includes additional constraints that limit certain properties to specific values and patterns or exclude certain properties entirely. Furthermore, these templates contain placeholders that you fill in with runtime values, such as the account ID, which your Hook provides as part of the <a href="https://docs.aws.amazon.com/cloudformation-cli/latest/hooks-userguide/guard-hooks-write-rules.html#:~:text=%3A%0A%20%20%20%20%20%20%20%20%7BPreviousResourceProperties%7D-,HookContext,the%20resource%20change%20was%20initiated%20by%20Cloud%20Control%20API%2C%20or%20the%20create-stack%2C%20update-stack%2C%20or%20delete-stack%20operations.,-Resources" rel="noopener" target="_blank">Hook context</a>.</p> 
<div class="wp-caption aligncenter" id="attachment_36854" style="width: 790px;">
 <img alt="Figure 5: Resource configuration enforcement (RCFGE) CloudFormation Hook flow" class="size-large wp-image-36854" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/12/img5-1024x518.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-36854">Figure 5: Resource configuration enforcement (RCFGE) CloudFormation Hook flow</p>
</div> 
<p>As shown in Figure 5, the flow within the RCFGE CloudFormation Hook involves the following steps:</p> 
<ol> 
 <li>The CloudFormation Hook is invoked with the Hook context and the resource’s configuration JSON object.</li> 
 <li>The Hook loads the restricted resource schema template from the S3 bucket and substitutes placeholders with the Hook context runtime values, producing a valid JSON schema.</li> 
 <li>The Hook validates the stack’s resource configuration JSON object against the schema. If it returns <code style="color: #000000;">OperationStatus.SUCCESS</code>, then CloudFormation proceeds with the provisioning process. If it returns <code style="color: #000000;">OperationStatus.FAILED</code>, then CloudFormation terminates the provisioning process.</li> 
</ol> 
<p>If a restricted resource schema template for a CloudFormation resource type isn’t found in the S3 bucket, the schema validation step fails by default.</p> 
<h3>Sample excerpt of a restricted schema template for an S3 bucket resource</h3> 
<p>The following is an excerpt from a restricted schema template for an S3 bucket. At runtime, your Hook processes this template, substituting the placeholders with relevant values from the Hook context. In this example, the Hook replaces the <code style="color: #ff0000; font-style: italic;">&lt;accountID&gt;</code> placeholder in the topic’s <code style="color: #000000;">pattern</code> with the actual account ID. The resulting JSON schema disallows additional properties beyond those defined by the schema and restricts the <a href="https://aws.amazon.com/sns/" rel="noopener" target="_blank">Amazon Simple Notification Service (Amazon SNS)</a> topics that can be used for event notifications.</p> 
<blockquote>
 <p><strong>Note</strong>: In the code samples that follow, we’ve omitted some code for brevity—we’ve indicated these omissions with three periods:&nbsp;<code style="color: #000000;">...</code></p>
</blockquote> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
  "type": "object",
  "required": [],
  "additionalProperties": false,
  "properties": {
        ...
      "NotificationConfiguration": {
          "$ref": "#/definitions/NotificationConfiguration"
      },
        ...
  },
  "definitions": {
        ...
      "NotificationConfiguration": {
          "type": "object",
          "additionalProperties": false,
          "properties": {
            ...
              "TopicConfigurations": {
                  "type": "array",
                  "uniqueItems": true,
                  "items": {
                      "$ref": "#/definitions/TopicConfiguration"
                  }
              }
          }
      },
        ...
      "TopicConfiguration": {
          "type": "object",
          "additionalProperties": false,
          "properties": {
        ...
              "Topic": {
                  "type": "string",
                  "<span style="background-color: yellow;">pattern": "^arn:aws:sns::$<span style="color: #ff0000; font-style: italic;">&lt;accountID&gt;</span>:.*$</span>"
              },
        ...
            }
      },
  }
}</code></pre> 
</div> 
<h3>CloudFormation template for an S3 bucket that adheres to the restricted schema</h3> 
<p>Let’s assume that your account ID is <code style="color: #000000;">111122223333</code>. The account ID is propagated to the Hook through the Hook context.</p> 
<p>The following is an excerpt from a CloudFormation template that aligns with the restricted schema for an S3 bucket instantiated from the template shown previously. As a result, your Hook allows the corresponding CloudFormation stack to proceed.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
   "AWSTemplateFormatVersion": "2010-09-09",
   "Resources": {
     "S3Bucket": {
       "Type": "AWS::S3::Bucket",
       "Properties": {
         "BucketName":
            "valid-bucket-sns-notification-configuration-template",
         "NotificationConfiguration": {
           "TopicConfigurations": [
             {
              "Topic":
                "<span style="background-color: yellow;">arn:aws:sns:eu-west-1:111122223333:this-is-my-topic-and-I-trust-it</span>",
              "Event": "s3:ObjectCreated:*"
             }
           ]
         }
       }
    }
  }
}</code></pre> 
</div> 
<h3>CloudFormation template for an S3 bucket that diverges from the restricted schema (example 1)</h3> 
<p>The following is an excerpt from a CloudFormation template that doesn’t align with the restricted schema for an S3 bucket instantiated from the template shown previously because it attempts to configure the&nbsp;Amazon SNS&nbsp;topic for the notification configuration, which uses an Amazon Resource Name (ARN) of another account. As a result, your Hook causes the corresponding CloudFormation stack to fail.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
   "AWSTemplateFormatVersion": "2010-09-09",
   "Resources": {
     "S3Bucket": {
       "Type": "AWS::S3::Bucket",
       "Properties": {
         "BucketName":
           "invalid-bucket-sns-notification-configuration-template",
         "NotificationConfiguration": {
            "TopicConfigurations": [
              {
               "Topic":
                 "<span style="background-color: yellow;">arn:aws:sns:eu-west-1:444455556666:this-is-not-my-topic</span>",
               "Event": "s3:ObjectCreated:*"
              }
            ]
         }
       }
     }
   }
}</code></pre> 
</div> 
<h3>CloudFormation template for an S3 bucket that diverges from the restricted schema (example 2)</h3> 
<p>The following is an excerpt from a CloudFormation template that doesn’t align with the restricted schema for an S3 bucket instantiated from the template shown previously. This time, it violates your feature gating objective by attempting to use a new, imaginary feature of an S3 bucket that isn’t approved for use by your restricted schema for an S3 bucket. As a result, your Hook causes the corresponding CloudFormation stack to fail.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Resources": {
    "S3Bucket": {
      "Type": "AWS::S3::Bucket",
      "Properties": {
        "BucketName":
           "valid-bucket-sns-notification-configuration-template",
        <span style="background-color: yellow;">"NewFeature": {</span>
           <span style="background-color: yellow;">"property-1": true,</span>
           <span style="background-color: yellow;">"property-2": "public"</span>
        <span style="background-color: yellow;">}</span>,                
        "NotificationConfiguration": {
          "TopicConfigurations": [
            {
              "Topic":
                 "<span style="background-color: yellow;">arn:aws:sns:eu-west-1:111122223333:this-is-my-topic-and-I-trust-it</span>",
              "Event": "s3:ObjectCreated:*"
            }
          ]
        }
      }
    }
  }
}</code></pre> 
</div> 
<h2>Protect your controls</h2> 
<p>If a security control itself isn’t protected adequately, it becomes a weak link in the security chain. For example, a surveillance camera (a physical security control) that isn’t securely mounted can be removed, rendering it useless. This principle also applies to your RCFGE solution.</p> 
<p>Next, we will show you how to isolate management activities to a dedicated account and use SCPs as preventative controls.</p> 
<h3>Isolate RCFGE management in a dedicated account</h3> 
<p>Organizing your AWS environment by using multiple accounts is a best practice because it enhances security, simplifies management, and allows for better resource isolation and cost tracking. Isolating the operation and management of your RCFGE solution in its own dedicated account is essential for securing the solution’s resources.</p> 
<p>With <a href="https://docs.aws.amazon.com/servicecatalog/latest/adminguide/using-stacksets.html" rel="noopener" target="_blank">AWS CloudFormation StackSets</a>, you can deploy and manage RCFGE stacks across multiple accounts and AWS Regions from a single central administrator account. This provides consistent and scalable infrastructure while maintaining centralized governance. With this functionality, you can deploy the RCFGE resources to existing accounts and automatically include new accounts as you add them to your organization, simplifying RCFGE management and providing uniformity across your environments. For more information, see&nbsp;<a href="https://aws.amazon.com/blogs/devops/deploy-cloudformation-hooks-to-an-organization-with-service-managed-stacksets/" rel="noopener" target="_blank">Deploy CloudFormation Hooks to an Organization with service-managed StackSets</a>.</p> 
<p>Figure 6 shows how to extend that idea so that you can operate the RCFGE solution at scale while maintaining isolation and the separation of duties. The solution operates across three key account types:</p> 
<ul> 
 <li><strong>Management account</strong> –use this account to create your organization and designate the CloudFormation StackSets delegated administrator account.</li> 
 <li><strong>Delegated administrator account</strong> – this account serves as the centralized management point for the RCFGE solution. It contains a continuous integration and continuous delivery (CI/CD) pipeline that provisions RCFGE resources across the organization by using CloudFormation StackSets with service managed permissions. The account hosts a centralized S3 bucket that stores the RCFGE restricted resource schema templates. The security engineering team uses this account to submit Hook code and restricted resource schema template changes, which trigger the CI/CD pipeline.</li> 
 <li><strong>Member accounts</strong> – each member account contains an RCFGE StackSet instance and an <a href="https://docs.aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> role for provisioning RCFGE resources. It also includes a CloudFormation Hook and an IAM role that allows the Hook to access the centralized S3 bucket with RCFGE restricted resource schema templates.</li> 
</ul> 
<p style="line-height: 1.25em;"></p>
<div class="wp-caption aligncenter" id="attachment_36858" style="width: 790px;">
 <img alt="Figure 6: Securely operate the RCFGE solution" class="size-large wp-image-36858" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/12/img6-1024x847.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-36858">Figure 6: Securely operate the RCFGE solution</p>
</div>
<p></p> 
<p>Let’s explore how the RCFGE solution architecture enforces resource configuration step by step, as shown in Figure 7.</p> 
<div class="wp-caption aligncenter" id="attachment_36859" style="width: 790px;">
 <img alt="Figure 7: CloudFormation stack deployment flow with RCFGE validation and enforcement" class="size-large wp-image-36859" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/12/img7-1024x359.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-36859">Figure 7: CloudFormation stack deployment flow with RCFGE validation and enforcement</p>
</div> 
<ol> 
 <li>DevOps initiates the deployment by specifying a CloudFormation template that defines the resources and configurations needed.</li> 
 <li>CloudFormation creates a new stack resource, initiating the resource provisioning process based on the provided template.</li> 
 <li>The RCFGE CloudFormation Hook is triggered for each resource defined in the CloudFormation template.</li> 
 <li>The Hook loads the corresponding restricted resource schema template from the S3 bucket.</li> 
 <li>The Hook validates a resource configuration: 
  <ul> 
   <li>The Hook processes the restricted resource schema template to create a JSON schema.</li> 
   <li>It uses this JSON schema to validate the current resource in the CloudFormation template.</li> 
   <li>If the resource is invalid according to the schema, the provisioning process is terminated.</li> 
  </ul> </li> 
 <li>If the current resource passes validation, CloudFormation proceeds with the resource provisioning process by creating and configuring the resources as specified in the template.</li> 
</ol> 
<h3>Use SCPs as preventive controls for your organization to help protect RCFGE</h3> 
<p>The following SCP excerpt accomplishes three objectives:</p> 
<ul> 
 <li>Implements a statement (see&nbsp;<code style="color: #000000;">AllowedListActions</code>) to explicitly specify the access that is allowed while other access is implicitly blocked.</li> 
 <li>Implements control objectives to help prevent changes to resources set up by the RCFGE solution (see&nbsp;<code style="color: #000000;">ProtectRCFGEResources</code> and&nbsp;<code style="color: #000000;">ProtectStackSetExecutionRole</code>).</li> 
 <li>Makes sure that AWS resource provisioning does not occur outside of CloudFormation (see&nbsp;<code style="color: #000000;">ProvisionResourcesViaCloudFormationOnly</code>).</li> 
</ul> 
<p>In this SCP excerpt, the <code style="color: #000000;">ProvisionResourcesViaCloudFormationOnly</code> statement restricts CloudFormation stacks to being managed only through <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_forward_access_sessions.html" rel="noopener" target="_blank">forward access sessions (FAS)</a> in AWS IAM.</p> 
<p>The <code style="color: #000000;">ProvisionResourcesViaCloudFormationOnly</code> statement explicitly prohibits direct create, update, and delete actions for all supported resources used in your environment. If needed, split this statement into multiple parts so you don’t exceed <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_reference_limits.html#min-max-values" rel="noopener" target="_blank">SCP size limits</a>, while providing comprehensive coverage of your resources to make sure that they are provisioned and managed only through CloudFormation.</p> 
<p>The <code style="color: #000000;">ProtectStackSetExecutionRole</code> statement in this example assumes that <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-orgs-activate-trusted-access.html" rel="noopener" target="_blank">CloudFormation trusted access</a> is activated with <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a>, which is required by StackSets to deploy across accounts and Regions by using service managed permissions.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowedListActions",
      "Effect": "Allow",
      "Action": [
        "s3:CreateBucket",
        "s3:DeleteBucket",
        "s3:DeleteBucketPolicy",
        "s3:PutAnalyticsConfiguration",
        "s3:PutBucketLogging",
        "s3:PutBucketNotification",
        "s3:PutBucketObjectLockConfiguration",
        "s3:PutBucketPolicy",
        "s3:PutBucketTagging",
        "s3:PutBucketVersioning",
        "s3:PutLifecycleConfiguration",
        "s3:PutMetricsConfiguration",
        "s3:PutReplicationConfiguration",
        "s3:GetObject",
        ...
      ],
      "Resource": "*"
    },
    {
      "Sid": "ProtectRCFGEResources",
      "Effect": "Deny",
      "Action": "*",
      "Resource": [
        "arn:aws:cloudformation:*:*:stack/RCFGEStackSet",
        "arn:aws:cloudformation:*:*:*/hook/RCFGEHook/*",
        "arn:aws:iam::*:role/RCFGEHookExecutionRole"
      ],
      "Condition": {
        "ArnNotLike": {
          "aws:PrincipalArn": [
            "arn:aws:iam::*:role/stacksets-exec-*"
          ]
        }
      }
    },
    {
      "Sid": "ProtectStackSetExecutionRole",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "arn:aws:iam::*:role/stacksets-exec-*"
    },
    {
      "Sid": "ProvisionResourcesViaCloudFormationOnly",
      "Effect": "Deny",
      "Action": [
        "s3:CreateBucket",
        "s3:DeleteBucket",
        "s3:DeleteBucketPolicy",
        "s3:PutAnalyticsConfiguration",
        "s3:PutBucketLogging",
        "s3:PutBucketNotification",
        "s3:PutBucketObjectLockConfiguration",
        "s3:PutBucketPolicy",
        "s3:PutBucketTagging",
        "s3:PutBucketVersioning",
        "s3:PutLifecycleConfiguration",
        "s3:PutMetricsConfiguration",
        "s3:PutReplicationConfiguration",
        ...
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:CalledViaFirst": "cloudformation.amazonaws.com"
        }
      }
    }
  ]
}</code></pre> 
</div> 
<p>To allow the Hook to retrieve the necessary restricted resource schema templates, member accounts must be able to access the S3 bucket that contains the RCFGE templates. The following code sample shows the bucket policy for the S3 bucket that contains the RCFGE templates.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowRCFGEHookExecutionRoleGetRCFGETemplates",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Principal": "*",
      "Resource": "arn:aws:s3:::RCFGETemplates/*",
      "Condition": {
        "StringEquals": {
          "aws:PrincipalOrgID": "o-abcdef0123"
        },
        "ArnLike": {
          "aws:PrincipalArn": "arn:aws:iam::*:role/RCFGEHookExecutionRole"
        }
      }
    }
  ]
}</code></pre> 
</div> 
<p>As shown in the following code sample, the <code style="color: #000000;">RCFGEHookExecutionRole</code> IAM role in member accounts has a policy that grants read-only access to the RCFGE templates that are stored in an S3 bucket in the RCFGE delegated administrator account, where <code style="color: #000000;">555555555555</code> represents the account ID.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowRCFGEHookExecutionRoleGetRCFGETemplates",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::RCFGETemplates/*",
      "Condition": {
        "StringEquals": {
          "aws:ResourceAccount": "555555555555"
        }
      }
    }
  ]
}</code></pre> 
</div> 
<p>In the following code sample, the <code style="color: #000000;">RCFGEHookExecutionRole</code> IAM role in member accounts has a trust policy that allows it to be assumed only by the relevant CloudFormation <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-services" rel="noopener" target="_blank">service principals</a>, where <code style="color: #000000;">444455556666</code> represents the account ID of the member account.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
  "Version": "2012-10-17",
  "Statement": {
    "Sid": "AllowRCFGEHookExecutionRoleGetRCFGETemplatesTrust",
    "Effect": "Allow",
    "Principal": {
      "Service": [
        "hooks.cloudformation.amazonaws.com",
        "resources.cloudformation.amazonaws.com"
      ]
    },
    "Action": "sts:AssumeRole",
    "Condition": {
      "ArnLike": {
        "aws:SourceArn": "arn:aws:cloudformation:eu-west-1:444455556666:type/hook/RCFGEHook/*"
      }
    }
  }
}</code></pre> 
</div> 
<h3>Define baseline configuration for RCFGE and continuous monitoring with AWS Config</h3> 
<p>Defense in depth is an effective strategy because if one line of defense fails, additional layers are in place to help stop threats at subsequent points. With <a href="https://aws.amazon.com/config/" rel="noopener" target="_blank">AWS Config</a>, you can capture the configuration of RCFGE resources over time. You can set up&nbsp;<a href="https://docs.aws.amazon.com/config/latest/developerguide/evaluate-config_develop-rules.html" rel="noopener" target="_blank">AWS Config custom rules</a>&nbsp;to automatically assess the compliance of your RCFGE resources against predefined policies. For example, you can use an AWS Config custom rule to make sure that the RCFGE Hook hasn’t been altered or removed.</p> 
<h2>Conclusion</h2> 
<p>In this post, you learned how to use CloudFormation Hooks to create a resource configuration enforcement (RCFGE) solution on AWS that is designed to be secure and scalable and that supports feature gating. Using this approach, you, as a security administrator, can maintain strict control over resource configurations and feature adoption across your AWS environments. The solution provides a balanced approach to governance, so that DevOps teams have the flexibility to work within approved boundaries while making sure that new AWS features are only accessible after explicit approval.</p> 
<p>If you have feedback about this post, submit comments in the <strong>Comments </strong>section. For questions, start a new thread on the <a href="https://repost.aws/tags/TAm3R3LNU3RfSX9L23YIpo3w/aws-cloudformation" rel="noopener" target="_blank">CloudFormation re:Post</a> or contact <a href="https://console.aws.amazon.com/support/home" rel="noopener" target="_blank">AWS Support</a>.<br />&nbsp;</p> 
<footer> 
 <div class="blog-author-box">
  <img alt="Yossi Cohen" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/12/ycohen.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Yossi Cohen</span>
  <br />Yossi is a Senior Security Specialist Solutions Architect at AWS for the public sector in the EMEA region. Yossi has over two decades of experience in cloud-native architecture development, design, operations, technical due diligence, and governance in highly regulated environments. At AWS, Yossi collaborates closely with defense, intelligence, government, and public sector clients, helping them navigate their unique threat landscapes.
 </div> 
 <div class="blog-author-box">
  <img alt="Yaniv Rozenboim" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/12/12/yanivr.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Yaniv Rozenboim</span>
  <br />Yaniv is a Senior Solutions Architect at AWS with extensive experience in cloud architecture and security. He specializes in designing and implementing secure, scalable, and efficient cloud infrastructures. Yaniv works closely with clients to help them achieve their business goals through AWS technologies.
 </div> 
</footer>
