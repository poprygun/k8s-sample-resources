# Experimental Supply Chain to execute Kubesec Scan

## Setup Instructions

```shell
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.9/git-clone.yaml
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-cli/0.4/git-cli.yaml

kubectl apply  -f tasks/kubesec-scanner.yaml
kubectl apply  -f tasks/list-files.yaml
kubectl apply  -f kubesec-scanner-pipeline.yaml
kubectl apply  -f kubesec-scanner-pipeline-run.yaml
kubectl apply -f kubesec-scanner-cluster-source-template.yaml
kubectl apply -f supply-chain-kubesec-scan.yaml
```

## Supply Chain Customization

### Get a hold of current installed templates

Capture the image listed under ootb-templates

```shell
kubectl get app -n tap-install |grep ootb-
ootb-delivery-basic                  Reconcile succeeded   4m31s          22h
ootb-supply-chain-testing-scanning   Reconcile succeeded   9m57s          22h
ootb-templates                       Reconcile succeeded   9m10s          22h
```

```shell
kubectl get app ootb-supply-chain-testing-scanning \
  -n tap-install \
  -o yaml | yq .spec.fetch
```

Extract templates from image

```shell
imgpkg pull \
  --registry-verify-certs=false \
  -b registry.tanzu.vmware.com/tanzu-application-platform/tap-packages@sha256:d0712e2aa1bf343836f9be4f6cddd80a4abeb41e91639e29753c087cfde6d223 \
  -o ootb-supply-chain-testing-scanning
```

### Get a hold of current installed BASIC templates

Capture the image listed under ootb-supply-chain-basic

```shell
kubectl get app ootb-supply-chain-basic \
  -n tap-install \
  -o yaml | yq .spec.fetch
```

Extract templates from image

```shell
imgpkg pull \
  --registry-verify-certs=false \
  -b harbor.h2o-4-9446.h2o.vmware.com/rpm/harbor.h2o-4-9446.h2o.vmware.com/rpm/tap-packages@sha256:3e1273c6af946c3e2b7a7f33eb87faad24e8b01622884f55185cfaa5ef6c11ae \
  -o ootb-supply-chain-basic
```

Apply changes after modifying the supply chain

```shell
ytt \
  --ignore-unknown-comments \
  --file ./ootb-supply-chain-testing-scanning/config/supply-chain-kubesec-scan.yaml \
  --data-value registry.server=registry.tanzu.vmware.com \
  --data-value registry.repository=tap-packages |
  kubectl apply -n tekton-pipelines -f-
```

### Setup ssh credentials for github

Add the contents of created .ssh/id_rsa_custom.base64 to `PipelineRun.data.id_rsa`

```shell
ssh-keygen -t rsa -b 4096 -C "poprygun@hotmail.com" -f .ssh/id_rsa_custom
openssl enc -base64 -in .ssh/id_rsa_custom -out .ssh/id_rsa_custom.base64
```

Add the contents of created .ssh/ssh_config64.base64 to `PipelineRun.data.known_hosts`

```shell
ssh-keyscan -t rsa github.com >> known_hosts
openssl enc -base64 -in .ssh/known_hosts -out .ssh/known_hosts.base64
```

Create github donfig file as follows and add to `PipelineRun.data.config` after encoding to base64

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
tkn pipelinerun logs kubesec-scanner-pipeline-run 
```

```shell
k logs kubesec-scanner-pipeline-run-fetch-repository-pod 
k logs kubesec-scanner-pipeline-run-kubesec-scanner-run-pod 
k logs kubesec-scanner-pipeline-run-list-scannned-files-pod -c step-list-files 
k logs kubesec-scanner-pipeline-run-list-scannned-files-pod -c step-write-file 
k logs kubesec-scanner-pipeline-run-push-kubesec-results-pod  
k logs kubesec-scanner-pipeline-run-commit-push-pod  
```

### Debugging pipelines execution

```shell
k apply -f kubesec-scanner-pipeline.yaml
k delete -f kubesec-scanner-pipeline.yaml
```

