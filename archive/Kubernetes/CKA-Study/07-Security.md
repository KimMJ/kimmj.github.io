---
title: "07 Security"
date:  2020-04-25T20:26:41+09:00
weight: 7
draft: false
tags: ["kubernetes", "cka"]
pre: "<i class='fas fa-minus'></i>&nbsp;"
---

## Kubernetes Security Primitives

* host와 cluster 자체의 security
  * 당연히 host는 그 자체로 안전해야하고, root access는 disable 되어야 하며 password based authentication은 disabled되고 SSH key based authentication만 가능해야 한다.
  * Kubernetes를 실행시키고 있는 호스트에 대한 physical, virtual infrastructure에 대한 보안도 필요하다.
* 쿠버네티스와 관련된 security
  * `kube-apiserver`는 모든것을 하는 주체가 되기 때문에 이곳에 대한 보안부터 챙겨야 한다.
  * Who can access? / What can they do?
  * API 서버에 누가 접근을 할 수 있게 할 것인가?
    * Files - Username and Passwords
    * Files - Username and Tokens
    * Certificates
    * External Authentication providers - LDAP
    * Service Accounts
  * What can they do
    * RBAC Authorization
    * ABAC Authorization
    * Node Authorization
    * Webhook Mode
  * TLS Certificates
    * 모든 controller component들은 TLS Encryption이 되어있다.
  * Cluster 내의 application에서의 communication
    * 모든 파드는 클러스터 내의 모든 파드로 access 할 수 있다.
    * `Network Policies`를 통해 access를 제한할 수 있다.

## Authentication

* Admin, Developers, End Users, Third Party가 클러스터에 접근한다.
* 내부 컴포넌트에서의 통신에 보안성을 추가하는 방법과 authentication, authorization 메카니즘으로 클러스터에 접근하는 것에 security management를 할 것이다.
* 그 중 authentication 메커니즘에 대해 집중할 것이다.
* End user에 대한 보안은 어플리케이션 내에서 해야한다.
* 두가지 방식으로 구분해보면 관리자, 개발자같은 사람인 User와 자동화 프로세스 등을 하는 로봇인 Service Accounts로 구분지을 수 있다.
* 쿠버네티스는 유저관리같은 것을 원래 지원하지는 않는다.
* 하지만 Service Account는 쿠버네티스가 지원한다.
* `kube-apiserver`는 authenticate user -> proccessing 의 순서로 동작한다.
* authenticate machanisms
  * Static Password File
  * Static Token File
  * Certificates
  * Identity Services(LDAP)
* static password, token file
  * `kube-apiserver`에 `csv`형태로 `<password>,<user name>,<user ID>`를 저장한뒤 전달하면 이를 사용할 수 있다.
    * `--basic-auth-file=user-details.csv` 옵션을 `kube-apiserver`에 추가하여 적용. restart 필요
    * `kubeadm`을 사용했다면 pod definition file을 수정해야 한다.
    * 이는 `curl` 사용 시 `-u "user1:password123"`의 형태로 사용할 수 있다.
    * `csv` 파일에 4번째 column을 추가할 수 있는데 이는 `<group ID>`를 의미한다.
  * 이와 비슷하게 `csv` 형태로 `<token>,<user name>,<user ID>,<group ID>`를 저장할 수 있다.
    * `--token-auth-file=user-details.csv` 형태로 적용 가능
    * `curl` 사용 시 `--header "Authorization: Bearer <token>"` 형태로 적용이 가능.
* static file로 설정하는 것은 추천하는 방법은 아니다.
* consider volume mount while providing the auth file in a kubeadm setup.

## TLS Basics

* transaction에서 두 party를 guarantee하기 위해 사용된다.
* user가 web server에 접근한다고 생각해보면 둘 간의 통신이 encrypt 되었음을 보장하기 위해 TLS를 사용한다.
* 데이터를 서버에 전송할 때 key를 통해 encrypt 되기 때문에 이를 받는 서버 또한 key를 가지고 있어야 복호화가 능하다.
  * 여기서 암호화/복호화에 동일한 key를 사용하는 것을 Symmetric Encription이라고 한다.
  * 해커가 key를 전송하는 과정 중에 이를 탈취해서 복호화할 가능성이 있다.
* asymmetric encryption
  * private key와 public key 두개로 구성
  * 간단하게 생각한다면 private key - public lock의 구성이라고 보면 된다.
  * private key는 나만 가지고 있고 public key는 공개하여 누구나 암호화할 수 있도록 한다.
* SSH를 asynmmetric으로 접근하기
  * `ssh-keygen`을 통해 private key와 public key를 생성한다.
  * 서버를 public key를 통해 암호화한다.
    * public key를 서버에 추가하면 된다.
    * `~/.ssh/authorized_keys`에 추가.
    * 그 뒤 `ssh -i <private key file> user@server`로 접속하면 된다.
  * 더 많은 서버에 접속한다면 pulic key만 서버들에 대해 복사하면 된다.
  * 다른 사람도 접근하고자 한다면 private key를 추가하면 된다.
* symmetric encryption에서 key를 함께 보낼 때 key를 해커가 탐지할 수 있다.
  * 이를 asymmetric encryption으로 key를 암호화한 뒤 보내면 해결 가능.
  * private key 생성: `openssl genrsa -out my-bank.key 1024`
  * public key 생성: `openssl rsa -in my-bank.key -pubout > mybank.pem`
* 유저가 https로 서버에 최초 접근하면 public key를 서버로부터 받게 된다.
  * 이 때 자신의 symmetric key를 서버의 Public key로 encryption한 뒤 전송한다.
  * 이를 받으면 서버는 자신의 private key로 복호화를 진행한다.
  * 해커가 이를 가져가도 어떤것을 할 수 없다.
  * 이를 통해 서버와 유저는 symmetric key를 통해 서로 통신한다.
* 해커는 replica 서버를 만들어서 유저의 symmetric key를 탈취하려 할 것이다.
  * 해커의 서버를 보면 key와 함께 certificate를 전송한다.
  * 이 certificate 안에는 Issuer가 있다.
    * 이는 그 서버의 identity의 유효성을 검증하는데 매우 중요하다.
  * Subject은 그 certificate에서 인증하는 서버를 나타낸다.
* certificate는 누구나 만들 수 있기 때문에 누가 certificate를 sign하는지가 중요하다.
  * 스스로 만들었다면 자신이 sign할 것이고 이를 self-signed certificates라고 한다.
  * 이 self-signed certificates는 안전하지 않은 인증서이다.
  * 따라서 해커가 사용하는 certificates도 self-signed certificates이다.
* 웹 브라우저는 자동으로 이런 self-signed certificates에 대해 경고를 해준다.
* 다른 3자에게 인증을 받고자 한다면 Certificates Authority(CA)가 필요하다.
  * 이를 위해서는 Certificate Signing Request(CSR)을 전송해야한다.
  * `openssl req -new -key my-bank.key -out my-bank.csr -subj "/C=US/ST=CA/O=MyOrg, Inc./CN=my-bank.com`
  * CA는 이를 적절히 확인하고 sign 후 응답을 준다.
  * 해커가 CA로부터 인증을 받으려 하면 validation에서 fail이 날 것이다.
  * 그러면 web browser에서 올바른 certificate임을 간주한다.
* Web browser는 어떻게 CA가 올바른 CA라고 판정을 하는가
  * CA가 만약 fake CA라면?
  * 어떻게 certificate에 서명된 CA가 진짜 CA라고 판정할 수 있는가.
  * CA는 private key를 브라우저 내에 심어놓는다.
  * 이를 통해 web browser는 자신에게 온 sign이 진짜 CA가 전송한 것인지 확인이 가능해진다.
* PKI(Public Key Infrastructure)
* nameing
  * Public Key는 보통 `*.crt`, `*.pem` 형식이다.
  * Private Key는 보통 `*.key`, `*-key.pm`
    * 보통 key라는 단어를 섞는다.

## TLS in Kubernetes - Certificate Creation

* certificate 생성 방법: `easyrsa`, `openssl`, `cfssl`
* certificate 생성 절차:
  * public key 생성: `openssl genrsa -out ca.key 2048`
  * certificate signing request: `openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr`
  * sign certificates: `openssl x509 -req -in ca.csr -singkey ca.key -out ca.crt`
* client certificate 생성 절차(admin user)
  * private key 생성: `openssl genrsa -out admin.key 2048`
  * certificate signing request: `openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr`
    * 반드시 `kube-admin`일 필요는 없고 `kubectl`에 사용되는 이름이면 된다.
  * sign certificates: `openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt`
    * `ca.key`로 certificate를 validate한다.
  * admin user라는 것을 certificate에 담으려면 group을 추가하면 된다.
    * `system-master`를 사용해야 한다.
    * certificate sigining request에 `/O=system:masters`를 추가하여 나타냄.
* system component들은 certificate에서 앞에 `SYSTEM:`이라는 prefix가 필요하다.
  * `kube-scheduler`
  * `kube controller manager`
  * `kube-proxy`
* 여기서 생성한 certificate를 사용하는 방법
  * `curl <url> --key <private key> --cert <certificate> --cacert ca.crt`
  * 또는 `kubeconfig`에 이 내용을 추가한다.
* CA certificate는 우리가 만든 것이기 때문에 파라미터로 넣을때 같이 넣어줘야 인증이 원할하게 된다.
* `kube-apiserver`는 IP, domain name이 다양하다.
  * `kubernetes`, `kubernetes.default`, `kubernetes.default.svc`, `kubernetes.default.svc.cluster.local`
  * 이런 것들이 모두 certificate에 기록되어야 한다.
  * 방법
    * certificate signing request시 config 파일을 두어 여기에다가 기록해둔다.
    * ```config
      [req]
      req_extensions = v3_req
      [v3_req]
      basicConstraints = CA:FALSE
      keyUsage = nonRepudiation,
      subjectAltName = @alt_names
      [alt_names]
      DNS.1 = kubernetes
      DNS.2 = kubernetes.default
      DNS.3 = kubernetes.default.svc
      DNS.4 = kubernetes.default.svc.cluster.local
      IP.1 = 10.96.0.1
      IP.2 = 172.17.0.87
      ```
    * 그 다음 `openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr -config openssl.cnf`처럼 config 파일을 지정한다.
* `kubelet`의 경우 노드마다 떠있게 되는데 여기에 대한 certificate의 name은 노드의 이름을 따온다.
  * `kubelet-config.yaml`에 certificate에 대한 정보를 지정한다.
  * 또한 `system:node:` prefix를 넣는다.
  * 그리고 `SYSTEM:NODES` 그룹에 넣어야 한다.

## View Certificate Details

* 이미 설치된 클러스터 내에서 certificate을 확인하는 방법
* 쿠버네티스가 어떤 방식으로 설치되어 있는지 알아야 한다.
* scartch로 구성을 할 경우 모든 certificate을 스스로 생성한다.
* `kubeadm`같은 automation tool을 이용할 경우 모든 certificate을 알아서 구성해준다.
* `kubeadm`
  * `/etc/kubernetes/manifests/kube-apiserver.yaml`
  * 여기서 `--client-ca-file`, `--etcd-cafile`, `--etcd-certfile`, `--etcd-keyfile`, `--kubelet-client-certificate`, `--kubelet-client-key`, `--tls-cert-file`, `--tls-private-key` 확인.
  * `--tls-cert-file=/etc/kubernetes/pki/apiserver.crt`로 설정되어 있을 경우 다음의 명령어로 조회 가능.
    * `openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout`
* `journalctl -u etcd.service -l`로 로그 확인

## Certificates API

* 새로운 admin이 들어오면, Certificates Request를 작성하게 한 뒤 기존 admin에게 전송하고, 이를 CA 서버에 전달하여 certificate을 생성한 뒤 되돌려준다.
  * 이 경우 certificate이 expire되면 다시 이 프로세스대로 반복해야한다.
* CA 서버가 있으면 여기에 접속할 수 있는 사람들이 certificate를 마음껏 생성할 수 있다.
  * 매우 안전한 서버에다가 구성해야한다.
  * certificate에 접속하고자 할 경우 반드시 그 서버를 통해야 한다.
* 쿠버네티스는 Certificate API를 보유하고 있어서 다음과 같은 작업을 할 수 있다.
  * admin이 Certificate Singing Request를 받으면 마스터 노드에 접속해서 직접 sign을 받는 것이 아니라, API를 통해서 한다.
  * `CertificateSigningRequest` Object를 생성한다.
  * 이를 통해 Request를 받고 Approve한 뒤 User에게 Certificate를 전송한다.
* Flow
  * `openssl genrsa -out jane.key 2048`
  * `openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr`
  * 이를 `base64`로 인코딩해서 `request`에 담는다.
  * `kubectl get csr`을 통해 조회해보면 `status.certificates`에 결과가 나온다.
  * 이 결과를 `base64`로 decode한다.
* csr에 관련된 것은 controller manager에서 관리한다.

## Kubeconfig

* 일반적으로 curl 명령어를 통해 `kube-apiserver`와 통신을 할 수 있다.
  * 
    ```bash
    curl https://my-kube-playground:6443/api/v1/pods \
        --key admin.key \
        --cert admin.crt \
        --cacert ca.crt
    ```
* `kubectl`을 이용할 때는 다음과 같이 사용할 수 있다.
  * 
    ```bash
    kubectl get pods --server my-kube-playground:6443 \
        --client-key admin.key \
        --client-certificate admin.crt \
        --certificate-authority ca.crt
    ```
  * 이를 매번 치는 것은 귀찮으니 이를 `kubeconfig` 파일에 넣을 수 있다.
* `kubeconfig`의 default directory는 `$HOME/.kube/config`
* 3가지 섹션으로 나뉨.
  * cluster
    * 접속할 클러스터
  * users
    * 클러스터에 접속하는 유저
  * contexts
    * 클러스터와 유저를 연결하는 것
* 이미 존재하는 user를 골라야 한다.
* `kubeconfig`에서 `contexts`에 특정한 네임스페이스를 사용하도록 지정할 수 있다.

## API Groups

* 쿠버네티스에서 api는 각각의 목적에 따라 그룹으로 구성되어 있다.
* 크게 `core` 그룹과 `named` 그룹으로 나뉜다.
* `core` group
  * `/api` -> `/v1` -> `namespaces`, `pods` ...
* 새로운 feature같은 것들은 `named` 그룹
  * `/apis` -> `/apps`, `extensions`, `/networking.k8s.io`, `storage.k8s.io`, `/authentication.k8s.io`, `certificates.k8s.io`
  * 이 `/apps`, `/extensions` 등을 API group이라고 한다.
  * 그 아래에 해당하는 api들을 resources라고 부르고, 각 resource는 특정한 verbs를 가지고 있다.
* api 문서를 보면 그룹에 관한 자세한 정보가 있다
* `kube-apiserver`로 요청을 보내면 사용할 수 있는 api 버전들을 보여준다.
* 그러나 certificates들을 기입해야 한다.
* 이를 안하려면 `kubectl proxy`를 사용하면 된다.
* 그러면 certificates를 사용하지 않을 수 있다.
* `kubeproxy`와 `kubectl proxy`는 다른 것이다.

## Role Based Access Controls

* `kind=Role`을 통해 어떤 api group, resources, verbs를 사용할 수 있는지 정의할 수 있다.
* `core` 그룹을 사용하려면 api group을 공백으로 두면 된다.
* 이렇게 정의한 role을 유저에게 적용시키려면 `kind=RoleBinding`이라는 것을 정의하면 된다.
* `kubectl get roles` 및 `kubectl get rolebindings`를 통해 확인할 수 있다.
* `kubectl auth can-i`를 통해 어떤 것을 할 수 있는지 확인할 수 있다.
  * `--as`를 통해 user를 특정할 수 있다.
  * `--namespace`를 통해 네임스페이스에 관해서도 볼 수 있다.
* `resourceNames` 필드를 통해 role이 적용되는 resource의 이름을 지정할 수 있다.

## Cluster Roles

* `Role`과 `RoleBindings`는 네임스페이스 아래에 있다.
* 네임스페이스를 지정하지 않으면 default 네임스페이스에 대해서 적용된다.
* 노드는 clusterwide resource이기 때문에 특정 네임스페이스에 대해서 지정할 수 없다.
* 리소스는 네임스페이스 또는 클러스터 스코프로 정의된다.
* cluster scoped
  * `nodes`, `PV`, `clusterroles`, `clusterrolebindings`, `certificatesigningrequests` `namespaces`
* `kubectl api-resources --namespaced=true`, `kubectl api-resources --namespaced=false`
* cluster wide인 리소스에 대해서 `Role`과 `RoleBindings`를 설정하는 방법은?
* `ClusterRole`을 설정한다.
  * 이걸로 전체 네임스페이스에 관한 resource에 대한 권한을 가지기 위해 설정할 수도 있다.

## Image Security

* private repository를 구축해서 외부로 노출되지 않은 이미지 저장소를 만들 수 있다.
* image path를 full path를 써서 사용하면 된다.
* credential이 필요하다면 `secret`을 만들어 `docker-registry` type을 사용하면 된다. 이는 built-in type이다.
  * 이를 `podSpec`의 `imagePullSecrets`에 추가하면 된다.

## Security Contexts

* `securityContexts`를 지정하면 기존 docker image에 정의된 security context를 overwrite할 수 있다.
  * `capability`를 설정하여 특정한 capability를 부여할 수 있다.
  * 이는 파드가 아닌 컨테이너 단위로만 설정 가능하다.

## Network Policy

* web - api - db가 있을 때 웹을 기준으로 user로부터 들어오는 트래픽을 `ingress traffic`, app server로 나가는 트래픽을 `egress traffic`이라고 한다.
  * 요청에 대한 응답은 크게 관련 없음.
* 파드는 라우팅같은 세팅을 통해 어떤 파드에도 접근할 수 있어야 한다.
* network policy는 리소스로, 파드에 대해 특정한 룰을 통과하는 트래픽만 흐르게 해주고 나머지는 블럭처리한다.
* `labels`로 `network policy`와 `pod`를 연결시킨다.
* Flannel은 `network policies`를 지원하지 않는다.

## Tips

* Service Account는 CKAD의 영역.
