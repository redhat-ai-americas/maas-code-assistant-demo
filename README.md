# Code Assistant Demo with MaaS

## Pre-reqs

A cluster with OpenShift AI 3.4.2 with MaaS and observability setup.

See [ai-accelerator](https://github.com/redhat-ai-services/ai-accelerator) as one option to configure your cluster.

An NVIDIA GPU such as an L40S with minimum 48Gb of vRAM.

## Setup

Deploy the model-server namespace:

```
oc apply -f manifests/llm/namespace.yaml
```

Deploy the model server:

```
oc apply -f manifests/llm/llminferenceservice.yaml
```

Wait for the model server pod to become available (this will take some time)

```
watch oc get pods -n model-server
```

Create the subscription and auth policies:

```
oc apply -f manifests/llm/subscription.yaml
```

## Demo

Log into the OpenShift AI dashboard and navigate to `Gen AI studio` > `API keys`.

Use the option to `Create API key` and copy the value.

From cursor, open the prompt menu click the model dropdown menu, flip the switch for Auto off if it is currently on.  Select the option to `Add models`.

Under `OpenAI API Key` enable to option, and select the option for `Override OpenAI Base URL`.

Enter the base URL such as the following:

```
https://maas.apps.cluster-85d49.85d49.sandbox5181.opentlc.com/model-server/gemma-4-31b-it-nvfp4/v1
```

Enter the API key you got from MaaS.

Click `More Models` in the models menu and select `Add Custom Model`.

Enter the following model name:

```
gemma-4-31b-it-nvfp4
```

## Prompts

Code Gen Prompts:

Create new project:

```
Create a new python project using uv.
It should be a simple fastapi application.
It should include a simple hello world endpoint.
Be sure to include a containerfile to build a container image.
Include tooling like ruff and be sure to configure a pre-commit hook for it.
Additionally, setup some initial tests with pytest.

Additionally, create a .gitignore file and init a new git repo.
After the project is setup, be sure to execute the tests to make sure they are passing and provide a command to start the web server in my local environment.
```

Create a helm chart:
```
Create a helm chart to deploy the application on openshift. Be sure to use a route instead of ingress.
```


Lightspeed Prompt:

```
Check the status of the DataScienceCluster object and report any errors with recommended fixes.
```

```
Investigate errors in the logs with the pod lightspeed-app-server in the openshift-lightspeed namespace and report any errors that you find, along with a recommended fix.
```

## Known Issues

### Cluster Observability Operator

The Cluster Observability Operator has known issues with OpenShift AI 3.4.2 which requires the user to deploy the 1.4.x version of Cluster Observability.  1.5.x versions will not be able to fully deploy with OpenShift AI 3.4.2.

### No Usage Data for MaaS

3.4.2 has a known issue that causes no data to be reported in the Observability dashboard.

As a work around run the following:

```
# 1. Scale down controller
oc scale deploy maas-controller -n redhat-ods-applications --replicas=0

# 2. Delete and recreate TelemetryPolicy without broken labels
oc delete telemetrypolicy maas-telemetry -n openshift-ingress
oc apply -f - <<'EOF'
apiVersion: extensions.kuadrant.io/v1alpha1
kind: TelemetryPolicy
metadata:
    name: maas-telemetry
    namespace: openshift-ingress
labels:
    app.kubernetes.io/part-of: maas-observability
    maas.opendatahub.io/tenant-name: default-tenant
    maas.opendatahub.io/tenant-namespace: models-as-a-service
spec:
    targetRef:
        group: gateway.networking.k8s.io
        kind: Gateway
        name: maas-default-gateway
    metrics:
        default:
        labels:
            subscription: auth.identity.selected_subscription
            user: auth.identity.userid
            model: responseBodyJSON("/model")
EOF

# 3. Restart gateway
oc delete pod -n openshift-ingress -l gateway.networking.k8s.io/gateway-name=maas-default-gateway
```

This issue should be resolved in 3.4.4.
