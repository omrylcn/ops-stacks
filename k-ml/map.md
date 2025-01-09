
# 3 Building an ML platform in Kubernetes

After learning the fundamentals of Kubernetes in the previous chapter, you are now prepared to deploy the toolchains that will power a scalable, secure, and resilient system for your ML projects. While in the previous chapter, you may have used a cluster running on your machine to run a Kubernetes cluster, in this chapter, you will deploy a production-grade Kubernetes cluster using Amazon EKS.

When operating in the cloud, it is best to utilize managed services (like Amazon EKS) whenever they suit your needs. Managed services allow organizations to outsource the management of infrastructure resources to the cloud provider. For example, creating a Kubernetes cluster from scratch is a complicated process. But in the cloud, you can create a production-grade Kubernetes cluster in minutes. By leveraging managed services, you reduce your operational burden and benefit from the expertise and resources of the cloud provider. These services have an edge over self-managed setups, for they are typically more reliable, scalable, and secure. These services are built and maintained by teams of experts who specialize in those services.

It is not easy to look for alternatives to propose in this book cloud-instances. Therefore, we won't be increasing our commitment to open-source tooling in the pursuit of cloud-native excellence.

To keep our promise of being cloud-agnostic, we have forced ourselves to limit the usage of managed services and chosen to deploy resources within the Kubernetes cluster wherever possible. While we'll adhere to our commitment to using open-source tooling for building the system, we'll use the following cloud specific services to build the environment and provide fundamental networking functions:

- Amazon EKS to create a Kubernetes cluster
- Amazon Route 53 for DNS
- AWS Certificate Manager to obtain a wildcard TLS certificate
- Elastic Load Balancer to provision load balancers
- Amazon Elastic Block Storage (EBS) for providing block storage
- Amazon Elastic File System (EFS) for providing NFS-based shared storage

While we have chosen these specific cloud services for this project, you are free to use alternative services that may be available in your own cloud or on-premises environment.

As we have established, an open-source ML system is composed of multiple interconnected components. Each component providing a specific functionality. Since there is no one tool or system that serves all the needs of an ML system, such a system is an amalgamation of tools that function together to provide the necessary capabilities data and ML scientists need to perform for data engineering and model development.

This chapter focuses on setting up the fundamental infrastructure required to run the system. Once you create the base infrastructure, we'll move into building a scalable notebook environment that allows data scientists and ML engineers to perform data analysis and model development. We'll do this by deploying a JupyterHub environment on Kubernetes. The JupyterHub environment we'll create will have the following properties:

Scalable Compute Resources – Leverage Kubernetes to dynamically allocate compute resources based on user demand, ensuring each user has access to the necessary resources for their ML workloads.

Secure Authentication and Authorization – Implement a robust authentication and authorization system using Keycloak to ensure that only authorized users can access the platform and that their actions are properly controlled based on their roles and permissions.

Customizable Development Environment – Provide users with the ability to customize their development environments using Jupyter Notebooks, allowing them to install the libraries, frameworks, and tools required for their specific ML projects.

Collaborative Workspace – Enable users to share and collaborate on ML projects by providing shared workspaces and the ability to manage access control at a granular level.

In the upcoming chapters, we will show how to integrate this environment with other components of the ML toolchain, such as data storage, workflow orchestrators, and model tracking.

The goal of this book is to equip you with the knowledge and skills needed to create a system that closely mirrors a scaled-down version of real-world implementations. By the end, we will demonstrate how to build a system in the cloud without hiding any base infrastructure details, as this is the knowledge environment for most large-scale ML platforms.

## 3.1 Creating a Kubernetes cluster using Amazon EKS

Amazon Elastic Kubernetes Service (Amazon EKS) is a managed Kubernetes service to run Kubernetes in the AWS cloud and on-premises. EKS provides a managed Kubernetes control plane that simplifies provisioning and maintaining clusters. You can use a variety of tools such as the AWS Management Console, AWS CLI, infrastructure-as-code tools (CloudFormation, Terraform, Pulumi, and CDK). The easiest way to create a cluster is using eksctl, which is a command line tool to create EKS clusters and associated infrastructure such as VPCs, subnets, nodes. In this chapter, you will use Terraform to deploy an Amazon EKS cluster.

**Note**
**AWS provides Terraform (<https://github.com/aws-ia/terraform-aws-eks-blueprints>) and CDK templates (<https://github.com/aws-quickstart/cdk-eks-blueprints>) to create clusters.**

Before you can create a cluster, you’ll need to have AWS API credentials configured and AWS CLI installed on your local machine. Please see AWS documentation for configuring credentials (<https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html>). Once you’ve installed AWS CLI and configured, you can run this command to check if it’s functioning properly:

```bash
$ aws sts get-caller-identity
Account: XXXXXXXXXXXX
Arn: arn:aws:iam::XXXXXXXXXXXX:user/realvarez
UserId: AIDAJHI4KEXAY6NXUESUO
```

The IAM role or user you use for this book will need permissions to create resources like Amazon EC2 instances, storage volumes, and a Kubernetes cluster. To keep things simple, we recommend attaching the Power User policy (<https://docs.aws.amazon.com/aws-managed-policy/latest/reference/PowerUserAccess.html>) to your role or user. This practice is acceptable in a lab environment. Such permissive policies shouldn’t be used in production. Please see the following document to learn best practices about securing IAM: <https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html>

To create the EKS cluster and other infrastructure resources it needs, we’ll use Terraform. Terraform (<https://www.terraform.io>) is an open-source infrastructure as code (IaC) tool that allows you to define and provision infrastructure resources across multiple cloud providers using a simple, declarative language. It enables you to manage and version your infrastructure configuration, making it easier to collaborate, track changes, and apply consistent configurations across different environments. Terraform supports a wide range of cloud providers, including AWS, Azure, Google Cloud, and many others, allowing you to manage infrastructure resources in a cloud-agnostic manner.

Please install Terraform on your local machine. You can find the instructions to install Terraform in its documentation (<https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli>).

Clone the book’s code repository and go to the “Chapter 3” directory:

```bash
git clone https://github.com/mlops-on-kubernetes/Book.git
cd 'Book/Chapter 3/eks'
```

The first step of deploying Terraform is to initialize Terraform configuration, which prepares the working directory for use with Terraform by installing required plugins, initializing the backend, and downloading any modules specified in the configuration. It must be run after writing a new Terraform configuration or modifying the provider requirements, backend configuration, or module sources. Initialize Terraform by running:

```bash
terraform init
```

Next, deploy the infrastructure into your AWS account by running:

```bash
terraform apply -target="module.vpc" -auto-approve
terraform apply -target="module.eks" -auto-approve
terraform apply -auto-approve
```

**New AWS accounts are subject to service quotas, which limit resources (like virtual machines, storage volumes, and databases) you can create in an account. In case you are unable to create resources because of exceeding quotas, you can request an increase as described here: <https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html>**

The first module (“module.vpc”) creates network resources required to run a Kubernetes cluster. The second module (“module.eks”) provisions an EKS cluster on top of the first module.

It takes about 10-15 minutes for the deployment to complete. Once Terraform finishes the deployment successfully, configure kubectl on your local machine to use the new cluster you just created:

```bash
aws eks --region us-west-2 update-kubeconfig --name machine-learning
```

Once your cluster is ready, you can check out what’s running in it already:

```bash
$ kubectl get pods -A
NAMESPACE            NAME                             READY   STATUS
kube-system          aws-node-5n7gk                   2/2     Running
kube-system          aws-node-kcszl                   2/2     Running
kube-system          coredns-774bc96b49-ds4mj         1/1     Running
kube-system          coredns-774bc96b49-l8ddg         1/1     Running
```

There are two worker nodes running these Pods. The provisioned cluster runs the following resources:

CoreDNS – Runs as a Deployment with two replicas.
kube-proxy, VPC CNI (aws-node), eks-pod-identity-agent – Run as a DaemonSet (once per node).
Karpenter – Runs as Deployment with two replicas.
EBS CSI Driver – Includes a controller Deployment and a DaemonSet to add block storage volumes for workloads.
EFS CSI Driver – Similar to the EBS CSI Driver, provides shared storage for workloads.

## 3.2 Enable integration with cloud services

When operating in a cloud environment, it is vital to utilize cloud infrastructure services for integration systems. These fundamental infrastructure building blocks like compute, storage, and networking services provide functionality that distributed systems often need. Since they are available in all cloud providers, integrating with these systems doesn't tie you into a specific cloud provider.

In this section, we will enable the provisioning of load balancers, storage volumes, and Kubernetes node level autoscaling, which entails adding and removing worker nodes based on what's running in your cluster.

### 3.2.1 Enabling load balancing

As we learned in chapter 2, workloads running in a Kubernetes cluster are isolated, scalable and within the cluster's boundaries. By default, you cannot connect to these services. To make services available to users, you must attach a load balancer. The load balancer makes services running in a Kubernetes cluster available publicly (from the internet) or privately (from a corporate network). When you need a highly available application, which can have multiple active replicas to avoid a single point of failure, a load balancer also distributes traffic to the backend Pods by DNS and distributes them evenly across the replicas.

Besides the controllers (part of Controller Manager) that are part of a vanilla Kubernetes controller, you can install additional controllers that enable Kubernetes to integrate with external systems. AWS Load Balancer Controller is a Kubernetes controller that allows Kubernetes to create AWS load balancers (both Network Load Balancer and Application Load Balancer). Once installed, the controller can provision and configure load balancers whenever you create a service of type "LoadBalancer" or an Ingress.

Unless you are an account administrator, you need permissions to provision resources in AWS. These permissions are managed in AWS Identity and Access Management (IAM), which is a service that allows you to securely control access to AWS resources by creating and managing users, groups, roles, and permissions.

Note:
In AWS, you can assign IAM roles to Pods so Pods can create resources like a Network Load Balancer. According to the principle of least privilege, assigning IAM roles to Pods allows you to grant limited permissions to Pod. This limits the damage an intruder can inflict in your environment should they gain unauthorized access by exploiting a vulnerability. For more information please see IAM roles for service accounts in EKS documentation (<https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html>).

Create an IAM policy that grants permissions to manage AWS load balancers:

```bash
$ curl -o aws_load_balancer_controller_iam_policy.json https://raw.githubusercontent.com/mlops-on-kubernetes/Book/refs/heads/main/Chapter%203/eks/aws_load_balancer_controller_iam_policy.json

$ aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://aws_load_balancer_controller_iam_policy.json 
```

**Note**
We recommend using the latest version of the policy. You can find the latest IAM policy document in AWS documentation (<https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html>)

Create environment variables for AWS account ID and cluster name:

```bash
AWS_ACCOUNT=$(aws sts get-caller-identity --query 'Account' --output text)
CLUSTER_NAME=$(kubectl config view --minify -o jsonpath='{.clusters[].name}' | cut -d '/' -f2)
```

To assign an IAM role to a Pod, you associate an IAM role to a service account. The service account can then be assigned to one or more Pods. A service account provides an identity for processes that run in a Pod.

Install eksctl (<https://eksctl.io>) on your local machine. Then, using eksctl, create a Kubernetes service account that will give AWS Load Balancer Controller Pods the IAM permissions to manage load balancers:

```bash
$ eksctl create iamserviceaccount \
  --cluster=${CLUSTER_NAME} \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name ${CLUSTER_NAME}-AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::${AWS_ACCOUNT}:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

Next, get the ID of the Virtual Private Cloud (VPC) that's associated with your EKS cluster. We need to supply this information when installing the AWS Load Balancer Controller:

```bash
VPC_ID=$(aws eks describe-cluster --name $CLUSTER_NAME --query "cluster.resourcesVpcConfig.vpcId" --output text)
```

Install Helm (<https://helm.sh>) on your local machine and deploy the AWS Load Balancer Controller Helm chart:

```bash
$ helm repo add eks https://aws.github.io/eks-charts
$ helm repo update eks
$ helm upgrade -i \
  aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=${CLUSTER_NAME} \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set vpcId=${VPC_ID} \
  --set region=us-west-2
```

Validate that the controller is running:

```bash
$ kubectl get deployment -n kube-system aws-load-balancer-controller
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
aws-load-balancer-controller   2/2     2            2           4m22s
```

When you see 2/2 in ready state, the controller is running properly.

Now that you can install AWS Load Balancer using Platform as well. In fact, the Platform team installs Karpenter using a Helm chart. In production, it is a best practice to create and manage resources using infrastructure as code. The steps to deploy the AWS Load Balancer Controller are included to illustrate how you can deploy controllers in Kubernetes using Helm and secure them using IAM roles.

### 3.2.2 Enabling cluster autoscaling

Cluster autoscaling in Kubernetes refers to the ability to automatically add or remove nodes from a cluster based on the resource demands of the running workloads. This is typically achieved using the Kubernetes Cluster Autoscaler (<https://kubernetes.io/docs/concepts/cluster-administration/cluster-autoscaling/>), which monitors the cluster and makes decisions to scale up or down the number of nodes accordingly.

Karpenter is an open-source Kubernetes node autoscaler. It is an alternative to the Kubernetes Cluster Autoscaler. It automatically adds new nodes to a cluster when there are Pods that cannot be scheduled due to insufficient resources in the current nodes. It also periodically scans the cluster for nodes that are underutilized to move or running pods to be rescheduled to other existing nodes, Karpenter will terminate these underutilized nodes to reduce costs.

**At the time of writing, Karpenter is only available in AWS. If you want to run it in AWS, we'll install the Kubernetes Cluster Autoscaler (<https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler>) in your cluster to enable autoscaling.**

Even though we've installed Karpenter, it can't start to autoscale our cluster. Create Karpenter custom resources to enable cluster autoscaling:

```bash
kubectl apply -f karpenter.yaml
```

Listing 3.1 shows a snippet of the Karpenter NodePool configuration. A NodePool specifies the type, size, and capacity that Karpenter can create when the cluster has Pods that cannot be scheduled (because the cluster has no more available capacity). This configuration enables Karpenter to create any EC2 instances of type "C" (Compute Optimized), "M" (Memory Optimized), and "R" (Memory Optimized) by default. For workloads that require GPUs, Karpenter create nodes with GPUs. Some EC2 instances have different capabilities. For example, G and P instances are optimal for machine learning workloads as they have GPUs.

Listing 3.1 Snippet of Karpenter NodePool configuration

```yaml
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: default
spec:
  template:
    spec:
      requirements:
        - key: "karpenter.k8s.aws/instance-category"
          operator: In
          values: ["c", "m", "r"]
…
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: gpu
spec:
  template:
    spec:
      requirements:
        - key: "karpenter.k8s.aws/instance-category"
          operator: In
          values: ["g", "p"]
…
```

Under this configuration, Karpenter will determine the best instance to create when new nodes are required to add to the cluster due to capacity. Karpenter will dynamically select the best node to create based on the workload to be deployed in your cluster.

### 3.2.3 Providing Persistent Storage

In Chapter 2, we introduced the concept of Container Storage Interface (CSI), which enables Kubernetes to integrate with external storage systems for providing persistent storage to workloads. It is common to host a Kubernetes cluster that hosts workloads with mixed storage requirements.

For example, workloads such as databases require block storage that can sustain high input/output operations per second (IOPS). In AWS, you can use block storage for workloads using Amazon Elastic Block Storage (EBS). However, you can only attach an EBS volume to one EC2 at a time. As a result, when you have a storage that multiple Pods can read and write to, you cannot use EBS. When multiple Pods need access to a shared storage, you'll need other storage services such as Amazon Elastic File System (EFS), Amazon FSx for Lustre, or Amazon Simple Storage Service (S3).

When bootstrapping the Kubernetes cluster, our Platform team will set up the CSI drivers for Amazon EBS and Amazon EFS. Whenever a Pod requires block storage, the EBS CSI driver dynamically provisions an EBS volume and attaches it to the Pod.

To provide a shared storage for multiple Pods, we'll use Amazon EFS. Instead of dynamically creating volumes, with EFS we will use static provisioning. Static provisioning is ideal for reusing existing or shared volumes. In static provisioning, the filesystem or volume is created and configured in advance, before being needed by Pods. Static provisioning involves a cluster administrator pre-creating PersistentVolume (PV) objects with specific storage configurations. These PVs are only available for binding to PersistentVolumeClaims (PVCs) when Pods need storage.

Besides setting up the CSI drivers, Platform also creates the EFS StorageClass, PV, and a PVC.

## 3.3 Setting up an identity system**

Building a platform for machine learning (ML) systems in a large organization is a complex undertaking. This is especially true in multi-tenant environments where multiple teams share a Kubernetes cluster. With multiple teams involved in data analytics, model development, and other ML-related tasks, scalability and accessibility are top priorities. Additionally, ensuring that access to tools and systems is limited to authorized users is crucial to maintain data integrity and protect sensitive information.

Consider that you're tasked with building a platform for ML systems in a large organization. Your organization has several teams that will use this platform for data analytics, model development, and other ML-related tasks. Therefore, the system needs to be scalable. Teams need different tools such as Jupyter notebooks, workflow orchestration systems, and monitoring tools. The goal is to ensure users can easily access these tools to perform their duties. At the same time, you must secure access to these tools and systems to ensure that only authenticated users have access to the tools.

We've learned that Kubernetes is a great place to run a wide variety of workloads and tools. However, building a platform is much more than just installing a bunch of tools and letting users have their way to the complexity. A platform improves developer experience by removing all points of friction that can impede software development.

To get started to build the next ML platform on Kubernetes, one of the first challenges you'll need to address is how to provide a robust authentication and authorization system for users. While ML tools you'll deploy in Kubernetes provide their own API interfaces for the users and administrators, most of these tools don't have an in-built identity and access management and rely on external services to provide this functionality. Chances are your organization already has a centralized identity management for authentication and authorization. We can use Keycloak (<https://www.keycloak.org>) to integrate with your organization's existing identity management system or as a standalone identity provider for your ML platform.

*Keycloak* is an enterprise-grade, open-source identity and access management solution that provides features such as single sign-on (SSO), user federation, and role-based access control (RBAC). It supports various authentication protocols, including OpenID Connect, OAuth 2.0, and SAML 2.0, making it compatible with a wide range of applications and services.

By integrating Keycloak with your ML tools, you can:

- Centralize authentication – Users can access all the ML tools in your platform using a single set of centralized credentials, eliminating the need to manage separate accounts for each tool.
- Enforce access control – Define roles and permissions in Keycloak to control user access to specific resources and actions within your ML tools, ensuring that users can only access what they're authorized to.
- Integrate with existing identity management – Keycloak can federate with your organization's existing identity management system, such as Active Directory or LDAP, allowing you to reuse existing user accounts and groups.
- Enable single sign-on (SSO) – Users can authenticate once and access multiple ML tools without being prompted for credentials again, providing a seamless user experience.

You'll need a public domain for Keycloak. If you don't already have a public domain, you can purchase one from Route 53 (<https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/registrar.html>) or another registrar like GoDaddy. You can also use any domain name service. Please ensure that the registrar you choose allows you to change name servers, as you can manage it using Route 53.

## 3.4 Creating a self-service development environment**

To maximize developer productivity and eliminate manual processes that hinder access to necessary tools, we'll create a self-service system for ML and data scientists to provision development environments on-demand. In this section, we'll explore deploying JupyterHub (<https://jupyter.org/hub>) in our Kubernetes cluster and using Keycloak for user authentication.

By providing a self-service platform, ML and data scientists can quickly provision notebook environments on their own, without waiting for manual provisioning or approvals. As a bonus of this deployment method, we'll have standardized the development environment ensuring consistent tooling, libraries, and configurations across projects, reducing compatibility issues and setup time.

### 3.4.1 Deploy JupyterHub**

We'll deploy JupyterHub in a new Kubernetes namespace. Namespaces are not only for the logical grouping of resources, but they can help to isolate workloads in Kubernetes. Furthermore, you can implement quotas (<https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/>) to ensure that workloads running in a namespace don't consume all available CPU and memory resources in a cluster, starving workloads running in other namespaces.

First, let's set up the environment variables needed for JupyterHub:

```bash
export JH_COOKIE_SECRET=$(openssl rand -hex 32)
export JH_HOSTNAME=platform.${DOMAIN}
export KEYCLOAK_HOSTNAME=auth.${DOMAIN}
export JH_VERSION=3.3.5
```

Go to the Chapter 3/jupyterhub directory and generate a Helm values file using the template:

```bash
cd ../jupyterhub
envsubst < jupyterhub-values.yaml.template > jupyterhub-values.yaml
```

Install JupyterHub using Helm:

```bash
$ helm repo add jupyterhub https://hub.jupyter.org/helm-chart/
$ helm repo update
$ helm upgrade --cleanup-on-fail \
  --install jhub jupyterhub/jupyterhub \
  --namespace jupyter \
  --version=${JH_VERSION} \
  --values jupyterhub-values.yaml
```

Now you'll create a record in the Route 53 hosted zone, so when a user needs a Jupyter notebook, they can simply open their browser and go to <https://platform>.<your_domain>.<tld>.

```bash
sh create-a-record.sh $DOMAIN platform
```

When you go to your URL (for example, <https://platform.mlopsbook.online/jupyter>), you'll be presented with a login page.

This setup creates a secure, scalable Jupyter environment that integrates with our authentication system, allowing users to spin up their own development environments while maintaining security and resource controls.

Let me translate and explain this important section about authentication and persistence in JupyterHub:

**Login and Authentication**
You can now log in using username "admin" and the password you set during the Keycloak setup. If you're already logged into Keycloak as the "admin" user, you'll need to logout of your current Keycloak session.

**3.4.2 Role-Based Access Control with Keycloak**

Keycloak's user management capabilities are particularly valuable when building an environment that needs to support a diverse range of developers with varying roles and responsibilities. A key feature that Keycloak offers is role-based access control (RBAC), which provides a sophisticated way to manage permissions based on users' job functions while preventing unauthorized access.

With RBAC, you can create a hierarchy of roles that reflect different levels of access and responsibilities within your organization. For example, you might define roles such as:

- "Developer" for those building models
- "Project Manager" for those overseeing projects
- "Data Scientist" for those analyzing data
- "Administrator" for those managing the platform

Each role can be assigned specific permissions that determine what applications, resources, or actions users with that role are allowed to access or perform.

Once you've defined these roles and their associated permissions, you can assign them to individual users or groups within Keycloak. This process is typically straightforward and can be done through Keycloak's user interface or programmatically via its APIs. For instance, you might assign the "Developer" role to software engineers, granting them access to the integrated development environment (IDE), version control system, and deployment tools. Similarly, you could assign the "Data Scientist" role to data analysts, allowing them to access data processing and analysis tools while restricting their access to the codebase.

**3.4.3 Providing Persistence to Notebook Servers**

Let's understand how JupyterHub handles user sessions. When a user signs into JupyterHub for the first time, JupyterHub creates a new notebook Pod based on a predefined template. This Pod serves as the user's personal notebook server. If the user disconnects from the session and logs in again later, JupyterHub will reconnect them to their existing notebook server.

To ensure users don't lose their work and data in the notebook workspace, each user's Pod is assigned a persistent volume backed by a storage service like Amazon EBS (<https://aws.amazon.com/ebs/>). In other cloud and hybrid environments, you can use a similar storage service. If a user's notebook server crashes or moves to another node, Kubernetes will reattach the persistent volume, preventing any data loss during infrastructure failure events.

This setup creates a robust and user-friendly environment where data scientists can work continuously without worrying about losing their work, even in the case of system failures or maintenance events.

![alt text](image.png)

JupyterHub provides flexibility to handle multiple persistent volumes of notebooks. This functionality becomes particularly useful when users need a shared storage to exchange data and artifacts.

Let's explore the common storage patterns for notebook servers:

1. **Block Storage**
JupyterHub notebook servers can use a persistent volume backed by block storage services like Amazon EBS. Block storage offers high throughput and low latency storage, but it can become expensive as you provision fixed-size volumes per user. For example, every JupyterHub pod might require a 10 GB volume to store configuration data.

1. **Shared Storage**
Notebook servers can store data using a persistent volume backed by shared storage services like NFS or Amazon EFS. While shared storage is more cost-effective than block storage, it doesn't provide the same performance levels as block storage.

1. **Combined Block and Shared Storage**
Notebook servers can use multiple persistent volumes - one backed by block storage for workspace data, and another shared volume that acts as a repository for shared data and artifacts.

Beyond block storage, all notebook servers have access to shared storage. The shared NFS filesystem is mounted at `/home/shared`. When users need to exchange data, they can do so using this shared directory.

**Customizing User Environment**

By default, notebook servers that are created run as the "jovyan" user in the Unix environment of their cluster. Since this user doesn't have access to a graphics processing unit (GPU), these notebook servers don't have GPU resources. However, you have the flexibility to add GPU support to your notebook servers if needed.

Kubernetes allows you to create nodes with GPUs and instruct it to run notebook servers on nodes that have GPUs attached. This way, users who require GPU acceleration for their workloads, such as training deep learning models or running computationally intensive simulations, can leverage the power of GPUs seamlessly.

It's important to note that GPU resources can be expensive, and not every user might require them. If you provision GPU nodes for all users, you might end up paying for expensive resources that go underutilized. To optimize costs, you can adopt a more targeted approach.

JupyterHub allows you to create custom profiles for notebook servers where you can specify the CPU, RAM, and memory requirements, runtime, and startup scripts for notebook servers.

For example, consider three notebook profiles:

1. All Stack Notebook - For data scientists who use Python, Scala, and R
2. Python Only - For data science users who only use Python
3. TensorFlow - For users who will create models using TensorFlow and need a GPU

These profiles can be specified in your `jupyterhub-values.yaml` file as shown in Listing 3.2:

```yaml
singleuser:
  profileList:
    - display_name: "All Spark environment"
      description: "Python, Scala, R and Spark Jupyter Notebook Stack"
      default: true
      kubespawner_override:
        image: jupyter/all-spark-notebook
    
    - display_name: "Python only"
      description: "Data Science Jupyter Notebook Python Stack"
      kubespawner_override:
        image: jupyter/datascience-notebook
    
    - display_name: "TensorFlow with GPU"
      description: "Jupyter Notebook Python Stack with TensorFlow"
      kubespawner_override:
        image: jupyter/tensorflow-notebook
        extra_resource_guarantees:
          nvidia.com/gpu: "1"
        extra_resource_limits:
          nvidia.com/gpu: "1"
```

This configuration allows users to select the environment that best matches their needs while efficiently managing computational resources.

Let me help decode and translate that text to English. It appears to be about enabling GPU workloads in Kubernetes. Here's the English translation of the content:

### 3.4.5 Enabling GPU Workloads in Kubernetes

Out of the box, your Kubernetes cluster is not ready to run GPU workloads. If you attempt to start a “TensorFlow with GPU” notebook, the server will fail to start. Just like you need drivers on your local machine to use GPUs, you must install the drivers on worker nodes before Kubernetes can run GPU workloads. Thankfully, NVIDIA makes it easy to install drivers in Kubernetes environment. The NVIDIA GPU Operator (<https://github.com/NVIDIA/gpu-operator>) is a Kubernetes operator that simplifies the installation of NVIDIA GPU drivers and configuration.

Install NVIDIA GPU Operator using Helm:

```
$ helm repo add nvidia https://helm.ngc.nvidia.com/nvidia
$ helm repo update
$ helm upgrade -i gpuo nvidia/gpu-operator \
  -n kube-system \
  --values nvidia-gpu-operator.yaml
```

The NVIDIA GPU Operator plays a crucial role in managing GPU resources. Among other things, it runs the GPU Feature Discovery (<https://github.com/NVIDIA/gpu-feature-discovery>) tool, which detects GPUs on a node and advertises their availability to Kubernetes using node labels. This enables machines enables Kubernetes to identify and allocate GPU resources to workloads.

When you attempt to start a "TensorFlow with GPU" notebook server in JupyterHub, the corresponding Pod will initially remain in the "Pending" state. This is because none of the existing nodes in the cluster has a GPU available. To address this situation, Autopilot will automatically provision a new node with a GPU.

The process of creating a new node with a GPU and installing the necessary NVIDIA utilities can take up to five minutes. During this time, the following steps occur:

1. Node Provisioning – Autopilot handles the creation of a new node in the cluster, ensuring that it meets the GPU requirements specified by the Pod.
2. GPU Driver Installation – Once the new node is provisioned, the NVIDIA GPU Operator installs the proprietary GPU drivers and utilities on the node.
3. GPU Feature Discovery – The GPU Feature Discovery runs on the new node, detecting the available GPUs and advertising their capabilities using Kubernetes node labels.
4. Pod Scheduling – Once the GPU resources are available and enabled, Kubernetes can schedule the "TensorFlow with GPU" notebook server Pod to the newly provisioned node.

To accommodate the potentially longer startup time required for provisioning a new node with a GPU and setting up the necessary software, we have increased the notebook server startup timeout to ten minutes (using the Helm values file). This configuration ensures that JupyterHub does not prematurely terminate the notebook server startup process, allowing sufficient time for the GPU resources to become available and for the Pod to be scheduled successfully.

```
singleuser:
  startTimeout: 600
```

![alt text](image-1.png)

By leveraging the NVIDIA GPU Operator and Autopilot's scaling capabilities, your Kubernetes cluster can dynamically provision GPU resources on-demand. This approach ensures efficient resource utilization and enables seamless access to GPU-accelerated workloads, such as those required for machine learning or scientific computing tasks.

### 3.4.6 Reducing wastage by shutting down idle notebooks

Cloud environments, whether public or private, offer elastic compute capacity, allowing workloads to seamlessly scale up and down based on demand. This scalability is particularly beneficial in scenarios like the JupyterHub environment deployed in your Kubernetes cluster. Initially, with only one user, JupyterHub creates a single Pod to host the user's notebook server. However, as more users onboard, JupyterHub dynamically creates additional Pods to accommodate their notebook servers.

While this on-demand provisioning of resources is advantageous, it can also lead to inefficiencies if not managed properly. Once a user's notebook server is provisioned, it continues to consume compute resources, even when the user is not actively utilizing the server or has completed their work. This idle resource consumption can result in wasted resources and unnecessary costs.

To address this challenge, JupyterHub provides a feature that allows you to stop a user's notebook server after it has been idle for a specified period. By implementing this idle server timeout, you can effectively optimize resource utilization and reduce waste.

In jupyterhub-values.yaml, we have enabled culling user notebooks after they have been idle for more than 3600 seconds. This configuration is specified as follows:


Note
every user's notebook server in JupyterHub has an associated persistent volume. This persistent volume ensures that user data remains intact even when their notebook server is terminated. When JupyterHub terminates a user's notebook server, whether due to inactivity, resource constraints, or other reasons, the persistent volume containing the user's data is not affected. The data remains safely stored and available for future use. Subsequently, when the user logs in again after their notebook server has been terminated, JupyterHub will create a new Pod for their notebook server. However, this new Pod will have the same persistent volume attached, allowing the user to seamlessly pick up where they left off.

Setting maximum age helps to ensure that the containers used for notebook servers run the most up-to-date versions of JupyterHub and included libraries, thereby improving the security and reliability of the notebook servers.

By regularly terminating and recreating notebook servers, you ensure that the latest security patches and updates are applied, reducing the risk of vulnerabilities in outdated software versions. Users also benefit from a consistent and up-to-date environment, reducing the likelihood of encountering issues caused by outdated software or configurations.

As JupyterHub terminates idle notebook server Pods, Karpenter will terminate any unused nodes automatically. This dynamic resource management capability helps optimize resource utilization and reduce costs by ensuring that resources are allocated only when needed.


Let me help translate that text about GPU node utilization in Kubernetes:

### 3.4.7 Optimizing GPU node utilization

For GPU resources, their efficient utilization is crucial for cost-effective operation. By default, Kubernetes does not differentiate between GPU and non-GPU workloads when scheduling, which can lead to suboptimal use of GPU nodes. To ensure that only workloads requiring GPUs are scheduled on GPU nodes, you can leverage Kubernetes's taints and toleration mechanism.

*Taints* are applied to nodes to mark them as having special properties that should not accept most pods unless those pods explicitly tolerate the taints. In our Kubernetes setup, we have added a taint to nodes with GPUs:

```
taints:
- key: nvidia.com/gpu
  effect: NoSchedule
```

With this configuration, Kubernetes adds a taint to the GPU nodes with the "gpu" key value, indicating to Kubernetes that no pods should be scheduled on these nodes unless they tolerate the taint.

*Tolerations* are applied to Pods to allow them to be scheduled on nodes with specific taints. To ensure that only GPU workloads (for example the "TensorFlow with GPU" notebook server) are scheduled on GPU nodes, a toleration is added to the GPU Pods. For example, in the "jupyter-values.yaml" file, there's a toleration added to "TensorFlow with GPU" notebook profile as shown in listing 3.3.

**Listing 3.3 Snippet of JupyterHub configuration specifying tolerations for GPU pods**

```
- display_name: "TensorFlow with GPU"
  description: "Jupyter Notebook Python Stack with TensorFlow"
  kubespawner_override:
    image: jupyter/tensorflow-notebook
    extra_resource_guarantees:
      nvidia.com/gpu: "1"
    extra_resource_limits:
      nvidia.com/gpu: "1"
    tolerations:
    - key: nvidia.com/gpu
      operator: Exists
      effect: NoSchedule
```

By utilizing taints and tolerations, Kubernetes ensures that GPU nodes are dedicated only to pods in the cluster requiring GPU resources. Without such mechanism, Kubernetes could potentially schedule non-GPU Pods on GPU nodes. This would prevent Kubernetes from shutting down these nodes when they are not needed for GPU workloads.

To avoid such scenarios, the taints and tolerations ensure that non-GPU workloads are only scheduled on nodes without a GPU. This way, GPU nodes remain dedicated to GPU-intensive workloads, and Kubernetes can efficiently scale down the GPU pool when not needed for GPU resources anymore.