# Amazon EKS Kubernetes Versions<a name="kubernetes-versions"></a>

The Kubernetes project is rapidly evolving with new features, design updates, and bug fixes\. The community releases new Kubernetes minor versions, such as 1\.12, as generally available approximately every three months, and each minor version is supported for approximately one year after it is first released\. 

## Available Amazon EKS Kubernetes Versions<a name="available-versions"></a>

The following Kubernetes versions are currently available for new clusters in Amazon EKS:
+ 1\.12\.6
+ 1\.11\.8
+ 1\.10\.13

**Important**  
Amazon EKS will deprecate Kubernetes version 1\.10 on July 22, 2019\. On this day, you will no longer be able to create new 1\.10 clusters and all Amazon EKS clusters running Kubernetes version 1\.10 will be updated to the latest available platform version of Kubernetes version 1\.11\. For more information, see [Amazon EKS Version Deprecation](#version-deprecation)\.

Unless your application requires a specific version of Kubernetes, we recommend that you choose the latest available Kubernetes version supported by Amazon EKS for your clusters\. As new Kubernetes versions become available in Amazon EKS, we recommend that you proactively update your clusters to use the latest available version\. For more information, see [Updating an Amazon EKS Cluster Kubernetes Version](update-cluster.md)\.

## Amazon EKS Version Deprecation<a name="version-deprecation"></a>

In line with the Kubernetes community support for Kubernetes versions, Amazon EKS is committed to running at least three production\-ready versions of Kubernetes at any given time, with a fourth version in deprecation\. 

We will announce the deprecation of a given Kubernetes minor version at least 60 days before the deprecation date\. Because of the Amazon EKS qualification and release process for new Kubernetes versions, the deprecation of a Kubernetes version on Amazon EKS will be on or after the date the Kubernetes project stops supporting the version upstream\.

On the deprecation date, Amazon EKS clusters running the version targeted for deprecation will begin to be updated to the next Amazon EKS\-supported version of Kubernetes\. This means that if the deprecated version is 1\.10, clusters will be automatically updated to version 1\.11\. If a cluster is automatically updated by Amazon EKS, you must also update the version of your worker nodes after the update is complete\. For more information, see [Worker Node Updates](update-workers.md)\.

Kubernetes supports compatibility between masters and workers for at least 2 minor versions, so 1\.10 workers will continue to operate when orchestrated by a 1\.11 control plane\. For more information, see [Kubernetes Version and Version Skew Support Policy](https://kubernetes.io/docs/setup/version-skew-policy/) in the Kubernetes documentation\.