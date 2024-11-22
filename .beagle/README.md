# kasm-chart

<https://kasmweb.com/docs/latest/install/kubernetes.html>

```bash
git remote add upstream git@github.com:kasmtech/kasm-helm.git

git fetch upstream

git merge upstream/release/1.16.1
```

## Deploy

```bash
# 1.install
helm package ./kasm-single-zone

kubectl create ns beagle-kasm

helm install \
  --namespace=beagle-kasm \
  kasm \
  /etc/kubernetes/charts/kasm-single-zone-1.16.1-develop.tgz \
  -f /etc/kubernetes/charts/kasm-single-zone-1.16.1-develop.yaml

# 2. Upgrade
helm upgrade \
  --namespace=beagle-kasm \
  kasm \
  /etc/kubernetes/charts/kasm-single-zone-1.16.1-develop.tgz \
  -f /etc/kubernetes/charts/kasm-single-zone-1.16.1-develop.yaml

# 3. Uninstall
helm uninstall \
  --namespace=beagle-kasm \
  kasm

# 4. Template
helm template \
  --namespace=beagle-kasm \
  kasm \
  ./kasm-single-zone \
  -f .beagle/values-amd64.yaml
```

## images

```bash
# kasmweb/proxy
docker pull kasmweb/proxy:1.16.1 && \
docker tag kasmweb/proxy:1.16.1 registry.cn-qingdao.aliyuncs.com/wod/kasmweb:proxy-1.16.1 && \
docker push registry.cn-qingdao.aliyuncs.com/wod/kasmweb:proxy-1.16.1

# kasmweb/postgres
docker pull kasmweb/postgres:1.16.1 && \
docker tag kasmweb/postgres:1.16.1 registry.cn-qingdao.aliyuncs.com/wod/kasmweb:postgres-1.16.1 && \
docker push registry.cn-qingdao.aliyuncs.com/wod/kasmweb:postgres-1.16.1

# kasmweb/api
docker pull kasmweb/api:1.16.1 && \
docker tag kasmweb/api:1.16.1 registry.cn-qingdao.aliyuncs.com/wod/kasmweb:api-1.16.1 && \
docker push registry.cn-qingdao.aliyuncs.com/wod/kasmweb:api-1.16.1

# kasmweb/manager
docker pull kasmweb/manager:1.16.1 && \
docker tag kasmweb/manager:1.16.1 registry.cn-qingdao.aliyuncs.com/wod/kasmweb:manager-1.16.1 && \
docker push registry.cn-qingdao.aliyuncs.com/wod/kasmweb:manager-1.16.1

# kasmweb/kasm-guac
docker pull kasmweb/kasm-guac:1.16.1 && \
docker tag kasmweb/kasm-guac:1.16.1 registry.cn-qingdao.aliyuncs.com/wod/kasmweb:guac-1.16.1 && \
docker push registry.cn-qingdao.aliyuncs.com/wod/kasmweb:guac-1.16.1

# kasmweb/rdp-gateway
docker pull kasmweb/rdp-gateway:1.16.1 && \
docker tag kasmweb/rdp-gateway:1.16.1 registry.cn-qingdao.aliyuncs.com/wod/kasmweb:rdp-gateway-1.16.1 && \
docker push registry.cn-qingdao.aliyuncs.com/wod/kasmweb:rdp-gateway-1.16.1

# kasmweb/rdp-https-gateway
docker pull kasmweb/rdp-https-gateway:1.16.1 && \
docker tag kasmweb/rdp-https-gateway:1.16.1 registry.cn-qingdao.aliyuncs.com/wod/kasmweb:rdp-https-gateway-1.16.1 && \
docker push registry.cn-qingdao.aliyuncs.com/wod/kasmweb:rdp-https-gateway-1.16.1

# kasmweb/share
docker pull kasmweb/share:1.16.1 && \
docker tag kasmweb/share:1.16.1 registry.cn-qingdao.aliyuncs.com/wod/kasmweb:share-1.16.1 && \
docker push registry.cn-qingdao.aliyuncs.com/wod/kasmweb:share-1.16.1

# kasmweb/rdp-https-gateway
docker pull redis:5-alpine && \
docker tag redis:5-alpine registry.cn-qingdao.aliyuncs.com/wod/kasmweb:redis-1.16.1 && \
docker push registry.cn-qingdao.aliyuncs.com/wod/kasmweb:redis-1.16.1
```
