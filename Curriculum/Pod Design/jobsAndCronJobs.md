# Jobs and CronJobs

## Job

Both of these are exceptionally painful to write from scratch in yaml, so we're going to use `kk run` as a dry-run to get the base layout for us. Strictly speaking, jobs and cron jobs are pretty straight forward. They create a pod and can run at a specific time, in parallel, etc. I'll walk through one and then add some specs to it.

Create a job named j1 that prints out the image's environment variables.

    kk run j1 --image=busybox --restart=OnFailure --command \
    --dry-run -o yaml -- printenv > j1.yaml

A couple notes about this:

With `kk run` setting `—restart=OnFailure` will turn this into a job. Very easy for a quick yaml job.

The `—command` extension will just add `printenv` under `command` and not `args`. Either should work fine, however. In Kubernetes, `command` == shell commands and `args` == arguments passed to the command. Generally, it seems both will work fine though.

I'm going to strip the yaml of unnecessary bits, so it's easier to read when we add a couple features.

    apiVersion: batch/v1
    kind: Job
    metadata:
      labels:
        run: j1
      name: j1
    spec:
      template:
        metadata:
          labels:
            run: j1
        spec:
          containers:
          - command:
            - printenv
            image: busybox
            name: j1
          restartPolicy: OnFailure

After running this job, you should see a pod, so let's check that and practice narrowing it down with a label.

    kk get po -l run

Now check the logs for that pod to confirm the environment variables were printed to STDOUT

    kk logs <j1-your-pod's-unique-hash>

We'll add a few features to the spec of the job now. If you want to see all the options available to you

    kk explain job.spec

We'll add 3 additional specs

1. Completions:  declares how many times to run the job. 
2. Parallelism:  declares how many pods can run at the same time
3. activeDeadlineSeconds:  declares how long the job can run

```
apiVersion: batch/v1
kind: Job
metadata:
  labels:
    run: j1
  name: j1
spec:
  completions: 10
  parallelism: 2
  activeDeadlineSeconds: 15
  template:
    metadata:
      labels:
        run: j1
    spec:
      containers:
      - command:
        - printenv
        image: busybox
        name: j1
      restartPolicy: OnFailure
```

Since this job is named the same as the previous one you just created, delete it first

    kk delete job --all

Now run the job and check the pods. 

    kk get po

You should see see two pods running at a time, which was determined by the **parallelism** spec. However, we set a **deadline** of only 15 seconds to create 10 pods with only 2 running at a time. That just isn't going to happen. So, let's check out the description of the job and see what it says.

    kk get job
    ---
    NAME   COMPLETIONS   DURATION   AGE
    j1     4/10          116s       116s

So I only had 4 complete out of the 10. Let's also check out the description and see what it says.

    kk describe job j1
    ---
    Warning  DeadlineExceeded  2m8s   job-controller  Job was active longer than specified deadline

So, that makes sense. The Job was stopped because we set a deadline. 

## CronJob

Really not much difference between the 2, as CRON just runs based on the "* * * * *" format you choose. The yaml is even worse, however, so we'll use `kk run` again to make our lives easier

    kk run cj1 --image=busybox --command --dry-run --schedule="*/1 * * * *" \
    -o yaml -- printenv > cj1.yaml

That generates this yaml below, with a few things stripped for readability. 

Only two things to point out, really:

1. `—schedule="*/1 * * * *"` turns this from a into a cronjob
2. If you're making this template yourself, watch out for `spec` as cron jobs have 3 different locations for it. CronJob → Job → Pod

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  labels:
    run: cj1
  name: cj1
spec:
  concurrencyPolicy: Allow
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            run: cj1
        spec:
          containers:
          - command:
            - printenv
            image: busybox
            name: cj1
          restartPolicy: Always
  schedule: '*/1 * * * *'
```
