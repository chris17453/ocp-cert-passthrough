# OCP SSL passthrough for APPS

```

login to cloud shell for the ROKS cluster. (from ocp console, see 9-box icon and click "Manage Cluster" link)
obtain oc login token and login to ocp console

oc extract secret/$(oc get route console -n openshift-console -o jsonpath="{.spec.host}" | awk -F'.' '{print $2}') -n ibm-cert-store --confirm --to=./
awk 'BEGIN {c=0;} /BEGIN CERT/{c++} { print > "cert-" c ".crt"}' < tls.crt
oc delete certificate route-cert
oc delete secret route-tls-secret
oc create secret generic route-tls-secret --from-file=ca.crt=./cert-2.crt  --from-file=tls.crt=./cert-1.crt  --from-file=tls.key=./tls.key
oc patch AutomationUIConfig iaf-system --type merge --patch '{"spec":{"tls":{"issuerRef":{"name":""}}}}'
oc delete certificate iaf-system-automationui-aui-zen-ca
oc delete certificate iaf-system-automationui-aui-zen-cert
oc delete secret external-tls-secret
oc create secret generic external-tls-secret --from-file=cert.crt=./tls.crt  --from-file=cert.key=./tls.key
oc delete pod -l app=auth-idp
oc delete pod -l component=ibm-nginx
oc delete route cpd
oc create route passthrough cpd --service=ibm-nginx-svc --port=ibm-nginx-https-port
```
