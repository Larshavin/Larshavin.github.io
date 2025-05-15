---
title: 1. Hello, k8s Dev World!
date: 2025-05-14
hero: images/hero/쿠버네티스가 무엇일까.png
description: "쿠버네티스를 개발자 관점에서 바라보는 글입니다."
menu:
  sidebar: 
    weight: 1
    parent: kubernetes-deep-dive
---

https://github.com/kubernetes/kubernetes

### 쿠버네티스 딥 다이브를 시도하는 이유

시중에 쿠버네티스(이하 K8s)를 엔지니어 관점에서 활용하는 예시는 무궁무진하다. 배포, 모니터링, 유지 관리 등등 .
특히 K8s 클러스터를 배포하는 것은 kubeadm, kubespray, k3s, k0s, kind, minikube 등등 엄청나게 방법이 다양하다.

하지만 K8s를 개발의 관점에서 접근하는 경우는 드물다. 당장 `k8s 개발 환경 구축` 키워드로 검색해보아도, 대부분 엔지니어 관점에서의 게시글이다. 

잘 만들어졌거나, 충분히 완성도가 높을 것이라 기대되는 K8s를 굳이 빌드해보는 이유는 무엇일까? 이에 대한 나의 선제적 답은 엔지니어와 개발자 두 가지 관점으로 있다.

엔지니어의 관점에서, 소프트웨어 내부 구조와 동작 원리를 얼마나 깊이 이해하고 있느냐에 따라 그 도구를 사용하는 효율성은 크게 달라진다. 특히, 복잡한 시스템일수록 내부 로직을 깊이 이해하는 것은 단순한 활용 이상의 가치를 제공한다.

이러한 이해는 단순히 현재의 요구 사항을 충족시키는 데서 그치지 않고, 예상치 못한 장애 상황이나 극한의 환경에서도 시스템의 성능과 안정성을 유지할 수 있는 확장성과 대응 능력을 제공한다. 예를 들어, 대규모 트래픽 폭증, 비정상적인 노드 장애, 또는 네트워크 단절과 같은 상황은 일반적인 환경에서는 쉽게 재현할 수 없지만, 이러한 시나리오에 대비한 설계와 배포 전략을 세우는 데에는 내부 로직에 대한 깊은 이해가 필수적이다.

또한, K8s를 코드 수준으로 분석하면서 얻게 되는 경험은 단순히 작동 방식을 넘어, 시스템의 다양한 구성 요소가 어떻게 상호작용하는지, 이를 최적화하거나 커스터마이징하는 방법은 무엇인지에 대한 통찰력을 제공한다. 이는 특정 프로젝트의 요구 사항에 맞는 맞춤형 솔루션을 설계하거나, 시스템 병목을 제거하고 성능을 극대화하는 데 직접적인 도움을 준다.

개발자 관점에서는 Kubernetes가 단순히 컨테이너 오케스트레이션 도구로서의 가치를 넘어 오픈소스 생태계와 Go 언어 기반 프로젝트의 대표적인 성공 사례로 꼽힌다는 점이다. 이는 단순히 소프트웨어를 빌드하고 사용하는 것을 넘어, 오픈소스 협업의 구조, 코드 품질 관리, 대규모 커뮤니티 기여 방식 등을 깊이 이해하는 데 훌륭한 케이스 스터디가 될 수 있다.

특히, Kubernetes는 복잡하고 정교하게 설계된 코드베이스를 가지고 있어, Go 언어의 고급 활용 사례를 학습하고, 효율적이고 확장 가능한 소프트웨어 설계를 탐구하기에 적합하다. 이를 통해 단순히 Kubernetes 자체의 동작 원리를 배우는 것뿐만 아니라, 대규모 오픈소스 프로젝트의 개발 및 운영 방식을 체득할 수 있다.

내 실력으로 쿠버네티스 생태계를 얼마나 깊은 수준으로 탐험할 수 있을까 걱정이지만, 조심스러운 여정을 시작해보려 한다.

### 개발 환경 구축
구체적인 방법은 다음 주소에서 확인할 수 있다. 
https://github.com/kubernetes/community/blob/master/contributors/devel/README.md
- [Building Kubernetes with Docker](https://github.com/kubernetes/kubernetes/blob/8b1fe81dfa5ca3cc0b2c1088a1a8df2db0880b10/build/README.md)
- [hack/local-up-cluster.sh](https://github.com/kubernetes/community/blob/master/contributors/devel/running-locally.md)
- 예시 : https://medium.com/@ElieXU/k8s-cluster-setup-from-source-code-9e38354a24fd
#### 환경 셋업
아래 환경에서 프로젝트를 생성하여 진행하였다.
cpu : 16 코어
mem : 60
os : rocky 9.4
go 1.23.4
#### 빌드
깃허브 레포지토리 readme를 확인하면 아래와 같은 두 가지 방법이 존재한다.
##### golang 기반 환경
``` shell
git clone https://github.com/kubernetes/kubernetes
cd kubernetes
make
```

실제로 빌드 시에 시간 많이 소요되진 않는다. 거의 금방 완료된다.
golang 기반으로 빌드 하였을 때 결과물은 다음과 같다.

``` shell
$ pwd
/home/syyang/projects/kubernetes/_output/bin

$ ls
apiextensions-apiserver  e2e.test  go-runner  kube-aggregator  kube-controller-manager  kubectl-convert  kube-log-runner  kube-proxy      mounter
e2e_node.test            ginkgo    kubeadm    kube-apiserver   kubectl                  kubelet          kubemark         kube-scheduler
```
##### docker 환경
[hack/local-up-cluster.sh](https://github.com/kubernetes/community/blob/master/contributors/devel/running-locally.md) 문서를 따라하기 위해 다음 방법을 사용한다.
``` shell
git clone https://github.com/kubernetes/kubernetes
cd kubernetes
make quick-release
```
해당 방식은 10~15분 정도 시간이 소요 된다.

``` shell
$ make quick-release
+++ [0120 14:01:32] Verifying Prerequisites....
+++ [0120 14:01:32] Building Docker image kube-build:build-9bf87f211d-5-v1.33.0-go1.23.4-bullseye.0
+++ [0120 14:05:22] Creating data container kube-build-data-9bf87f211d-5-v1.33.0-go1.23.4-bullseye.0
+++ [0120 14:05:22] Syncing sources to container
+++ [0120 14:05:28] Running build command...
+++ [0120 14:05:31] Building go targets for linux/amd64
    k8s.io/apiextensions-apiserver (static)
    k8s.io/component-base/logs/kube-log-runner (static)
    k8s.io/kube-aggregator (static)
    k8s.io/kubernetes/cluster/gce/gci/mounter (static)
    k8s.io/kubernetes/cmd/kube-apiserver (static)
    k8s.io/kubernetes/cmd/kube-controller-manager (static)
    k8s.io/kubernetes/cmd/kube-proxy (static)
    k8s.io/kubernetes/cmd/kube-scheduler (static)
    k8s.io/kubernetes/cmd/kubeadm (static)
    k8s.io/kubernetes/cmd/kubelet (non-static)
+++ [0120 14:06:39] Building go targets for linux/amd64
    k8s.io/component-base/logs/kube-log-runner (static)
    k8s.io/kubernetes/cmd/kube-proxy (static)
    k8s.io/kubernetes/cmd/kubeadm (static)
    k8s.io/kubernetes/cmd/kubelet (non-static)
+++ [0120 14:06:45] Building go targets for linux/amd64
    k8s.io/kubernetes/cmd/kubectl (static)
    k8s.io/kubernetes/cmd/kubectl-convert (static)
+++ [0120 14:06:51] Building go targets for linux/amd64
    github.com/onsi/ginkgo/v2/ginkgo (non-static)
    k8s.io/kubernetes/test/conformance/image/go-runner (non-static)
    k8s.io/kubernetes/test/e2e/e2e.test (test)
+++ [0120 14:07:13] Building go targets for linux/amd64
    github.com/onsi/ginkgo/v2/ginkgo (non-static)
    k8s.io/kubernetes/cmd/kubemark (static)
    k8s.io/kubernetes/test/e2e_node/e2e_node.test (test)
+++ [0120 14:07:34] Syncing out of container
+++ [0120 14:07:37] Building tarball: src
+++ [0120 14:07:37] Building tarball: manifests
+++ [0120 14:07:37] Starting tarball: client linux-amd64
+++ [0120 14:07:37] Waiting on tarballs
gtar: Removing leading `/' from member names
gtar: Removing leading `/' from hard link targets
gtar: Removing leading `/' from member names
gtar: Removing leading `/' from member names
gtar: Removing leading `/' from hard link targets
gtar: Removing leading `/' from member names
+++ [0120 14:07:42] Building tarball: node linux-amd64
+++ [0120 14:07:42] Building images: linux-amd64
+++ [0120 14:07:42] Starting docker build for image: kube-apiserver-amd64
+++ [0120 14:07:42] Starting docker build for image: kube-controller-manager-amd64
+++ [0120 14:07:42] Starting docker build for image: kube-scheduler-amd64
+++ [0120 14:07:42] Starting docker build for image: kube-proxy-amd64
+++ [0120 14:07:42] Starting docker build for image: kubectl-amd64
+++ [0120 14:07:46] Deleting docker image registry.k8s.io/kube-proxy-amd64:v1.33.0-alpha.0.536_8b1fe81dfa5ca3
+++ [0120 14:07:47] Deleting docker image registry.k8s.io/kubectl-amd64:v1.33.0-alpha.0.536_8b1fe81dfa5ca3
+++ [0120 14:07:47] Deleting docker image registry.k8s.io/kube-scheduler-amd64:v1.33.0-alpha.0.536_8b1fe81dfa5ca3
+++ [0120 14:07:47] Deleting docker image registry.k8s.io/kube-controller-manager-amd64:v1.33.0-alpha.0.536_8b1fe81dfa5ca3
+++ [0120 14:07:49] Deleting docker image registry.k8s.io/kube-apiserver-amd64:v1.33.0-alpha.0.536_8b1fe81dfa5ca3
+++ [0120 14:07:49] Docker builds done
+++ [0120 14:07:49] Building tarball: server linux-amd64
+++ [0120 14:08:21] Building tarball: final
+++ [0120 14:08:21] Waiting on test tarballs
+++ [0120 14:08:21] Starting tarball: test linux-amd64
+++ [0120 14:08:30] Building tarball: test portable
```
#### hack/local-up-cluster.sh
로컬환경에 클러스터를 배포하는 스크립트이다. 
etcd 가 설치되어 있어야 한다. 아래는 etcd가 없어서 새로 설치 후 다시 진행했다. 

``` shell
$ export. CONTAINER_RUNTIME_ENDPOINT="unix:///run/containerd/containerd.sock"

$ sudo -E PATH=$PATH hack/local-up-cluster.sh
make: Entering directory '/home/syyang/projects/kubernetes'
+++ [0120 14:13:07] Building go targets for linux/amd64
    k8s.io/kubernetes/cmd/cloud-controller-manager (non-static)
    k8s.io/kubernetes/cmd/kube-apiserver (static)
    k8s.io/kubernetes/cmd/kube-controller-manager (static)
    k8s.io/kubernetes/cmd/kubectl (static)
    k8s.io/kubernetes/cmd/kubelet (non-static)
    k8s.io/kubernetes/cmd/kube-proxy (static)
    k8s.io/kubernetes/cmd/kube-scheduler (static)
make: Leaving directory '/home/syyang/projects/kubernetes'
[sudo] password for syyang: 

etcd must be in your PATH

You can use 'hack/install-etcd.sh' to install a copy in third_party/.

$ ./hack/install-etcd.sh
Downloading https://github.com/etcd-io/etcd/releases/download/v3.5.17/etcd-v3.5.17-linux-amd64.tar.gz succeed
  PATH="$PATH:/home/syyang/projects/kubernetes/third_party/etcd"

$ PATH="$PATH:/home/syyang/projects/kubernetes/third_party/etcd"
```

`/etc/containerd/config.toml` 파일에서 `SystemdCgroup = true` 라인을 지워야 하고 ( 해당 환경에 과거 k8s 설치이력이 있었다면, 클러스터 성분 및 kubelet 등을 깔끔하게 제거 하는 것을 추천한다 ) sudo 권한을 사용하여 `sudo -E PATH=$PATH hack/local-up-cluster.sh` 로 실행하는 게 좋다.

``` shell
...
pod/coredns-76b7578cff-2qlv5 condition met
deployment.apps/coredns condition met
6
Create default storage class for 
storageclass.storage.k8s.io/standard created
Local Kubernetes cluster is running. Press Ctrl-C to shut it down.

Configurations:
  /tmp/local-up-cluster.sh.7bEbu8/kube-audit-policy-file
  /tmp/local-up-cluster.sh.7bEbu8/kube_egress_selector_configuration.yaml
  /tmp/local-up-cluster.sh.7bEbu8/kubelet.yaml
  /tmp/local-up-cluster.sh.7bEbu8/kube-proxy.yaml
  /tmp/local-up-cluster.sh.7bEbu8/kube-scheduler.yaml
  /tmp/local-up-cluster.sh.7bEbu8/kube-serviceaccount.key

Logs:
  /tmp/etcd.log
  /tmp/kube-apiserver.log
  /tmp/kube-controller-manager.log
  
  /tmp/kube-proxy.log
  /tmp/kube-scheduler.log
  /tmp/kubelet.log

To start using your cluster, you can open up another terminal/tab and run:

  export KUBECONFIG=/var/run/kubernetes/admin.kubeconfig
  cluster/kubectl.sh

Alternatively, you can write to the default kubeconfig:

  export KUBERNETES_PROVIDER=local

  cluster/kubectl.sh config set-cluster local --server=https://localhost:6443 --certificate-authority=/var/run/kubernetes/server-ca.crt
  cluster/kubectl.sh config set-credentials myself --client-key=/var/run/kubernetes/client-admin.key --client-certificate=/var/run/kubernetes/client-admin.crt
  cluster/kubectl.sh config set-context local --cluster=local --user=myself
  cluster/kubectl.sh config use-context local
  cluster/kubectl.sh
```

코드 수정 후 매번 이런 식으로 클러스터를 배포해 볼 수 있다. 다만 이 과정 자체가 금방 수행된다는 느낌은 아니고 일부 수정 후에 항상 전체를 재배포해야 한다는 불만 요소가 있다. kind 등을 통해 컨테이너 수준으로 배포시에 파일 일부 변화만 감지하여 재배포하는 프로세스 등을 시도해봄직 하다.

우선은 간단한 방법론을 알아보았는데, 더 편리한 배포 방식은 추후에 더 심도있게 응용해야할 때가 있을 더 고민을 해볼 필요가 있을 듯하다.

#### kind
K8s in Docker 라는 이름의 뜻 https://kind.sigs.k8s.io
{{< img src="/posts/images/kubernetes-deep-dive1/kubernetes-deep-dive.png" align="center" >}}

`kind supports building Kubernetes release builds from source` 라는 문장을 문서 어딘가에서 확인하였는데, 실제로 변경된 코드 사항을 빠르게 클러스터로 곧장 배포할 수 있다면 쿠버네티스 개발에 큰 도움이 되지 않을까 싶다.

다만, 아직 정확한 방법을 찾지는 못 하였는데, 예상컨데 소스 빌드 후 kind 노드 이미지를 새로 빌드하여 kind 클러스터를 배포하는 방식이 있을 듯하고,
가장 좋은 것은 배포된 환경에서 일부 컴포넌트만 수정된 컴포넌트로 교체하는 방식이 존재하는 것이다.

공식 페이지 resource 카테고리 확인시, CI로 사용하는 예제에 대한 영상이 많다.

### 코드 훑어보기
가장 먼저 K8s를  코드 수준으로 분석하기 위해 git 코드베이스를 펼쳐보았을 때 겪은 문제가 있었다. 코드를 어디서부터 확인해야 하는지 모르겠다는 점이다. K8s 레포의 구조는 다음과 같다. 

``` shell
$ ls
api    CHANGELOG     cluster  code-of-conduct.md  docs    go.sum   go.work.sum  LICENSE   logo      _output  OWNERS_ALIASES  plugin     SECURITY_CONTACTS  SUPPORT.md  third_party
build  CHANGELOG.md  cmd      CONTRIBUTING.md     go.mod  go.work  hack         LICENSES  Makefile  OWNERS   pkg             README.md  staging            test        vendor
```

[golang의 표준 레포 레이아웃](https://github.com/golang-standards/project-layout/blob/master/README_ko.md)은 해당 주소를 확인해보면 좋다. 공식적인 표준은 아니라는 점을 주의해야 한다.

위 게시글을 참고로 보았을 때, 코드를 확인할 때 가장 먼저 확인하면 좋은 영역은 역시 `cmd` 하위이다.

#### cmd

```
$ ls
clicheck                  dependencyverifier  genkubedocs         genutils    import-boss     kube-apiserver           kubectl-convert  kube-proxy      preferredimports
cloud-controller-manager  fieldnamedocscheck  genman              genyaml     importverifier  kube-controller-manager  kubelet          kube-scheduler  prune-junit-xml
dependencycheck           gendocs             genswaggertypedocs  gotemplate  kubeadm         kubectl                  kubemark         OWNERS          yamlfmt
```

익숙한 컴포넌트와 익숙하지 않은 컴포넌트들이 동시에 눈에 띈다.
예를 들면, kube 로 시작하는 컴포넌트들은 익숙하다. kubeadm, kube-apiserver, kube-controller-manager, kubectl, kubelet, kube-proxy, kube-scheduler 등등이다. 

{{< img src="/posts/images/kubernetes-deep-dive1/kubernetes-deep-dive-1.png" align="center" >}}

쿠버네티스를 구성하는 핵심 컴포넌트들 모두가 하나의 레포에 모여 있다. 이를 모노레포 전략이라고 부른다. OpenStack은 기본적으로 여러 서비스에 대한 관리를 멀티 레포지토리로 처리한다. 예를 들면, Nova 코드의 원리를 확인하고 싶으면 Nova 레포지토리를 확인하면 된다. 

눈에 익숙치 않은 컴포넌트 들도 존재한다. 이름으로 짐작컨데, 문서 작성이나 의존성 체크 등 프로젝트 관리에 도움이 되는 요소라고 추측한다.

이 수 많은 컴포넌트 중에, 쿠버네티스 구조에서 가장 중요하다고 생각이 되는 kube-apiserver 안을 살펴보자. 생각보다 아기자기한 구성이다. apiserver.go 와 app 폴더가 메인이고 특징적으로 .import-restrictions 파일이 존재한다.

``` shell
ls -al
.  ..  apiserver.go  app  .import-restrictions  OWNERS
```

apiserver.go 파일은 main 패키지의 역할이다. 일반적인 오픈소스들의 코드 시작부는 정말 간단한 것 같다. 추후 프로젝트를 시작할 때 고려해 볼만한 사항이다. 
``` go
/*
Copyright 2014 The Kubernetes Authors.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/

// APIServer is the main API server and master for the cluster.
// It is responsible for serving the cluster management API.
package main

import (
	"os"
	_ "time/tzdata" // for timeZone support in CronJob

	"k8s.io/component-base/cli"
	_ "k8s.io/component-base/logs/json/register"          // for JSON log format registration
	_ "k8s.io/component-base/metrics/prometheus/clientgo" // load all the prometheus client-go plugins
	_ "k8s.io/component-base/metrics/prometheus/version"  // for version metric registration
	"k8s.io/kubernetes/cmd/kube-apiserver/app"
)

func main() {
	command := app.NewAPIServerCommand()
	code := cli.Run(command)
	os.Exit(code)
}
```
핵심 로직은 `"k8s.io/kubernetes/cmd/kube-apiserver/app"` 쪽에 담겨 있음을 짐작할 수 있고, 프로그램의 시작이 cobra 기반의 cli tool로 되어 있다는 사실이 매력적이다. cobra 기반임을 알 수 있는 건 `"k8s.io/component-base/cli"` 쪽을 미리 확인했기 때문이다. 이를 별도의 component-base 경로의 cli 패키지로 정의했다는 것은 여러 프로젝트에서 공통적으로 사용하는 디자인이라고 생각할 수 있다.
``` go
package cli

import (
	"fmt"
	"os"

	"github.com/spf13/cobra"

	cliflag "k8s.io/component-base/cli/flag"
	"k8s.io/component-base/logs"
	"k8s.io/klog/v2"
)
```

#### staging
자연스럽게 staging 폴더로 넘어가보자. `"k8s.io/component-base/cli"` 파일 경로는 staging/src 폴더 하위에 있었다. staging 폴더는 Kubernetes 코드베이스의 **모듈화 및 재사용성**을 위한 중요한 구조적 요소이다.

staging 폴더에 있는 코드는 Kubernetes 코드베이스 내에서 개발 및 테스트되지만, 최종적으로는 외부 리포지토리(예: https://github.com/kubernetes/client-go)로 [퍼블리싱 봇](https://github.com/kubernetes/publishing-bot)에 의해 복사된다.

Kubernetes 프로젝트는 staging 디렉토리에 정의된 모듈을 내부적으로 사용하며, Go 모듈의 replace 지시문을 활용하여 경로를 지정한다. 이를 통해 Kubernetes 프로젝트는 자체 코드를 의존성으로 처리하고, 외부에 독립적인 라이브러리로 제공할 수 있다.

staging 구조는 Kubernetes의 모노레포를 점진적으로 분리하여, 독립적인 마이크로 리포지토리로 전환하는 전략의 일부다. 이는 코드 재사용성과 커뮤니티 기여를 촉진하고, 특정 라이브러리의 업데이트가 Kubernetes 전체 코드베이스에 영향을 미치지 않도록 설계되었다.

관련한 전체적인 개발 프로세스는 다음과 같다.

1. 개발자가 Kubernetes 메인 리포지토리에 코드 변경(PR 생성).
2. staging 코드 변경 사항 감지:
	- Publishing Bot이 변경을 감지하고, 동기화 규칙에 따라 작업 실행.
3. 외부 리포지토리로 코드 복사:
	- rules.yaml에 지정된 대로 외부 리포지토리에 PR 생성.
4. 의존성 업데이트:
	- 관련 리포지토리 간 의존성을 업데이트하고, 테스트를 통해 검증.

꽤나 멋진 구조라고 생각한다.

#### pkg
 cmd/kube-apiserver/.import-restrictions 를 체크해보면, k8s.io/kubernetes/pkg 항목이 존재하는 것을 볼 수 있다.

```
rules:
  - selectorRegexp: k8s[.]io/kubernetes
    allowedPrefixes:
      - k8s.io/kubernetes/cmd/kube-apiserver
      - k8s.io/kubernetes/pkg
      - k8s.io/kubernetes/plugin
      - k8s.io/kubernetes/test/utils
      - k8s.io/kubernetes/third_party

```

pkg 폴더는 golang에서 흔히 사용되는 레포지토리 레이아웃 관점에서 **내부 구현 세부사항**과 **공통 유틸리티**를 담는 공간이지만, 근본적으로 외부 사용자가 사용할 수 있는 코드 세트이다. 즉, 이 폴더는 프로젝트 내부와 외부 모두에서 재사용 가능한 코드를 패키지 형태로 제공한다.

실제 사용 예시로 cmd/kube-apiserver/app/server.go 파일의 임포트 항목을 살펴보면 다음과 같다. 거의 대부분의 임포트 항목의 경로에 pkg 폴더가 함유 됨을 알 수 있다.
``` go
package app

import (
	"context"
	"fmt"
	"net/url"
	"os"

	"github.com/spf13/cobra"
	apiextensionsv1 "k8s.io/apiextensions-apiserver/pkg/apis/apiextensions/v1"
	utilerrors "k8s.io/apimachinery/pkg/util/errors"
	utilruntime "k8s.io/apimachinery/pkg/util/runtime"
	"k8s.io/apiserver/pkg/admission"
	genericapifilters "k8s.io/apiserver/pkg/endpoints/filters"
	genericapiserver "k8s.io/apiserver/pkg/server"
	"k8s.io/apiserver/pkg/server/egressselector"
	serverstorage "k8s.io/apiserver/pkg/server/storage"
	utilfeature "k8s.io/apiserver/pkg/util/feature"
	"k8s.io/apiserver/pkg/util/notfoundhandler"
	"k8s.io/apiserver/pkg/util/webhook"
	clientgoinformers "k8s.io/client-go/informers"
	"k8s.io/client-go/rest"
	cliflag "k8s.io/component-base/cli/flag"
	"k8s.io/component-base/cli/globalflag"
	"k8s.io/component-base/featuregate"
	"k8s.io/component-base/logs"
	logsapi "k8s.io/component-base/logs/api/v1"
	_ "k8s.io/component-base/metrics/prometheus/workqueue"
	"k8s.io/component-base/term"
	utilversion "k8s.io/component-base/version"
	"k8s.io/component-base/version/verflag"
	"k8s.io/klog/v2"
	aggregatorapiserver "k8s.io/kube-aggregator/pkg/apiserver"
	"k8s.io/kubernetes/cmd/kube-apiserver/app/options"
	"k8s.io/kubernetes/pkg/capabilities"
	"k8s.io/kubernetes/pkg/controlplane"
	controlplaneapiserver "k8s.io/kubernetes/pkg/controlplane/apiserver"
	"k8s.io/kubernetes/pkg/controlplane/reconcilers"
	kubeapiserveradmission "k8s.io/kubernetes/pkg/kubeapiserver/admission"
)
```

golang을 다루다보면 패키지의 순환 의존성을 조심할 필요가 있다. 특정 패키지에서 호출(import)되는 패키지는, 자신을 호출하는 패키지를 호출 하면 안된다. 특히 모노레포 생태계인 K8s 생태계에서는 조심해야한 성질이며, 이러한 부분을 아마도 현명하게 컨트롤하고 있을 것이라 생각한다. `.import-restrictions` 가 그 노력 중 하나이다. **빌드 프로세스에서 위반 여부를 자동으로 검증** 한다고 한다.

또한, 패키지 간 호출의 변화는 코드 리뷰 프로세스를 통해 검증 된다. **OWNERS 파일** 에서는 패키지 간의 관리자를 정하고 있는데, 특정 디렉터리와 관련된 코드 변경은 해당 디렉터리의 전문가(Reviewer 또는 Approver)에 의해 검토되게 되는 시스템이다. 대표적으로 pkg/api/OWNERS 를 살펴보면 다음과 같다.

```
# See the OWNERS docs at https://go.k8s.io/owners

# Disable inheritance as this is an api owners file
options:
  no_parent_owners: true
filters:
  ".*":
    approvers:
      - api-approvers
    reviewers:
      - api-reviewers
  # examples:
  #   pkg/api/types.go
  #   pkg/api/*/register.go
  "([^/]+/)?(register|types)\\.go$":
    labels:
      - kind/api-change

```

패키지 구조는 가급적이면 2~3단계 이상의 폴더보다 더 깊게 생성되지 않는다. 또한 테스트 파일도 많이 작성해 둔다. kubernetes/pkg/api 경로를 예를 들면 이러하다. 

``` shell
$ tree .
.
├── endpoints
│   └── testing
│       └── make.go
├── job
│   ├── warnings.go
│   └── warnings_test.go
├── legacyscheme
│   └── scheme.go
├── node
│   ├── util.go
│   └── util_test.go
├── OWNERS
├── persistentvolume
│   ├── util.go
│   └── util_test.go
├── persistentvolumeclaim
│   ├── OWNERS
│   ├── util.go
│   └── util_test.go
├── pod
│   ├── OWNERS
│   ├── testing
│   │   └── make.go
│   ├── util.go
│   ├── util_test.go
│   ├── warnings.go
│   └── warnings_test.go
├── service
│   ├── OWNERS
│   ├── testing
│   │   └── make.go
│   ├── util.go
│   ├── util_test.go
│   ├── warnings.go
│   └── warnings_test.go
├── servicecidr
│   ├── servicecidr.go
│   └── servicecidr_test.go
├── storage
│   ├── util.go
│   └── util_test.go
├── testing
│   ├── applyconfiguration_test.go
│   ├── backward_compatibility_test.go
│   ├── compat
│   │   └── compatibility_tester.go
│   ├── conversion.go
│   ├── conversion_test.go
│   ├── copy_test.go
│   ├── deep_copy_test.go
│   ├── defaulting_test.go
│   ├── doc.go
│   ├── fuzzer.go
│   ├── install.go
│   ├── meta_test.go
│   ├── node_example.json
│   ├── OWNERS
│   ├── replication_controller_example.json
│   ├── serialization_proto_test.go
│   ├── serialization_test.go
│   └── unstructured_test.go
└── v1
    ├── endpoints
    │   ├── util.go
    │   └── util_test.go
    ├── OWNERS
    ├── persistentvolume
    │   ├── util.go
    │   └── util_test.go
    ├── pod
    │   ├── util.go
    │   └── util_test.go
    ├── resource
    │   ├── helpers.go
    │   └── helpers_test.go
    └── service
        ├── util.go
        └── util_test.go
```

#### 이외
위에서 살펴본 폴더 이외에도 test, hack, build, api, plugin 등등의 여러 폴더가 존재한다. 재미 삼아  api/openapi-spec/swagger.json 파일을 열어보면, 8만 줄에 가까운 엄청난 길의 파일도 만나볼 수 있다. ( 아마도 자동으로 생성될 것이라 예상한다 ) 

K8s는 탄생 10주년을 맞이하는 정말 방대한 프로젝트이다. 한 번에 모든 컴포넌트를 분석할 순 없다. 처음부터 코드를 분석하는 것은 아주 어려운 일이다. 거의 대부분의 경우 필요에 의해 코드를 찾아가는 것이 일반적이다. 그러다 우연히 처음 보는 영역에 발을 들이는 순간에 각 폴더 별로 존재하는 README.md 파일을 첫 지침서로서 활용하는 편이 좋겠다.

### 정리
Kubernetes는 컨테이너 오케스트레이션을 넘어 분산 시스템의 다양한 요구를 충족시키는 플랫폼이다. 이번 글에서는 Kubernetes 코드베이스의 주요 구조를 살펴보았으며, cmd, staging, pkg 디렉터리의 역할과 목적을 분석했다. 이를 통해 Kubernetes 내부 구조를 이해하고, 필요에 따라 기능을 커스터마이징하거나 디버깅하는 데 활용할 수 있다.

Kubernetes를 제대로 이해하려면 코드 구조를 아는 것에서 그치지 않고, 클러스터가 실제로 동작하는 원리를 탐구해야 한다. 특히, “파드 생성 과정”은 Kubernetes의 핵심 기능 중 하나로, 클러스터의 주요 컴포넌트가 어떻게 상호작용하며 워크로드를 배포하는지를 보여주는 중요한 흐름이다.

다음 글에서는 Kubernetes의 파드 생성 과정을 다루며, 클러스터 내부에서 일어나는 동작 원리를 구체적으로 파악 해보고자 한다.