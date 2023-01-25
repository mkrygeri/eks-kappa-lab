
# If you want to automatically onboard your AWS infrastructure into Kentik and create a cluster using terraform 
## You MUST have the following 
- kentik api credentials
- AWS cli tools configured
- git 
- terraform


## important!! You must have AWS CLI configured as well as a working terraform!!!

## Configure AWS CLI with provided credentials run the following command and put in us-east-2 as the region
```
aws configure
```
git clone [eks-kappa-lab](https://github.com/mkrygeri/eks-kappa-lab)
```

## Modify your variables.tf via your favorite editor
These variables are self explanatory. 

## Kubectl configuration
AWS CLI provides a way to dump out the connection details for an EKS cluster. This populates your $HOME/.kube/config file. This config file will be provided to each person, so nobody will actually need to login to AWS to manage thier cluster.

### Note: When deploying your own k8 cluster, you only need this command.  
```
aws eks update-kubeconfig --name=[CLUSTER_NAME]  
```

### Note: with EKS, you can output the .kube/config file without writing the config with the following command:
```
aws eks update-kubeconfig --name $your_cluster_name --dry-run 
```

From here, follow the instruction for installing kappa.


###  Credits - Hashicorp k8 learning lab
This excercise is based partly on the lab found [here](https://github.com/hashicorp/learn-terraform-provision-eks-cluster/blob/main/eks-cluster.tf). 
The Kentik terraform was taken from Kentik's portal configuration, but the module can be found [here](https://github.com/kentik/config-snippets-cloud/tree/master/cloud_AWS). I used the terraform code generated in the portal and combined it with Hashicorp's

