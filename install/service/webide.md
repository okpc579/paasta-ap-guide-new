### [Index](https://github.com/okpc579/paasta-guide-new/blob/main/README.md) > [AP Install](../README.md) > WEB IDE Service

## Table of Contents
1. [문서 개요](#1)  
  1.1. [목적](#1.1)  
  1.2. [범위](#1.2)  
  1.3. [시스템 구성도](#1.3)  
  1.4. [참고자료](#1.4)  

2. [WEB IDE 설치](#2)  
  2.1. [Prerequisite](#2.1)   
  2.2. [Stemcell 확인](#2.2)    
  2.3. [Deployment 다운로드](#2.3)   
  2.4. [Deployment 파일 수정](#2.4)  
  2.5. [서비스 설치](#2.5)    
  2.6. [서비스 설치 - 다운로드 된 PaaS-TA Release 파일 이용 방식](#2.6)   
  2.7. [서비스 설치 확인](#2.7)  

3. [WEB-IDE의 PaaS-TA 포털사이트 연동](#3)  
  3.1. [WEB-IDE 서비스 브로커 등록](#3.1)

4. [WEB-IDE 에서 CF CLI 사용법](#4)  
  4.1. [WEB-IDE New Project 화면](#4.1)  
  4.2. [WEB-IDE Workspace 화면](#4.2)  
  4.3. [WEB-IDE Teminal에서의 CF CLI 실행](#4.3)  

5. [WEB IDE IP 증설](#5)  
  5.1. [서비스 확인](#5.1)   
  5.2. [Deployment 파일 수정](#5.2)   
  5.3. [서비스 재 설치](#5.3)    
  5.4. [서비스 설치 확인](#5.4)    


## <div id="1"/> 1. 문서 개요

### <div id='1.1'/>1.1. 목적

본 문서는 PaaS-TA에서 사용할 수 있는 WEB-IDE의 설치를 Bosh를 이용하여 설치 하는 방법과 PaaS-TA 포털에서 WEB-IDE 서비스를 사용하는 방법을 기술하였다.
PaaS-TA 3.5 버전부터는 Bosh2.0 기반으로 deploy를 진행하며 기존 Bosh1.0 기반으로 설치를 원할경우에는 PaaS-TA 3.1 이하 버전의 문서를 참고한다.

### <div id='1.2'/> 1.2. 범위
설치 범위는 WEB-IDE 사용을 검증하기 위한 기본 설치를 기준으로 작성하였다.

### <div id='1.3'/> 1.3. 시스템 구성도
본 장에서는 WEB-IDE의 시스템 구성에 대해 기술하였다. Browser(PaaS-TA Portal), WEB IDE Server, Workspace, Desktop IDE로 최소사항을 구성하였다.<br />
WEB-IDE 는 0개 부터 N개 까지 VM INSTANCE 를 생성, 삭제 할 수 있다. <br />
(설치시 확보된 PUBLIC IP 갯수 안에서 가능함)

![](/service-guide/images/webide/web-ide-on-01.png)

| 구분 | Resource Pool | 스펙 |
|--------|-------|-------|
| web-ide | resource\_pools | 1vCPU / 2GB RAM / 10GB Disk |



### <div id='1.4'/>1.4. 참고자료

> [**http://bosh.io/docs**](http://bosh.io/docs) <br>
> [**http://docs.cloudfoundry.org/**](http://docs.cloudfoundry.org/) <br>
> [**https://www.eclipse.org/che/technology/**](https://www.eclipse.org/che/technology/) <br>


## <div id='2'/> 2. WEB IDE 설치  

### <div id="2.1"/> 2.1. Prerequisite 

본 설치 가이드는 Linux 환경에서 설치하는 것을 기준으로 하였다. 서비스 설치를 위해서는 BOSH 2.0과 PaaS-TA 5.0 이상이 설치되있어야 한다.

### <div id="2.2"/> 2.2. Stemcell 확인  

Stemcell 목록을 확인하여 서비스 설치에 필요한 Stemcell이 업로드 되어 있는 것을 확인한다.  (PaaS-TA 5.5.2 과 동일 stemcell 사용)

> $ bosh -e micro-bosh stemcells  

```
Using environment '10.0.1.6' as client 'admin'

Name                                     Version  OS             CPI  CID  
bosh-aws-xen-hvm-ubuntu-xenial-go_agent  621.94*  ubuntu-xenial  -    ami-0297ff649e8eea21b  

(*) Currently deployed

1 stemcells

Succeeded
```  

### <div id="2.3"/> 2.3. Deployment 다운로드

서비스 설치에 필요한 Deployment를 Git Repository에서 받아 서비스 설치 작업 경로로 위치시킨다.  

- Service Deployment Git Repository URL : https://github.com/PaaS-TA/service-deployment/tree/v5.1.0

```
# Deployment 다운로드 파일 위치 경로 생성 및 설치 경로 이동
$ mkdir -p ~/workspace/paasta-5.5.2/deployment
$ cd ~/workspace/paasta-5.5.2/deployment

# Deployment 파일 다운로드
$ git clone https://github.com/PaaS-TA/service-deployment.git -b v5.1.0
```

### <div id="2.4"/> 2.4. Deployment 파일 수정

BOSH Deployment manifest는 Components 요소 및 배포의 속성을 정의한 YAML 파일이다.
Deployment 파일에서 사용하는 network, vm_type, disk_type 등은 Cloud config를 활용하고, 활용 방법은 BOSH 2.0 가이드를 참고한다.   

- Cloud config 설정 내용을 확인한다.   

> $ bosh -e micro-bosh cloud-config   

```
Using environment '10.0.1.6' as client 'admin'

azs:
- cloud_properties:
    availability_zone: ap-northeast-2a
  name: z1
- cloud_properties:
    availability_zone: ap-northeast-2a
  name: z2

... ((생략)) ...

disk_types:
- disk_size: 1024
  name: default
- disk_size: 1024
  name: 1GB

... ((생략)) ...

networks:
- name: default
  subnets:
  - az: z1
    cloud_properties:
      security_groups: paasta-security-group
      subnet: subnet-00000000000000000
    dns:
    - 8.8.8.8
    gateway: 10.0.1.1
    range: 10.0.1.0/24
    reserved:
    - 10.0.1.2 - 10.0.1.9
    static:
    - 10.0.1.10 - 10.0.1.120

... ((생략)) ...

vm_types:
- cloud_properties:
    ephemeral_disk:
      size: 3000
      type: gp2
    instance_type: t2.small
  name: minimal
- cloud_properties:
    ephemeral_disk:
      size: 10000
      type: gp2
    instance_type: t2.small
  name: small

... ((생략)) ...

Succeeded
```

- common_vars.yml을 서버 환경에 맞게 수정한다. 
- WEB IDE에서 사용하는 변수는 bosh_url, bosh_client_admin_id, bosh_client_admin_secret, bosh_director_port,  bosh_oauth_port, bosh_version, system_domain, paasta_admin_username, paasta_admin_password 이다.

> $ vi ~/workspace/paasta-5.5.2/deployment/common/common_vars.yml
```
# BOSH INFO
bosh_ip: "10.0.1.6"				# BOSH IP
bosh_url: "https://10.0.1.6"			# BOSH URL (e.g. "https://00.000.0.0")
bosh_client_admin_id: "admin"			# BOSH Client Admin ID
bosh_client_admin_secret: "ert7na4jpew"		# BOSH Client Admin Secret('echo $(bosh int ~/workspace/paasta-5.5.2/deployment/paasta-deployment/bosh/{iaas}/creds.yml --path /admin_password)' 명령어를 통해 확인 가능)
bosh_director_port: 25555			# BOSH director port
bosh_oauth_port: 8443				# BOSH oauth port
bosh_version: 271.2				# BOSH version('bosh env' 명령어를 통해 확인 가능, on-demand service용, e.g. "271.2")

# PAAS-TA INFO
system_domain: "61.252.53.246.nip.io"		# Domain (nip.io를 사용하는 경우 HAProxy Public IP와 동일)
paasta_admin_username: "admin"			# PaaS-TA Admin Username
paasta_admin_password: "admin"			# PaaS-TA Admin Password
paasta_nats_ip: "10.0.1.121"
paasta_nats_port: 4222
paasta_nats_user: "nats"
paasta_nats_password: "7EZB5ZkMLMqT7"		# PaaS-TA Nats Password (CredHub 로그인후 'credhub get -n /micro-bosh/paasta/nats_password' 명령어를 통해 확인 가능)
paasta_nats_private_networks_name: "default"	# PaaS-TA Nats 의 Network 이름
paasta_database_ips: "10.0.1.123"		# PaaS-TA Database IP (e.g. "10.0.1.123")
paasta_database_port: 5524			# PaaS-TA Database Port (e.g. 5524(postgresql)/13307(mysql)) -- Do Not Use "3306"&"13306" in mysql
paasta_database_type: "postgresql"		# PaaS-TA Database Type (e.g. "postgresql" or "mysql")
paasta_database_driver_class: "org.postgresql.Driver"	# PaaS-TA Database driver-class (e.g. "org.postgresql.Driver" or "com.mysql.jdbc.Driver")
paasta_cc_db_id: "cloud_controller"		# CCDB ID (e.g. "cloud_controller")
paasta_cc_db_password: "cc_admin"		# CCDB Password (e.g. "cc_admin")
paasta_uaa_db_id: "uaa"				# UAADB ID (e.g. "uaa")
paasta_uaa_db_password: "uaa_admin"		# UAADB Password (e.g. "uaa_admin")
paasta_api_version: "v3"

# UAAC INFO
uaa_client_admin_id: "admin"			# UAAC Admin Client Admin ID
uaa_client_admin_secret: "admin-secret"		# UAAC Admin Client에 접근하기 위한 Secret 변수
uaa_client_portal_secret: "clientsecret"	# UAAC Portal Client에 접근하기 위한 Secret 변수

# Monitoring INFO
metric_url: "10.0.161.101"			# Monitoring InfluxDB IP
elasticsearch_master_ip: "10.0.1.146"           # Logsearch의 elasticsearch master IP
elasticsearch_master_port: 9200                 # Logsearch의 elasticsearch master Port
syslog_address: "10.0.121.100"			# Logsearch의 ls-router IP
syslog_port: "2514"				# Logsearch의 ls-router Port
syslog_transport: "relp"			# Logsearch Protocol
saas_monitoring_url: "61.252.53.248"		# Pinpoint HAProxy WEBUI의 Public IP
monitoring_api_url: "61.252.53.241"		# Monitoring-WEB의 Public IP

### Portal INFO
portal_web_user_ip: "52.78.88.252"
portal_web_user_url: "http://portal-web-user.52.78.88.252.nip.io" 

### ETC INFO
abacus_url: "http://abacus.61.252.53.248.nip.io"	# abacus url (e.g. "http://abacus.xxx.xxx.xxx.xxx.nip.io")

```



- Deployment YAML에서 사용하는 변수 파일을 서버 환경에 맞게 수정한다.

> $ vi ~/workspace/paasta-5.5.2/deployment/service-deployment/web-ide/vars.yml

```
deployment_name: "web-ide"                                                # 서비스 배포 명

# STEMCELL
stemcell_os: "ubuntu-xenial"                                              # stemcell os
stemcell_version: "621.94"                                                # stemcell version
stemcell_alias: "default"                                                 # stemcell alias

# NETWORK
private_networks_name: "default"                                          # private network name
public_networks_name: "vip"                                               # public network name

# ECLIPSE-CHE
eclipse_che_azs: [z7]                                                     # eclipse-che : azs
eclipse_che_instances: 0                                                  # eclipse-che : instances (1), ondemand service default 0
eclipse_che_vm_type: "large"                                              # eclipse-che : vm type
eclipse_che_public_ips: "<ECLIPSE_CHE_INIT_PUBLIC_IPS>"                   # eclipse-che : public ips (e.g. ["00.00.00.00" , "11.11.11.11"])
eclipse_che_buffer_ips: "<ECLIPSE_CHE_BUFFER_PUBLIC_IPS>"                 # eclipse-che : OnDemand 에서 사용할 여분의 public ips
eclipse_che_instance_name: "eclipse-che"                                  # eclipse-che : 작업 이름

# MARIA_DB
mariadb_azs: [z4]                                                         # mariadb : azs
mariadb_instances: 1                                                      # mariadb : instances (1) 
mariadb_vm_type: "small"                                                  # mariadb : vm type
mariadb_persistent_disk_type: "10GB"                                      # mariadb : persistent disk type
mariadb_port: "<MARIADB_PORT>"                                            # mariadb : database port (e.g. 31306) -- Do Not Use "3306"
mariadb_admin_password: "<MARIADB_ADMIN_PASSWORD>"                        # mariadb : database admin password (e.g. "Paasta@2018")

# SERVICE-BROKER
broker_azs: [z4]                                                          # service-broker : azs
broker_instances: 1                                                       # service-broker : instances (1)
broker_vm_type: "medium"                                                  # service-broker : vm type
broker_port: "<BROKER_PORT>"                                              # service-broker : broker port (e.g. "8080")
serviceDefinition_id: "<SERVICE_GUID>"                                    # serviceDefinition_id : service guid (e.g. "af86588c-6212-11e7-907b-b6006ad3webide0")
serviceDefinition_name: "WEB IDE"
serviceDefinition_plan1_id: "<SERVICE_PLAN_ID>"                           # serviceDefinition_plan1_id : service plan id (e.g. "a5930564-6212-11e7-907b-b6006ad3webide1")
serviceDefinition_plan1_name: "<SERVICE_PLAN_NAME>"                       # serviceDefinition_plan1_name : service plan name (e.g. "dedicated-vm")
serviceDefinition_plan1_desc: "WEB IDE SERVICE"
serviceDefinition_bullet_name: "Web IDE OnDemand Server Use"
serviceDefinition_bullet_desc: "Web IDE Service Using a OnDemand Server"
serviceDefinition_org_limitation: "-1"                                    # serviceDefinition_org_limitation : 조직별 서비스 제한
serviceDefinition_space_limitation: "-1"                                  # serviceDefinition_space_limitation : 공간별 서비스 제한

# CF
cloudfoundry_sslSkipValidation: "true"
```

### <div id="2.5"/> 2.5. 서비스 설치

- 서버 환경에 맞추어 Deploy 스크립트 파일의 VARIABLES 설정을 수정한다.  
  (선택) -o operations/cce.yml (CCE 조치를 적용하여 설치) 

> $ vi ~/workspace/paasta-5.5.2/deployment/service-deployment/web-ide/deploy.sh

```
#!/bin/bash
  
# VARIABLES
COMMON_VARS_PATH="<COMMON_VARS_FILE_PATH>"       # common_vars.yml File Path (e.g. ../../common/common_vars.yml)
CURRENT_IAAS="${CURRENT_IAAS}"					 # IaaS Information (PaaS-TA에서 제공되는 create-bosh-login.sh 미 사용시 aws/azure/gcp/openstack/vsphere 입력)
BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"			 # bosh director alias name (PaaS-TA에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

# DEPLOY
bosh -e ${BOSH_NAME} -n -d web-ide deploy --no-redact web-ide.yml \
    -o operations/${IAAS}-network.yml \
    -o operations/cce.yml \
    -l ${COMMON_VARS_PATH} \
    -l vars.yml      
```

- 서비스를 설치한다.  
```
$ cd ~/workspace/paasta-5.5.2/deployment/service-deployment/web-ide
$ sh ./deploy.sh  
```  

### <div id="2.6"/> 2.6. 서비스 설치 - 다운로드 된 PaaS-TA Release 파일 이용 방식  

- 서비스 설치에 필요한 릴리즈 파일을 다운로드 받아 Local machine의 서비스 설치 작업 경로로 위치시킨다.  
  
  - 설치 릴리즈 파일 다운로드 : [paas-ta-webide-release-2.1.0.tgz](https://nextcloud.paas-ta.org/index.php/s/kYSnWLNCRDBRcrA/download)

```
# 릴리즈 다운로드 파일 위치 경로 생성
$ mkdir -p ~/workspace/paasta-5.5.2/release/service

# 릴리즈 파일 다운로드 및 파일 경로 확인
$ ls ~/workspace/paasta-5.5.2/release/service
paasta-webide-release-2.0.tgz
```
  
- 서버 환경에 맞추어 Deploy 스크립트 파일의 VARIABLES 설정을 수정하고 Option file 및 변수를 추가한다.  
     (추가) -o operations/use-offline-releases.yml  
     (추가) -v releases_dir="<RELEASE_DIRECTORY>"  
     
> $ vi ~/workspace/paasta-5.5.2/deployment/service-deployment/web-ide/deploy.sh

```
#!/bin/bash

# VARIABLES
COMMON_VARS_PATH="<COMMON_VARS_FILE_PATH>"       # common_vars.yml File Path (e.g. ../../common/common_vars.yml)
CURRENT_IAAS="${CURRENT_IAAS}"					 # IaaS Information (PaaS-TA에서 제공되는 create-bosh-login.sh 미 사용시 aws/azure/gcp/openstack/vsphere 입력)
BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"			 # bosh director alias name (PaaS-TA에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)


# DEPLOY
bosh -e ${BOSH_NAME} -n -d web-ide deploy --no-redact web-ide.yml \
    -o operations/use-offline-releases.yml \
    -o operations/${IAAS}-network.yml \
    -o operations/cce.yml \
    -l ${COMMON_VARS_PATH} \
    -l vars.yml\
    -v releases_dir="/home/ubuntu/workspace/paasta-5.5.2/release"  
```  

- 서비스를 설치한다.  
```
$ cd ~/workspace/paasta-5.5.2/deployment/service-deployment/web-ide
$ sh ./deploy.sh  
```  

### <div id="2.7"/> 2.7. 서비스 설치 확인

설치 완료된 서비스를 확인한다.  

> $ bosh -e micro-bosh -d web-ide vms

```
Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

Task 7872. Done

Deployment 'web-ide'

Instance                                            Process State  AZ  IPs            VM CID                                   VM Type  Active
mariadb/ec34aa5b-c7cc-4297-9e2d-babf05d83832        running        z3  10.30.56.55    vm-9e1631af-b6c8-481e-aad3-3fd713f106a9  small    true
webide-broker/a641df99-d36a-49ee-8329-018fe10fa23d  running        z3  10.30.56.56    vm-eb784964-48cd-4e4c-b080-53675d3738c2  medium   true

3 vms

Succeeded
```



## <div id='3'/> 3. WEB-IDE의 PaaS-TA 포털사이트 연동

### <div id='3.1'/> 3.1. WEB-IDE 서비스 브로커 등록

>`$ cf create-service-broker {서비스팩 이름} {서비스팩 사용자ID} {서비스팩 사용자비밀번호} http://{서비스팩 URL(IP)}`

  **서비스팩 이름** : 서비스 팩 관리를 위해 PaaS-TA에서 보여지는 명칭이다. 서비스 Marketplace에서는 각각의 API 서비스 명이 보여지니 여기서 명칭은 서비스팩 리스트의 명칭이다.<br>
  **서비스팩 사용자ID** / 비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID입니다. 서비스팩도 하나의 API 서버이기 때문에 아무나 접근을 허용할 수 없어 접근이 가능한 ID/비밀번호를 입력한다.<br>
  **서비스팩 URL** : 서비스팩이 제공하는 API를 사용할 수 있는 URL을 입력한다.

>`$ cf create-service-broker webide-service-broker admin cloudfoundry http://10.30.56.56:8080`
```
$ cf create-service-broker webide-service-broker admin cloudfoundry http://10.30.56.56:8080
Creating service broker webide-service-broker as admin...
OK
```
<br>

##### 등록된 WEB-IDE 서비스 브로커를 확인한다.
>`$ cf service-brokers`
```
$ cf service-brokers
Getting service brokers as admin...

name                          url
webide-service-broker         http://10.30.56.56:8080
```
<br>

#### 접근 가능한 서비스 목록을 확인한다.
>`$ cf service-access`
```
$ cf service-access
Getting service access as admin...
broker: webide-service-broker
   offering   plan           access   orgs
   webide     dedicated-vm   none      
```
<br>

- 서비스 브로커 등록시 최초에는 접근을 허용하지 않는다. 따라서 access는 none으로 설정된다.

#### 특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)
>`$ cf enable-service-access webide`<br>
>`$ cf service-access`
```
$ cf enable-service-access webide
Enabling access to all plans of service webide for all orgs as admin...
OK
```
```
$ cf service-access
Getting service access as admin...

broker: webide-service-broker
   offering   plan           access   orgs
   webide     dedicated-vm   all      
```
<br>

#### PaaS-TA Marketplace에서 서비스가 있는지 확인을 한다.

>`$ cf marketplace`
```
$ cf marketplace
Getting services from marketplace in org system / space dev as admin...
OK

offering   plans          description                                                                 broker
webide     dedicated-vm   A paasta web ide service for application development.provision parameters   webide-service-broker
```
<br>

#### Marketplace에서 원하는 서비스가 있으면 서비스 신청(Provision)을 한다.

>`$ cf create-service {서비스명} {서비스 플랜} {내 서비스명}`
- **서비스명** : webide로 Marketplace에서 보여지는 서비스 명칭이다.
- **서비스플랜** : 서비스에 대한 정책으로 plans에 있는 정보 중 하나를 선택한다. webide 서비스는 standard plan만 지원한다.
- **내 서비스명** : 내 서비스에서 보여지는 명칭이다. 이 명칭을 기준으로 환경 설정 정보를 가져온다.


>`$ cf create-service webide dedicated-vm webide-service`
```
$ cf create-service webide dedicated-vm paasta-webide-service
Creating service instance paasta-webide-service in org system / space dev as admin...
OK

Create in progress. Use 'cf services' or 'cf service webide' to check operation status.
```
<br>

#### 생성된 WEB-IDE VM 인스턴스를 확인한다.

>`bosh -e micro-bosh -d web-ide vms`
```
$ bosh -e micro-bosh -d web-ide vms
Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

Task 7872. Done

Deployment 'web-ide'

Instance                                            Process State  AZ  IPs            VM CID                                   VM Type  Active
eclipse-che/ed136540-c650-47a2-918b-bb7f6020469d    running        z7  10.30.56.54    vm-5a3a2b10-d0c9-47c8-97f0-6ea64c339df8  large    true
							               115.68.46.178
mariadb/ec34aa5b-c7cc-4297-9e2d-babf05d83832        running        z3  10.30.56.55    vm-9e1631af-b6c8-481e-aad3-3fd713f106a9  small    true
webide-broker/a641df99-d36a-49ee-8329-018fe10fa23d  running        z3  10.30.56.56    vm-eb784964-48cd-4e4c-b080-53675d3738c2  medium   true

3 vms

Succeeded
```
<br>

#### 생성된 WEB-IDE 서비스 인스턴스를 확인한다.

>`$ cf services`
```
$ cf services
Getting services in org system / space dev as admin...

name     service   plan           bound apps   last operation     broker                  upgrade available
webide   webide    dedicated-vm                create succeeded   webide-service-broker   
```
<br>

## <div id='4'/> 4. WEB-IDE 에서 CF CLI 사용법

### <div id='4.1'/> 4.1. WEB-IDE New Project 화면
***※ [PaaS-TA 운영자 포탈 4.3.3 카탈로그 관리 서비스 가이드](/use-guide/portal/PAAS-TA_ADMIN_PORTAL_USE_GUIDE_V1.1.md#4.3.3) 참고***  

- 사용할 언어를 선택하고 Create workspace and project 로 새로운 프로젝트를 시작한다.

![](/service-guide/images/webide/web-ide-08-1.png)

<br>


### <div id='4.2'/> 4.2. WEB-IDE Workspace 화면

- WEB-IDE를 처음 생성 시 Workspace를 새로 생성하는 화면이 열리고 프로젝트를 설정에 맞게 생성하여 작업을 진행한다.

![](/service-guide/images/webide/web-ide-on-02.jpeg)

![](/service-guide/images/webide/web-ide-on-03.jpeg)

![](/service-guide/images/webide/web-ide-on-04.jpeg)

<br>

### <div id='4.3'/> 4.3. WEB-IDE Teminal에서의 CF CLI 실행

##### -cf api 명령을 이용해 endpoint를 지정한다.

> ![](/service-guide/images/webide/web-ide-12.png)

##### cf login 명령어로 로그인하고 조직과 공간을 선택한다.

> ![](/service-guide/images/webide/web-ide-13.png)

##### cf push 를 이용해 cf에 앱을 업로드한다.

> ![](/service-guide/images/webide/web-ide-14.png)




## <div id='5'/> 2. WEB IDE IP 증설
### <div id="5.1"/> 5.1. 서비스 확인

현재 생성된 WEB-IDE VM 인스턴스를 확인한다.

>`bosh -e micro-bosh -d web-ide vms`
```
$ bosh -e micro-bosh -d web-ide vms
Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

Task 7872. Done

Deployment 'web-ide'

Instance                                            Process State  AZ  IPs            VM CID                                   VM Type  Active
eclipse-che/ed136540-c650-47a2-918b-bb7f6020469d    running        z7  10.30.56.54    vm-5a3a2b10-d0c9-47c8-97f0-6ea64c339df8  large    true
							               115.68.46.178
mariadb/ec34aa5b-c7cc-4297-9e2d-babf05d83832        running        z3  10.30.56.55    vm-9e1631af-b6c8-481e-aad3-3fd713f106a9  small    true
webide-broker/a641df99-d36a-49ee-8329-018fe10fa23d  running        z3  10.30.56.56    vm-eb784964-48cd-4e4c-b080-53675d3738c2  medium   true

3 vms

Succeeded
```
<br>



### <div id="5.2"/> 5.2. Deployment 파일 수정

기존 설치할때 사용했던 Deployment YAML에서 eclipse_che_instances의 값을 배포된 eclipse-che의 수만큼 변경을 해주고 eclipse_che_public_ips에 설치된 public ip를 입력한다.  
그리고 WEB-IDE에 추가시킬 IP를 eclipse_che_buffer_ips에 추가한다.

> $ vi ~/workspace/paasta-5.5.2/deployment/service-deployment/web-ide/vars.yml

```
.....

# ECLIPSE-CHE
eclipse_che_azs: [z7]                                                   # eclipse-che : azs
eclipse_che_instances: 1                                                # eclipse-che : instances (1), ondemand service default 0
eclipse_che_vm_type: "large"                                            # eclipse-che : vm type
eclipse_che_public_ips: ["115.68.46.178"]                               # eclipse-che : public ips (e.g. ["00.00.00.00" , "11.11.11.11"])
eclipse_che_buffer_ips: ["115.68.46.178", "52.153.36.143"]              # eclipse-che : OnDemand 에서 사용할 여분의 public ips
eclipse_che_instance_name: "eclipse-che"                                # eclipse-che : 작업 이름

........

```


### <div id="5.3"/> 5.3. 서비스 재 설치

- 서비스를 재 설치한다.  
```
$ cd ~/workspace/paasta-5.5.2/deployment/service-deployment/web-ide
$ sh ./deploy.sh  

Using environment '10.0.1.6' as client 'admin'

Using deployment 'web-ide'

Release 'paas-ta-webide-release/2.0' already exists.

  instance_groups:
  - name: webide-broker
    properties:
      network:
        static_ips:
+       - 52.153.36.143
Task 581

Task 581 | 02:20:43 | Preparing deployment: Preparing deployment (00:00:02)
Task 581 | 02:20:45 | Preparing deployment: Rendering templates (00:00:01)
Task 581 | 02:20:46 | Preparing package compilation: Finding packages to compile (00:00:00)
Task 581 | 02:20:46 | Updating instance webide-broker: webide-broker/f47c5c19-92d3-4b84-86da-89e8e53090fc (0) (canary) (00:00:13)

Task 581 Started  Mon Jan 11 02:20:43 UTC 2021
Task 581 Finished Mon Jan 11 02:20:59 UTC 2021
Task 581 Duration 00:00:16
Task 581 done

```  


### <div id="5.4"/> 5.4. 서비스 설치 확인

설치 완료된 서비스를 확인한다.  

> $ bosh -e micro-bosh -d web-ide vms

```
Using environment '10.30.40.111' as user 'admin' (openid, bosh.admin)

Task 7872. Done

Deployment 'web-ide'

Instance                                            Process State  AZ  IPs            VM CID                                   VM Type  Active
eclipse-che/ed136540-c650-47a2-918b-bb7f6020469d    running        z7  10.30.56.54    vm-5a3a2b10-d0c9-47c8-97f0-6ea64c339df8  large    true
							               115.68.46.178
mariadb/ec34aa5b-c7cc-4297-9e2d-babf05d83832        running        z3  10.30.56.55    vm-9e1631af-b6c8-481e-aad3-3fd713f106a9  small    true
webide-broker/a641df99-d36a-49ee-8329-018fe10fa23d  running        z3  10.30.56.56    vm-eb784964-48cd-4e4c-b080-53675d3738c2  medium   true

3 vms

Succeeded
```


### [Index](https://github.com/okpc579/paasta-guide-new/blob/main/README.md) > [AP Install](../README.md) > WEB IDE Service