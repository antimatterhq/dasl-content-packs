# AWS CloudTrail IAM Events

This preset processes gzipped CloudTrail JSON log files, extracting only IAM-related events while excluding all others. The filtered data is then transformed into structured silver tables, and ultimately into OCSF compliant gold tables. See the sections below for details on how events are categorized and mapped into the silver and gold layers.

## Unity Catalog Targets

### Silver

The following silver tables store normalized IAM-related CloudTrail data. Each table corresponds to a specific OCSF class and serves as an intermediate processing layer before populating the gold tables. These tables allow for targeted querying and analysis of IAM operations, broken down by category and use case.


| Table Name                      | Description                                                                 |
|--------------------------------|-----------------------------------------------------------------------------|
| aws_cloudtrail_account_change  | Parsed and normalized CloudTrail events related to IAM account lifecycle actions, such as user creation, deletion, and credential updates. |
| aws_cloudtrail_entity_management | Parsed and normalized CloudTrail events focused on reading, updating, etc., IAM entities. |
| aws_cloudtrail_user_access     | Parsed and normalized CloudTrail events related to the management of user permissions/access. |
| aws_cloudtrail_group_management| Parsed and normalized CloudTrail events for group membership operations, including adding or removing users and managing groups. |
| aws_cloudtrail_api_activity    | Parsed and normalized CloudTrail events related to IAM CRUD and other actions. |



### Gold

The following gold tables contain data that conforms to the [OCSF (Open Cybersecurity Schema Framework)](https://ocsf.io) specification. Each table maps to a specific OCSF class and provides high-quality, schema-aligned datasets for downstream analytics, detection engineering, and threat hunting.

| Table Name              | OCSF Class Name        | OCSF Class UID | Description                                                                 |
|-------------------------|------------------------|----------------|-----------------------------------------------------------------------------|
| account_change          | Account Change         | 3001           | Events related to modifications of user accounts, such as creation, deletion, or updates to account details. |
| entity_management       | Entity Management      | 3004           | Events involving the management of entities, such as users, groups, and roles, including their creation and deletion. |
| user_access_management  | User Access Management | 3005           | Events related to the management of user access, including permission assignments and policy changes. |
| group_management        | Group Management       | 3006           | Events concerning the management of groups, including group creation, deletion, and membership changes. |
| api_activity            | API Activity           | 6003           | Events related to general CRUD (Create, Read, Update, Delete) and other API activities. |


## Event Type Descriptions

### Account Change (3001)

Changes to IAM identities, credentials, or account properties. Canonical activities include: `Create`, `Enable`, `Password Change`, `Password Reset`, `Disable`, `Delete`, `Attach Policy`, `Detach Policy`, `Lock`, `MFA Factor Enable`, `MFA Factor Disable` and `Unlock`. See the [OCSF documentation](https://schema.ocsf.io/1.4.0/classes/account_change?extensions=) for a more detailed description of these values.

| IAM Event Type                  | Comment                                           | OCSF Activity Name       |
|---------------------------------|---------------------------------------------------|--------------------------|
| ChangePassword                  | User-initiated password change                    | Password Change          |
| CreateLoginProfile              | Creates a password for an IAM user                | Create                   |
| DeleteLoginProfile              | Removes IAM user password profile                 | Delete                   |
| UpdateLoginProfile              | Changes IAM user password or settings             | Password Change          |
| CreateUser                      | Adds a new IAM user                               | Create                   |
| DeleteUser                      | Removes an IAM user                               | Delete                   |
| UpdateUser                      | Modifies user name or path                        | Other                    |
| CreateVirtualMFADevice          | Sets up a virtual MFA device                      | Other                    |
| DeactivateMFADevice             | Disables MFA for a user                           | MFA Factor Disable       |
| DeleteVirtualMFADevice          | Deletes a virtual MFA device                      | MFA Factor Disable       |
| EnableMFADevice                 | Enables MFA on an IAM user                        | MFA Factor Enable        |
| ResyncMFADevice                 | Synchronizes MFA token devices                    | Other                    |
| UpdateSSHPublicKey              | Modifies uploaded SSH key metadata                | Other                    |
| UploadSSHPublicKey              | Adds an SSH public key to IAM user                | Other                    |
| DeleteSSHPublicKey              | Removes SSH key from IAM user                     | Other                    |
| UploadSigningCertificate        | Adds an X.509 cert for IAM user                   | Other                    |
| DeleteSigningCertificate        | Removes X.509 cert from IAM user                  | Other                    |
| UpdateSigningCertificate        | Updates the status of a signing cert              | Other                    |
| CreateServiceSpecificCredential | Creates credentials for a specific AWS service    | Other                    |
| DeleteServiceSpecificCredential | Deletes service-specific credentials              | Other                    |
| ResetServiceSpecificCredential  | Resets password for a service-specific credential | Password Reset           |
| UpdateServiceSpecificCredential | Changes status for a service-specific credential  | Other                    |
| CreateAccessKey                 | Generates a new IAM access key                    | Other                    |
| DeleteAccessKey                 | Deletes an IAM access key                         | Other                    |
| UpdateAccessKey                 | Enables/disables an access key                    | Other                    |
| CreateAccountAlias              | Creates a friendly alias for the AWS account      | Other                    |
| DeleteAccountAlias              | Removes an account alias                          | Other                    |
| UpdateAccountPasswordPolicy     | Sets account-wide password rules                  | Other                    |
| DeleteAccountPasswordPolicy     | Removes account-wide password rules               | Other                    |
| AttachRolePolicy                | Binds managed policy to role                      | Attach Policy            |
| DetachRolePolicy                | Unbinds managed policy from role                  | Detach Policy            |
| PutRolePolicy                   | Creates inline policy for role                    | Attach Policy            |
| DeleteRolePolicy                | Removes inline role policy                        | Detach Policy            |
| PutRolePermissionsBoundary      | Adds a role permissions boundary                  | Attach Policy            |
| DeleteRolePermissionsBoundary   | Removes role boundary                             | Detach Policy            |
| TagUser                         | Adds tags to user                                 | Other                    |
| UntagUser                       | Removes tags from user                            | Other                    |
| TagRole                         | Adds tags to role                                 | Other                    |
| UntagRole                       | Removes tags from role                            | Other                    |


### Entity Management (3004)

Operations or reporting about IAM entities. Canonical activities include: `Create`, `Read`, `Update`, `Delete`, `Move`, `Enroll`, `Unenroll`, `Enable`, `Disable`, `Activate`, `Deactivate`, `Suspend`, `Resume`.

| IAM Event Type                                | Comment                                             | OCSF Activity Name |
|-----------------------------------------------|-----------------------------------------------------|--------------------|
| GetGroup                                      | Returns group details                               | Read               |
| GetGroupPolicy                                | Gets group inline policy                            | Read               |
| GetUser                                       | Returns IAM user details                            | Read               |
| GetUserPolicy                                 | Gets user inline policy                             | Read               |
| GetRole                                       | Returns IAM role details                            | Read               |
| GetRolePolicy                                 | Gets role inline policy                             | Read               |
| GetPolicy                                     | Returns managed policy metadata                     | Read               |
| GetPolicyVersion                              | Gets a specific version of a managed policy         | Read               |
| GetInstanceProfile                            | Returns IAM instance profile details                | Read               |
| GetLoginProfile                               | Returns login profile for IAM user                  | Read               |
| GetOpenIDConnectProvider                      | Returns OIDC provider metadata                      | Read               |
| GetSAMLProvider                               | Returns SAML provider metadata                      | Read               |
| GetServerCertificate                          | Returns SSL certificate metadata                    | Read               |
| ListUserTags                                  | Lists tags attached to a user                       | Read               |
| ListGroupsForUser                             | Groups a user belongs to                            | Read               |
| ListRoleTags                                  | Lists tags on a role                                | Read               |
| ListInstanceProfilesForRole                   | Profiles associated with a role                     | Read               |
| ListInstanceProfileTags                       | Tags on an instance profile                         | Read               |
| ListSAMLProviderTags                          | Tags for a SAML provider                            | Read               |
| ListOpenIDConnectProviderTags                 | Tags for OIDC provider                              | Read               |
| ListServerCertificateTags                     | Tags for certs                                      | Read               |
| ListMFADevices                                | Lists MFA devices                                   | Read               |
| ListMFADeviceTags                             | MFA tags                                            | Read               |
| ListSigningCertificates                       | X.509 signing certs                                 | Read               |
| ListSSHPublicKeys                             | SSH keys                                            | Read               |
| ListServiceSpecificCredentials                | Credentials scoped to AWS services                  | Read               |
| ListPolicyVersions                            | Versions of a managed policy                        | Read               |
| ListPolicyTags                                | Tags on policies                                    | Read               |
| ListEntitiesForPolicy                         | Lists entities attached to a policy                 | Read               |
| ListAttachedUserPolicies                      | Managed policies attached to a user                 | Read               |
| ListAttachedGroupPolicies                     | Policies attached to group                          | Read               |
| ListAttachedRolePolicies                      | Policies attached to role                           | Read               |
| GetMFADevice                                  | Retrieves details about an MFA device               | Read               |
| ListAccessKeys                                | Lists the access keys for an IAM user               | Read               |
| ListGroupPolicies                             | Lists inline policies of a group                    | Read               |
| ListOrganizationsFeatures                     | Lists enabled org features                          | Read               |
| ListRolePolicies                              | Lists inline policies attached to a role            | Read               |
| ListUserPolicies                              | Lists inline policies attached to a user            | Read               |
| ListVirtualMFADevices                         | Lists virtual MFA devices in the account            | Read               |
| TagMFADevice                                  | Tags MFA device                                     | Update             |
| UntagMFADevice                                | Removes MFA tags                                    | Update             |
| TagOpenIDConnectProvider                      | Tags OIDC provider                                  | Update             |
| UntagOpenIDConnectProvider                    | Untag OIDC provider                                 | Update             |
| TagSAMLProvider                               | Tags SAML provider                                  | Update             |
| UntagSAMLProvider                             | Untag SAML provider                                 | Update             |
| TagServerCertificate                          | Tags server certificate                             | Update             |
| UntagServerCertificate                        | Untag server certificate                            | Update             |
| SetDefaultPolicyVersion                       | Changes default policy version                      | Update             |
| TagPolicy                                     | Adds tags to policy                                 | Update             |
| UntagPolicy                                   | Removes tags from policy                            | Update             |
| TagInstanceProfile                            | Adds tags to instance profile                       | Update             |
| UntagInstanceProfile                          | Removes instance profile tags                       | Update             |
| AddClientIDToOpenIDConnectProvider            | Adds a client ID to an existing OIDC provider       | Update             |
| AddRoleToInstanceProfile                      | Adds a role to an instance profile                  | Update             |
| RemoveClientIDFromOpenIDConnectProvider       | Removes a client ID from an OIDC provider           | Update             |
| RemoveRoleFromInstanceProfile                 | Removes a role from an instance profile             | Update             |
| UpdateAssumeRolePolicy                        | Updates the trust relationship for a role           | Update             |
| UpdateOpenIDConnectProviderThumbprint         | Updates the thumbprint list for an OIDC provider    | Update             |
| UpdateRole                                    | Modifies an existing IAM role                       | Update             |
| UpdateRoleDescription                         | Changes the description of a role                   | Update             |
| UpdateSAMLProvider                            | Modifies the metadata document for a SAML provider  | Update             |
| UpdateServerCertificate                       | Updates the name or path of a server cert           | Update             |
| CreateInstanceProfile                         | Creates a new instance profile                      | Create             |
| CreateOpenIDConnectProvider                   | Creates a new OIDC identity provider                | Create             |
| CreatePolicy                                  | Creates a managed IAM policy                        | Create             |
| CreatePolicyVersion                           | Creates a new version of an existing managed policy | Create             |
| CreateRole                                    | Creates a new IAM role                              | Create             |
| CreateSAMLProvider                            | Creates a new SAML identity provider                | Create             |
| CreateServiceLinkedRole                       | Creates a service-linked IAM role                   | Create             |
| UploadServerCertificate                       | Uploads a new server certificate                    | Create             |
| DeleteInstanceProfile                         | Deletes an existing instance profile                | Delete             |
| DeleteOpenIDConnectProvider                   | Deletes an existing OIDC identity provider          | Delete             |
| DeletePolicy                                  | Deletes a managed IAM policy                        | Delete             |
| DeletePolicyVersion                           | Deletes a specific version of a managed policy      | Delete             |
| DeleteRole                                    | Deletes an existing IAM role                        | Delete             |
| DeleteSAMLProvider                            | Deletes a SAML identity provider                    | Delete             |
| DeleteServerCertificate                       | Deletes an uploaded server certificate              | Delete             |
| DeleteServiceLinkedRole                       | Deletes a service-linked role                       | Delete             |
| DisableOrganizationsRootCredentialsManagement | Disables root credential management for the org     | Disable            |
| DisableOrganizationsRootSessions              | Disables sessions for the org root                  | Disable            |
| EnableOrganizationsRootCredentialsManagement  | Enables root credential management for the org      | Enable             |
| EnableOrganizationsRootSessions               | Enables sessions for the org root                   | Enable             |



### User Access Management (3005)

Managing permissions and policies attached to IAM users, roles. Canonical activities include: `Assign Privileges`, `Revoke Privileges`.

| IAM Event Type                | Comment                                       | OCSF Activity Name   |
|-------------------------------|-----------------------------------------------|----------------------|
| AttachUserPolicy              | Binds a managed policy to a user              | Assign Privileges    |
| DetachUserPolicy              | Unbinds policy from a user                    | Revoke Privileges    |
| PutUserPolicy                 | Creates or updates inline user policy         | Assign Privileges    |
| DeleteUserPolicy              | Deletes an inline user policy                 | Revoke Privileges    |
| PutUserPermissionsBoundary    | Sets permissions boundary for user            | Assign Privileges    |
| DeleteUserPermissionsBoundary | Removes user's permissions boundary           | Revoke Privileges    |


### Group Management (3006)

Creation and modification of IAM groups, their memberships and permissions. Canonical activities include: `Assign Privileges`, `Revoke Privileges`, `Add User`, `Remove User`, `Delete`, and `Create`.

| IAM Event Type      | Comment                               | OCSF Activity Name   |
|---------------------|---------------------------------------|----------------------|
| CreateGroup         | Creates a new IAM group               | Create               |
| DeleteGroup         | Deletes an IAM group                  | Delete               |
| UpdateGroup         | Renames or moves a group              | Other                |
| AddUserToGroup      | Adds user to a group                  | Add User             |
| RemoveUserFromGroup | Removes user from a group             | Remove User          |
| AttachGroupPolicy   | Attaches managed policy to group      | Assign Privileges    |
| DetachGroupPolicy   | Detaches managed policy from group    | Revoke Privileges    |
| PutGroupPolicy      | Creates inline group policy           | Assign Privileges    |
| DeleteGroupPolicy   | Deletes inline group policy           | Revoke Privileges    |


### API Activity (6003)

CRUD type actions typically performed through programmatic interfaces, such as AWS SDKs or CLI.  Canonical activities include: `Create`, `Read`, `Update` and `Delete`.

| IAM Event Type                            | Comment                                         | OCSF Activity Name |
|-------------------------------------------|-------------------------------------------------|--------------------|
| GenerateCredentialReport                  | Creates CSV report of IAM credentials           | Create             |
| GenerateOrganizationsAccessReport         | Creates report for org-level access analyzer    | Create             |
| GenerateServiceLastAccessedDetails        | Starts analysis of service usage by entity      | Create             |
| GetCredentialReport                       | Retrieves most recent credential report         | Read               |
| GetOrganizationsAccessReport              | Fetches an org access analyzer report           | Read               |
| GetServiceLastAccessedDetails             | Gets results of last access analysis            | Read               |
| GetServiceLastAccessedDetailsWithEntities | Breaks down access by entity                    | Read               |
| GetServiceLinkedRoleDeletionStatus        | Checks SL role deletion readiness               | Read               |
| GetAccountAuthorizationDetails            | Returns permission summaries by entity          | Read               |
| GetAccountSummary                         | Overview of IAM resources in account            | Read               |
| GetAccountPasswordPolicy                  | Returns current password policy                 | Read               |
| GetAccessKeyLastUsed                      | Shows when access key was last used             | Read               |
| GetSSHPublicKey                           | Returns SSH key details                         | Read               |
| GetContextKeysForCustomPolicy             | Gets context keys referenced in a custom policy | Read               |
| GetContextKeysForPrincipalPolicy          | Gets context keys from a principal's policies   | Read               |
| ListUsers                                 | Lists all IAM users                             | Read               |
| ListGroups                                | Lists all IAM groups                            | Read               |
| ListRoles                                 | Lists all IAM roles                             | Read               |
| ListInstanceProfiles                      | Lists instance profiles                         | Read               |
| ListSAMLProviders                         | Lists SAML identity providers                   | Read               |
| ListOpenIDConnectProviders                | OIDC identity providers                         | Read               |
| ListServerCertificates                    | Lists uploaded SSL certs                        | Read               |
| ListPolicies                              | Lists all policies                              | Read               |
| ListPoliciesGrantingServiceAccess         | Shows what grants service access to user        | Read               |
| ListAccountAliases                        | List friendly names for account                 | Read               |
| SetSecurityTokenServicePreferences        | Defines global STS session duration prefs       | Update             |
| SimulateCustomPolicy                      | Simulates policy evaluation for access test     | Other              |
| SimulatePrincipalPolicy                   | Simulates a principal's effective permissions   | Other              |


## Other Resources

https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-record-contents.html
A complete reference/data dictionary for the CloudTrail events. Note that the reference does not have information for service specific schemas nested within the CloudTrail record.

https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-events.html#cloudtrail-management-events
A short guide explaining the different CloudTrail event types (i.e., management, data, insight, etc.)


https://github.com/aws/aws-sdk-ruby/blob/version-3/apis/iam/2010-05-08/api-2.json
A collection of IAM API event definitions maintained by Amazon in JSON format.