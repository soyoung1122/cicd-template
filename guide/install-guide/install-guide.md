# 배포 가이드

[1. 사전 설정](#1-사전-설정)<br>
- [1.1. 네임스페이스 생성](#11-네임스페이스-생성)<br>
- [1.2. 설정 파일 다운로드](#12-설정-파일-다운로드)<br>
[2. Gitlab 설정](#2-gitlab-배포-및-설정)<br>
- [2.1. Gitlab 배포](#21-gitlab-배포)<br>
- [2.2. 접속 정보 확인](#22-접속-정보-확인)<br>
[3. ArgoCD 설정](#3-argocd-배포-및-설정)<br>
- [3.1. ArgoCD 배포](#31-argocd-배포)<br>
- [3.2. 접속 정보 확인](#32-접속-정보-확인)<br>
[4. Jenkins 배포 및 설정](#4-jenkins-배포-및-설정)<br>
- [4.1. Jenkins 배포](#41-jenkins-배포)<br>
- [4.2. 접속 정보 확인](#42-접속-정보-확인)<br>

# 소개

쿠버네티스 클러스터에 자체 CI/CD 파이프라인을 구축하는 방법에 대한 가이드를 다루고 있다. 

<br>

## 주요 소프트웨어 (참고)

<br>

|주요 소프트웨어|버전|
|---|---|
|ArgoCD|3.0.11|
|Gitlab|15.5.9|
|Jenkins|2.504.3|

<br>

# 1. 사전 설정

## 1.1. 네임스페이스 생성

##### 클러스터에 접근하여 `cicd` 네임스페이스를 생성한다.

    $ kubectl create ns cicd

## 1.2. 설정 파일 다운로드

##### 아래의 명령어를 실행하여 설정 파일을 다운로드한다.

    $ cd ~ 
    $ git clone https://github.com/soyoung1122/cicd-template.git # 추후 변경

##### 다운로드를 확인한다.

    $ cd cicd-template/manifests # 추후 변경    
    $ ls
    argocd.yaml  gitlab.yaml  jenkins.yaml

# 2. Gitlab 배포 및 설정

## 2.1. Gitlab 배포

##### 아래의 명령어를 실행하여 Gitlab을 배포한다.

    $ kubectl apply -f gitlab.yaml 
    persistentvolumeclaim/gitlab-config created
    persistentvolumeclaim/gitlab-data created
    secret/gitlab-root-password created
    serviceaccount/gitlab created
    role.rbac.authorization.k8s.io/gitlab-secret-editor created
    rolebinding.rbac.authorization.k8s.io/gitlab-secret-editor-binding created
    deployment.apps/gitlab created
    service/gitlab created

##### 배포 확인

    $ kubectl get pods -n cicd -l app=gitlab
    NAME                     READY   STATUS    RESTARTS   AGE
    gitlab-7c8f4bcb8-thlg9   1/1     Running   0          2m20s

## 2.2. 접속 정보 확인 

##### 아래의 명령어를 실행하여 비밀번호를 확인한다. 

    $ kubectl -n cicd get secret gitlab-root-password -o jsonpath="{.data.password}" | base64 -d; echo
    1YNO1nRY1ob5wF4

##### 아래의 명령어를 실행하여 접속 포트 번호를 확인한다. 

    $ kubectl get svc -n cicd -l app=gitlab
    NAME     TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                     AGE
    gitlab   NodePort   198.19.211.101   <none>        80:30819/TCP,22:31532/TCP   4m5s

##### 접속 정보 예시

    URL : http://<Node Public IP>:30819
    ID  : root
    PW  : 1YNO1nRY1ob5wF4

# 3. ArgoCD 배포 및 설정

## 3.1. ArgoCD 배포

##### 아래의 명령어를 실행하여 ArgoCD를 배포한다.

    $ kubectl apply -f argocd.yaml
    serviceaccount/argocd-sa created
    role.rbac.authorization.k8s.io/argocd-role created
    rolebinding.rbac.authorization.k8s.io/argocd-rb created
    job.batch/argocd-redis-secret-init created
    secret/argocd-notifications-secret created
    secret/argocd-secret created
    configmap/argocd-cm created
    configmap/argocd-cmd-params-cm created
    configmap/argocd-gpg-keys-cm created
    configmap/argocd-notifications-cm created
    configmap/argocd-rbac-cm created
    configmap/argocd-ssh-known-hosts-cm created
    configmap/argocd-tls-certs-cm created
    configmap/argocd-redis-health-configmap created
    customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
    customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io created
    customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
    clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
    clusterrolebinding.rbac.authorization.k8s.io/argocd-crb created
    service/argocd-applicationset-controller created
    service/argocd-repo-server created
    service/argocd-server created
    service/argocd-dex-server created
    service/argocd-redis created
    deployment.apps/argocd created

##### 배포 확인

    $ kubectl get pods -n cicd -l app.kubernetes.io/name=argocd
    NAME                             READY   STATUS      RESTARTS      AGE
    argocd-c5f5f5d7-fhkr6            7/7     Running     1 (79s ago)   87s

## 3.2. 접속 정보 확인

##### 아래의 명령어를 실행하여 비밀번호를 확인한다.

    $ kubectl -n cicd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
    7e0DRTASxZpH-RU6

##### 아래의 명령어를 실행하여 접속 포트 번호를 확인한다. 

    $ kubectl get svc -n cicd -l app=argocd
    NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
    argocd-server   NodePort   198.19.198.128   <none>        80:30723/TCP,443:30774/TCP   20m

##### 접속 정보 예시

    URL : http://<Node Public IP>:30713
    ID  : admin 
    PW  : 7e0DRTASxZpH-RU6

# 4. Jenkins 배포 및 설정

## 4.1. Jenkins 배포

##### 아래의 명령어를 실행하여 Jenkins를 배포한다. 

    $ kubectl apply -f jenkins.yaml 
    serviceaccount/jenkins created
    secret/jenkins created
    configmap/jenkins created
    configmap/jenkins-jenkins-jcasc-config created
    persistentvolumeclaim/jenkins created
    role.rbac.authorization.k8s.io/jenkins-schedule-agents created
    role.rbac.authorization.k8s.io/jenkins-casc-reload created
    rolebinding.rbac.authorization.k8s.io/jenkins-schedule-agents created
    rolebinding.rbac.authorization.k8s.io/jenkins-watch-configmaps created
    service/jenkins-agent created
    service/jenkins created
    statefulset.apps/jenkins created

##### 배포 확인

    $ kubectl get pods -n cicd -l app=jenkins
    NAME        READY   STATUS    RESTARTS   AGE
    jenkins-0   2/2     Running   0          10m

## 4.2. 접속 정보 확인

##### 아래의 명령어를 실행하여 비밀번호를 확인한다. 

    $ kubectl get secret jenkins -n cicd \
        -o jsonpath="{.data.jenkins-admin-password}" | base64 --decod && echo
    9kMBAYeQTHLwsuqsk0GGGI

##### 아래의 명령어를 실행하여 접속 포트 번호를 확인한다.

    $ kubectl get svc -n cicd -l app=jenkins
    NAME      TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
    jenkins   NodePort   198.19.154.9   <none>        8080:31883/TCP   11m

##### 접속 정보 예시

    URL : http://<Node Public IP>:31883
    ID  : admin
    PW  : 9kMBAYeQTHLwsuqsk0GGGI

