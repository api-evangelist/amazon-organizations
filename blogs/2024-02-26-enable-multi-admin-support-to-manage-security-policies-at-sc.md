---
title: "Enable multi-admin support to manage security policies at scale with AWS Firewall Manager"
url: "https://aws.amazon.com/blogs/security/enable-multi-admin-support-to-manage-security-policies-at-scale-with-aws-firewall-manager/"
date: "Mon, 26 Feb 2024 21:23:24 +0000"
author: "Mun Hossain"
feed_url: "https://aws.amazon.com/blogs/security/tag/aws-organizations/feed/"
---
<p>The management of security services across organizations has evolved over the years, and can vary depending on the size of your organization, the type of industry, the number of services to be administered, and compliance regulations and legislation. When compliance standards require you to set up scoped administrative control of event monitoring and auditing, we find that single administrator support on management consoles can present several challenges for large enterprises. In this blog post, I’ll dive deep into these security policy management challenges and show how you can optimize your security operations at scale by using <a href="https://aws.amazon.com/firewall-manager/" rel="noopener" target="_blank">AWS Firewall Manager</a> to support multiple administrators.</p> 
<p>These are some of the use cases and challenges faced by large enterprise organizations when scaling their security operations:</p> 
<h3>Policy enforcement across complex organizational boundaries</h3> 
<p>Large organizations tend to be divided into multiple organizational units, each of which represents a function within the organization. Risk appetite, and therefore security policy, can vary dramatically between organizational units. For example, organizations may support two types of users: central administrators and app developers, both of whom can administer security policy but might do so at different levels of granularity. The central admin applies a baseline and relatively generic policy for all accounts, while the app developer can be made an admin for specific accounts and be allowed to create custom rules for the overall policy. A single administrator interface limits the ability for multiple administrators to enforce differing policies for the organizational unit to which they are assigned.</p> 
<h3>Lack of adequate separation across services</h3> 
<p>The benefit of centralized management is that you can enforce a centralized policy across multiple services that the management console supports. However, organizations might have different administrators for each service. For example, the team that manages the firewall could be different than the team that manages a web application firewall solution. Aggregating administrative access that is confined to a single administrator might not adequately conform to the way organizations have services mapped to administrators.</p> 
<h3>Auditing and compliance</h3> 
<p>Most security frameworks call for auditing procedures, to gain visibility into user access, types of modifications to configurations, timestamps of incremental changes, and logs for periods of downtime. An organization might want only specific administrators to have access to certain functions. For example, each administrator might have specific compliance scope boundaries based on their knowledge of a particular compliance standard, thereby distributing the responsibility for implementation of compliance measures. Single administrator access greatly reduces the ability to discern the actions of different administrators in that single account, making auditing unnecessarily complex.</p> 
<h3>Availability</h3> 
<p>Redundancy and resiliency are regarded as baseline requirements for security operations. Organizations want to ensure that if a primary administrator is locked out of a single account for any reason, other legitimate users are not affected in the same way. Single administrator access, in contrast, can lock out legitimate users from performing critical and time-sensitive actions on the management console.</p> 
<h3>Security risks</h3> 
<p>In a single administrator setting, the ability to enforce the policy of least privilege is not possible. This is because there are multiple operators who might share the same levels of access to the administrator account. This means that there are certain administrators who could be granted broader access than what is required for their function in the organization.</p> 
<h2>What is multi-admin support?</h2> 
<p>Multi-admin support for <a href="https://aws.amazon.com/firewall-manager/" rel="noopener" target="_blank">Firewall Manager</a> allows customers with multiple organizational units (OUs) and accounts to create up to 10 Firewall Manager administrator accounts from <a href="https://aws.amazon.com/organizations/" rel="noopener" target="_blank">AWS Organizations</a> to manage their firewall policies. You can delegate responsibility for firewall administration at a granular scope by restricting access based on OU, account, policy type, and AWS Region, thereby enabling policy management tasks to be implemented more effectively.</p> 
<p>Multi-admin support provides you the ability to use different administrator accounts to create administrative scopes for different parameters. Examples of these administrative scopes are included in the following table.</p> 
<table style="border-collapse: collapse; border: 1px solid #808080;" width="100%"> 
 <tbody>
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%"><strong>Administrator</strong></td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%"><strong>Scope</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Default Administrator</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Full Scope (Default)</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Administrator 1</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">OU = “Test 1”</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Administrator 2</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Account IDs = “123456789, 987654321”</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Administrator 3</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Policy-Type = “Security Group”</td> 
  </tr> 
  <tr> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Administrator 4</td> 
   <td style="padding: 8px; border: 1px solid #808080;" width="50%">Region = “us-east-2”</td> 
  </tr> 
 </tbody>
</table> 
<h2>Benefits of multi-admin support</h2> 
<p>Multi-admin support helps alleviate many of the challenges just discussed by allowing administrators the flexibility to implement custom configurations based on job functions, while enforcing the principle of least privilege to help ensure that corporate policy and compliance requirements are followed. The following are some of the key benefits of multi-admin support:</p> 
<h3>Improved security</h3> 
<p>Security is enhanced, given that the principle of least privilege can be enforced in a multi-administrator access environment. This is because the different administrators using Firewall Manager will be using delegated privileges that are appropriate for the level of access they are permitted. The result is that the scope for user errors, intentional errors, and unauthorized changes can be significantly reduced. Additionally, you attain an added level of accountability for administrators.</p> 
<h3>Autonomy of job functions</h3> 
<p>Companies with organizational units that have separate administrators are afforded greater levels of autonomy within their AWS Organizations accounts. The result is an increase in flexibility, where concurrent users can perform very different security functions.</p> 
<h3>Compliance benefits</h3> 
<p>It is easier to meet auditing requirements based on compliance standards in multi-admin accounts, because there is a greater level of visibility into user access and the functions performed on the services when compared to a multi-eyes approval workflow and approval of all policies by one omnipotent admin. This can simplify routine audits through the generation of reports that detail the chronology of security changes that are implemented by specific admins over time.</p> 
<h3>Administrator Availability</h3> 
<p>Multi-admin management support helps avoid the limitations of having a single point of access and enhances availability by providing multiple administrators with their own levels of access. This can result in fewer disruptions, especially during periods that require time-sensitive changes to be made to security configurations.</p> 
<h3>Integration with AWS Organizations</h3> 
<p>You can enable trusted access using either the Firewall Manager console or the AWS Organizations console. To do this, you sign in with your AWS Organizations management account and configure an account allocated for security tooling within the organization as the Firewall Manager administrator account.&nbsp;After this is done, subsequent multi-admin Firewall Manager operations can also be performed using AWS APIs. With accounts in an organization, you can quickly allocate resources, group multiple accounts, and apply governance policies to accounts or groups.&nbsp;This simplifies operational overhead for services that require cross-account management.</p> 
<h2>Key use cases</h2> 
<p>Multi-admin support in Firewall Manager unlocks several use cases pertaining to admin role-based access. The key use cases are summarized here.</p> 
<h3>Role-based access</h3> 
<p>Multi-admin support allows for different admin roles to be defined based on the job function of the administrator, relative to the service being managed. For example, an administrator could be tasked to manage network firewalls to protect their VPCs, and a different administrator could be tasked to manage web application firewalls (<a href="https://aws.amazon.com/waf/" rel="noopener" target="_blank">AWS WAF</a>), both using Firewall Manager.</p> 
<h3>User tracking and accountability</h3> 
<p>In a multi-admin configuration environment, each Firewall Manager administrator’s activities are logged and recorded according to corporate compliance standards. This is useful when dealing with the troubleshooting of security incidents, and for compliance with auditing standards.</p> 
<h3>Compliance with security frameworks</h3> 
<p>Regulations specific to a particular industry, such as Payment Card Industry (PCI), and industry-specific legislation, such as HIPAA, require restricted access, control, and separation of tasks for different job functions. Failure to adhere to such standards could result in penalties. With administrative scope extending to policy types, customers can assign responsibility for managing particular firewall policies according to user role guidelines, as specified in compliance frameworks.</p> 
<h3>Region-based privileges</h3> 
<p>Many state or federal frameworks, such as the California Consumer Privacy Act (CCPA), require that admins adhere to customized regional requirements, such as data sovereignty or privacy requirements. Multi-admin Firewall Manager support helps organizations to adopt these frameworks by making it easier to assign admins who are familiar with the regulations of a particular region to that region.</p> 
<div class="wp-caption aligncenter" id="attachment_33466" style="width: 790px;">
 <img alt="Figure 1: Use cases for multi-admin support on AWS Firewall Manager" class="size-full wp-image-33466" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/02/21/img1-5.png" style="border: 1px solid #bebebe;" width="780" />
 <p class="wp-caption-text" id="caption-attachment-33466">Figure 1: Use cases for multi-admin support on AWS Firewall Manager</p>
</div> 
<h2>How to implement multi-admin support with Firewall Manager</h2> 
<p>To configure multi-admin support on Firewall Manager, use the following steps:</p> 
<ol> 
 <li>In the AWS Organizations console of the organization’s managed account, expand the <strong>Root</strong> folder to view the various accounts in the organization. Select the <a href="https://docs.aws.amazon.com/waf/latest/developerguide/enable-integration.htm" rel="noopener" target="_blank">Default Administrator account</a> that is allocated to delegate Firewall Manager administrators. The Default Administrator account should be a dedicated security account separate from the AWS Organizations management account, such as a Security Tooling account.<br />&nbsp; 
  <div class="wp-caption aligncenter" id="attachment_33467" style="width: 750px;">
   <img alt="Figure 2: Overview of the AWS Organizations console" class="size-full wp-image-33467" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/02/21/img2-2.png" style="border: 1px solid #bebebe;" width="749" />
   <p class="wp-caption-text" id="caption-attachment-33467">Figure 2: Overview of the AWS Organizations console</p>
  </div> </li> 
 <li>Navigate to Firewall Manager and in the left navigation menu, select <strong>Settings</strong>.<br />&nbsp; 
  <div class="wp-caption aligncenter" id="attachment_33468" style="width: 750px;">
   <img alt="Figure 3: AWS Firewall Manager settings to update policy types" class="size-full wp-image-33468" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/02/21/img3-1.jpg" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-33468">Figure 3: AWS Firewall Manager settings to update policy types</p>
  </div> </li> 
 <li>In <strong>Settings</strong>, choose an account. Under <strong>Policy types</strong>, select <strong>AWS Network Firewall</strong> to allow an admin to manage a specific firewall across accounts and across Regions.&nbsp;Select <strong>Edit</strong> to show the <strong>Details </strong>menu in Figure 4.<br />&nbsp; 
  <div class="wp-caption aligncenter" id="attachment_33469" style="width: 690px;">
   <img alt="Figure 4: Select AWS Network Firewall as a policy type that can be managed by this administration account" class="size-full wp-image-33469" height="822" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/02/21/img4-3.png" style="border: 1px solid #bebebe;" width="680" />
   <p class="wp-caption-text" id="caption-attachment-33469">Figure 4: Select AWS Network Firewall as a policy type that can be managed by this administration account</p>
  </div> <p>The results of your selection are shown in Figure 5. The admin has been granted privileges to set AWS Network Firewall policy across all Regions and all accounts.<br />&nbsp;</p> 
  <div class="wp-caption aligncenter" id="attachment_33470" style="width: 750px;">
   <img alt="Figure 5: The admin has been granted privileges to set Network Firewall policy across all Regions and all accounts" class="size-full wp-image-33470" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/02/21/img5-2.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-33470">Figure 5: The admin has been granted privileges to set Network Firewall policy across all Regions and all accounts</p>
  </div> </li> 
 <li>In this second use case, you will identify a number of sub-accounts that the admin should be restricted to.&nbsp;As shown in Figure 6, there are no sub-accounts or OUs that the admin is restricted to by default until you choose <strong>Edit</strong> and select them.<br />&nbsp; 
  <div class="wp-caption aligncenter" id="attachment_33471" style="width: 750px;">
   <img alt="Figure 6: The administrative scope details for the admin" class="size-full wp-image-33471" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/02/21/img6-2.png" style="border: 1px solid #bebebe;" width="740" />
   <p class="wp-caption-text" id="caption-attachment-33471">Figure 6: The administrative scope details for the admin</p>
  </div> <p>In order to achieve this second use case, you choose <strong>Edit</strong>, and then add multiple sub-accounts or an OU that you need the admin restricted to, as shown in Figure 7.<br />&nbsp;</p> 
  <div class="wp-caption aligncenter" id="attachment_33472" style="width: 658px;">
   <img alt="Figure 7: Add multiple sub-accounts or an OU that you need the admin restricted to" class="size-full wp-image-33472" height="814" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/02/21/img7-1.png" style="border: 1px solid #bebebe;" width="648" />
   <p class="wp-caption-text" id="caption-attachment-33472">Figure 7: Add multiple sub-accounts or an OU that you need the admin restricted to</p>
  </div> </li> 
 <li>The third use case pertains to granting the admin privileges to a particular AWS Region. In this case, you go into the <strong>Edit administrator account </strong>page once more, but this time, for <strong>Regions</strong>, select <strong>US West (N California)</strong> in order to restrict admin privileges only to this selected Region.<br />&nbsp; 
  <div class="wp-caption aligncenter" id="attachment_33473" style="width: 492px;">
   <img alt="Figure 8: Restricting admin privileges only to the US West (N California) Region" class="size-full wp-image-33473" height="678" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/02/21/img8-1.png" style="border: 1px solid #bebebe;" width="482" />
   <p class="wp-caption-text" id="caption-attachment-33473">Figure 8: Restricting admin privileges only to the US West (N California) Region</p>
  </div> </li> 
</ol> 
<h2>Conclusion</h2> 
<p>Large enterprises need strategies for operationalizing security policy management so that they can enforce policy across organizational boundaries, deal with policy changes across security services, and adhere to auditing and compliance requirements. Multi-admin support in Firewall Manager provides a framework that admins can use to organize their workflow across job roles, to help maintain appropriate levels of security while providing the autonomy that admins desire.</p> 
<p>You can get started using the multi-admin feature with Firewall Manager using the AWS Management Console. To learn more, refer to the service&nbsp;<a href="https://docs.aws.amazon.com/waf/latest/developerguide/fms-administrators.html" rel="noopener" target="_blank">documentation</a>.</p> 
<p>&nbsp;<br />If you have feedback about this post, submit comments in the<strong> Comments</strong> section below. If you have questions about this post, <a href="https://console.aws.amazon.com/support/home" rel="noopener noreferrer" target="_blank">contact AWS Support</a>.</p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image"> 
   <img alt="Mun Hossain" class="aligncenter size-full wp-image-33465" height="120" src="https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2024/02/21/munahoss.jpg" width="90" /> 
  </div> 
  <h3 class="lb-h4">Mun Hossain</h3> 
  <p>Mun is a Principal Security Service Specialist at AWS. Mun sets go-to-market strategies and prioritizes customer signals that contribute to service roadmap direction. Before joining AWS, Mun was a Senior Director of Product Management for a wide range of cybersecurity products at Cisco. Mun holds an MS in Cybersecurity Risk and Strategy and an MBA.</p> 
  <p></p>
 </div> 
</footer>
