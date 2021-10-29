### [Index](../../../README.md) > [AP - Architecture](../README.md) > Portal Architecture - VM Type

## Table of Contents
- [목적]()
- [시스템 구성도]()
<br>

## 목적
본 문서(Portal Architecture - VM Type)는 AP Portal - VM TYPE의 Architecture를 제공한다.
<br>

## 시스템 구성도
VM TYPE의 AP Portal은 Portal API와 Portal UI로 나뉘어있다.  
Portal API, Portal UI의 구성과 스펙은 다음과 같다.  

| Deployment | 구분  | 스펙 |
|------------|-------|-----|
| portal-api | binary_storage | 1vCPU / 512MB RAM / 4GB Disk 10GB(영구적 Disk) |
| portal-api | haproxy | 1vCPU / 512MB RAM / 4GB Disk|
| portal-api | mariadb | 1vCPU / 512MB RAM / 4GB Disk +10GB(영구적 Disk) |
| portal-api | paas-ta-portal-registration | 1vCPU / 512MB RAM / 4GB Disk |
| portal-api | paas-ta-portal-gateway | 1vCPU / 512MB RAM / 4GB Disk |
| portal-api | paas-ta-portal-api | 1vCPU / 1GB RAM / 4GB Disk |
| portal-api | paas-ta-portal-common-api | 1vCPU / 512MB RAM / 4GB Disk |
| portal-api | paas-ta-portal-log-api | 1vCPU / 512MB RAM / 4GB Disk |
| portal-api | paas-ta-portal-storage-api | 1vCPU / 512MB RAM / 4GB Disk |
| portal-ui | haproxy | 1vCPU / 512MB RAM / 4GB Disk|
| portal-ui | mariadb | 1vCPU / 512MB RAM / 4GB Disk +10GB(영구적 Disk) |
| portal-ui | paas-ta-portal-webadmin | 1vCPU / 512MB RAM / 4GB Disk |
| portal-ui | paas-ta-portal-webuser | 1vCPU / 512MB RAM / 4GB Disk|



![Portal Architecture - VM Type](image/portal_architecture_vm.png)







### [Index](../../../README.md) > [AP - Architecture](../README.md) > Portal Architecture - VM Type