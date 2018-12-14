# SecurityContexts

Container security should be considered in any type of environment with sensitive data, and there a number of considerations to be taken into account. Is the container running as privileged, is there a PID safety net (aka not running at PID1 - init), what's the UID the container is running as? It's worth taking a jaunt through common security practices in Docker too when looking at this stuff too.

In this case, the k8s [documentation](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) has pretty much everything you need to build a Pod with securityContext specs. 

There are two different securityContext sections that can be added. 

1. Pod level - `kk explain pod.spec.securityContext`
2. Container level - `kk explain pod.spec.containers.securityContext`

Let's make a pod where both the user id and group filesystem are different in each and see what happens. 

    apiVersion: v1
    kind: Pod
    metadata:
      name: p1
    spec:
      securityContext:
        runAsUser: 1500
        fsGroup: 2000
      containers:
      - image: busybox
        args:
        - /bin/sh
        - -c
        - sleep 3600
        name: p1
        securityContext:
          runAsUser: 1000

Once the pod is up and running, let's `exec` into it and check out the id and group.

    kk exec -it p1 -- sh

You'll notice one thing right off the bat. You are no longer running as root, as indicated by the `$` instead of `#`

Type in `id` and you should get the following details:

    uid=1000 gid=0(root) groups=2000

Based off this, we can see that **container** securityContext takes precedence over **pod** securityContext. By doing this, you're adding additional layers of protection between your host OS and the running container. This is implemented with the user namespace from your kernel.

Be careful with `GID` manipulation. I believe some areas require a specific GID of 2000 and this can cause potential issues. I'll come back to this if I learn more about it.
