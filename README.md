# k8s-sample-resources

tanzu apps workload create k8s-sample-resources \
--git-repo git@github.com:poprygun/k8s-sample-resources.git \
--type k8s-resource --git-branch master --namespace default \
--label "app.kubernetes.io/part-of=k8s-sample-resources" \
--tail \
--yes