k8s-cleanup
===========

Here are 3 cleanups you can apply on your kubernetes cluster:

* Cleans up exited containers and dangling images/volumes running as a DaemonSet (`docker-clean.yml`).
* Cleans up old replica sets, finished jobs and unrecycled evicted pods as a CronJob (`k8s-clean.yml`).
* Cleans up empty directory (not used anymore) in etcd as a CronJob (`etcd-empty-dir-cleanup.yml`).

You must have `batch/v1beta1` enabled on your k8s API server runtime config in order to run the CronJob.

Env vars
--------

In the DaemonSet (`docker-clean.yml`) you can set `DOCKER_CLEAN_INTERVAL` to modify the interval when it cleans up exited containers and dangling images/volumes; defaults to 30min (1800s).

In the CronJob (`k8s-clean.yml`) you can set `DAYS` to modify the maximum age of replica sets; defaults to 7 days.

Build and deploy with helm
--------------------------

* get project sources

```bash
git clone https://github.com/cnaslain/k8s-cleanup.git
```

* Private docker repository

```bash
export VERSION=0.1.0
export DockerImage=docker-registry.your-repo.com/k8s-cleanup:${VERSION}
docker build -t ${DockerImage} .
docker push ${DockerImage}
```

* Deploy with helm the built version of Docker hub with tag 0.1.0 in a custom namespace

```bash
export VERSION=0.1.0
export DockerImage=cnaslain/k8s-cleanup:${VERSION}
export NAMESPACE=k8s-cleanup

cd helm

echo "apiVersion: v1" > k8s-cleanup/Chart.yaml && echo "name: k8s-cleanup" >> k8s-cleanup/Chart.yaml && echo "version: ${VERSION}" >> k8s-cleanup/Chart.yaml

helm template k8s-cleanup \
    --debug \
    --set namespace=${NAMESPACE} \
    --set DockerImage=${DockerImage} \
    k8s-cleanup

kubectl create namespace ${NAMESPACE}

helm upgrade --install k8s-cleanup \
    --set namespace=${NAMESPACE} \
    --set DockerImage=${DockerImage} \
    --wait --namespace=${NAMESPACE} \
    k8s-cleanup
```
