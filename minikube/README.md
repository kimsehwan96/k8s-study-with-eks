# Minikube (macos)

`brew install minikube` 

`minkube start --driver docker`

`cat ~/.kube/config`

```config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/gimsehwan/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Thu, 17 Feb 2022 22:14:10 KST
        provider: minikube.sigs.k8s.io
        version: v1.25.1
      name: cluster_info
    server: https://127.0.0.1:51754
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Thu, 17 Feb 2022 22:14:10 KST
        provider: minikube.sigs.k8s.io
        version: v1.25.1
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /Users/gimsehwan/.minikube/profiles/minikube/client.crt
    client-key: /Users/gimsehwan/.minikube/profiles/minikube/client.key
```

