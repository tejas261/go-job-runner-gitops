# Disposable AKS sizing validation

Uses the existing job-runner Helm chart, Argo CD automated sync/prune/self-heal, and existing pinned kube-prometheus-stack application. Only this experiment branch is watched by the test Applications; main is unchanged.

Three isolated namespaces each get one app replica and a disposable PostgreSQL database. Separate databases prevent multiple LISTEN/NOTIFY consumers from processing the same jobs. Control: 20m/32Mi requests, 250m/128Mi limits. Oversized: 1000m/1Gi requests. Undersized: 8Mi memory limit. The low memory limit deliberately risks OOMKilled only in the test namespace. Databases use bounded emptyDir storage and contain generated test jobs only.

Fresh cluster-specific SealedSecrets provide app/database credentials; no original database credentials are reused. The AKS kubelet uses AcrPull on the temporary private registry. No service uses LoadBalancer ingress. Access is through localhost port forwarding.

Bootstrap Argo CD and Sealed Secrets, apply the sealed secrets and Applications, and verify Argo reports the expected Git revision. Controlled traffic must target local in-cluster endpoints only. Export sanitized metrics and analyser findings, then delete both Azure test resource groups and registry after validation.

Changing requests is not itself a cloud saving. Short runs establish detection integration, not production sizing, business SLOs or multiweek savings.

The single-node test has a 30-pod limit. Argo CD ApplicationSet, Dex and notifications controllers are scaled to zero (plain Applications and local access only). The monitoring override disables Alertmanager and node-exporter while retaining Prometheus, kube-state-metrics, kubelet/container metrics and Grafana. These are experiment-only resource constraints.
