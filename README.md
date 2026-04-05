# Day-57-Resource-Requests-Limits-and-Probes

# Task

Your Pods are running, but Kubernetes has no idea how much CPU or memory they need — and no way to tell if they are actually healthy. Today you set resource requests and limits for smart scheduling, then add probes so Kubernetes can detect and recover from failures automatically.

# Expected Output

- A Pod with CPU and memory requests and limits
- OOMKilled observed when exceeding memory limits
- Liveness, readiness, and startup probes tested
- A markdown file: day-57-resources-probes.md

# Challenge Tasks

# Task 1: Resource Requests and Limits

1. Write a Pod manifest with resources.requests (cpu: 100m, memory: 128Mi) and resources.limits (cpu: 250m, memory: 256Mi)
2. Apply and inspect with kubectl describe pod — look for the Requests, Limits, and QoS Class sections
3. Since requests and limits differ, the QoS class is Burstable. If equal, it would be Guaranteed. If missing, BestEffort.

CPU is in millicores: 100m = 0.1 CPU. Memory is in mebibytes: 128Mi.

<img width="1152" height="902" alt="Image" src="https://github.com/user-attachments/assets/9407aa31-f672-45a0-b544-d92e04c3ad66" />



# Task 2: OOMKilled — Exceeding Memory Limits

1. Write a Pod manifest using the polinux/stress image with a memory limit of 100Mi
2. Set the stress command to allocate 200M of memory: command: ["stress"] args: ["--vm", "1", "--vm-bytes", "200M", "--vm-       hang", "1"]
3. Apply and watch — the container gets killed immediately

CPU is throttled when over limit. Memory is killed — no mercy.

Check kubectl describe pod for Reason: OOMKilled and Exit Code: 137 (128 + SIGKILL).

<img width="1014" height="246" alt="Image" src="https://github.com/user-attachments/assets/968c2e3c-79c2-4081-b220-a81803a0ec57" />

<img width="980" height="860" alt="Image" src="https://github.com/user-attachments/assets/bcaf8691-70ca-4145-8f6d-d586ddccf908" />



# Task 3: Pending Pod — Requesting Too Much

1. Write a Pod manifest requesting cpu: 100 and memory: 128Gi
2. Apply and check — STATUS stays Pending forever
3. Run kubectl describe pod and read the Events — the scheduler says exactly why: insufficient resources

<img width="994" height="964" alt="Image" src="https://github.com/user-attachments/assets/96ac7933-3d32-4533-9554-dff8dbe0b196" />



# Task 4: Liveness Probe

A liveness probe detects stuck containers. If it fails, Kubernetes restarts the container.

1. Write a Pod manifest with a busybox container that creates /tmp/healthy on startup, then deletes it after 30 seconds
2. Add a liveness probe using exec that runs cat /tmp/healthy, with periodSeconds: 5 and failureThreshold: 3
3. After the file is deleted, 3 consecutive failures trigger a restart. Watch with kubectl get pod -w



# Task 5: Startup Probe

A startup probe gives slow-starting containers extra time. While it runs, liveness and readiness probes are disabled.

1. Write a Pod manifest where the container takes 20 seconds to start (e.g., sleep 20 && touch /tmp/started)
2. Add a startupProbe checking for /tmp/started with periodSeconds: 5 and failureThreshold: 12 (60 second budget)
3. Add a livenessProbe that checks the same file — it only kicks in after startup succeeds



<img width="1006" height="985" alt="Image" src="https://github.com/user-attachments/assets/9dab4e53-5eb3-40c8-a959-eacd72b622fb" />

<img width="961" height="994" alt="Image" src="https://github.com/user-attachments/assets/e12e60ee-581a-46da-932c-634314f6b501" />
