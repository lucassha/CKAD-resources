# ServiceAccounts

[https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

When you (a human) access the cluster (for example, using kubectl), you are authenticated by the apiserver as a particular User Account (currently this is usually admin, unless your cluster administrator has customized your cluster). Processes in containers inside pods can also contact the apiserver. When they do, they are authenticated as a particular Service Account (for example, default).

Can't really say that better than above. Basically, the apiserver needs authentication, and you're usually running as admin but individual users can be added with a serviceAccount. This is something you would implement when having many users on a cluster with many namespaces.

Let's take a look at the yaml before actually creating it 

    kk create sa mysa --dry-run -o yaml
    ---
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      creationTimestamp: null
      name: mysa

Looks pretty straightforward. Let's build it now without the `—dry-run` portion.

    kk create sa mysa

Then add it to a Pod's spec. pod.spec.serviceAccount is deprecated, so we'll use **serviceAccountName** instead

    apiVersion: v1
    kind: Pod
    metadata:
      name: p1
    spec:
      serviceAccountName: mysa
      containers:
      - image: nginx
        name: c1
