# k8s-sample-resources

tanzu apps workload create k8s-sample-resources \
--git-repo https://github.com/poprygun/k8s-sample-resources.git \
--type k8s-resource --git-branch master --namespace default \
--label "app.kubernetes.io/part-of=k8s-sample-resources" \
--namespace my-apps

