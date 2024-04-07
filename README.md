

## Quick Start

Builds ansible in docker image
```
docker -f Dockerfile.ansible -t ansible:6.4.0 .
```

Playbooks are in <cwd>
```
docker run -v <cwd>:/workspace -it ansible:6.4.0 bash
cd /workspace
export ANSIBLE_HOST_KEY_CHECKING=False
ansible-playbook -k -i inventory.ini playbook.yaml 
```

## Helm in offline environments
It's an absolute pain in the head but the following instructions should help ease the pain a little. Using `kube-prometheus-stack` as an example`.

Online:

1. Add a repository.
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

2. Download the chart.
```
helm pull prometheus-community/kube-prometheus-stack
```

3. View docker images in the chart.

`helm template [REPO]/[CHART] | sls image: | sort -unique` should be sufficient for the default chart. If you enabled certain extra features that requires additional containers by configuring `values.yaml`, please pass it into the command line using the `-f` flag (e.g. `helm template prometheus-community/kube-prometheus-stack -f values.yaml`) 
```
# In powershell,
helm template prometheus-community/kube-prometheus-stack | sls image: | sort -unique

# Expected output:
#          image: "docker.io/grafana/grafana:10.4.0"
#          image: "quay.io/kiwigrid/k8s-sidecar:1.26.1"
#          image: "quay.io/prometheus-operator/prometheus-operator:v0.73.0"
#          image: quay.io/prometheus/node-exporter:v1.7.0
#          image: registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20221220-controller-v1.5.1-58-g787ea74b6
#        image: registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.12.0
#      image: "docker.io/bats/bats:v1.4.1"
#  image: "quay.io/prometheus/alertmanager:v0.27.0"
#  image: "quay.io/prometheus/prometheus:v2.51.1"
```
Download each of them manually into tar files with `docker save`.
TODO: probably can write a script to automatically pull and save the docker images

4. Transfer the downloaded helm chart `XXX.tgz` and docker images `YYY.tar` to the offline machine.

Offline:

5. Load the docker images created in step 4 with `docker load -i YYY.tar`. You may optionally upload these images to your private docker repository, if any, with `docker tag` and `docker push`.
6. (Optional) If there's a private helm repository (e.g. http://my-private-helm.abc.net), you may upload the helm chart `XXX.tgx` to it. After it's uploaded to the private repository, update the repository by:
```
helm repo add private-repo http://my-private-helm.abc.net
helm repo update
```
7. Update `values.yaml` of the chart as required. If you've retagged and push the docker images to your private docker repository, please replace the repository accordingly in the chart values.
8. Install the chart away!
```
# With private helm repository
helm install [RELEASE_NAME] private-repo/kube-prometheus-stack

# With local downloaded chart
helm install [RELEASE_NAME] XXX.tgz
```
