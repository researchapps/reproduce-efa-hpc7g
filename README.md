# Reproducing EFA + eksctl Failure

This will reproduce using eksctl with the elastic fiber adapter, namely that we cannot
get it to work. The current steps require:

```bash
$ aws --version
aws-cli/2.9.18 Python/3.9.11 Linux/5.15.0-75-generic exe/x86_64.ubuntu.22 prompt/off

$ kubectl version --short
Client Version: v1.25.9-dispatcher
Kustomize Version: v4.5.7

$ go version
go version go1.20.3 linux/amd64
```

If you need instructions for installation, [go is here](https://go.dev/doc/install), 
awscli is via `pip install awscli`, and [kubectl instructions are here](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/).


## Setup

You likely want to clone the repository here first:

```bash
git clone https://github.com/researchapps/reproduce-efa-hpc7g
cd reproduce-efa-hpc7g
```

### Build eksctl

You will first want to build eksctl from [this branch](https://github.com/weaveworks/eksctl/pull/6743#pullrequestreview-1507361538)
that has the new nodes added, and a bugfix for the plugin adapter. You can do this as follows.

```bash
git clone -b add/hpc7g-node-arm-support https://github.com/researchapps/eksctl
cd ./eksctl
```

You'll want to build the binary. Note that this will install go dependencies and build a local `eksctl`

```bash
make binary
```

I added this directory to my path and verified it was the eksctl that was being hit, e.g.,

```bash
$ export PATH=$PWD:$PATH
# should return the path to the binary you just built!
$ which eksctl
```

For build environment, I am running Ubuntu 22.04 Linux.

### Deploy with eksctl

Note that the [eks-config.yaml](eks-config.yaml) is going to create the cluster.
Of note:

 - efaEnabled: is set to true
 - we are using hpc7g.16xlarge (not the 4xlarge)
 - we are deploying 12 nodes
 
The scale (12 nodes) is important to reproduce in case it's an issue with the number of nodes,
as is the exact instance type (and not just the family). Here is how to create the cluster
Also note that you can either remove or update the ssh public key path:

```yaml
ssh:
      allow: true
      publicKeyPath: ~/.ssh/id_eks.pub
```

Very likely you don't have the same key as me! I will also note I am
using a dedicated `$HOME/.aws/config` and not the environment credentials,
and I mention this because I've hit bugs using the first. Here is how to create the cluster:

```bash
$ eksctl create cluster -f ./eks-config.yaml
```

Complete output is shown below.

<details>

<summary>Cluster creation details</summary>

```console
$ eksctl create cluster -f ./eks-config.yaml
2023-07-01 20:26:41 [ℹ]  eksctl version 0.148.0-dev+994a2d27f.2023-07-01T20:14:14Z
2023-07-01 20:26:41 [ℹ]  using region us-east-1
2023-07-01 20:26:41 [ℹ]  subnets for us-east-1a - public:192.168.0.0/19 private:192.168.64.0/19
2023-07-01 20:26:41 [ℹ]  subnets for us-east-1b - public:192.168.32.0/19 private:192.168.96.0/19
2023-07-01 20:26:41 [ℹ]  nodegroup "workers" will use "" [AmazonLinux2/1.27]
2023-07-01 20:26:41 [ℹ]  using SSH public key "/home/vanessa/.ssh/id_eks.pub" as "eksctl-scaling-study-efa-nodegroup-workers-4e:93:d9:47:eb:81:3e:4f:1b:e0:44:ac:af:c6:ac:b3" 
2023-07-01 20:26:42 [ℹ]  using Kubernetes version 1.27
2023-07-01 20:26:42 [ℹ]  creating EKS cluster "scaling-study-efa" in "us-east-1" region with managed nodes
2023-07-01 20:26:42 [ℹ]  1 nodegroup (workers) was included (based on the include/exclude rules)
2023-07-01 20:26:42 [ℹ]  will create a CloudFormation stack for cluster itself and 0 nodegroup stack(s)
2023-07-01 20:26:42 [ℹ]  will create a CloudFormation stack for cluster itself and 1 managed nodegroup stack(s)
2023-07-01 20:26:42 [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-east-1 --cluster=scaling-study-efa'
2023-07-01 20:26:42 [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "scaling-study-efa" in "us-east-1"
2023-07-01 20:26:42 [ℹ]  CloudWatch logging will not be enabled for cluster "scaling-study-efa" in "us-east-1"
2023-07-01 20:26:42 [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=us-east-1 --cluster=scaling-study-efa'
2023-07-01 20:26:42 [ℹ]  
2 sequential tasks: { create cluster control plane "scaling-study-efa", 
    2 sequential sub-tasks: { 
        wait for control plane to become ready,
        create managed nodegroup "workers",
    } 
}
2023-07-01 20:26:42 [ℹ]  building cluster stack "eksctl-scaling-study-efa-cluster"
2023-07-01 20:26:43 [ℹ]  deploying stack "eksctl-scaling-study-efa-cluster"
2023-07-01 20:27:13 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-cluster"
2023-07-01 20:27:43 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-cluster"
2023-07-01 20:28:43 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-cluster"
2023-07-01 20:29:44 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-cluster"
2023-07-01 20:30:44 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-cluster"
2023-07-01 20:31:44 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-cluster"
2023-07-01 20:32:45 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-cluster"
2023-07-01 20:33:45 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-cluster"
2023-07-01 20:34:45 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-cluster"
2023-07-01 20:35:46 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-cluster"
2023-07-01 20:36:46 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-cluster"
2023-07-01 20:37:46 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-cluster"
2023-07-01 20:39:49 [ℹ]  building managed nodegroup stack "eksctl-scaling-study-efa-nodegroup-workers"
2023-07-01 20:39:49 [ℹ]  skipping us-east-1b from selection because it doesn't support the following instance type(s): hpc7g.16xlarge
2023-07-01 20:39:49 [ℹ]  EFA requires all nodes be in a single subnet, arbitrarily choosing one: [subnet-0af2a381b46f2c174]
2023-07-01 20:39:50 [ℹ]  deploying stack "eksctl-scaling-study-efa-nodegroup-workers"
2023-07-01 20:39:50 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-nodegroup-workers"
2023-07-01 20:40:20 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-nodegroup-workers"
2023-07-01 20:41:01 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-nodegroup-workers"
2023-07-01 20:42:37 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-nodegroup-workers"
2023-07-01 20:44:17 [ℹ]  waiting for CloudFormation stack "eksctl-scaling-study-efa-nodegroup-workers"
2023-07-01 20:44:17 [ℹ]  waiting for the control plane to become ready
2023-07-01 20:44:17 [✔]  saved kubeconfig as "/home/vanessa/.kube/config"
2023-07-01 20:44:17 [ℹ]  1 task: { install EFA device plugin }
W0701 20:44:18.439148 1335956 warnings.go:70] spec.template.metadata.annotations[scheduler.alpha.kubernetes.io/critical-pod]: non-functional in v1.16+; use the "priorityClassName" field instead
2023-07-01 20:44:18 [ℹ]  created "kube-system:DaemonSet.apps/aws-efa-k8s-device-plugin-daemonset"
2023-07-01 20:44:18 [ℹ]  as you have enabled EFA, the EFA device plugin was automatically installed.
2023-07-01 20:44:18 [✔]  all EKS cluster resources for "scaling-study-efa" have been created
2023-07-01 20:44:18 [ℹ]  nodegroup "workers" has 12 node(s)
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-0-180.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-12-191.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-12-23.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-16-172.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-18-84.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-19-78.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-20-170.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-26-0.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-3-94.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-4-49.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-6-170.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-7-236.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  waiting for at least 12 node(s) to become ready in "workers"
2023-07-01 20:44:18 [ℹ]  nodegroup "workers" has 12 node(s)
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-0-180.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-12-191.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-12-23.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-16-172.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-18-84.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-19-78.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-20-170.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-26-0.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-3-94.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-4-49.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-6-170.ec2.internal" is ready
2023-07-01 20:44:18 [ℹ]  node "ip-192-168-7-236.ec2.internal" is ready
2023-07-01 20:44:22 [ℹ]  kubectl command should work with "/home/vanessa/.kube/config", try 'kubectl get nodes'
2023-07-01 20:44:22 [✔]  EKS cluster "scaling-study-efa" in "us-east-1" region is ready

```

</details>


Note this takes 15-20 minutes, and the logs indicate that we are creating efa.

### View efa daemonset pods 

At this point we should be able to see the daemonset pods with kubectl:

```bash
$ kubectl get pods -n kube-system | grep efa
```
```console
kube-system   aws-efa-k8s-device-plugin-daemonset-2f68m   0/1     Completed          4 (55s ago)   109s
kube-system   aws-efa-k8s-device-plugin-daemonset-2j682   0/1     Completed          4 (58s ago)   109s
kube-system   aws-efa-k8s-device-plugin-daemonset-2p8kh   0/1     Completed          4 (60s ago)   109s
kube-system   aws-efa-k8s-device-plugin-daemonset-cg5pd   0/1     Completed          4 (62s ago)   109s
kube-system   aws-efa-k8s-device-plugin-daemonset-dbwm2   0/1     Completed          4 (61s ago)   109s
kube-system   aws-efa-k8s-device-plugin-daemonset-dm44z   0/1     Completed          4 (55s ago)   109s
kube-system   aws-efa-k8s-device-plugin-daemonset-dwsc8   0/1     Completed          4 (59s ago)   109s
kube-system   aws-efa-k8s-device-plugin-daemonset-fcclh   0/1     CrashLoopBackOff   4 (15s ago)   109s
kube-system   aws-efa-k8s-device-plugin-daemonset-h66h6   0/1     Completed          4 (63s ago)   109s
kube-system   aws-efa-k8s-device-plugin-daemonset-n2k82   0/1     Completed          4 (63s ago)   109s
kube-system   aws-efa-k8s-device-plugin-daemonset-rns26   0/1     Completed          4 (59s ago)   109s
kube-system   aws-efa-k8s-device-plugin-daemonset-shddz   0/1     CrashLoopBackOff   4 (19s ago)   109s
```

Note that mine start crash loop back-offing or registering as "completed" fairly quickly.
Eventually they all crash loop backoff.

```console
aws-efa-k8s-device-plugin-daemonset-2f68m   0/1     CrashLoopBackOff   4 (58s ago)   2m37s
aws-efa-k8s-device-plugin-daemonset-2j682   0/1     CrashLoopBackOff   4 (55s ago)   2m37s
aws-efa-k8s-device-plugin-daemonset-2p8kh   0/1     CrashLoopBackOff   4 (60s ago)   2m37s
aws-efa-k8s-device-plugin-daemonset-cg5pd   0/1     CrashLoopBackOff   4 (58s ago)   2m37s
aws-efa-k8s-device-plugin-daemonset-dbwm2   0/1     CrashLoopBackOff   4 (58s ago)   2m37s
aws-efa-k8s-device-plugin-daemonset-dm44z   0/1     CrashLoopBackOff   4 (59s ago)   2m37s
aws-efa-k8s-device-plugin-daemonset-dwsc8   0/1     CrashLoopBackOff   4 (53s ago)   2m37s
aws-efa-k8s-device-plugin-daemonset-fcclh   0/1     CrashLoopBackOff   4 (63s ago)   2m37s
aws-efa-k8s-device-plugin-daemonset-h66h6   0/1     CrashLoopBackOff   4 (60s ago)   2m37s
aws-efa-k8s-device-plugin-daemonset-n2k82   0/1     CrashLoopBackOff   4 (60s ago)   2m37s
aws-efa-k8s-device-plugin-daemonset-rns26   0/1     CrashLoopBackOff   4 (60s ago)   2m37s
aws-efa-k8s-device-plugin-daemonset-shddz   0/1     CrashLoopBackOff   4 (67s ago)   2m37s
```

I can inspect a log for any one and see this message I've shared before:

```
2023/07/02 02:47:18 Fetching EFA devices.
2023/07/02 02:47:18 No devices found.
```

We can see the daemonset is likely re-creating them because none are ready:

```
$ kubectl get daemonset --all-namespaces
NAMESPACE     NAME                                  DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
kube-system   aws-efa-k8s-device-plugin-daemonset   12        12        0       12           0           <none>          3m50s
kube-system   aws-node                              12        12        12      12           12          <none>          14m
kube-system   kube-proxy                            12        12        12      12           12          <none>          14m
```

And verify that:

```bash
Name:           aws-efa-k8s-device-plugin-daemonset
Selector:       name=aws-efa-k8s-device-plugin
Node-Selector:  <none>
Labels:         <none>
Annotations:    deprecated.daemonset.template.generation: 1
Desired Number of Nodes Scheduled: 12
Current Number of Nodes Scheduled: 12
Number of Nodes Scheduled with Up-to-date Pods: 12
Number of Nodes Scheduled with Available Pods: 0
Number of Nodes Misscheduled: 0
Pods Status:  12 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:           name=aws-efa-k8s-device-plugin
  Annotations:      scheduler.alpha.kubernetes.io/critical-pod: 
  Service Account:  default
  Containers:
   aws-efa-k8s-device-plugin:
    Image:        602401143452.dkr.ecr.us-east-1.amazonaws.com/eks/aws-efa-k8s-device-plugin:v0.3.3
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:
      /var/lib/kubelet/device-plugins from device-plugin (rw)
  Volumes:
   device-plugin:
    Type:               HostPath (bare host directory volume)
    Path:               /var/lib/kubelet/device-plugins
    HostPathType:       
  Priority Class Name:  system-node-critical
Events:
  Type    Reason            Age                    From                  Message
  ----    ------            ----                   ----                  -------
  Normal  SuccessfulCreate  5m16s                  daemonset-controller  Created pod: aws-efa-k8s-device-plugin-daemonset-dbwm2
  Normal  SuccessfulCreate  5m16s                  daemonset-controller  Created pod: aws-efa-k8s-device-plugin-daemonset-h66h6
  Normal  SuccessfulCreate  5m16s                  daemonset-controller  Created pod: aws-efa-k8s-device-plugin-daemonset-2f68m
  Normal  SuccessfulCreate  5m16s                  daemonset-controller  Created pod: aws-efa-k8s-device-plugin-daemonset-2j682
  Normal  SuccessfulCreate  5m16s                  daemonset-controller  Created pod: aws-efa-k8s-device-plugin-daemonset-fcclh
  Normal  SuccessfulCreate  5m16s                  daemonset-controller  Created pod: aws-efa-k8s-device-plugin-daemonset-cg5pd
  Normal  SuccessfulCreate  5m16s                  daemonset-controller  Created pod: aws-efa-k8s-device-plugin-daemonset-dwsc8
  Normal  SuccessfulCreate  5m16s                  daemonset-controller  Created pod: aws-efa-k8s-device-plugin-daemonset-rns26
  Normal  SuccessfulCreate  5m16s                  daemonset-controller  Created pod: aws-efa-k8s-device-plugin-daemonset-n2k82
  Normal  SuccessfulCreate  5m16s (x3 over 5m16s)  daemonset-controller  (combined from similar events): Created pod: aws-efa-k8s-device-plugin-daemonset-dm44z
```

### Additional metadata

In case it's needed, I've saved additional metadata about the nodes in [nodes.json](nodes.json)
as follows:

```bash
kubectl get node -o json > nodes.json
```

I've also saved the [daemonset.json](daemonset.json) to verify the hpc7g instance types are included.

```bash
$ kubectl get daemonset -n kube-system aws-efa-k8s-device-plugin-daemonset -o json > daemonset.json
```

### Clean Up

Delete the cluster with eksctl

```bash
$ eksctl delete cluster -f ./eks-config.yaml 
```
