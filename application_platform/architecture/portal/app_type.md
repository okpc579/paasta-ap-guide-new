### [Index](../../../README.md) > [AP - Architecture](../README.md) > Portal Architecture - APP Type

## 목적
본 문서(Portal Architecture - APP Type)는 AP Portal - APP TYPE의 Architecture를 제공한다.
<br><br>

## 시스템 구성도
APP TYPE의 AP Portal은 Portal Infra와 Portal APP으로 나뉘어있다.  
Portal Infra, Portal APP의 구성과 스펙은 다음과 같다.  
<br>



![Portal Architecture - APP Type](./image/portal_architecture_app.png)

<br>

* Paas-TA Portal infra VM   

| 구분 | 스펙 |
|---------|-------|
| infra (mariadb / binary storage) | 1vCPU / 512MB RAM / 10GB Disk 20GB(영구적 Disk) |

* Paas-TA Portal App

| App명 | 인스턴스 수 | 메모리 | 디스크 |
|--------|-------|-------|-------|
| portal-registration | 1 | 1G | 1G|
| portal-gateway | 1 | 1G | 1G|
| portal-api | N | 2G | 2G|
| portal-common-api | N | 1G | 1G|
| portal-storage-api | N | 1G | 1G|
| portal-log-api | N | 1G | 1G|
| portal-web-admin | N | 1G | 1G|
| portal-web-user | N | 1G | 1G|  



### [Index](../../../README.md) > [AP - Architecture](../README.md) > Portal Architecture - VM Type
