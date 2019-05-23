# Amazon EKS Troubleshooting<a name="troubleshooting"></a>

This chapter covers some common errors that you may see while using Amazon EKS and how to work around them\.

## Insufficient Capacity<a name="ICE"></a>

If you receive the following error while attempting to create an Amazon EKS cluster, then one of the Availability Zones you specified does not have sufficient capacity to support a cluster\.

`Cannot create cluster 'example-cluster' because us-east-1d, the targeted availability zone, does not currently have sufficient capacity to support the cluster. Retry and choose from these availability zones: us-east-1a, us-east-1b, us-east-1c`

Retry creating your cluster with subnets in your cluster VPC that are hosted in the Availability Zones returned by this error message\.

## `aws-iam-authenticator` Not Found<a name="no-auth-provider"></a>

If you receive the error `"aws-iam-authenticator": executable file not found in $PATH`, then your kubectl is not configured for Amazon EKS\. For more information, see [Installing `aws-iam-authenticator`](install-aws-iam-authenticator.md)\.

## Worker Nodes Fail to Join Cluster<a name="worker-node-fail"></a>

There are two common reasons that prevent worker nodes from joining the cluster:
+ The `aws-auth-cm.yaml` file does not have the correct IAM role ARN for your worker nodes\. Ensure that the worker node IAM role ARN \(not the instance profile ARN\) is specified in your `aws-auth-cm.yaml` file\. For more information, see [Launching Amazon EKS Worker Nodes](launch-workers.md)\.
+ The **ClusterName** in your worker node AWS CloudFormation template does not exactly match the name of the cluster you want your worker nodes to join\. Passing an incorrect value to this field results in an incorrect configuration of the worker node's `/var/lib/kubelet/kubeconfig` file, and the nodes will not join the cluster\.

## Unauthorized or Access Denied \(`kubectl`\)<a name="unauthorized"></a>

If you receive one of the following errors while running kubectl commands, then your kubectl is not configured properly for Amazon EKS or the IAM user or role credentials that you are using do not map to a Kubernetes RBAC user with sufficient permissions in your Amazon EKS cluster\.
+ `could not get token: AccessDenied: Access denied`
+ `error: You must be logged in to the server (Unauthorized)`
+ `error: the server doesn't have a resource type "svc"`

This could be because the cluster was created with one set of AWS credentials \(from an IAM user or role\), and kubectl is using a different set of credentials\.

When an Amazon EKS cluster is created, the IAM entity \(user or role\) that creates the cluster is added to the Kubernetes RBAC authorization table as the administrator \(with `system:master` permissions\). Initially, only that IAM user can make calls to the Kubernetes API server using kubectl\. For more information, see [Managing Users or IAM Roles for your Cluster](add-user-role.md)\. Also, the [AWS IAM Authenticator for Kubernetes](https://github.com/kubernetes-sigs/aws-iam-authenticator) uses the AWS SDK for Go to authenticate against your Amazon EKS cluster\. If you use the console to create the cluster, you must ensure that the same IAM user credentials are in the AWS SDK credential chain when you are running kubectl commands on your cluster\.

If you install and configure the AWS CLI, you can configure the IAM credentials for your user\. If the AWS CLI is configured properly for your user, then the [AWS IAM Authenticator for Kubernetes](https://github.com/kubernetes-sigs/aws-iam-authenticator) can find those credentials as well\. For more information, see [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) in the *AWS Command Line Interface User Guide*\.

If you assumed a role to create the Amazon EKS cluster, you must ensure that kubectl is configured to assume the same role\. Use the following command to update your kubeconfig file to use an IAM role\. For more information, see [Create a `kubeconfig` for Amazon EKS](create-kubeconfig.md)\.

```
aws --region region eks update-kubeconfig --name cluster_name --role-arn arn:aws:iam::aws_account_id:role/role_name
```

To map an IAM user to a Kubernetes RBAC user, see [Managing Users or IAM Roles for your Cluster](add-user-role.md)\.

## AccessDeniedException<a name="iam-error"></a>

If you receive an `AccessDeniedException` when calling an AWS API operation, then the AWS Identity and Access Management \(IAM\) user or role credentials that you are using do not have the required permissions to make that call\. 

```
An error occurred (AccessDeniedException) when calling the DescribeCluster operation: 
User: arn:aws:iam::111122223333:user/user_name is not authorized to perform: 
eks:DescribeCluster on resource: arn:aws:eks:us-west-2:111122223333:cluster/cluster_name
```

In the above example message, the user does not have permissions to call the Amazon EKS `DescribeCluster` API operation\. To provide Amazon EKS admin permissions to a user, see [Creating Amazon EKS IAM Policies](EKS_IAM_user_policies.md)\.

For more general information about IAM, see [Controlling Access Using Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_controlling.html) in the *IAM User Guide*\.

## `hostname doesn't match`<a name="python-version"></a>

Your system's Python version must be 2\.7\.9 or greater\. Otherwise, you receive `hostname doesn't match` errors with AWS CLI calls to Amazon EKS\. For more information, see [What are "hostname doesn't match" errors?](http://docs.python-requests.org/en/master/community/faq/#what-are-hostname-doesn-t-match-errors) in the Python Requests FAQ\.

## `getsockopt: no route to host`<a name="troubleshoot-dicker-cidr"></a>

Docker runs in the `172.17.0.0/16` CIDR range in Amazon EKS clusters\. We recommend that your cluster's VPC subnets do not overlap this range\. Otherwise, you will receive the following error:

```
Error: : error upgrading connection: error dialing backend: dial tcp 172.17.nn.nn:10250: getsockopt: no route to host
```

## CNI Log Collection Tool<a name="troubleshoot-cni"></a>

The Amazon VPC CNI plugin for Kubernetes has its own troubleshooting script \(which is available on worker nodes at `/opt/cni/bin/aws-cni-support.sh`\) that you can use to collect diagnostic logs for support cases and general troubleshooting\.

The script collects the following diagnostic information:
+ L\-IPAMD introspection data
+ Metrics
+ Kubelet introspection data
+ `ifconfig` output
+ `ip rule show` output
+ `iptables-save` output
+ `iptables -nvL` output
+ `iptables -nvL -t nat` output
+ A dump of the CNI configuration
+ Kubelet logs
+ Stored `/var/log/messages`
+ Worker node's route table information \(via `ip route`\)
+ The `sysctls` output of `/proc/sys/net/ipv4/conf/{all,default,eth0}/rp_filter`

Use the following command to run the script on your worker node:

```
sudo bash /opt/cni/bin/aws-cni-support.sh
```

**Note**  
If the script is not present at that location, then the CNI container failed to run\. You can manually download and run the script with the following command:  

```
curl https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/master/scripts/aws-cni-support.sh | sudo bash
```

The diagnostic information is collected and stored at `/var/log/aws-routed-eni/aws-cni-support.tar.gz`\.
