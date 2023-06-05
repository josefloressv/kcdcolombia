# 5 maneras de implementar un Cluster de Kubernetes con Amazon EKS

## 1. Consola de AWS
https://console.aws.amazon.com/

## 2. AWS CLI (kubectl)
```bash
eksctl create cluster \
    --name kcdcolombia \
    --region=us-east-1 \
    --zones=us-east-1a,us-east-1b,us-east-1d \
    --nodegroup-name kcdcolombia-workers \
    --node-type t3.medium \
    --nodes 2 \
    --nodes-min 1 \
    --nodes-max 4 \
    --managed
```

## 3. Terraform
```hcl

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "3.19.0"

  name = "${var.app_name}-vpc"

  cidr = "10.0.0.0/16"
  azs  = var.azs_available

  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.4.0/24", "10.0.5.0/24", "10.0.6.0/24"]

  enable_nat_gateway   = true
  single_nat_gateway   = true
  enable_dns_hostnames = true

  public_subnet_tags = {
    "kubernetes.io/cluster/kcdcolombia" = "shared"
    "kubernetes.io/role/elb"                      = 1
  }

  private_subnet_tags = {
    "kubernetes.io/cluster/kcdcolombia" = "shared"
    "kubernetes.io/role/internal-elb"             = 1
  }
}

module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "19.5.1"

  cluster_name    = "kcdcolombia"
  cluster_version = "1.24"

  vpc_id                         = module.vpc.vpc_id
  subnet_ids                     = module.vpc.private_subnets
  cluster_endpoint_public_access = true

  eks_managed_node_group_defaults = {
    ami_type = "AL2_x86_64"

  }

  eks_managed_node_groups = {
    one = {
      name = "node-group-1"

      instance_types = ["t3.medium"]

      min_size     = 1
      max_size     = 3
      desired_size = 2
    }
  }
}

```
Ejemplo completo: 
https://github.com/josefloressv/learn-terraform-provision-eks-cluster/blob/main/main.tf

## 4. CDK
```python

        # Create a VPC for the EKS cluster
        # https://docs.aws.amazon.com/cdk/api/v2/python/aws_cdk.aws_ec2/Vpc.html
        vpc = ec2.Vpc(
            self,
            "EksClusterVpc",
            cidr="10.0.0.0/16",
            max_azs=2,
            nat_gateways=1,
        )

       # create eks admin role
       # https://docs.aws.amazon.com/cdk/api/v2/python/aws_cdk.aws_iam/Role.html
        eks_master_role = iam.Role(self, 'EksMasterRole',
            role_name='EksAdminRole',
            assumed_by=iam.AccountRootPrincipal()
        )

        # Create the EKS cluster
        # https://docs.aws.amazon.com/cdk/api/v2/python/aws_cdk.aws_eks/Cluster.html
        cluster = eks.Cluster(
            self,
            "EksCluster",
            version=eks.KubernetesVersion.V1_20,
            cluster_name="my-cluster",
            masters_role=eks_master_role,
            default_capacity=0,
            vpc=vpc,
        )

        # Add a node group to the cluster
        # https://docs.aws.amazon.com/cdk/api/v2/python/aws_cdk.aws_eks/Cluster.html#aws_cdk.aws_eks.Cluster.add_nodegroup_capacity
        nodegroup = cluster.add_nodegroup_capacity(
            "NodeGroup",
            instance_types=[ec2.InstanceType("t3.medium")],
            min_size=1,
            max_size=5,
            desired_size=2,
        )

```
Ejemplo completo: https://github.com/josefloressv/eks-cdk-101 \
EKS Blueprints for CDK Workshop\
https://catalog.workshops.aws/eks-blueprints-for-cdk/en-US

## 5. Cloud Formation
```json
```

## Recursos Adicionales
https://github.com/josefloressv/eks/
https://github.com/josefloressv/learn-terraform-provision-eks-cluster


# KCD El Salvador 2023
https://community.cncf.io/events/details/cncf-kcd-el-salvador-presents-kcd-el-salvador-2023/
