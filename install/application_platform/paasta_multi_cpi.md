### [Index](https://github.com/okpc579/paasta-guide-new/blob/main/README.md) > [AP Install](../README.md) > PaaS-TA Multi CPI

## Table of Contents

1. [개요](#1)  
 1.1. [목적](#1.1)  
 1.2. [범위](#1.2)  
 1.3. [참고 자료](#1.3)  
2. [Multi CPI](#2)  
 2.1. [Prerequisite](#2.1)  
 2.2. [설치 파일 다운로드](#2.2)  
 2.3. [OpenVPN](#2.3)  
　2.3.1. [변수 설정](#2.3.1)       
　2.3.2. [인증서 생성](#2.3.2)       
　2.3.3. [OpenVPN 설치](#2.3.3)  
　2.3.4. [OpenVPN 연결 확인](#2.3.4)  
　2.3.5. [정적 라우팅 추가 (선택)](#2.3.5)  
 2.4. [Multi CPI 설정](#2.4)   
　2.4.1. [BOSH 설치](#2.4.1)    
　2.4.2. [Stemcell 업로드](#2.4.2)    
　2.4.3. [CPI Config 설정](#2.4.3)    
　　2.4.3.1. [같은 IaaS를 사용할 경우](#2.4.3.1)    
　　2.4.3.2. [다른 IaaS를 사용할 경우](#2.4.3.2)    
　2.4.4. [Cloud Config 설정](#2.4.4)    
　　2.4.4.1. [같은 IaaS를 사용할 경우](#2.4.4.1)    
　　2.4.4.2. [다른 IaaS를 사용할 경우](#2.4.4.2)    
　2.4.5. [Multi CPI 설정 테스트](#2.4.5)    
 
# <div id='1'/>1.  문서 개요

## <div id='1.1'/>1.1. 목적
본 문서는 BOSH2(이하 BOSH)의 Multi CPI 설정 가이드 문서로, 하나의 BOSH를 통하여 상이한 IaaS 환경에서 VM을 배포하는 Multi CPI를 설정하고 사용하는 방법에 관해서 설명하였다.

<br>

## <div id='1.2'/>1.2. 범위
multi-cpi-deployment는 paasta-deployment v5.6.2의 설치를 기준으로 가이드를 작성하였다.  
multi-cpi-deployment는 AWS, Openstack, vSphere에서 설정이 가능하다.

<br>

## <div id='1.3'/>1.3. 참고 자료

본 문서는 Cloud Foundry의 BOSH Document와 Open VPN을 참고로 작성하였다.

BOSH Document: [https://bosh.io](https://bosh.io)  
BOSH cpi-config Document: [https://bosh.io/docs/cpi-config](https://bosh.io/docs/cpi-config)  
BOSH guide-multi-cpi-aws Document: [https://bosh.io/docs/guide-multi-cpi-aws](https://bosh.io/docs/guide-multi-cpi-aws)  
BOSH Deployment: [https://github.com/cloudfoundry/bosh-deployment](https://github.com/cloudfoundry/bosh-deployment)  
OpenVPN : [https://openvpn.net/](https://openvpn.net/)  

<br><br>

# <div id='2'/>2.  Multi CPI
BOSH에 Multi CPI를 설정할 경우 하나의 BOSH를 통하여 상이한 IaaS 환경에서 VM을 각각 배포할 수 있다.  
본 가이드에서는 OpenVPN을 상이한 IaaS 환경에 각각 설치한 후, BOSH를 설치한 뒤 Multi CPI 설정을 진행한다.  
이 때 IaaS의 종류가 같을경우(e.g. A OpenStack <-> B OpenStack) 기존 BOSH 설치 가이드와 동일하게 BOSH를 설치하며, IaaS의 종류가 다를경우(e.g. Openstack <-> AWS) 추가 옵션을 설정하여 BOSH를 설치한다.  

## <div id='2.1'/>2.1. Prerequisite

본 가이드는 Linux 환경에서 진행하는 것을 기준으로 하였다.  
본 가이드는 BOSH에 대한 기본 이해도가 있다는 전제 하에 가이드를 진행하였다.  
또한 Multi CPI 설정를 위해서는 먼저 BOSH CLI가 설치 되어 있어야 한다.  
BOSH에 대한 기본 이해도가 없거나 BOSH CLI가 설치 되어 있지 않을 경우 먼저 BOSH 설치 가이드 문서를 참고 하여 BOSH CLI를 설치를 하고 사용법을 숙지 해야 한다.  

<br>


## <div id='2.2'/>2.2. 설치 파일 다운로드

- BOSH를 설치하기 위한 paasta-deployment와 Multi CPI 설정을 위한 multi-cpi-deployment가 존재하지 않는다면 다운로드 받는다

```
$ mkdir -p ~/workspace
$ cd ~/workspace
$ git clone https://github.com/PaaS-TA/paasta-deployment.git -b v5.6.2
$ git clone https://github.com/PaaS-TA/multi-cpi-deployment.git -b v5.6.2
```

<br>

## <div id='2.3'/>2.3. OpenVPN
BOSH가 상이한 IaaS와의 통신을 진행하기 위하여 OpenVPN을 각각 상이한 IaaS 환경에 설치를 진행한다.

### <div id='2.3.1'/>2.3.1. 변수 설정
```
# OpenVPN 설치를 위한 deployment 폴더 이동
$ cd ~/workspace/multi-cpi-deployment/openvpn
```

- BOSH를 설치 할 IaaS 환경에 설치되는 OpenVPN az1 변수를 설정한다.  
  (OpenVPN을 설치할 IaaS 환경에 대한 주석을 해제하고 변수를 설정한다.)
> $ vi vars-az1.yml
```
### openvpn default
wan_ip: "XXX.XXX.XXX.XXX"                         # Used by OpenVPN Server-1 ip 
push_routes: ["XXX.XXX.XXX.XXX 255.255.255.0"]    # ex) 10.0.10.0 255.255.255.0
lan_gateway: "XXX.XXX.XXX.XXX"                    # ex) 10.0.10.1
lan_ip: "XXX.XXX.XXX.XXX"                         # ex) 10.0.10.10
lan_network: "XXX.XXX.XXX.XXX"                    # ex) 10.0.10.0
lan_network_mask_bits: "24"                       # ex) 24
vpn_network: "XXX.XXX.XXX.XXX"                    # ex) 192.168.0.0 (az-1: 192.168.0.0, az-2: 192.168.1.0)
vpn_network_mask: "XXX.XXX.XXX.XXX"               # ex) 255.255.255.0
vpn_network_mask_bits: "24"                       # ex) 24
remote_network_cidr_block: "XXX.XXX.XXX.XXX/24"   # ex) 20.0.20.0/24
remote_vpn_ip: "XXX.XXX.XXX.XXX"                  # Used by OpenVPN Server-2 ip 

#### IaaS :: AWS
#access_key_id: "XXXXXXXXXXXXXXX"                  # AWS Access Key
#secret_access_key: "XXXXXXXXXXXXXXX"              # AWS Secret Key
#region: "ap-northeast-2"                          # AWS Region
#availability_zone: "ap-northeast-2a"              # AWS Region
#subnet_id: "paasta-subnet"                        # AWS Subnet ex) subnet-0ebc.....
#default_security_groups: ["bosh-sg"]              # AWS Security-Group
#bootstrap_ssh_key_name: "bosh-key"                # AWS SSH Private Key Name
#bootstrap_ssh_key_path: "/.ssh/bosh-key.pem"      # AWS SSH Private Key Path
#route_table_id: "XXXXXXXXXXXXXXX"                 # AWS Route table id ex) rtb-4127..

#### IaaS :: openstack 
#auth_url: "http://XXX.XXX.XXX.XXX:5000/v3/"       # Openstack Keystone URL
#az: "nova"                                        # Openstack AZ Zone
#default_key_name: "bosh-key"                      # Openstack Key Name
#default_security_groups: ["bosh-sg"]              # Openstack Security Group
#net_id: "XXXXXXXXXXXXXXX"                         # Openstack Network ID ex) 70f151db-274....
#openstack_password: "XXXXXXXXXXXXXXX"             # Openstack User Password
#openstack_username: "XXXXXXXXXXXXXXX"             # Openstack User Name
#openstack_domain: "default"                       # Openstack Domain Name
#openstack_project: "paasta"                       # Openstack Project
#private_key: "/.ssh/bosh-key.pem"                 # Openstack SSH Private Key Path
#region: "RegionOne"                               # Openstack Region

#### IaaS :: vsphere 
#wan_gateway: "XXX.XXX.XXX.XXX"                    # Used by OpenVPN gateway ex) 172.100.100.100
#wan_network: "XXX.XXX.XXX.XXX"                    # Used by OpenVPN Network ex) 172.100.100.0
#wan_network_mask_bits: "24"                       # Used by OpenVPN Mask bits ex) 24
#wan_network_name: "Public Network"                # vCenter Public Network Name 
#lan_network_name: "Private Network"               # vCenter Private Network Name 
#vcenter_dc: "DataCenter"                          # vCenter Data Center Name
#vcenter_cluster: "CLUSTER"                        # vCenter Cluster Name
#vcenter_ip: "XXX.XXX.XXX.XXX"                     # vCenter Private IP
#vcenter_user: "XXXXXXXXXXXXXXX"                   # vCenter User Name
#vcenter_password: "XXXXXXXXXXXXXXX"               # vCenter User Password
#vcenter_ds: "DataStorage"                         # vCenter Data Storage Name
#vcenter_templates: "Templates"                    # vCenter Templates Name
#vcenter_vms: "Vms"                                # vCenter VMS Name
#vcenter_disks: "Disks"                            # vCenter Disk Name
```

- BOSH가 설치되지 않은 상이한 IaaS 환경 설치되는 OpenVPN az2 변수를 설정한다.  
  (OpenVPN을 설치할 IaaS 환경에 대한 주석을 해제하고 변수를 설정한다.)
> $ vi vars-az2.yml
```
### openvpn default
wan_ip: "XXX.XXX.XXX.XXX"                         # Used by OpenVPN Server-2 ip 
push_routes: ["XXX.XXX.XXX.XXX 255.255.255.0"]    # ex) 20.0.20.0 255.255.255.0
lan_gateway: "XXX.XXX.XXX.XXX"                    # ex) 20.0.20.1
lan_ip: "XXX.XXX.XXX.XXX"                         # ex) 20.0.20.10
lan_network: "XXX.XXX.XXX.XXX"                    # ex) 20.0.20.0
lan_network_mask_bits: "24"                       # ex) 24
vpn_network: "XXX.XXX.XXX.XXX"                    # ex) 192.168.1.0 (az-1: 192.168.0.0, az-2: 192.168.1.0)
vpn_network_mask: "XXX.XXX.XXX.XXX"               # ex) 255.255.255.0
vpn_network_mask_bits: "24"                       # ex) 24
remote_network_cidr_block: "XXX.XXX.XXX.XXX/24"   # ex) 10.0.10.0/24
remote_vpn_ip: "XXX.XXX.XXX.XXX"                  # Used by OpenVPN Server-1 ip 

... ((생략)) ...

```

### <div id='2.3.2'/>2.3.2. 인증서 생성
OpenVPN에서 사용 할 인증서를 generate_ca.sh을 실행하여 생성한다.
```
$ source generate_ca.sh

$ ll creds
drwxrwxr-x 2 ubuntu ubuntu  4096 Jul 22 02:12 ./
drwxrwxr-x 4 ubuntu ubuntu  4096 Jul 21 01:47 ../
-rw-rw-r-- 1 ubuntu ubuntu 18098 Jul 14 04:22 vpn-server-az1.yml
-rw-rw-r-- 1 ubuntu ubuntu 18102 Jul 14 04:22 vpn-server-az2.yml
```

### <div id='2.3.3'/>2.3.3. OpenVPN 설치
생성된 인증서를 사용하여 OpenVPN az1, OpenVPN az2 설치를 진행한다.
```
### OpenVPN az1 설치
$ source deploy-vpn-az1.sh

### OpenVPN az2 설치
$ source deploy-vpn-az2.sh
```

### <div id='2.3.4'/>2.3.4. OpenVPN 연결 확인
OpenVPN이 설치가 완료되면 상호간 연결이 가능한지 확인한다.
```
# OpenVPN 인증키 생성
## OpenVPN az1.key
$ bosh int creds/vpn-deploy-az1.yml --path /ssh/private_key > openvpn-az1.key 
$ chmod 600 openvpn-az1.key

## OpenVPN az2.key
$ bosh int creds/vpn-deploy-az2.yml --path /ssh/private_key > openvpn-az2.key 
$ chmod 600 openvpn-az2.key


# 연결 확인
## openVPN az1
$ ssh openvpn@{openvpn-az1-ip} -i openvpn-az1.key 

## ping 연결 확인
$ ping {openvpn-az2-ip}

## ifconfig 확인 (네트워크 인터페이스 tun0, tun2 확인)
$ sudo su
$ ifconfig 
<<<<<<<<<<<<<<<<<<<< 수정 예정(실제 배포 후 결과 드래그 예정)
```

### <div id='2.3.5'/>2.3.5. 정적 라우팅 추가 (선택)
클라이언트 터널을 사용하기 위하여 정적 라우팅을 추가할 수 있다.  
IaaS에서 ???을 지원하지 않는 경우 ??? VM에서 해당 명령어를 통하여 정적 라우팅을 설정한다.
```
$ sudo ip route add {remote_network_cidr_block} via {lan_ip}
e.g.) sudo ip route add 20.0.20.0/24 via 10.0.10.10

$ ping {remote_network_ip}
```

 2.4. [Multi CPI 설정](#2.4)   
　2.4.1. [BOSH 설치](#2.4.1)    
　2.4.2. [Stemcell 업로드](#2.4.2)    
　2.4.3. [CPI Config 설정](#2.4.3)    
　　2.4.3.1. [같은 IaaS를 사용 할 경우](#2.4.3.1)    
　　2.4.3.2. [다른 IaaS를 사용 할 경우](#2.4.3.2)    
　2.4.4. [Cloud Config 설정](#2.4.4)    
　　2.4.4.1. [같은 IaaS를 사용 할 경우](#2.4.4.1)    
　　2.4.4.2. [다른 IaaS를 사용 할 경우](#2.4.4.2)    
　2.4.5. [Multi CPI 설정 테스트](#2.4.5)    

## <div id='2.4'/>2.4. Multi CPI 설정
BOSH를 설치하고 Multi CPI를 설정하여 하나의 BOSH로 상이한 IaaS 환경에서 VM을 배포할 수 있다.  
이 때 같은 IaaS의 상이한 IaaS 환경(e.g. A OpenStack <-> B OpenStack)의 경우 다른 옵션을 추가하지 않고 BOSH를 설치하며, IaaS의 종류가 다를경우(e.g. Openstack <-> AWS) 옵션을 추가하여 BOSH 설치를 진행한다.  

### <div id='2.4.1'/>2.4.1. BOSH 설치
본 가이드에서는 추가되는 옵션에대한 설명을 진행하고 BOSH 설치에대한 상세 내용은 BOSH 설치 가이드를 참고한다.

- Multi CPI 관련 Option 파일

|파일명|설명|
|------|---|
| deploy-cpi-aws-secondary.yml | 새 인프라가 AWS 일 경우 사용 |
| deploy-cpi-openstack-secondary.yml	 | 새 인프라가 OpenStack 일 경우 사용 |
| deploy-cpi-vsphere-secondary.yml	 | 새 인프라가 vSphere 일 경우 사용 |
| deploy-cpi-registry-secondary.yml | 기존 인프라가 vSphere 일 경우 사용 |

- 예제1. AWS - Openstack BOSH 설치
> vi ~/workspace/bosh/deploy-aws.sh
```
 bosh create-env bosh.yml \
 	--state=aws/state.json \
 	--vars-store=aws/creds.yml \
 	-o aws/cpi.yml \
+ 	-o multi-cpi/deploy-cpi-openstack-secondary.yml \
 	-o uaa.yml \
 	-o credhub.yml \
 	-o jumpbox-user.yml \
 	-o cce.yml \
 	-l aws-vars.yml
```









### <div id='2.4.2'/>2.4.2. Stemcell 업로드
### <div id='2.4.3'/>2.4.3. CPI Config 설정
#### <div id='2.4.3.1'/>2.4.3.1. 같은 IaaS를 사용 할 경우
#### <div id='2.4.3.2'/>2.4.3.2. 다른 IaaS를 사용 할 경우
### <div id='2.4.4'/>2.4.4. Cloud Config 설정
#### <div id='2.4.4.1'/>2.4.4.1. 같은 IaaS를 사용 할 경우
#### <div id='2.4.4.2'/>2.4.4.2. 다른 IaaS를 사용 할 경우

### <div id='2.4.5'/>2.4.5. Multi CPI 설정 테스트
AP 말고 다른 가벼운거 Deplyoyment로 테스트해도 가능할지??? 문의



### [Index](https://github.com/okpc579/paasta-guide-new/blob/main/README.md) > [AP Install](../README.md) > PaaS-TA Multi CPI
