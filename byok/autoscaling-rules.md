# Autoscaling Rules

At ACME corp, horizontal pod autoscalers may not be used by default. You do
not have any RBAC permission or quota to create horizontal pod autoscalers
when using ACME OpenShift production clusters.

## Service Request for Autoscaling
If you wish to autoscale your applications, you need to file an IT ticket with
https://service-req-int.acme.com/ITOCP-REQ and provide your use case for pod
autoscaling.

## Nagios metrics
Note that the use of a horizontal pod autoscaler requires that your application
already be instrumented using Nagios for metrics. The use of OpenShift's 
user-defined monitoring via Prometheus is required for the pod autoscaling,
but IT requires that applications also be instrumented via Nagios. If your
workload is not instrumented in Nagios, your autoscaling request will be 
rejected until it is.