# Using Terraform to build ROSA cluster with Private Link enabled, STS and identity provider configuration

Red Hat Openshift Service on AWS (ROSA) is a fully-managed turnkey application platform. A ROSA cluster can be created without any requirements on public subnets, internet gateways, or network address translation (NAT) gateways. In this configuration, Red Hat uses AWS PrivateLink to manage and monitor a cluster to avoid all public ingress network traffic.

To deploy Red Hat OpenShift Service on AWS (ROSA) into your existing Amazon Web Services (AWS) account, Red Hat requires several prerequisites to be met. There are several [requirements](https://docs.openshift.com/rosa/rosa_planning/rosa-sts-aws-prereqs.html#rosa-sts-aws-prereqs), [Review AWS service quota](https://docs.openshift.com/rosa/rosa_planning/rosa-sts-required-aws-service-quotas.html#rosa-sts-required-aws-service-quotasr) and [enable ROSA in your AWS account Access](https://docs.openshift.com/rosa/rosa_planning/rosa-sts-setting-up-environment.html#rosa-sts-setting-up-environment).


NOTE: STS(secure token service) allows us to deploy ROSA without needing a ROSA admin account, instead it uses roles and policies with Amazon STS to gain access to the AWS resources needed to install and operate the cluster.

In this series, we use Terraform to provision all resources in AWS to deploy a ROSA cluster with Privatelink, STS and IdP configiration.  

## Create the AWS Virtual Private Cloud (VPCs), Pub/Private Subnets, TGW and configure IdP

This install.sh script provisions 2 VPCs(VPC for ROSA cluster and egress VPC), 3 subnets, a bastion, IGW, NGW, and a forward proxy to control the cluster's egress traffic. Meanwhile, the script configures the OKTA account by creating an application, authorization server, groups, and all necessary components for OIDC configuration in OpenShift. This script adds three users to the OpenShift cluster as cluster_admin,  dedicated_admin, and restricted_user, which should be configured in the [oidc](./oidc/oidc.tf) file



![architecture diagram showing privatelink with TGW](./images/ROSA_PrivateLink_TGW_Proxy.png)

## Setup

Using the code in the repo will require having the following tools installed:

- The Terraform CLI
- The AWS CLI
- The ROSA CLI
- The OC CLI
- [sshuttle](https://github.com/sshuttle/sshuttle)

## Create OKTA account
[Create an Okta developer account](https://developer.okta.com/signup/)

[Create the API token](https://developer.okta.com/docs/guides/create-an-api-token/main/)
  
    1. Sign in to your Okta organization as a user with administrator privileges . ...
    2. Access the API page: In the Admin Console, select API from the Security menu and then select the Tokens tab.
    3. Click Create Token.
    4. Name your token and click Create Token.
    5. Record the token value.

update terraform variables in [oidc](./oidc/oidc.tf).

**NOTE:** be sure to update at least api_token, org_name, base_url, okta_admin_email, OpenShift cluster-admin, dedicated-admin and restricted user's emails.  
## Deploy cluster

update terraform variable in [rosa](./rosa/rosa_sts_prvlnk.tf) and [oidc](./oidc/oidc.tf)

   ```
   install.sh
   ```

 Check that you can access the Console by opening the console url in your browser.

   ```
   rosa describe cluster -c <clustername>
   ```
**NOTE 1:** if you get disconnected from the cluster you can use connect.sh to reconnect.

**NOTE 2:** if you need to connect to jumphost:
   ```
   1. eval $(ssh-agent)
   2. ssh-add
   3. ssh -A ec2-user@<bastion host public ip>
   3. ssh ec2-user@<jump host ip address>
   ```
## Cleanup

   Delete cluster and OKTA configuration.
   ```
   uninstall.sh
   ```

