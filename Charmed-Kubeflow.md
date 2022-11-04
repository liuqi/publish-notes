## Deploy Charmed Kubeflow on US-based VMs

Following the following steps:
[Quick start guide to Kubeflow | Documentation | Charmed Kubeflow](https://charmed-kubeflow.io/docs/quickstart)

```bash
# how to install snap
sudo apt install snapd
```

How to handle [docker.io](http://docker.io) pull limit issue. 

login [docker.io](http://docker.io) with the following method
[MicroK8s - DockerHub download rate limits | MicroK8s](https://microk8s.io/docs/dockerhub-limits)

How to access the network on remote host
```bash
# setup sock5 proxy on port 1080
ssh -D localhost:1080 remote-host
```

## The only difficulty for CN-based VM is image pulling

```bash
# import tar
ls *.tar | xargs -I {} docker load -i {}

# cp tag & remove the old one
docker tag old-tag new-tag
docker rmi old-tag

# pull and save image
PACKAGES=`cat ./package-list.txt`;

for package in $PACKAGES; do docker pull "$package"; done

docker images | tail -n +2 | grep -v "<none>" | awk '{printf("%s:%s\n", $1, $2)}' | while read IMAGE; do
    for package in $PACKAGES;
    do
        if [[ $package != *[':']* ]];then package="$package:latest"; fi

        if [ $IMAGE == $package ];then
            echo "[find image] $IMAGE"
            filename="$(echo $IMAGE| tr ':' '-' | tr '/' '-').tar"
            echo "[save as] $filename"
            docker save ${IMAGE} -o $filename
        fi
    done
done

# save image
docker images | tail -n +2 | grep -v "<none>" | awk '{printf("%s:%s\n", $1, $2)}' | while read IMAGE; do
    echo "[find image] $IMAGE"
    filename="$(echo $IMAGE| tr ':' '-' | tr '/' '-').tar"
    echo "[save as] $filename"
    docker save ${IMAGE} -o $filename
done
```

```bash
# import and list images
ls *.tar | xargs -I {} microk8s.ctr image import {}
microk8s.ctr images ls
```

- [解决microk8s拉不到外网image的问题](https://zhuanlan.zhihu.com/p/368512329)
- [通过 MicroK8s 搭建你的 K8s 环境](https://zhuanlan.zhihu.com/p/81648464)