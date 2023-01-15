# eks-kappa-lab

## prerequisites

- [kubectl](https://kubernetes.io/docs/tasks/tools/) v1.24.0 or newer. This Allows you to communicate with your cluster via commandline.
- Git in order to clone repos to deploy into your cluster
- Basic understanding of Kubernetes terms e.g. Containers, Pods, namespaces, deployments, daemonsets etc.. 

## Goal of this session
The goal here is to emulate what someone might do if they own a Kubernetes environement but don't have access to the underlying infrastructure. This means you will not need to login to the AWS console or connect to any nodes via ssh.

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
This excercise is based on the lab found [here](https://github.com/hashicorp/learn-terraform-provision-eks-cluster/blob/main/eks-cluster.tf). 
The Kentik terraform was taken from Kentik's portal configuration, but the module can be found [here](https://github.com/kentik/config-snippets-cloud/tree/master/cloud_AWS).  

a configuration file has been provided for each user to connect to thier own cluster. Place this file in the ~/.kube/ directory.


## Testing your configuration
You can do a basic test of the configuratipn by running the following command:
```
kubectl get pods -A
```
Example:
```% k get pods -A
NAMESPACE     NAME                       READY   STATUS    RESTARTS   AGE
kube-system   aws-node-4pjv4             1/1     Running   0          29m
kube-system   aws-node-7gwdl             1/1     Running   0          29m
kube-system   aws-node-m962p             1/1     Running   0          29m
kube-system   coredns-5c5677bc78-cbqct   1/1     Running   0          34m
kube-system   coredns-5c5677bc78-vm7fm   1/1     Running   0          34m
kube-system   kube-proxy-cxp6k           1/1     Running   0          29m
kube-system   kube-proxy-jb858           1/1     Running   0          29m
kube-system   kube-proxy-p4x9f           1/1     Running   0          29m
```


This will give you a listing of all the running pods in all Namespaces.

Once we are confident that we can communicate with our assigned clusters, we can move onto deploying Kappa to Kube.

1. Go to your commandline and clone the Master Kentik Kappa repository.
```
git clone https://github.com/kentik/kappa-cfg
```
2. Login to the provided Kentik Portal and add a device name it something with your name so we know who belongs to each device. Set the device type to "Process-aware Telemetry agent". Make note of the device name and plan-id. 

3. CD into the kappa-cfg directory and use your favorite editor to open up the Kustomization file. This is the file that you need to edit in order to point the output to Kentik. There are 2 sections that need to be nodified for now.

One is the know as a "configmap". This helps to define what interfaces we are looking at for traffic. 
```
configMapGenerator:
  - name: kappa-config
    literals:
      - capture=ens3|veth.*
      - device=kappa_test
      - region=US
      - plan=1234
      - bytecode=x86_64/kappa_bpf-ubuntu-5.4.o
 ```
**** Note that you can configure a list of interface names. This is important and depends on the OS used. If you are unsure you can ask your AWS administrator. In mine, I had to add eth* to the list.


Second is the kentik credentials used. You can use credentials shown in the kproxy config if you want to be sure no other users will change the token. Alternatively, create your own user account in this portal.  

```
secretGenerator:
  - name: kentik-api-secrets
    literals:
      - email=test@example.com
      - token=asdf1234
```

Once everything has been filled in, it is now time to deploy.

4. From the kappa-cfg directory run the following:
```
kubectl apply -k .
```
*** -k tells kubectl that there is customization to be done. "." just means the local directory.

you'll see output like this if all goes well:
```
% kubectl apply -k .
namespace/kentik created
serviceaccount/kubeinfo created
clusterrole.rbac.authorization.k8s.io/kubeinfo created
clusterrolebinding.rbac.authorization.k8s.io/kubeinfo created
configmap/kappa-bytecode-5mmmmcb2dt created
configmap/kappa-config-4fd6b66m8d created
configmap/kappa-init-5c79hkbb6c created
secret/kentik-api-secrets-8fb929gg68 created
service/kappa-agg created
deployment.apps/kappa-agg created
deployment.apps/kubeinfo created
daemonset.apps/kappa-agent create
```

5. Let's Verify things worked. One simple way is to look in the portal to see that the device is now on your list of devices. Or we can verify it is working by looking at pod logs. 
Use kubectl to find the pod with "kappa-agg" in the name. Does anything look off when you list the pods?

k logs kappa-agg-xxxx-xxxx

You will see that it created a device and each of the agents has connected to it.

```
[2023-01-15T19:52:34Z INFO  kappa] initializing kappa 1.1.1
[2023-01-15T19:52:34Z DEBUG kappa::export] creating device aws_kappa_krygeris
[2023-01-15T19:52:34Z DEBUG kappa::export] device Device { id: 144660, name: "aws_kappa_krygeris", kind: "host-nprobe-dns-www", subtype: "kappa", 
...
[2023-01-15T19:52:37Z DEBUG kappa::augment::augment] connection from 10.0.3.140:50420
[2023-01-15T19:52:42Z DEBUG kappa::agg] connection from 10.0.3.223:42098
[2023-01-15T19:52:43Z DEBUG kappa::agg] connection from 10.0.1.29:51348
[2023-01-15T19:52:44Z DEBUG kappa::agg] connection from 10.0.1.65:44682
```

## Let's talk about all the parts.

namespace/kentik 
This is a unique place that kappa live in inside kubernetes.

serviceaccount/kubeinfo 
This is the account that kappa uses to interact with kubernetes and gather infomration about kubernetes

clusterrole.rbac.authorization.k8s.io/kubeinfo 
This the role used by kubeinfo RBAC

clusterrolebinding.rbac.authorization.k8s.io/kubeinfo
role binding for kubeinfo

configmap/kappa-bytecode-xxxx
This is the eBPF bytecode that is used to collect stats. Used by kappa agent.

configmap/kappa-config-xxxx 
This is the configuration that is used by the agents in the daemonset (kappa agents)

configmap/kappa-init-xxxx

secret/kentik-api-secrets-xxxx
These are the kentic credentials that are safely encrypted at rest

service/kappa-agg 
This is the service that makes sure kappa-agg keeps going.

deployment.apps/kappa-agg
This is the pod that takes all the data from all the nodes and aggregates it. 

deployment.apps/kubeinfo 

daemonset.apps/kappa-agent 
This is the actual agent that resides on each node in the cluster. It uses both packets and ebpf to gather data



## About eBPF



## Removing Kappa. 
You may have noticed that there isnt much talk about removeing it. But sometime you want to start over. Fortunately this is simeple. From your kappa-cfg diretory, simply run the following:
```
kubectl apply -k .
```



