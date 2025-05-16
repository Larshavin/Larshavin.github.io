---
title: Harbor
date: 2022-07-31
hero: images/hero/harbor.png
description: "컨테이너 레지스트리 프로젝트 : Harbor"
menu:
  sidebar: 
    parent: k8s-cncf-landscape
---

# Harbor

{{< img src="/posts/images/cncf/harbor/0.png" align="center" >}}

- 참고자료
    
    https://happycloud-lee.tistory.com/165
    
    https://goharbor.io/docs/2.1.0/install-config/ 
    

쿠버네티스에서 파드를 생성할 때 필수적으로 필요로 하는 것이 바로 컨테이너 이미지 입니다. 컨테이너 이미지는 로컬에 저장할 수도 있고 [도커 허브](https://hub.docker.com)와 같은 원격 환경을 이용할 수도 있습니다. 원격 환경이 다른 사람들에게 이미지를 공유하거나 할 때 편리하지만, 문제는 도커 허브 이용에 제한이 생겼다는 것 입니다. 특정 시간 동안 특정 횟수의 다운로드 제한이 있습니다.

유료 모델을 사용하는 것도 하나의 방법이지만, 그보다 무료 이미지 저장소를 로컬에 설치 하는 것이 더 바람직할 듯 합니다. Harbor는 그 목적에 아주 적합 합니다.

### Installation

쿠버네티스에서 HA하게 하버를 이용하기 위해, [Helm을 통해 설치](https://goharbor.io/docs/2.8.0/install-config/harbor-ha-helm/)하는 것 권장하고 있습니다.

미리 준비 해두어야 할 조건은 다음과 같습니다.

- Kubernetes cluster 1.10+ ⇒ 이번 테스트에서는 1.27 버전을 사용하고 있습니다.
- Helm 2.8.0+ ⇒ Helm 3을 사용하고 있습니다.
- High available ingress controller (Harbor does not manage the external endpoint) ⇒ 노드 포트를 사용할 수 있습니다. nginx 인그레스 컨트롤러 앞에 proxy를 붙혔을 때 왠지 모를 ssl 인증오류가 생겨서, 노드 포트를 열고 이를 proxy에 등록해줄 것 입니다.
- High available PostgreSQL 9.6+ (Harbor does not handle the deployment of HA of database) ⇒ 파드로 만들어 놓아야 하는 걸까요?
- High available Redis (Harbor does not handle the deployment of HA of Redis) ⇒ 위와 동일한 질문이 있습니다. 답을 먼저 말씀 드리자면, 꼭 미리 준비해 둘 필요는 없습니다.
- PVC that can be shared across nodes or external object storage ⇒ 준비 된 PVC가 필요한데, 저희는 동적 프로비저닝 기능을 사용할 것입니다.

```bash
# 헬름 리포를 등록하고, 폴더를 내려 받습니다.
helm repo add harbor https://helm.goharbor.io
helm fetch harbor/harbor --untar
```

명령어 실행한 경로에 `/harbor` 폴더가 생성되어 있습니다. 

```bash
# cd ./harbor && ls -al

rwxr-xr-x   4 root root    123  7월 31 18:03 .
dr-xr-x---. 64 root root   4096  7월 31 18:03 ..
-rw-r--r--   1 root root     57  7월 31 18:03 .helmignore
-rw-r--r--   1 root root    567  7월 31 18:03 Chart.yaml
-rw-r--r--   1 root root  11357  7월 31 18:03 LICENSE
-rw-r--r--   1 root root 192242  7월 31 18:03 README.md
drwxr-xr-x   2 root root     58  7월 31 18:03 conf
drwxr-xr-x  15 root root   4096  7월 31 18:03 templates
-rw-r--r--   1 root root  33874  7월 31 18:03 values.yaml
```

`values.yaml` 파일을 수정해야 합니다. **Ingress rule**, **External URL**, **External PostgreSQL**, **External Redis,** **Storage** 부분들에 변화가 필요하다고 합니다. 허나, 위에서 언급했듯 저는 PostgreSQL와 Redis도 내부 저장소 사용할 예정입니다. 결국 따로 필요로 하는 것은 Storage 설정입니다.

#### Dynamic provisioning : pvc with NFS

기본적인 PV / PVC 사용 순서는 미리 PV를 만들어 놓고, "저 PV를 쓸거야"라는 선언의 목적으로 PVC를 만들어 놓아야 합니다. 하지만 동적 프로비저닝을 사용하면 PVC 하나만 적용시켜도 자동으로 PV가 생성되게 됩니다. 참고로 설치에 사용한 볼륨은 13G 정도 입니다. 

1. NFS 서버 시작하기
    
    ```bash
    # nfs-server를 다운 받습니다.
    systemctl start nfs-server
    systemctl enable nfs-server
    
    # nfs 저장소로 사용할 폴더를 생성해 줍니다.
    # 경로는 자유입니다. 저는 nfs 폴더를 생성하였습니다.
    mkdir nfs
    
    # /etc/exports 파일에 경로를 등록합니다.
    # [path] = 파일 경로
    # [cidr] = ex) 192.168.15.0/24 
    echo '[path] [cidr](rw,sync,no_root_squash)' >> /etc/exports
    
    #설정을 적용하고 결과를 확인합니다.
    exportfs -r
    showmount -e
    ```
    
2. 동적 프로비저닝 설정하기
    
    참고자료 → [Kubernetes k8s Volume 동적 프로비저닝 with NFS 기본 스토리지 클래스](https://nayoungs.tistory.com/entry/Kubernetes-k8s-Volume-동적-프로비저닝-with-NFS-기본-스토리지-클래스)
    
    ⇒ 쿠버네티스 노드 모두에 nfs 클라이언트를 받아놓아야합니다.
    위의 블로그에 과정이 자세하게 나와 있습니다. 굳이 디폴트 스토리지 클래스 까지 등록해주실 필요는 없습니다.
    
    궁금한 점 중 하나는 [깃허브](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner) 사이트에 보면 헬름으로도 이를 만들어줄 수 있는 듯 한데, 만약 여러 경로를 사용하려면, 여러 번 헬름 install을 수행하면 되는 것인가,, 라는 생각이 듭니다.
    

#### values.yaml

 `/harbor` 경로에 있는 폴더에 들어가보면 `values.yaml` 이라는 파일이 존재합니다. 하버 뿐만이 아니라, 여러 헬름 프로젝트에서는 설치에 필요한 각종 정보들은 위의 파일로 관리 합니다. 

 파일을 열어보면 1000줄 가까이의 무지막지한 라인들이 저를 반겨줍니다. 이 설정들을 하나하나 뜯어봐야하는 공포에 사로잡히지만, 다행이 수정할 부분은 그리 많지 않습니다.

가장 먼저 expose.type을 nodePort로 바꿔봅시다. 인그레스를 활용한 케이스는 [이 블로그](https://velog.io/@wkfwktka/k8s위에서-Harbor-설정ingress-Helm)를 살펴보시면 좋을 것 같습니다. 저도 처음에 nginx ingress 컨트롤러에 연결해 보았다가, 프록시와의 이유 모를 통신 이슈로 nodePort로 선회 했습니다.

인그레스 이외의 설정("clusterIP", "nodePort" or "loadBalancer")을 사용한다면, tls 인증을 위해 expose.tls.auto.commonName을 입력해 줍시다. 저는 프록시에서 사용할 도메인을 입력해 주었습니다. tls 인증이 harbor CA 라는 곳에서 이뤄지던데, 정확이 어떤 곳인지는 잘 모르겠습니다.  

또한 externalURL 쪽에도 https://[domain]을 입력해 주었습니다.

그 다음 볼륨 설정이 필요합니다. persistence.persistentVolumeClaim 쪽에서 registry, jobservice, database, redis, trivy 부분 아래 공통적으로 storageClass을 동적 프로비저닝 스토리지의 클래스로 지정해줘야 합니다. 

 설정이 끝났습니다.

#### Helm install

helm을 이용한 설치는 언제나 편리합니다. 다만, 명령어 실행 부분이 harbor 폴더를 바라보고 있어야 합니다.

```bash
# 네임스페이스 생성
kubectl create namespace harbor

# harbor 네임스페이스에 harbor 폴더의 설정 파일을 이용해 harbor를 설치
helm install -n harbor harbor harbor/
```

설정이 바뀌었을 때 적용 방법입니다.

```bash
helm upgrade -n harbor harbor harbor/
```

설치를 확인 해봅시다.

```bash
$ k get all,pv,pvc -n harbor 
NAME                                        READY   STATUS    RESTARTS       AGE
pod/harbor-core-7db5b74c86-nctk2            1/1     Running   0              148m
pod/harbor-database-0                       1/1     Running   0              15h
pod/harbor-jobservice-865b66687b-mg9dr      1/1     Running   3 (148m ago)   148m
pod/harbor-nginx-5cb465f58f-fmmrz           1/1     Running   0              148m
pod/harbor-notary-server-854944cd7d-kvsnf   1/1     Running   1 (143m ago)   148m
pod/harbor-notary-signer-76b6978d97-h7gfl   1/1     Running   0              148m
pod/harbor-portal-d7dbcc46c-94rxr           1/1     Running   0              15h
pod/harbor-redis-0                          1/1     Running   0              15h
pod/harbor-registry-d7c5dbf6f-7pzxc         2/2     Running   0              148m
pod/harbor-trivy-0                          1/1     Running   0              15h

NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                     AGE
service/harbor                 NodePort    10.96.115.165    <none>        80:30002/TCP,443:30003/TCP,4443:30004/TCP   148m
service/harbor-core            ClusterIP   10.96.117.194    <none>        80/TCP                                      15h
service/harbor-database        ClusterIP   10.104.0.73      <none>        5432/TCP                                    15h
service/harbor-jobservice      ClusterIP   10.102.37.222    <none>        80/TCP                                      15h
service/harbor-notary-server   ClusterIP   10.108.64.69     <none>        4443/TCP                                    15h
service/harbor-notary-signer   ClusterIP   10.111.161.104   <none>        7899/TCP                                    15h
service/harbor-portal          ClusterIP   10.101.185.44    <none>        80/TCP                                      15h
service/harbor-redis           ClusterIP   10.98.124.114    <none>        6379/TCP                                    15h
service/harbor-registry        ClusterIP   10.100.29.39     <none>        5000/TCP,8080/TCP                           15h
service/harbor-trivy           ClusterIP   10.107.210.23    <none>        8080/TCP                                    15h

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/harbor-core            1/1     1            1           15h
deployment.apps/harbor-jobservice      1/1     1            1           15h
deployment.apps/harbor-nginx           1/1     1            1           148m
deployment.apps/harbor-notary-server   1/1     1            1           15h
deployment.apps/harbor-notary-signer   1/1     1            1           15h
deployment.apps/harbor-portal          1/1     1            1           15h
deployment.apps/harbor-registry        1/1     1            1           15h

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/harbor-core-7db5b74c86            1         1         1       148m
replicaset.apps/harbor-jobservice-865b66687b      1         1         1       148m
replicaset.apps/harbor-nginx-5cb465f58f           1         1         1       148m
replicaset.apps/harbor-notary-server-854944cd7d   1         1         1       148m
replicaset.apps/harbor-notary-signer-76b6978d97   1         1         1       148m
replicaset.apps/harbor-portal-d7dbcc46c           1         1         1       15h
replicaset.apps/harbor-registry-d7c5dbf6f         1         1         1       148m

NAME                               READY   AGE
statefulset.apps/harbor-database   1/1     15h
statefulset.apps/harbor-redis      1/1     15h
statefulset.apps/harbor-trivy      1/1     15h

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                    STORAGECLASS   REASON   AGE
persistentvolume/pvc-24fc670f-e4f2-435d-8205-55e8681cdaa2   5Gi        RWO            Delete           Bound    harbor/data-harbor-trivy-0               nfs-client              15h
persistentvolume/pvc-54c51073-e463-4337-bce7-4bf218e58e1d   1Gi        RWO            Delete           Bound    harbor/harbor-jobservice                 nfs-client              15h
persistentvolume/pvc-9b0f92ee-439f-4e5b-bb4c-46c51c6a6317   1Gi        RWO            Delete           Bound    harbor/data-harbor-redis-0               nfs-client              15h
persistentvolume/pvc-c3926a2b-eecb-4758-aabf-3d32aeb0dd52   1Gi        RWO            Delete           Bound    harbor/database-data-harbor-database-0   nfs-client              15h
persistentvolume/pvc-eee40456-6920-472a-827c-b9d58179e092   5Gi        RWO            Delete           Bound    harbor/harbor-registry                   nfs-client              15h

NAME                                                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/data-harbor-redis-0               Bound    pvc-9b0f92ee-439f-4e5b-bb4c-46c51c6a6317   1Gi        RWO            nfs-client     15h
persistentvolumeclaim/data-harbor-trivy-0               Bound    pvc-24fc670f-e4f2-435d-8205-55e8681cdaa2   5Gi        RWO            nfs-client     15h
persistentvolumeclaim/database-data-harbor-database-0   Bound    pvc-c3926a2b-eecb-4758-aabf-3d32aeb0dd52   1Gi        RWO            nfs-client     15h
persistentvolumeclaim/harbor-jobservice                 Bound    pvc-54c51073-e463-4337-bce7-4bf218e58e1d   1Gi        RWO            nfs-client     15h
persistentvolumeclaim/harbor-registry                   Bound    pvc-eee40456-6920-472a-827c-b9d58179e092   5Gi        RWO            nfs-client     15h
```

노드 포트를 사용했기 때문에 `[쿠버네티스 노드 ip]:30003` 경로를 https를 이용해 들어가보면 다음과 같이 대시보드 화면이 나타나는 것을 확인하실 수 있게 됩니다. 다만, 인증서가 신뢰할 수 없다고는 뜹니다.

{{< img src="/posts/images/cncf/harbor/1.png" align="center" >}}

저는 Envoy proxy 서버를 두었기 때문에 제가 가진 도메인과 잘 엮어서 외부에서도 접속 할 수 있게 설정을 마쳤습니다. 하버의 초기 로그인 정보는 ID : admin, PW : Harbor12345 입니다.

## How to use?

하버를 설치 해보았으니, 사용법도 익혀봅시다. 공식 문서에서는 [Working with project](https://goharbor.io/docs/2.8.0/working-with-projects/) 부분 입니다. 눈치 빠르신 분들은 [Harbor administration](https://goharbor.io/docs/2.8.0/working-with-projects/) 파트를 넘어 갔다는 걸 아셨을 텐데, 왠지 양이 많이 뛰어 넘고 싶은 느낌 입니다. 그래도 언젠가 한 번 심도 있게 어떤 기능이 있는지 살펴보긴 해야겠죠? 언젠간 말이에요

로그인을 하고 나면 다음과 같은 화면이 나타납니다.

{{< img src="/posts/images/cncf/harbor/2.png" align="center" >}}

**Project**

프로젝트가 생성되지 않으면 이미지를 하버에 저장할 수 없습니다. 프로젝트는 유저의 역할에 따라 사용할 수 있는 기능이 다르다고 하네요. (Role-Based Access Control : RBAC) 또 크게 Public과 Private로 구분된다고 합니다. 디폴트로 library 프로젝트가 만들어져 있습니다.

저는 지금 관리자의 권한을 가지고 있습니다. 프로젝트를 만들어봅시다. 프로젝트 네임과 quota limit이 필수 기입 조건인데, -1이 의미하는 것은 용량 제한을 두지 않겠다 라는 말입니다. 

{{< img src="/posts/images/cncf/harbor/3.png" align="center" >}}

위의 상태로 생성을 해보면, 프로젝트가 추가된 게 확인 됩니다. Private로 만들었습니다.

{{< img src="/posts/images/cncf/harbor/4.png" align="center" >}}

안에 들어와 보니 summary, repositories, helm charts, members, labels, scanner, p2p preheat, policy, robot accounts, logs 그리고 configuration 탭이 존재 합니다. 기능이 많습니다. 

{{< img src="/posts/images/cncf/harbor/5.png" align="center" >}}

Configuration 쪽을 클릭해보면, Public 전환 버튼과 취약점이 있을지도 모르는 이미지(Vulnerability image)에 관련된 버튼들이 존재 합니다.

{{< img src="/posts/images/cncf/harbor/6.png" align="center" >}}

이제 프로젝트에 유저를 추가해봅시다. 유저를 먼저 만들어 봐야겠죠? Administration에 NEW USER를 클릭합니다. 저는 openstack 이라는 이름으로 하나 만들어 보았습니다. 생성 후 project로 돌아와 Members에서 openstack을 추가 해줍시다. 선택 버튼이 아니라 입력인 것이 조금 이외입니다.

{{< img src="/posts/images/cncf/harbor/7.png" align="center" >}}

추후 Role을 바꾼다던가 유저를 프로젝트에서 없애려면 Action 버튼을 활용하면 될 것 같습니다.

openstack 유저로 로그인 해서 확인해보면, kaps 프로젝트가 리스트에 보이게 됩니다. 

LDAP(?) server을 하버에 연결 시켜 두었다면, 유저에 Group을 형성할 수 있는 듯 한데, 제 하버에는 그런 설정이 되어 있지 않습니다.

이외에도 프로젝트에서는, Robot 계정을 만들수도 있고, 웹훅 알람을 보낼 수도 있습니다. 자세한 내용은 [문서](https://goharbor.io/docs/2.8.0/working-with-projects/project-configuration/)를 참고해봅시다.

### Image

하버를 HTTP로 사용하거나, 인증서가 unknown CA certificate 일 때 도커 클라이언트에서 이미지를 보낼 수 없다고 합니다. 현재 제 서버가 어느 정도까지 허용 될지 모르겠으니, 우선 image push를 시도해봅시다.

```bash
$ sudo docker image list
REPOSITORY         TAG            IMAGE ID       CREATED        SIZE
envoyproxy/envoy   v1.27-latest   511f8ff2a1f9   5 days ago     147MB
hello-world        latest         9c7a54a9a43c   2 months ago   13.3kB
```

엔보이 프록시 이미지를 하버에 넣어봅시다. 저는 문제 없이 로그인 했습니다.

```bash
# docker login <harbor_address>
Username: admin
Password:
```

만약 문제가 생긴다면 [이 방법](https://goharbor.io/docs/2.8.0/install-config/run-installer-script/#connect-http)을 시도 해봅시다.

다운 받아두었던 이미지를 tag 명령으로 다음과 같이 바꿔줍시다. 

```bash
# docker tag envoyproxy/envoy:v1.27-latest <harbor_address>/kaps/envoyproxy/envoy:v1.27-latest
# docker image list
REPOSITORY                                TAG            IMAGE ID       CREATED        SIZE
envoyproxy/envoy                          v1.27-latest   511f8ff2a1f9   5 days ago     147MB
<harbor_address>/kaps/envoyproxy/envoy   v1.27-latest   511f8ff2a1f9   5 days ago     147MB
hello-world                               latest         9c7a54a9a43c   2 months ago   13.3kB
```

그리고 바꾼 이미지를 push 합니다.

```bash
docker push <harbor_address>/kaps/envoyproxy/envoy:v1.27-latest
The push refers to repository [<harbor_address>/kaps/envoyproxy/envoy]
5f70bf18a086: Pushed 
34049bcf227d: Pushed 
7f9bdb71d8f3: Pushed 
d44ad2051d84: Pushed 
d4dab6f2fcc7: Pushed 
dde1e8ed09f5: Pushed 
c04c5616301c: Pushed 
928c1abc363c: Pushed 
f5bb4f853c84: Pushed 
v1.27-latest: digest: sha256:fdc35b806570985ff51db9736017b76508873581223e5d8df92b3f8ad9c9c489 size: 2195
```

결과는 성공입니다!!!

{{< img src="/posts/images/cncf/harbor/8.png" align="center" >}}

가장 근본적이고, 중요한 기능을 동작 해보았습니다.

[문서](https://goharbor.io/docs/2.8.0/working-with-projects/working-with-images/)를 보면, 이미지를 다루는 다양한 기능들이 존재하니 언젠가 꼭  살펴 봐야겠습니다.

### API V2.0

대시보드 좌측 하단에 API 문서 링크가 있습니다. 이 역시 언젠가 마음을 다 잡고 한 번 살펴보는 것이 좋아 보입니다.

{{< img src="/posts/images/cncf/harbor/9.png" align="center" >}}

{{< img src="/posts/images/cncf/harbor/10.png" align="center" >}}