# Labels, Selectors, Annotations

## nodeSelector

Assign a pod to a specific node with the **nodeSelector** resource. In this example, the node must have the label `nodeName=test`

    apiVersion: v1
    kind: Pod
    metadata:
      name: nodetest
    spec:
      containers:
      - name: n1
        image: nginx
      nodeSelector:
        nodeName: test

## If your node isn't labeled, you can follow these steps to test the YAML above:

Get the node name you want to label:

    kk get nodes

Label the node with `nodeName=test` Careful with labeling. YAML treats numbers and true/false as truthy/falsy values similar to Python. Thus, if you're labeling them with numbers, remember to use quotes (e.g. - `version: "2"`). 

    kk label node <your-node-name> nodeName=test

## Label pods

For a more comprehensive list of examples:

    kk label -h | head -27

 Let's label the pod we created above.

    kk label pod nodetest name=shannon

Let's confirm it's labeled correctly. Either of these commands below will work.

    kk get pods -l name=shannon
    kk get pods --selector=name=shannon

Now let's remove the label.

    kk label pod nodetest name-

Let's make sure that label has been removed.

    kk get pods --show-labels

## Annotations

Both annotations and labels are defined in the `metadata` section of a Pod. Labels are used to loosely couple applications. In contrast, **annotations** **are not** used to loosely define any coupling of applications or identifying and selecting objects. They can be used in a declarative manner, such as version control or timestamping of a Pod. However, you cannot use the `kubectl` CLI to find or query a Pod labeled with a specific annotation.

    apiVersion: v1
    kind: Pod
    metadata:
      annotations:
        release_version: "0.1.12"
        date: "some date here"
      labels:
        tier: backend
      name: example
    spec:
      containers:
      - image: nginx
        name: n1

You can, however, use `kubectl` to annotate something imperatively. Take a peek at `kk annotate -h` for some practical examples on this.
