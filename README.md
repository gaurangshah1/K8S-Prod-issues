📘 Scenario #1: Zombie Pods Causing NodeDrain to Hang
Category: Cluster Management
Environment: K8s v1.23, On-prem bare metal, Systemd cgroups
Scenario Summary: Node drain stuck indefinitely due to unresponsive terminating pod.
What Happened: A pod with a custom finalizer never completed termination, blocking kubectl drain. Even after the pod was marked for deletion, the API server kept waiting because the finalizer wasn’t removed.
Diagnosis Steps:
	• Checked kubectl get pods --all-namespaces -o wide to find lingering pods.
	• Found pod stuck in Terminating state for over 20 minutes.
	• Used kubectl describe pod <pod> to identify the presence of a custom finalizer.
	• Investigated controller logs managing the finalizer – the controller had crashed.
Root Cause: Finalizer logic was never executed because its controller was down, leaving the pod undeletable.
Fix/Workaround:
kubectl patch pod <pod-name> -p '{"metadata":{"finalizers":[]}}' --type=merge
Lessons Learned: Finalizers should have timeout or fail-safe logic.
How to Avoid:
	• Avoid finalizers unless absolutely necessary.
	• Add monitoring for stuck Terminating pods.
	• Implement retry/timeout logic in finalizer controllers.
