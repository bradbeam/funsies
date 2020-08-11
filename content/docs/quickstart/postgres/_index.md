# Postgres

The purpose of this guide is to get you started with Red Sky Ops and Postgres.
We will cover
- Deploying a postgres to Kubernetes
- Exploring the postgres experiment
- Running trials to determine the best configuration

## Prerequisites

- Kubernetes cluster ( [minikube][minikube] cluster will be sufficient )
- [kubectl][kubectl] properly configured for your cluster
- [redskyctl][redskyctl]
- [kustomize][kustomize] v3.1.0+

## Initialize the Red Sky Ops Manager

Log in to your Red Sky Ops account:
```sh
$ redskyctl login
Opening your default browser to visit:

	https://auth.carbonrelay.io/authorize?audience=https%3A%2F%2Fapi.carbonrelay.io%2Fv1%2F&client_id=QJHVshlUUQJ40Rml7CUJDJVmRLHhE04Y&code_challenge=5JstkywCbFCwSC4qarX8WC_Uq_6ktsn1g28aY7ZtTDs&code_challenge_method=S256&redirect_uri=http%3A%2F%2F127.0.0.1%3A8085%2F&response_type=code&scope=register%3Aclients+offline_access&state=S-t7HVXShlUUSel6KAWMIQ

You are now logged in.
```

Initialize the manager in your cluster:

```sh
$ redskyctl init
namespace/redsky-system created
customresourcedefinition.apiextensions.k8s.io/experiments.redskyops.dev created
customresourcedefinition.apiextensions.k8s.io/trials.redskyops.dev created
clusterrole.rbac.authorization.k8s.io/redsky-manager-role created
clusterrolebinding.rbac.authorization.k8s.io/redsky-manager-rolebinding created
deployment.apps/redsky-controller-manager created
clusterrole.rbac.authorization.k8s.io/redsky-patching-role created
clusterrolebinding.rbac.authorization.k8s.io/redsky-patching-rolebinding created
secret/redsky-manager created
```

Verify manager is running:

```sh
$ kubectl wait --for condition=Ready=true po -n redsky-system -l app.kubernetes.io/name=redskyops
pod/redsky-controller-manager-5fb9f4cd4d-g2rn5 condition met
```

## Create the Experiment

We'll use the Postgres example [here][postgres-example].
This example will deploy the postgres application and configure an experiment to tune the memory and cpu limits for postgres.
This is done by running trials using `pgbench` to generate load against our postgres instance.
Each trial will test a different set of parameters provided by the Red Sky Ops ML servers.
The effectiveness of each trial is gauged by the metrics, in this case we contrast cost versus duration.

Deploy the postgres application and experiment using the following:
```sh
$ kustomize build github.com/redskyops/redskyops-recipes/postgres | kubectl apply -f -
# TODO show sample output
```

You can monitor the progress using `kubectl`:

```sh
$ watch -d kubectl get trials -o wide
# TODO show sample output
```

After the trial is complete, you will be able to view the parameters and the metrics generated from the trial.
The results can be reviewed as a visualization by running the following command:

```sh
$ redskyctl results
```

## Removing the Experiment

To clean up the data from your experiment, simply delete the experiment. The delete will cascade to the associated trials and other Kubernetes objects:

```sh
$ kubectl delete experiment postgres-example
```

Congratulations! You just ran your first experiment. You can move on to a more [advanced tutorial](tutorial.md) or browse the rest of the documentation to learn more about the Red Sky Ops Kubernetes experimentation product.

[redskyctl]: https://github.com/redskyops/redskyops-controller/releases/latest
[kustomize]: https://github.com/kubernetes-sigs/kustomize/releases/
[minikube]: https://kubernetes.io/docs/setup/learning-environment/minikube/
[kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/

[postgres-example]: https://github.com/redskyops/redskyops-recipes/tree/master/postgres

