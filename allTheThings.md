## Kubectl configuration (self-paced lab)
AWS CLI provides a way to dump out the connection details for an EKS cluster. This populates your $HOME/.kube/config file. This config file will be provided to each person, so nobody will actually need to login to AWS to manage thier cluster.

If you dont want to overwrite your existing config. you can run the command by specifying where the config is.

Note: with EKS, you can generate the .kube file with the followind command:
```
aws eks update-kubeconfig --name kl-krygeris-eks-lab --kubeconfig $HOME/my_directory/.kube 
```

### Note: When deploying your own k8 cluster, you only need this command.  
```
aws eks update-kubeconfig --name=[CLUSTER_NAME]  
```

###  Hashicorp k8 learning lab
This excercise is based partly on the lab found [here](https://github.com/hashicorp/learn-terraform-provision-eks-cluster/blob/main/eks-cluster.tf). 
The Kentik terraform was taken from Kentik's portal configuration, but the module can be found [here](https://github.com/kentik/config-snippets-cloud/tree/master/cloud_AWS). I used the terraform code generated in the portal and combined it with Hashicorp's

