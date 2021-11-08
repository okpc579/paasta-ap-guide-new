### [Index](https://github.com/okpc579/paasta-guide-new/blob/main/README.md) > [AP Architecture](../README.md) > PaaS-TA AP

## 목적
본 문서는 PaaS-TA Application Platform (AP)의 Architecture를 제공한다.
<br><br>

## 시스템 구성도
![PaaS-TA AP Component](image/ap_architecture_component.png)



| 구분  | 인스턴스 수| 스펙 |
|-------|----|-----|
| api | N | 1vCPU / 512MB RAM / 4GB Disk 10GB(영구적 Disk) |

## 설명
PaaS-TA AP는 개발자 프레임워크 및 앱 서비스를 선택할 수 있는 PaaS(Platform as a Service) 플랫폼이다.  
PaaS-TA AP를 사용하면 어플리케이션을 더 빠르고 쉽게 구축, 테스트 배포 및 확장할 수 있다.

다음은 어플리케이션 배포 흐름이다.
<br>  

![PaaS-TA AP run_map](image/ap_architecture_run_map.png)
![PaaS-TA AP run_map_detail](image/ap_architecture_run_map_detail.png)


### [Index](https://github.com/okpc579/paasta-guide-new/blob/main/README.md) > [AP Architecture](../README.md) > PaaS-TA AP
