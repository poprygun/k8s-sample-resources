# Experimental Supply Chain to execute Kubesec Scan

## Prerequisites

- Kubernetes Cluster

- Tekton Pipelines CRD installed

- Cartographer CRD installed

## Setup Instructions

- Provision Tekton tasks

```shell
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.9/git-clone.yaml
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-cli/0.4/git-cli.yaml
kubectl apply -f tasks/kubesec-scanner.yaml
```

- Provision the tekton pipeline and Cartographer templates

```shell
kubectl apply -f kubesec-scanner-pipeline.yaml
kubectl apply -f kubesec-scanner-cluster-source-template.yaml
kubectl apply -f supply-chain-kubesec-scan.yaml
```

### Setup ssh credentials for github

`git-cli` tekton task will need to authenticate with Github via ssh

- Add the contents of created .ssh/id_rsa_custom.base64 to `PipelineRun.data.id_rsa`

```shell
ssh-keygen -t rsa -b 4096 -C "poprygun@hotmail.com" -f .ssh/id_rsa_custom
openssl enc -base64 -in .ssh/id_rsa_custom -out .ssh/id_rsa_custom.base64
```

- Add the contents of created .ssh/ssh_config64.base64 to `PipelineRun.data.known_hosts`


```shell
ssh-keyscan -t rsa github.com >> known_hosts
openssl enc -base64 -in .ssh/known_hosts -out .ssh/known_hosts.base64
```

- Create github config file as follows and add to `PipelineRun.data.config` after encoding to base64

```yaml
Host github.com
  AddKeysToAgent yes
  IdentityFile /root/.ssh/id_rsa
```

```shell
openssl enc -base64 -in .ssh/config -out .ssh/config.base64
```

### Monitor pipeline run execution.


```shell
kubectl logs kubesec-scanner-pipeline-run-fetch-repository-pod 
kubectl logs kubesec-scanner-pipeline-run-kubesec-scanner-run-pod 
kubectl logs kubesec-scanner-pipeline-run-commit-push-pod  
```

### Debugging pipelines execution

```shell
k apply -f kubesec-scanner-pipeline.yaml
k delete -f kubesec-scanner-pipeline.yaml
```





# k8s-sample-resources

tanzu apps workload create k8s-sample-resources \
--git-repo git@github.com:poprygun/k8s-sample-resources.git \
--type k8s-resource --git-branch master --namespace default \
--label "app.kubernetes.io/part-of=k8s-sample-resources" \
--tail \
--yes