# standalone-fluentd
1. oc create -f template.yaml

2. Clean up old instances if any: 
OLD=standalone-fluentd; for object in dc bc is cm service; do oc delete $object/$OLD ; done

3. oc new-app standalone-fluentd (or use UI)

4. oc create secret generic my-credentials --from-file=.dockerconfigjson=$HOME/.docker/config.json --type=kubernetes.io/dockerconfigjson --namespace=openshift-logging ; oc import-image --from=registry.redhat.io/openshift3/ose-logging-fluentd:v3.11.98 openshift/ose-logging-fluentd --confirm



# Steps for the keys:

1. oc annotate service standalone-fluentd-splunk service.aplha.openshift.io/serving-sort-secret-name=standlaone-fluentd-splunk-tls

 
2. oc get secrets standalone-fluentd-splunk-tls -o go-template --template='{{index .data "tls.key"}}' | base64 -d > tls.key

3. openssl rsa -des -in tls.key -out tls1.key

writing RSA key

Enter PEM pass phrase:

Verifying - Enter PEM pass phrase:

4. oc patch secret standalone-fluentd-splunk-tls -p='{"data":{"tls.key": "'$(base64 -w 0 < tls1.key)'"}}'

secret/standalone-fluentd-splunk-tls patched
5. oc get secrets standalone-fluentd-splunk-tls -o go-template --template='{{index .data "tls.crt"}}' > tls.crt.64

6. base64 -d tls.crt.64 > tls.crt

7. oc patch secrets/logging-fluentd --type=json --patch "[{'op':'add','path':'/data/tls.crt','value':'$(base64 -w 0 tls.crt)'}]"

secret/logging-fluentd patched
8. oc patch secrets/logging-fluentd --type=json --patch "[{'op':'add','path':'/data/tls.key','value':'$(base64 -w 0 tls1.key)'}]"

secret/logging-fluentd patched

 
