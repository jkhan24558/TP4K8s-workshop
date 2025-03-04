# Platform Engineer Lab Prerequisites

## Workshop Owner pre-requisites
- vSphere environment(s) with vCenter v8.0.2c (8.0.2.00300) or newer.
    - Workload Management enabled with AVI
    - Supervisor cluster v1.27.5+vmware.1-fips.1 or newer.
    - ContentLibrary synced with TKr 1.28.8 available (1.29.4 should work too)
    - Compute capacity for one TKG clusters with 1 CP node and 3 worker nodes with 4VCPU & 16GBRAM per VM per attendee
    - At least 4 VM IPs available for all those k8s nodes per attendee
    - At least 3 AVI VIPs available per attendee: 1 VIP for the cluster CP, 2 VIP for ingress per cluster
    - Supervisor Namespace created with Storage Policy and VM Classes associated to it
- AWS Route53 Zone
    - Domain registered/delegated to Route53 zone
- Tanzu Platform for Kubernetes Org + Group + Project
    - AWS Account, EKS Credential and GSLB Credential configured
    - AWS EKS Overflow Clusters (labeled with `workshop-overflow: true`)
    - AT for Overflow Clusters
- Onboard all attendees on designated Tanzu Platform for k8s CSP org and project

## Workshop Attendee pre-requisites
- kubectl CLI v1.30.3 installed in workstation - [kubectl CLI install guide](https://kubernetes.io/docs/reference/kubectl/kubectl/)
- tanzu CLI v1.5.1 installed in workstation - [tanzu CLI installation guide](https://techdocs.broadcom.com/us/en/vmware-tanzu/cli/tanzu-cli/1-5/cli/index.html)
- tanzu CLI plugins for platform engineer installaed - [tanzu CLI PE plugins installation guide](https://techdocs.broadcom.com/us/en/vmware-tanzu/platform/tanzu-platform/saas/tnz-platform/spaces-getting-started-create-app-envmt.html)
```
tanzu plugin install --group vmware-tanzu/platform-engineer
```
