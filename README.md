# eks-kappa-lab

## prerequisites

- an AWS account - Provided by Kentik. Please see Ted if you need help with this.
- the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) v2.7.0/v1.24.0 or newer, installed and [configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)
- [AWS IAM Authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)
- [kubectl](https://kubernetes.io/docs/tasks/tools/) v1.24.0 or newer
- git in order to clone repos to deploy

## Kubectl configuration 
AWS CLI provides a way to dump out the connection details for an EKS cluster. This populates your $HOME/.kube/config file. This config file will be provided to each person, so nobody will actually need to login to AWS to manage thier cluster.

### Note: When deploying your own k8 cluster, you only need this command.  
aws eks update-kubeconfig --name=[CLUSTER_NAME]  

