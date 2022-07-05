# eks-fargate-quickstart

This example shows how to provision a serverless cluster (serverless data plane) using Fargate Profiles. 

Repo structure:

- terraform-aws-eks-blueprints: terraform modules to provision EKS clusters. The latest source code can be found at https://github.com/aws-ia/terraform-aws-eks-blueprints

- eks-fargate-quickstart: provision a fargate onlu EKS cluster.
  a. one EKS cluster is created in a VPC with private and public subnets
  b. EKS cluster is enbaled with OIDC authentication and with private and public access
  c. Fargate profiles are created with extra IAM roles for logging
  d. Kubernetes add-ons: `vpc-cni`, `kube-proxy`, `coredns` and `ArgoCD`
  e. Amazon Opensearch Service for logging
  f. Amazon managed prmetheus for monitoring 
  
- eks-fargate-addon: Helm charts code used by ArgoCD to deploy:
  a. metrics_server
  b. you can extend to use it to deploy more add-ons for your case

- fluentbit-openseach-logging includs a yaml file to configure built-in fluentbit in fargate and a yaml file for deploying an application for testing

- adot-amp includs a yaml file to deploy adot agents in fargate and a yaml file for deploying an application for testing

   
## supported regions for AMP
Europe (Stockholm)
Europe (London)
Europe (Ireland)
Asia Pacific (Tokyo)
Asia Pacific (Singapore)
Asia Pacific (Sydney)
Europe (Frankfurt)
US East (N. Virginia)
US East (Ohio)
US West (Oregon)


## Prerequisites:

Ensure that you have the following tools installed locally:

1. [aws cli](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
2. [kubectl](https://Kubernetes.io/docs/tasks/tools/)
3. [terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli)

## Deploy

1. Go to directory <your repo directory>/eks-fargate-quickstart

```sh
terraform init
terraform plan
terraform apply
```

Enter `yes` at command prompt to apply

- Validate
The following command will update the `kubeconfig` on your local machine and allow you to interact with your EKS Cluster using `kubectl` to validate the CoreDNS deployment for Fargate. 

Run `update-kubeconfig` command:

```sh
aws eks --region <REGION> update-kubeconfig --name <CLSUTER_NAME>
```

Test by listing all the pods running currently. The CoreDNS pod should reach a status of `Running` after approximately 60 seconds:

```sh
kubectl get pods -A

# Output should look like below
NAMESPACE     NAME                      READY   STATUS    RESTARTS   AGE
kube-system   coredns-dcc8d4c97-2jvfb   1/1     Running   0          2m28s

2. Configure and test logging components
- Go to directory <your repo directory>/fluentbit-openseach-logging, replace <your opensearch domain> and <your region> in fargate-cm.yaml with your setup values. then run  kubectl apply -f  fargate-cm.yaml
- Deploy a test app by running kubectl apply -f test-app.yaml
- Go to your Amazon opensearch service opensearch-demo
    a. click OpenSearch Dashboards URL, in the login page, use default username/password defined in eks-fargate-quickstart/variables.tf
    b. Unde left side panel OpenSearch Plugins/Security, add your fargate execution role arn into all_access adn security_manager Role
    c. Then you can create index pattern and start using. the default index pattern is fargate_log

3. Configure and test your monitor components
- Go to directory <your repo directory>/adot-amp,replace <your eks cluster amp-ingest-irsa role>, <your amp remote write endpoint> and <your region> in adot-collector-fargate.yaml with your setup values. then run  kubectl apply -f adot-collector-fargate.yaml
    a. <your eks cluster amp-ingest-irsa role> can be found in AWS IAM, go to Roles section, searhc amp-ingest-irsa and go to the one startin with your cluster name. copy ARN of that role.
    b. <your amp remote write endpoint> can be found in Amazon Opensearch service, go to opensearch-demo domain and copy Domain endpoint 
- Deploy a test app by running kubectl apply -f prometheus-sample-app.yaml
    
- setup grafana


## Destroy

To teardown and remove the resources created in this example:

```sh
terraform destroy -auto-approve
```
