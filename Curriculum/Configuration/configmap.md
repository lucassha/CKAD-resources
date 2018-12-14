# ConfigMaps

From the k8s definition, "ConfigMaps allow you to decouple configuration artifacts from image content to keep containerized applications portable"

So what are they really? Basically, they're the configuration portion of a microservice where you swap in and out environment variables. Want to point to a specific database, namespace, etc.? You're probably going to be doing that with ConfigMaps, unless it's more sensitive data in which you'll be using secrets.

Here's a quick Node.JS example connecting to a local MongoDB pod

    const uri = 'mongodb://mongo-cluster-ip-service:27017/docker-node';

Now, let's put that in terms of a ConfigMap

    const uri = 'mongodb://' + \
    process.env.DEV_DB_ENDPOINT + ':' + \
    process.env.DEV_DB_PORT + '/' + \
    process.env.DEV_DB_NAME

## Create a ConfigMap

`kk create cm` is the easiest solution for a quick job. As always, use the `-h` flag to get some practical examples.

There are a few different options you can select from, but I'm just going to choose `—from-literal=foo=bar`, as that's the easiest for this.

Your options:

1. `—from-file`
2. `—from-literal`
3. `—from-env-file`

Let's create a configmap called `cm1`

    kk create cm cm1 --from-literal=name=shannon --from-literal=config=map

This created a configmap with two **keys**: name and config. Thus, there are two **values**: shannon and map. 

Just to make sure:

- Name → cm1
- key(name):value(shannon)
- key(config):value(map)

Let's confirm they were created.

    kk get cm

Take a peek at the yaml too, so you can have some self-documenting code later if you want it.

    kk get cm cm1 -o yaml

Let's also confirm the key value pairs are in the configmap. The output format is, in my opinion, a little funky so keep in mind a dash is added between the key and value.

    kk describe cm cm1
    ---
    Name:         cm1
    Namespace:    default
    Labels:       <none>
    Annotations:  <none>
    
    Data
    ====
    config:
    ----
    map
    name:
    ----
    shannon
    Events:  <none>

Now, let's create a pod and add these env variables to them. Use `kk run` for a quick template. I'll be slashing a bit of the details from mine to keep this neater.

    apiVersion: v1
    kind: Pod
    metadata:
      name: p1
    spec:
      containers:
      - command:
        - printenv
        image: busybox
        name: bb1

First, we'll add an individual env variable from the cm and then we'll add all of them. Make sure to delete the pod before running this again unless you've changed the `name` in `metadata`.

    apiVersion: v1
    kind: Pod
    metadata:
      name: p1
    spec:
      containers:
      - command:
        - printenv
        image: busybox
        name: bb1
        env: # need to add everything here and below
          - name: MY_NAME  # the env variable name in your pod
            valueFrom:
              configMapKeyRef:
                name: cm1  # the cm name you created (cm1)
                key: name  # the key you created (name:shannon)

Run the pod and then check to see if the environment variables was added

    kk logs p1
    ---
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    HOSTNAME=p1
    MY_NAME=shannon
    ....

Now let's add all of them instead of just one. Essentially, you just use `envFrom:` instead of `env:`

    apiVersion: v1
    kind: Pod
    metadata:
      name: p1
    spec:
      containers:
      - command:
        - printenv
        image: busybox
        name: bb1
        envFrom: # need to add everything here and below
        - configMapRef:
            name: cm1

Check the logs again to see if both your environment variables are now logged

    kk logs p1
    ---
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    HOSTNAME=p1
    config=map
    name=shannon

## Cleaning up

Delete the cm

    kk delete cm cm1

Let's try to build the pod again and let's check what happens

    kk describe po p1
    ---
    Warning  Failed 4s   kubelet, <node>  Error: configmap "cm1" not found

If a configMap object does not exist, the pod cannot be created. If looking at all pods, a similar error will be generated.

    kk get po 
    ---
    NAME   READY   STATUS                       RESTARTS   AGE
    p1     0/1     CreateContainerConfigError   0          50s
