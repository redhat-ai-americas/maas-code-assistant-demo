Run the following to generate an API key and create a secret for Lightspeed.

```
oc create serviceaccount lightspeed -n openshift-lightspeed
SA_TOKEN=$(oc create token lightspeed -n openshift-lightspeed --duration=1h)
MAAS_GATEWAY=$(oc get route maas-gateway-route -n openshift-ingress -o jsonpath='{.spec.host}')
API_KEY=$(
  curl -sk -X POST "https://$MAAS_GATEWAY/maas-api/v1/api-keys" \
    -H "Authorization: Bearer $SA_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"name":"sa-test-key","subscription":"demo"}' \
  | jq -r '.key'
)
oc create secret generic openshift-ai-credentials -n openshift-lightspeed \
  --from-literal=apitoken="$API_KEY"
```
