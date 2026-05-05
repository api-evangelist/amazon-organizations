---
title: "Secure root user access for member accounts in AWS Organizations"
url: "https://aws.amazon.com/blogs/security/secure-root-user-access-for-member-accounts-in-aws-organizations/"
date: "Fri, 22 Nov 2024 14:17:18 +0000"
author: "Jonathan VanKim"
feed_url: "https://aws.amazon.com/blogs/security/tag/aws-organizations/feed/"
---
<blockquote>
 <p><strong>November 17, 2025: </strong>The MFA Security Key program, which provided eligible customers with free MFA devices, has been discontinued effective November 6th, 2025. While existing devices will continue to function normally, no new orders for MFA security keys will be accepted after the program closure date. </p>
</blockquote> 
<hr /> 
<p><a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> now <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-enable-root-access.html" rel="noopener" target="_blank">supports centralized management of root access</a> for member accounts in <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a>. With this capability, you can remove unnecessary root user credentials for your member accounts and automate some routine tasks that previously required root user credentials, such as restoring access to <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> buckets and <a href="https://aws.amazon.com/sqs/" rel="noopener" target="_blank">Amazon Simple Queue Service (Amazon SQS)</a> queues that have policies that deny all access.</p> 
<p>In this blog post, we show how you can centrally manage root credentials and perform tasks that previously required root credentials across member accounts in your organization.</p> 
<h2>Centralized root access</h2> 
<p>This new IAM capability has two features: root credentials management and privileged root actions in member accounts.</p> 
<p><strong>Root credentials management</strong> enables you to centrally monitor, remove, and disallow recovery of long-term root credentials across your member accounts in AWS Organizations. This helps to prevent unintended root access and improves account security at scale throughout your organization. It helps reduce the number of privileged credentials and multi-factor authentication (MFA) devices that you need to manage.</p> 
<blockquote>
 <p> <strong>Note:</strong> After you enable root credentials management in your organization, new AWS accounts you create from AWS Organizations will not have a root user password, and will not be eligible for the root user password recovery procedure until you re-enable account recovery. </p>
</blockquote> 
<p><strong>Privileged root actions in member accounts </strong>provide you with a way to centrally perform the most common privileged tasks that previously required root user credentials in your organization member accounts. Your security teams can support your member account users by performing privileged tasks such as unlocking a misconfigured S3 bucket or SQS queue centrally, through short-term (maximum 15 minutes) task-scoped root sessions. You can authorize the root session to perform only the actions that the session was intended for. For example, a root session that you initiate to unlock an S3 bucket policy can only unlock an S3 bucket policy, and cannot be used for other root tasks. The root sessions can only be initiated from your management account or from a delegated administrator account. An IAM principal requires permissions to the new IAM action <code style="color: #000000;">sts:AssumeRoot</code> in the management account or the delegated administrator account to create a root session.</p> 
<p>Service control policies, VPC endpoint policies, and other relevant policies remain effective during the root sessions. For example, you can restrict root sessions to only expected networks.</p> 
<p>You can scope temporary root sessions with one of the following <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/security-iam-awsmanpol.html#security-iam-awsmanpol-IAMCreateRootUserPassword" rel="noopener" target="_blank">AWS managed policies</a>:</p> 
<ul> 
 <li><code style="color: #000000;">policy/root-task/IAMDeleteRootUserCredentials</code> – The root session is scoped to allow the deletion of member root credentials (console passwords, access keys, signing certiﬁcates, and MFA devices).</li> 
 <li><code style="color: #000000;">policy/root-task/IAMCreateRootUserPassword</code> – The root session is scoped to allow the creation of a member root login profile.</li> 
 <li><code style="color: #000000;">policy/root-task/IAMAuditRootUserCredentials</code> – The root session is scoped to review root credentials.</li> 
 <li><code style="color: #000000;">policy/root-task/S3UnlockBucketPolicy</code> – The root session is scoped to allow deletion of an S3 bucket policy</li> 
 <li><code style="color: #000000;">policy/root-task/SQSUnlockQueuePolicy</code> – The root session is scoped to allow deletion of an SQS queue resource policy.</li> 
</ul> 
<h2>Enable centralized root access</h2> 
<p>In this section, we show you how to enable centralized management of root access. You must be signed in to your organization management account with Organizations admin permissions.</p> 
<p><strong>To enable centralized root access (console)</strong></p> 
<ol> 
 <li>In the IAM console, in the left navigation menu, choose <strong>Root access management</strong>.</li> 
 <li>Choose the <strong>Enable</strong> When you enable centralized root access by using the console, you also enable trusted access for IAM in AWS Organizations. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36648" style="width: 750px;">
   <img alt="Figure 1: Centralized root access capability in the IAM console" class="size-large wp-image-36648" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/21/img1-1024x475.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36648">Figure 1: Centralized root access capability in the IAM console</p>
  </div><p></p> <p>On the <strong>Centralized root access for member accounts</strong> page, both the <strong>Root credentials management</strong> and <strong>Privileged root actions in member accounts</strong> capabilities are selected by default, as shown in Figure 2. As a security best practice, AWS strongly recommends that you delegate the administration of this service to a dedicated member account used by your security team that is separate from AWS accounts that are used to host your workloads or applications. You can also use a delegated administrator account to avoid unnecessary access to your management account.</p> </li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_36649" style="width: 750px;">
 <img alt="Figure 2: Enable root access management" class="size-large wp-image-36649" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/21/img2-1024x475.png" style="border: 1px solid #bebebe;" width="740" />
 <p class="wp-caption-text" id="caption-attachment-36649">Figure 2: Enable root access management</p>
</div> 
<h2>Enable centralized root access (CLI)</h2> 
<p>From the Organizations management account, you can also enable centralized root access from the command line.</p> 
<p><strong>To enable centralized root access (CLI)</strong></p> 
<ol> 
 <li>First, make sure that you’ve <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html" rel="noopener" target="_blank">updated</a> to the latest AWS CLI so that the new APIs are available.</li> 
 <li>After you’ve verified your CLI version, turn on the feature by running the following command: 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">  aws organizations enable-aws-service-access \
--service-principal iam.amazonaws.com</code></pre> 
  </div> </li> 
 <li>To reduce unnecessary access to the management account, delegate the administration of this service to a dedicated <strong>Security</strong> member account by using the following command. Make sure to replace <code><span style="color: #ff0000; font-style: italic;">&lt;MEMBER_ACCOUNT_ID&gt;</span></code> with the member account ID where the delegated administrator will register. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">aws organizations register-delegated-administrator --service-principal iam.amazonaws.com --account-id <span style="color: #ff0000; font-style: italic;">&lt;MEMBER_ACCOUNT_ID&gt;</span></code></pre> 
  </div> </li> 
 <li>Next, enable root actions: 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">aws iam enable-organizations-root-sessions 
aws iam enable-organizations-root-credentials-management</code></pre> 
  </div> </li> 
</ol> 
<p>Centralized root access is now enabled and delegated to a dedicated <strong>Security</strong> member account. From that account, you can manage root credentials for member accounts or gain short-term task-scoped root access into member accounts in order to perform specific root actions. Sign in to the <strong>Security</strong> member account to follow the rest of the steps in this post.</p> 
<h2>Root credentials management</h2> 
<p>The first feature that we will discuss is root credentials management. Navigate to the new centralized root access management console page as described earlier, and you will see the organizational structure. As shown in Figure 3, there is a <strong>Root user credentials</strong> field for each AWS account, which tells you if the root user credential is present.</p> 
<div class="wp-caption aligncenter" id="attachment_36650" style="width: 790px;">
 <img alt="Figure 3: Preview of member accounts with root credential status" class="size-large wp-image-36650" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/21/img3-1024x553.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-36650">Figure 3: Preview of member accounts with root credential status</p>
</div> 
<p>From this console page, you can delete or create the root user console password (login profile) for each member account.</p> 
<p><strong>To delete or create the root user console password</strong></p> 
<ol> 
 <li>Under <strong>Accounts</strong>, select one account and choose the <strong>Take privileged action</strong> button.</li> 
 <li>Select either <strong>Delete root user credentials</strong> or <strong>Allow password recovery</strong> (for AWS accounts where root credentials do not exist). Note that creating a root user login profile does not restore the previous root user configurations, such as the previously set password and associated MFA device.</li> 
</ol> 
<p style="line-height: 1.25em;"></p>
<div class="wp-caption aligncenter" id="attachment_36651" style="width: 790px;">
 <img alt="Figure 4: The Delete root user credentials feature in the IAM console" class="size-large wp-image-36651" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/21/img4-1024x434.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-36651">Figure 4: The <strong>Delete root user credentials</strong> feature in the IAM console</p>
</div>
<p></p> 
<h2>Privileged root tasks</h2> 
<p>After you enable the privileged root actions feature, you (as a security admin) will be able to use the console or CLI to perform privileged tasks such as unlocking S3 bucket or SQS queue policies in member accounts.</p> 
<p><strong>To perform privileged root actions (console)</strong></p> 
<ol> 
 <li>From your delegated administrator account, navigate to the IAM console. In the left navigation menu, choose <strong>Root access management</strong>.</li> 
 <li>Select the account where your S3 bucket or SQS queue exists. Then choose the <strong>Take privileged action</strong> button.</li> 
 <li>Select the privileged task you want to perform on the member account and provide the details of the S3 bucket or SQS queue from which you would like remove the resource policy. In the example in Figure 5, we’ve selected the <strong>Delete S3 bucket policy</strong> action and entered the URI of the S3 bucket in the member account. <p style="line-height: 1.25em;"></p>
  <div class="wp-caption aligncenter" id="attachment_36654" style="width: 750px;">
   <img alt="Figure 5: Privileged root actions in the IAM console" class="size-large wp-image-36654" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/21/img5-1024x401.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-36654">Figure 5: Privileged root actions in the IAM console</p>
  </div><p></p> </li> 
 <li>Confirm your intent to delete the resource policy and then choose <strong>Delete bucket policy</strong>.</li> 
</ol> 
<p><strong>To perform privileged root actions (CLI)</strong></p> 
<ol> 
 <li>From a terminal, <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html" rel="noopener" target="_blank">update</a> to the latest AWS CLI to make sure that the new APIs are available.</li> 
 <li>From the delegated admin account, get a root session in a member account by using the <code style="color: #000000;">STS/AssumeRoot</code> API action, as shown following. The default and maximum duration for the root session is 15 minutes. Make sure to replace <code style="color: #ff0000; font-style: italic;">&lt;MEMBER_ACCOUNT_ID&gt;</code> with your member account ID. 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text"> aws sts assume-root \
--target-principal <span style="color: #ff0000; font-style: italic;">&lt;MEMBER_ACCOUNT_ID&gt;</span> \
--task-policy-arn arn=arn:aws:iam::aws:policy/root-task/S3UnlockBucketPolicy </code></pre> 
  </div> </li> 
 <li>Use the following command to load the new credentials in the CLI: 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">export AWS_ACCESS_KEY_ID=[from sts assume root response]
export AWS_SECRET_ACCESS_KEY=[from sts assume root response] 
export AWS_SESSION_TOKEN=[from sts assume root response]</code></pre> 
  </div> </li> 
 <li>Delete the locked S3 bucket policy, making sure to replace <code style="color: #ff0000; font-style: italic;">&lt;value&gt;</code> with the name of the bucket: 
  <div class="hide-language"> 
   <pre class="unlimited-height-code"><code class="lang-text">aws s3api delete-bucket-policy --bucket <span style="color: #ff0000; font-style: italic;">&lt;value&gt;</span></code></pre> 
  </div> </li> 
</ol> 
<p>Now the bucket policy is available, and the bucket owner can write a new policy.</p> 
<h2>Best practices for centralized root access</h2> 
<p>This section outlines security considerations for centralized root access and usage of temporary root sessions.</p> 
<h3>Restrict who can use root sessions</h3> 
<p>Only grant access to use the new root sessions with <code style="color: #000000;">AssumeRoot</code> to admins and automations that need access. Within your organization’s management and delegated admin account for root management, only grant <code style="color: #000000;">sts:AssumeRoot</code> permissions to the persons and automations who need it.</p> 
<p>You can further limit the root actions that an admin or automation principal can perform by using the <a href="https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html" rel="noopener" target="_blank">AWS Security Token Service (AWS STS)</a> condition key <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_iam-condition-keys.html#ck_taskpolicyarn" rel="noopener" target="_blank">sts:TaskPolicyArn</a>, as shown in the following policy statement.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">{
   "Sid": "AllowLaunchingRootSessionsforS3Action",
   "Effect": "Allow",
   "Action": "sts:AssumeRoot",
   "Resource": "*",
   "Condition": {
      "StringEquals": {
         "sts:TaskPolicyARN": "arn:aws:iam::aws:policy/root-task/S3UnlockBucketPolicy"
      }
   }
}</code></pre> 
</div> 
<h3>Provide break glass access for root access</h3> 
<p>Break glass access refers to an alternative method of gaining access for use in exceptional circumstances, such as <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html#root-user-tasks" rel="noopener" target="_blank">tasks that can only be performed with root access</a>. When you follow the recommendations for <a href="https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/break-glass-access.html" rel="noopener" target="_blank">break glass access</a>, root user access is not needed. Review and update your existing procedures that rely on the root user to reduce the dependency of break glass access on root credentials.</p> 
<h3>Automate routine root actions</h3> 
<p>Because the centralized root access feature launched with AWS API, CLI, and SDK support, you can build automations to save time and reduce the need for security teams to take manual actions. For example, you can build an <a href="https://aws.amazon.com/eventbridge/" rel="noopener" target="_blank">Amazon EventBridge</a> integration with your ticketing system, where an EventBridge rule triggers an <a href="https://aws.amazon.com/lambda/" rel="noopener" target="_blank">AWS Lambda</a> function when the queue or bucket owner submits a ticket with approval. The Lambda function then uses a task-scoped root session to delete the policy on an SQS queue or S3 bucket. The diagram in Figure 6 shows an example of this type of automation.</p> 
<div class="wp-caption aligncenter" id="attachment_36656" style="width: 790px;">
 <img alt="Figure 6: An automation to delete policies on SQS queues or S3 buckets upon ticket approval" class="size-full wp-image-36656" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/11/21/img6-1.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-36656">Figure 6: An automation to delete policies on SQS queues or S3 buckets upon ticket approval</p>
</div> 
<p>The flow of the automation is as follows:</p> 
<ol> 
 <li>When a ticket to delete a policy on an S3 bucket or SQS queue is approved in your ticketing system, an event is put on the EventBridge event bus and an EventBridge rule is triggered on your delegated admin account.</li> 
 <li>The EventBridge rule triggers and invokes a Lambda function, passing a copy of the event.</li> 
 <li>The Lambda function uses the <code style="color: #000000;">assumeRoot</code> action, with the scope as one of the centralized root access task policies.</li> 
 <li>AWS STS returns temporary credentials with the scope that was determined in the task policy in the preceding step.</li> 
 <li>Using the temporary credentials, the Lambda function performs the privileged root action of deletion of S3 bucket or SQS queue policies on your member account.</li> 
</ol> 
<h3>Review and update your root usage and root credentials management procedures</h3> 
<p>Now that the tasks that most commonly required root user access (S3 bucket recovery and SQS queue recovery) no longer require long-lived root user credentials, you should revisit your procedures for those use cases and migrate to using root sessions instead of long-lived root users.</p> 
<p>Because it is now possible to delete the root user’s login profile, you should revisit the credential management procedures for the root users of your organization’s member accounts. Rather than performing password rotation or MFA device management, you might be able to improve your overall security posture by deleting the root login profile so no credential can be used to access the root user, and no way to initiate the password recovery procedure.</p> 
<h3>Continue to follow root user best practices for the root user in your organization’s management account</h3> 
<p>The new capability allows you to more simply manage root credentials from your organization’s member accounts. However, the organization’s management account root user must still exist with a known credential. <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/root-user-best-practices.html" rel="noopener" target="_blank">See the IAM User Guide</a> to learn more about the best practices for managing the organization’s management account root user.</p> 
<p>If you don’t have an MFA device for your organization’s management account root user, AWS will provide a <a href="https://aws.amazon.com/security/amazon-security-initiatives/free-mfa-security-key/" rel="noopener" target="_blank">free MFA device to eligible customers</a>.</p> 
<h2>How to remove root credentials in a scalable manner</h2> 
<p>This section outlines an approach to securely remove your root user credentials at scale. First, get a summary of root credentials for your member accounts. Review the usage of root credentials and identify accounts where root credentials can be safely removed. Then build automation to remove unused root credentials at scale across your member accounts.</p> 
<h3>Get a summary of root credentials for your member accounts</h3> 
<p>First, verify whether the root account for your member accounts has credentials before you remove them. If you already have a security admin role in your member accounts, use the <a href="https://docs.aws.amazon.com/IAM/latest/APIReference/API_GetAccountSummary.html" rel="noopener" target="_blank">getAccountSummary</a> action to audit root credentials. If you don’t have such a role and can’t create an audit role across your member accounts, you can build automation that uses an assume-root session scoped for the <code style="color: #000000;">IAMAuditRootUserCredentials</code> task to determine whether root credentials exist, and the last time the persistent root credentials were used. The persistent root account can have two types of credentials, password and access keys. You need to check both.</p> 
<p>Below is a sample bash script that you can run from your delegated admin account to get a summary of the root credentials on your member accounts.</p> 
<p><strong>To use the bash script to get a summary of root credentials</strong></p> 
<ol> 
 <li>Make sure that you have the AWS CLI installed and are signed in to your delegated admin account using admin role credentials with permissions to the <code style="color: #000000;">organizations:ListAccounts</code> and <code style="color: #000000;">sts:AssumeRoot</code> actions.</li> 
 <li>Save the code that follows to <code style="color: #000000;">GetRootCredentialsSummary.sh</code>.</li> 
 <li>The profile used in the scripts is <code style="color: #000000;">root-access-management</code>. You can modify the scripts to use another profile.</li> 
 <li>Run <code style="color: #000000;">GetRootCredentialsSummary.sh</code> on your terminal.</li> 
 <li>The output will have a <code style="color: #000000;">.csv</code> file for the root accounts that lists their last login, for both password and access key. Use this information to determine which root accounts are safe to remove. If there is no last-used information, then the <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_finding-unused.html" rel="noopener" target="_blank">credentials are unused</a>, and you can proceed to remove them. If they were used, trace the actions for which they were used in <a href="https://aws.amazon.com/cloudtrail/" rel="noopener" target="_blank">AWS CloudTrail</a>. If the credentials were used for root actions, replace them with an <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/root-user-tasks.html#root-user-tasks" rel="noopener" target="_blank">alternative method</a> for member accounts. Identify accounts for which root credentials cannot be removed at this time and need to be excluded from the deletion process.</li> 
</ol> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">#!/bin/bash

# Specify the AWS profile to use
AWS_PROFILE="root-access-management"

# Specify the account IDs to exclude (comma-separated)
EXCLUDED_ACCOUNTS="111122223333,444455556666"

# Get the list of accounts in the organization
ACCOUNTS=$(aws organizations list-accounts  --profile $AWS_PROFILE --query 'Accounts[*].[Id]' --output text 2&gt;&amp;1) || handle_error $? $LINENO
# Open a CSV file for writing
: &gt; root_user_last_login.csv  # Create an empty file
echo "AccountId,MFADevices,AccountAccessKeysPresent,AccountMFAEnabled,AccountPasswordPresent,PasswordLastUsedTime" &gt;&gt; root_user_last_login.csv
# Set the assume-root parameters\
REGION="us-east-1"
TASK_POLICY_ARN="arn=arn:aws:iam::aws:policy/root-task/IAMAuditRootUserCredentials"
# Iterate over each account
while IFS=',' read -r account_id account_name; do
    # Check if the account is excluded
    if [[ ",$EXCLUDED_ACCOUNTS," == *,"$account_id",* ]]; then
        echo "Skipping account $account_id ($account_name) as it is excluded."
        continue
    fi
    # Set the role ARN and session name for the current account
    SESSION_NAME="session_${account_id}"
    TARGET_PRINCIPAL="${account_id}"
    # Assume the role and capture the JSON response
    # Assume the role
    ASSUME_ROLE_OUTPUT=$(aws  sts assume-root \
        --profile "$AWS_PROFILE" \
        --region $REGION \
        --task-policy-arn "$TASK_POLICY_ARN" \
        --target-principal "$TARGET_PRINCIPAL" \
        --output json )
        
    # Extract the temporary credentials from the JSON response
    ACCESS_KEY_ID=$(echo "$ASSUME_ROLE_OUTPUT" | jq -r '.Credentials.AccessKeyId')
    SECRET_ACCESS_KEY=$(echo "$ASSUME_ROLE_OUTPUT" | jq -r '.Credentials.SecretAccessKey')
    SESSION_TOKEN=$(echo "$ASSUME_ROLE_OUTPUT" | jq -r '.Credentials.SessionToken')
    # Export the temporary credentials as environment variables
    export AWS_ACCESS_KEY_ID="$ACCESS_KEY_ID"
    export AWS_SECRET_ACCESS_KEY="$SECRET_ACCESS_KEY"
    export AWS_SESSION_TOKEN="$SESSION_TOKEN"
    
    # Fetch IAM account summary using get-account-summary
    iam_summary=$(aws iam get-account-summary --query 'SummaryMap')
    
    # Extract relevant information
    mfa_devices=$(echo "$iam_summary" | jq -r '.MFADevices // "No"')
    account_accesskeys_present=$(echo "$iam_summary" | jq -r '.AccountAccessKeysPresent // "No"')
                       
    # Extract MFA and password status for the root user
    mfa_enabled=$(echo "$iam_summary" | jq -r '.AccountMFAEnabled // "No"')
    password_present=$(echo "$iam_summary" | jq -r '.AccountPasswordPresent // "No"')

    # Get the root user's password last used information
    ROOT_PASSWORD_LAST_USED=$(aws iam get-user  --query User.PasswordLastUsed --output text 2&gt;&amp;1)  
    # Unset temporary credentials for security
    unset AWS_ACCESS_KEY_ID
    unset AWS_SECRET_ACCESS_KEY
    unset AWS_SESSION_TOKEN

    # Write the account information to the CSV file
    echo "$account_id,$mfa_devices,$account_accesskeys_present,$mfa_enabled,$password_present,$ROOT_PASSWORD_LAST_USED" &gt;&gt; root_user_last_login.csv
    sleep .1 # Waits 0.1 second.
done &lt;&lt;&lt; "$ACCOUNTS"
echo "Root user last login information has been written to root_user_last_login.csv"</code></pre> 
</div> 
<h3>Remove root credentials at scale</h3> 
<p>After you determine which AWS accounts have persistent root credentials that you want to remove, use the new action, <code style="color: #000000;">assumeRoot</code>, to access these accounts and remove the root credentials.</p> 
<p>Below is a script that will remove root login profiles across your entire organization. You can exclude certain accounts by updating the variable <code style="color: #000000;">EXCLUDED_ACCOUNTS</code>.</p> 
<p><strong>To use the script to remove root credentials</strong></p> 
<ol> 
 <li>Make sure that you have the AWS CLI installed and are signed in to your delegated admin account using admin role credentials with permissions to the <code style="color: #000000;">organizations:ListAccounts</code> and <code style="color: #000000;">sts:AssumeRoot</code> actions.</li> 
 <li>Save the code that follows to <code style="color: #000000;">DeleteRootCredentials.sh</code>.</li> 
 <li>The profile used in the script is <code style="color: #000000;">root-access-management</code>. You can modify the script to use another profile.</li> 
 <li>Run <code style="color: #000000;">./DeleteRootCredentials.sh</code> on your terminal.</li> 
 <li>The output will have a <code style="color: #000000;">.csv</code> file for the root accounts (except the ones in <code style="color: #000000;">EXCLUDED_ACCOUNTS</code>) with the status for root login profile deletion.</li> 
</ol> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-text">#/bin/bash

# Specify the account IDs to exclude (comma-separated)
EXCLUDED_ACCOUNTS="111122223333,444455556666"

# Specify the AWS profile to use
AWS_PROFILE="root-access-management"

# Set the role name and additional parameters
REGION="us-east-1"
TASK_POLICY_ARN="arn=arn:aws:iam::aws:policy/root-task/IAMDeleteRootUserCredentials"

# Function to handle errors
handle_error() {
    echo "Error on line $2: Command exited with status $1" &gt;&amp;2
    exit $1
}

# Get the list of accounts in the organization
ACCOUNTS=$(aws organizations list-accounts  --profile $AWS_PROFILE  --query 'Accounts[*].[Id]' --output text 2&gt;&amp;1) || handle_error $? $LINENO

# Open a CSV file for writing
: &gt; root_user_deletion.csv  # Create an empty file
echo "AccountId,AccountName,RootUserDeleted" &gt;&gt; root_user_deletion.csv

# Iterate over each account
while IFS=$'\t' read -r account_id ; do
    # Check if the account is excluded
    if [[ ",$EXCLUDED_ACCOUNTS," == *,"$account_id",* ]]; then
        echo "Skipping account $account_id ($account_name) as it is excluded."
        continue
    fi

    SESSION_NAME="session_${account_id}"
    TARGET_PRINCIPAL="${account_id}"
    
    # Assume the role
    assume_role=$(aws  sts assume-root \
        --profile "$AWS_PROFILE" \
        --region $REGION \
        --task-policy-arn "$TASK_POLICY_ARN" \
        --target-principal "$TARGET_PRINCIPAL" \
        --output json)

    
    
    # Extract temporary credentials from the assume role response
    export AWS_ACCESS_KEY_ID=$(echo $assume_role | jq -r '.Credentials.AccessKeyId')
    export AWS_SECRET_ACCESS_KEY=$(echo $assume_role | jq -r '.Credentials.SecretAccessKey')
    export AWS_SESSION_TOKEN=$(echo $assume_role | jq -r '.Credentials.SessionToken')

    # Attempt to delete the root user
    ROOT_USER_DELETED="false"
    if aws iam delete-login-profile  ; then
        ROOT_USER_DELETED="true"
    fi

     # Unset temporary credentials for security
    unset AWS_ACCESS_KEY_ID
    unset AWS_SECRET_ACCESS_KEY
    unset AWS_SESSION_TOKEN
    
    # Write the account information to the CSV file
    echo "$account_id,$account_name,$ROOT_USER_DELETED" &gt;&gt; root_user_deletion.csv

done &lt;&lt;&lt; "$ACCOUNTS"

echo "Root user deletion results have been written to root_user_deletion.csv"</code></pre> 
</div> 
<p>You can extend this script to delete root access keys by using the <a href="https://docs.aws.amazon.com/cli/latest/reference/iam/delete-access-key.html" rel="noopener" target="_blank">delete-access-key</a> command. To do so, you retrieve the list of access keys by using the <a href="https://docs.aws.amazon.com/cli/latest/reference/iam/list-access-keys.html" rel="noopener" target="_blank">list-access-keys</a> command, iterate through the list of access keys to determine which keys to delete, and pass the resulting access key IDs to <a href="https://docs.aws.amazon.com/cli/latest/reference/iam/delete-access-key.html" rel="noopener" target="_blank">delete-access-key</a> to delete the access keys.</p> 
<p>Similarly, you can extend the script to delete MFA devices by doing the following. Retrieve the list of MFA devices by using <a href="https://docs.aws.amazon.com/cli/latest/reference/iam/list-mfa-devices.html" rel="noopener" target="_blank">list-mfa-devices</a>, iterate through the list to determine which MFA devices to delete, and pass the resulting device serial numbers to <a href="https://docs.aws.amazon.com/cli/latest/reference/iam/deactivate-mfa-device.html" rel="noopener" target="_blank">deactivate-mfa-device</a> <u>and </u><a href="https://docs.aws.amazon.com/cli/latest/reference/iam/delete-virtual-mfa-device.html" rel="noopener" target="_blank">delete-virtual-mfa-device</a> to deactivate the MFA devices and further delete the virtual MFA devices.</p> 
<h2>Conclusion</h2> 
<p>In this post, we showed you how to enable and use the various features of centralized root access. Additionally, we covered best practices for using this new capability and discussed considerations for adoption.</p> 
<p>To learn more about centralized root access and root user best practices, review our <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/root-user-best-practices.html" rel="noopener" target="_blank">documentation</a>. If you have questions, reach out to <a href="https://console.aws.amazon.com/support/home" rel="noopener" target="_blank">AWS Support</a> or post a question at <a href="https://repost.aws/?nc1=f_dr" rel="noopener" target="_blank">re:Post</a>.</p> 
<footer> 
 <div class="blog-author-box">
  <img alt="Jonathan VanKim" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2022/05/13/vankimj.jpg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Jonathan VanKim</span>
  <br />Jonathan is a Principal Solutions Architect who specializes in security and identity for AWS. In 2014, he started working at AWS Professional Services and transitioned to solutions architecture four years later. His AWS career has been focused on helping customers of all sizes build secure AWS architectures. He enjoys snowboarding, wakesurfing, travelling, and experimental cooking.
 </div> 
 <div class="blog-author-box">
  <img alt="Sowjanya Rajavaram" class="alignleft size-full" src="https://d2908q01vomqb2.cloudfront.net/5b384ce32d8cdef02bc3a139d4cac0a22bb029e8/2023/10/17/sowjir-1.jpeg" style="margin-left: 12px; margin-right: 18px; margin-top: 12px; margin-bottom: 6px; width: 93.750px; height: 125px;" />
  <span class="lb-h4" style="line-height: 2.1em; padding-top: 12px; margin-top: 24px;">Sowjanya Rajavaram</span>
  <br />Sowjanya is a Senior Solutions Architect who specializes in security and identity for AWS. Her career has been focused on helping customers of all sizes solve their identity and access management problems. She enjoys traveling and experiencing new cultures and food.
 </div> 
</footer>
