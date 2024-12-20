# Kasm on Kubernetes

**Non-Release branches are not intended for production**

Kasm has been modified to run inside Kubernetes. The service containers will automatically detect they are running in Kubernetes and they will talk directly to each other rather than assume they are talking through an NGINX server as is the case for a normal Kasm deployment. Additionally, components need to talk to the name of the service defined, not to individual containers. A Kubernetes service has a resolvable DNS name that all containers should be able to talk with. API containers will not talk to an individual rdp gateway or guac container, but rather be load balanced to all existing respective containers. The reverse is also true. The API servers have been modified to only return a single entry when guac or rdp gateways call to get a list of API servers.

## Branches

This project will contain a branch that matches the release version of the corresponding Kasm Workspaces release. For example, Kasm Workspaces 1.16.0 will have a branch `release/1.16.0` within this project. **Non-release branches should not be used for production.** Be sure to checkout the branch on this project that matches the version of Kasm Workspaces you intend to deploy. Use the default `develop` branch to deploy the [developer preview](https://kasmweb.com/docs/latest/developers/builds.html#developer-preview-builds) build of Kasm Workspaces.

## Current Limitations

The following limitations are still be worked out.
1. The RDP Gateway component provides native RDP proxying for RDP clients. It is currently not exposed and would require 3389 to be defined in the ingress. We are currently working on an update that will support RDP over HTTPS, which is supported by most RDP clients. Therefore, this will not be required in the future.
2. After deployment, the administrator needs to login to Kasm, go to Admin->Infrastructure->Zones and set the Upstream Auth 

## Helm

A helm chart is used to deploy Kasm. This project contains a Kasm helm chart in the `kasm-single-zone` directory with a templated deployment. Follow the instructions below to deploy this chart.

> ***NOTE:*** There are a few steps that can be performed manually like creating the [namespace](#optional-manually-create-namespace), generating [certificates](#optional-manually-generate-certs-for-deployment), or adding [Docker credentials](#optional-docker-hub-login). Check out the *(Optional)* sections below for reference. These can also be added directly to the `values.yaml` file. Review the settings documentation in the `values.yaml` file for reference.

## Deploy

The following will deploy Kasm in a single zone configuration.

### Edit values.yaml

1. Change the namespace variable if desired (you can also change this by using the `--namespace <namespace name>` when you run the Helm Chart if you do not wish to modify this value)
2. Add the Kubernetes Cluster Domain Name. To retrieve this value, either ask your Kubernetes Administrator, or refer to the [Retrieving the Cluster Domain Name](#retrieving-the-cluster-domain-name) below for more information on how to get this value.
3. Add a `global.hostname` value for the resolvable hostname and certificate registration values.
4. *(Optional)* Modify the `global.altHostnames` to add a list of Alternate hostnames for the Ingress certificate (e.g. "*.kasmweb.com")
5. If you require a [Docker Hub Login](#optional-docker-hub-login) set the credentials in the `global.image.pullCredentials` or manually create the secret in Kubernetes and ensure the `global.image.PullSecrets` value matches the `SECRET` name used
6. Configure Kasm passwords. If these values are left blank, Helm will automatically set them for you. If you are running a `helm upgrade` of this deployment, Helm will reuse any password values that already exist in the location of the secrets generated by this helm chart.
7. Review the notes for the remaining settings in the `values.yaml` file and make adjustments as necessary
8. Note: The default PullPolicy of IfNotPresent will need to be changed to Always in order to refresh existing images. Images are cached globally so even uninstalling the helm chart and installing the helm chart in a new namespace will reference the existing images.

### Deploy Helm Chart

```bash
## Use this if you manually created the namespace
helm install <release name> kasm-single-zone --namespace <namespace name>

## Use this if you want Helm to create the namespace
helm install <release name> kasm-single-zone --namespace <namespace name> --create-namespace
```

## Retrieving the Cluster Domain Name

> ***NOTE:*** This step is only required if you are running a Kasm agent in the same Kubernetes cluster as your management services via KubeVirt.

This write-up is provided to help you locate the value you need to use for the [Kasm Upstream Auth address](https://kasmweb.com/docs/latest/guide/zones/deployment_zones.html#id4) so your Kasm Agents will be able to communicate with the Kasm management services and you will be able to successfully launch Kasm Workspace containers on your KubeVirt Agents. Please refer to the [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) for more information related to this topic.

Kubernetes supports many different DNS Custom Resource Definitions (CRD), but one of the most common is CoreDNS. Since this is the most common, and the one discussed specifically in the [Kubernetes DNS documentation](https://kubernetes.io/docs/tasks/administer-cluster/coredns/) it is the one addressed here. To retrieve this information you will require read permissions on the `kube-system` namespace.

To discover the name of the CoreDNS ConfigMap used to configure the cluster DNS run the command below:

```bash
kubectl get configmaps -n kube-system

## Should return something like the below
NAME                                                   DATA   AGE
kube-apiserver-legacy-service-account-token-tracking   1      72d
extension-apiserver-authentication                     6      72d
local-path-config                                      4      72d
chart-content-traefik-crd                              0      72d
chart-content-traefik                                  0      72d
kube-root-ca.crt                                       1      72d
coredns                                                2      72d
```

From this list of ConfigMaps, the `coredns` is the one to view.

> ***NOTE:*** Sometimes this ConfigMap will be named something like `kube-dns` or `cluster-dns`. Basically, you're looking for a "DNS" ConfigMap in the kube-system namespace to get your cluster domain name.

Once the DNS ConfigMap name is known, use the command below to view the the contents.

```bash
kubectl get configmap -n kube-system coredns -o yaml

## Representative output of the above command
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          fallthrough in-addr.arpa ip6.arpa
        }
        forward . /etc/resolv.conf
        cache 300 {
          prefetch 10
        }
        loop
        reload
        loadbalance
        import custom/*.override
    }
    import custom/*.server
kind: ConfigMap
metadata:
  creationTimestamp: "2024-08-25T21:38:26Z"
  labels:
    c3.doks.digitalocean.com/component: coredns
    c3.doks.digitalocean.com/plane: data
    doks.digitalocean.com/managed: "true"
  name: coredns
  namespace: kube-system
  resourceVersion: "1537"
  uid: 69765275-5832-4652-a3e3-506949b3f793
```

From this output, the important information is the line `kubernetes cluster.local in-addr.arpa ip6.arpa`. The value immediately after `kubernetes` in this line is the cluster domain value - `cluster.local` in this instance.


## *(Optional)* Manually create namespace

```bash
NAMESPACE="<namespace name>"
kubectl create ns "${NAMESPACE}" --dry-run=client -o yaml | kubectl apply -f -
```

## *(Optional)* Docker Hub Login

If you are using private images in Docker Hub, you will need to login. Substitute with your creds.

```bash
NAMESPACE="<namespace name>"
SECRET="<secret name>"
USERNAME="<docker username>"
PASSWORD="<dckr_pat_xxxxxxxxxxx>"
EMAIL="<user@mail.com>"
kubectl create secret docker-registry "${SECRET}" --docker-server="https://index.docker.io/v2/" --docker-username "${USERNAME}" --docker-password="${PASSWORD}" --docker-email="${EMAIL}" --namespace="${NAMESPACE}"
```

## *(Optional)* Manually generate certs for deployment:

This step will generate self-signed certificates for your Kasm services.

> **NOTE:** If you manually run the steps below, make sure to modify the `values.yaml` file and set the `create` value in each of the `kasmCerts` objects to `false` so Helm doesn't attempt to change or overwrite your existing certs.

```bash
# Set the Kasm Domain Name, K8s Ingress IP, and K8s namespace as needed
IP="<k8s ingress ip>"
HOST="<kasm ingress domain name>"
NAMESPACE="<namespace name>"
NGINX_SERVICE_NAME="kasm-proxy"
RDPPROXY_SERVICE_NAME="rdp-gateway"
DB_SERVICE_NAME="db"

## Make necessary directories - required to prevent certificates from being overwritten
mkdir -p certs/{ingress,proxy,rdp,db}

## Generate certs
## Ingress cert
openssl req -x509 -newkey rsa:4096 -keyout certs/ingress/tls.key -out certs/ingress/tls.crt -sha256 -days 365 -nodes -extensions san -config \
  <(echo "[req]";
    echo distinguished_name=req;
    echo "[san]";
    echo subjectAltName=DNS:${HOST},IP:${IP}
    ) \
    -subj "/C=US/O=KASM/CN=${HOST}" &> /dev/null

## Nginx proxy cert
openssl req -x509 -nodes -days 1825 -newkey rsa:2048 -keyout certs/proxy/tls.key -out certs/proxy/tls.crt -subj "/C=US/ST=VA/L=None/O=None/OU=DoFu/CN=${NGINX_SERVICE_NAME}/emailAddress=none@none.none"

## RDP Gateway cert
openssl req -x509 -nodes -days 1825 -newkey rsa:2048 -keyout certs/rdp/tls.key -out certs/rdp/tls.crt -subj "/C=US/ST=VA/L=None/O=None/OU=DoFu/CN=${RDPPROXY_SERVICE_NAME}/emailAddress=none@none.none"

## Kasm DB cert
openssl req -x509 -nodes -days 1825 -newkey rsa:2048 -keyout certs/db/tls.key -out certs/db/tls.crt -subj "/C=US/ST=VA/L=None/O=None/OU=DoFu/CN=${DB_SERVICE_NAME}/emailAddress=none@none.none"

## Upload certs to K8s
## Ingress cert
kubectl create secret tls kasm-ingress-cert --namespace "${NAMESPACE}" --key certs/ingress/tls.key --cert certs/ingress/tls.crt

## Nginx cert
kubectl create secret tls kasm-nginx-proxy-cert --namespace "${NAMESPACE}" --key certs/proxy/tls.key --cert certs/proxy/tls.crt

# RDP Gateway cert
kubectl create secret tls kasm-rdpproxy-cert --namespace "${NAMESPACE}" --key certs/rdp/tls.key --cert certs/rdp/tls.crt

# Kasm DB cert
kubectl create secret tls kasm-db-cert --namespace "${NAMESPACE}" --key certs/db/tls.key --cert certs/db/tls.crt
```
