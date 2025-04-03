# Kubernetes Jobs and CronJobs

Jobs and CronJobs in Kubernetes provide a way to run tasks that execute for a finite duration, as opposed to continuously running workloads like Deployments or StatefulSets.

## What are Jobs?

A Job creates one or more Pods and ensures that a specified number of them successfully terminate. As pods successfully complete, the Job tracks the successful completions. When a specified number of successful completions is reached, the Job itself is complete.

![Job Diagram](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/controllers/job.svg)

## What are CronJobs?

A CronJob is a Job that runs on a recurring schedule, specified using [cron format](https://en.wikipedia.org/wiki/Cron). CronJobs are useful for creating periodic and recurring tasks, like running backups, sending emails, or cleaning up resources.

![CronJob Diagram](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/controllers/cronjob.svg)

## Key Features of Jobs

- **One-Time Execution**: Run tasks to completion rather than indefinitely
- **Parallelism Control**: Run multiple Pods in parallel
- **Completions Management**: Ensure a specific number of Pods complete successfully
- **Automatic Retry**: Restart Pods that fail until the specified completions are achieved
- **Deadlines**: Set a time limit after which the Job is terminated

## Key Features of CronJobs

- **Scheduled Execution**: Run Jobs according to a time-based schedule
- **Concurrency Policy**: Control how concurrent executions are handled
- **History Limits**: Control how many completed and failed Jobs are kept
- **Starting Deadlines**: Define a deadline for starting Jobs if they missed their scheduled time

## When to Use Jobs and CronJobs

### Jobs
- **Batch Processing**: Data processing, file conversions, rendering
- **One-Off Tasks**: Database migrations, setup operations, cleanup
- **Parallel Computations**: Distributed calculations, simulation runs
- **Work Queue Processing**: Processing items from a queue

### CronJobs
- **Scheduled Backups**: Regular database or file system backups
- **Reporting**: Generating and emailing reports on a schedule
- **Maintenance**: Cleanup tasks, index rebuilding, log rotation
- **Alerts and Notifications**: Regular system checks and notifications
- **Data Synchronization**: Periodic data synchronization between systems

## Jobs vs Other Kubernetes Objects

| Feature | Job | CronJob | Deployment | StatefulSet | DaemonSet |
|---------|-----|---------|------------|-------------|-----------|
| Purpose | Run to completion | Scheduled tasks | Long-running apps | Stateful apps | Node-level services |
| Execution | One-time/finite | Recurring | Continuous | Continuous | Continuous |
| Pod Lifecycle | Terminated when done | Terminated when done | Kept running | Kept running | Kept running |
| Scaling | Fixed number of completions | N/A | Manual/auto | Manual | Per-node |
| Use Case | Batch processing | Scheduled tasks | Stateless apps | Stateful apps | Node services |

## Job Specification

A basic Job YAML looks like this:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-calculator
spec:
  completions: 3          # Number of successful pod completions required
  parallelism: 2          # Number of pods to run in parallel
  backoffLimit: 4         # Number of retries before marking job as failed
  activeDeadlineSeconds: 300  # Time limit for job execution (5 minutes)
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34.0
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
        resources:
          requests:
            cpu: 100m
            memory: 50Mi
          limits:
            cpu: 500m
            memory: 100Mi
      restartPolicy: Never  # Important for Jobs: OnFailure or Never
```

### Key Fields in Job Spec

- **completions**: Number of successful pod completions required
- **parallelism**: Number of pods to run in parallel
- **backoffLimit**: Number of retries before marking the job as failed
- **activeDeadlineSeconds**: Time limit for the job execution
- **template**: Pod template for the job, similar to other controllers
- **restartPolicy**: Must be OnFailure or Never (not Always)

## CronJob Specification

A basic CronJob YAML looks like this:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: database-backup
spec:
  schedule: "0 2 * * *"     # Every day at 2:00 AM
  concurrencyPolicy: Forbid # Don't allow concurrent executions
  successfulJobsHistoryLimit: 3    # Keep 3 successful jobs in history
  failedJobsHistoryLimit: 1        # Keep 1 failed job in history
  startingDeadlineSeconds: 120     # Must start within 2 minutes of scheduled time
  jobTemplate:
    spec:
      backoffLimit: 2            # Retry failed pods twice
      activeDeadlineSeconds: 600 # Time limit (10 minutes)
      template:
        spec:
          containers:
          - name: backup
            image: postgres:14
            command: ["/bin/sh", "-c", "pg_dump > /backup/db-$(date +%Y-%m-%d).sql"]
            resources:
              requests:
                cpu: 200m
                memory: 200Mi
              limits:
                cpu: 500m
                memory: 500Mi
            volumeMounts:
            - name: backup-volume
              mountPath: /backup
          volumes:
          - name: backup-volume
            persistentVolumeClaim:
              claimName: backup-pvc
          restartPolicy: OnFailure
```

### Key Fields in CronJob Spec

- **schedule**: Cron-formatted schedule (`* * * * *` = minute, hour, day of month, month, day of week)
- **concurrencyPolicy**: How to handle concurrent executions (Allow, Forbid, or Replace)
- **suspend**: If true, no new jobs will be created
- **successfulJobsHistoryLimit**: How many successful jobs to keep in history
- **failedJobsHistoryLimit**: How many failed jobs to keep in history
- **startingDeadlineSeconds**: Deadline for starting jobs that missed their scheduled time
- **jobTemplate**: Template for the Job to run

## Cron Schedule Syntax

The schedule is specified using the standard cron syntax:

```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday = 0)
# │ │ │ │ │
# * * * * *
```

Common examples:
- `0 * * * *` - Every hour at minute 0
- `0 0 * * *` - Every day at midnight
- `0 0 * * 0` - Every Sunday at midnight
- `*/10 * * * *` - Every 10 minutes
- `0 2 * * 1-5` - At 2:00 AM, Monday through Friday

## Working with Jobs

### Creating a Job

```bash
kubectl apply -f job.yaml
```

### Checking Job Status

```bash
# Get all jobs
kubectl get jobs

# Get details about a specific job
kubectl describe job pi-calculator

# Get pods created by a job
kubectl get pods --selector=job-name=pi-calculator
```

### Viewing Job Results

```bash
# Get logs from a specific pod
kubectl logs pods/pi-calculator-x6zn9
```

### Deleting a Job

```bash
kubectl delete job pi-calculator
```

## Working with CronJobs

### Creating a CronJob

```bash
kubectl apply -f cronjob.yaml
```

### Checking CronJob Status

```bash
# List all cronjobs
kubectl get cronjobs

# Get details about a specific cronjob
kubectl describe cronjob database-backup

# Get jobs created by a cronjob
kubectl get jobs --selector=cronjob-name=database-backup
```

### Manually Triggering a CronJob

```bash
# Create a job from a cronjob
kubectl create job --from=cronjob/database-backup manual-backup
```

### Suspending and Resuming a CronJob

```bash
# Suspend a cronjob
kubectl patch cronjob database-backup -p '{"spec":{"suspend":true}}'

# Resume a cronjob
kubectl patch cronjob database-backup -p '{"spec":{"suspend":false}}'
```

### Deleting a CronJob

```bash
kubectl delete cronjob database-backup
```

## Job Patterns

### Work Queue Jobs

Processing items from a work queue:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: queue-processor
spec:
  parallelism: 5           # Run 5 pods in parallel
  template:
    spec:
      containers:
      - name: queue-worker
        image: my-queue-worker:latest
        env:
        - name: QUEUE_URL
          value: "redis://redis-service:6379/0"
      restartPolicy: OnFailure
```

### Indexed Jobs

Running jobs with index awareness (useful for data sharding):

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: indexed-job
spec:
  completions: 5
  parallelism: 5
  completionMode: Indexed  # Each pod gets a unique index
  template:
    spec:
      containers:
      - name: indexed-processor
        image: my-processor:latest
        env:
        - name: JOB_COMPLETION_INDEX  # Kubernetes injects this automatically
          valueFrom:
            fieldRef:
              fieldPath: metadata.annotations['batch.kubernetes.io/job-completion-index']
      restartPolicy: Never
```

## Advanced Job Features

### Time-Based Deadlines

Ensuring Jobs complete within a time limit:

```yaml
spec:
  activeDeadlineSeconds: 600  # 10 minutes
```

### Automatic Cleanup

Setting Time-to-Live (TTL) for jobs after completion:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-ttl
spec:
  ttlSecondsAfterFinished: 100  # Delete 100 seconds after completion
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

### Pod Failure Policy

Control how job handles different types of pod failures:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-with-pod-failure-policy
spec:
  backoffLimit: 6
  podFailurePolicy:
    rules:
    - action: FailJob
      onExitCodes:
        containerName: main
        operator: In
        values: [1, 2, 3]
    - action: Ignore
      onPodConditions:
      - type: DisruptionTarget
  template:
    spec:
      containers:
      - name: main
        image: busybox
        command: ["sh", "-c", "sleep 5; exit 0"]
      restartPolicy: Never
```

## CronJob Best Practices

### Concurrency Policies

Controlling how concurrent executions are handled:

- **Allow**: Default. Allows concurrent jobs to run
- **Forbid**: Skips the new job if the previous job is still running
- **Replace**: Cancels the currently running job and starts a new one

```yaml
spec:
  concurrencyPolicy: Forbid
```

### History Limits

Control how many completed and failed Jobs are kept:

```yaml
spec:
  successfulJobsHistoryLimit: 3  # Keep only 3 successful jobs
  failedJobsHistoryLimit: 1      # Keep only 1 failed job
```

### Starting Deadlines

Handle missed schedules:

```yaml
spec:
  startingDeadlineSeconds: 180  # Must start within 3 minutes of scheduled time
```

## Common Issues and Troubleshooting

### Jobs Not Completing

1. **Pod failures**:
   - Check pod logs: `kubectl logs <pod-name>`
   - Check pod status: `kubectl describe pod <pod-name>`

2. **Insufficient resources**:
   - Check for resource constraints: `kubectl describe node <node-name>`

3. **Deadlock or infinite loop**:
   - Use `activeDeadlineSeconds` to ensure jobs terminate eventually

### CronJobs Not Running

1. **Schedule issues**:
   - Check if your cron syntax is correct
   - Verify timezone issues (Kubernetes uses UTC)

2. **Missed schedules**:
   - Check if `startingDeadlineSeconds` is too small
   - Look for issues in previous job runs

3. **Suspended CronJobs**:
   - Check if the CronJob is suspended: `kubectl get cronjob <name> -o=jsonpath='{.spec.suspend}'`

## Best Practices

1. **Idempotency**: Ensure jobs can be run multiple times safely

2. **Resource Limits**: Always set CPU and memory limits for your job pods

3. **Retry Limits**: Set appropriate `backoffLimit` to prevent infinite retries

4. **Cleanup**: Use `ttlSecondsAfterFinished` to automatically clean up completed jobs

5. **Monitoring**: Set up monitoring for job completion and failure rates

6. **CronJob Scheduling**: Avoid scheduling many CronJobs at the exact same time

7. **History Limits**: Set appropriate history limits to control resource usage

8. **Timezone Awareness**: Remember that CronJob schedules are in UTC

## Job and CronJob Examples

See the [examples](./examples/) directory for sample Job and CronJob configurations.

## Next Steps

- Learn about [ConfigMaps and Secrets](../08-configmaps-secrets/) for configuration
- Explore [Persistent Volumes](../09-persistent-volumes/) for persistent storage
- Study [Resource Management](../11-resource-management/) for CPU and memory control 