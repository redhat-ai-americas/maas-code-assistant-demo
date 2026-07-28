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
