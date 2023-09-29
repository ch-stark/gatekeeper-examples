# gatekeeper-examples

This example (WIP) will show ACM Gatekeeper Integration


As a prerequisite execute the setup-gitops folder.

It will create an ArgoCD Instance on the Hub-Cluster in a namespace called 'policies' and create ACM-Policies which are wrapping Gatekeeper Policies


The following features are highlighted

- Dependency between policies
1. First Gatekeeper will be installed
2. then an instance will be configured and namespaces excluded
3. we check Gatekeeper is running fine 
4. then we install a Gatekeeper-library
5. then we install Custom Contraint Templates
6. then we install a Custom Contraint

- Policy Templates, e.g. we check only Kubernetes Clusters with a certain version
- Placement.  Gatekeeper-Files will be distributed to all Clusters with a Certain label

All configuration is done in a central file: 

https://github.com/stolostron/policy-generator-plugin/blob/main/docs/policygenerator-reference.yaml



