### [Index](https://github.com/okpc579/paasta-guide-new/blob/main/README.md) > [AP Install](../README.md) > MySQL Service

## Table of Contents  

1. [문서 개요](#1)  
  1.1. [목적](#1.1)  
  1.2. [범위](#1.2)  
  1.3. [참고자료](#1.3)  
  
2. [MySQL 서비스 설치](#2)  
  2.1. [Prerequisite](#2.1)   
  2.2. [Stemcell 확인](#2.2)    
  2.3. [Deployment 다운로드](#2.3)   
  2.4. [Deployment 파일 수정](#2.4)  
  2.5. [서비스 설치](#2.5)    
  2.6. [서비스 설치 확인](#2.6)   
  
3. [MySQL 연동 Sample Web App 설명](#3)  
  3.1. [서비스 브로커 등록](#3.1)  
  3.2. [Sample Web App 다운로드](#3.2)  
  3.3. [PaaS-TA에서 서비스 신청](#3.3)  
  3.4. [Sample Web App 배포 및 MySQL바인드 확인](#3.4)  
 

## <div id='1'> 1. 문서 개요
### <div id='1.1'> 1.1. 목적

본 문서(MySQL 서비스팩 설치 가이드)는 PaaS-TA에서 제공되는 서비스팩인 MySQL 서비스팩을 Bosh를 이용하여 설치 하는 방법을 기술하였다.

### <div id='1.2'> 1.2. 범위
설치 범위는 MySQL 서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다.

### <div id='1.3'> 1.3. 참고자료
BOSH Document: [http://bosh.io](http://bosh.io)  
Cloud Foundry Document: [https://docs.cloudfoundry.org](https://docs.cloudfoundry.org)  

## <div id='2'> 2. MySQL 서비스 설치

### <div id="2.1"/> 2.1. Prerequisite  

본 설치 가이드는 Linux 환경에서 설치하는 것을 기준으로 하였다.  
서비스팩 설치를 위해서는 먼저 BOSH CLI v2 가 설치 되어 있어야 하고 BOSH 에 로그인이 되어 있어야 한다.  
BOSH CLI v2 가 설치 되어 있지 않을 경우 먼저 BOSH2.0 설치 가이드 문서를 참고 하여 BOSH CLI v2를 설치를 하고 사용법을 숙지 해야 한다.  

### <div id="2.2"/> 2.2. Stemcell 확인

Stemcell 목록을 확인하여 서비스 설치에 필요한 Stemcell이 업로드 되어 있는 것을 확인한다.  
본 가이드의 Stemcell은 ubuntu-bionic 1.34를 사용한다.  

> $ bosh -e ${BOSH_ENVIRONMENT} stemcells

```
Using environment '10.0.1.6' as client 'admin'

Name                                       Version   OS             CPI  CID  
bosh-openstack-kvm-ubuntu-bionic-go_agent  1.34      ubuntu-bionic  -    ce507ae4-aca6-4a6d-b7c7-220e3f4aaa7d

(*) Currently deployed

1 stemcells

Succeeded
```

만약 해당 Stemcell이 업로드 되어 있지 않다면 [bosh.io 스템셀](https://bosh.io/stemcells/) 에서 해당되는 IaaS환경과 버전에 해당되는 스템셀 링크를 복사 후 다음과 같은 명령어를 실행한다.

```
# Stemcell 업로드 명령어 예제
bosh -e ${BOSH_ENVIRONMENT} upload-stemcell -n {STEMCELL_URL}
```

### <div id="2.3"/> 2.3. Deployment 다운로드  

서비스 설치에 필요한 Deployment를 Git Repository에서 받아 서비스 설치 작업 경로로 위치시킨다.  

- Service Deployment Git Repository URL : https://github.com/PaaS-TA/service-deployment/tree/v5.1.2

```
# Deployment 다운로드 파일 위치 경로 생성 및 설치 경로 이동
$ mkdir -p ~/workspace
$ cd ~/workspace

# Deployment 파일 다운로드
$ git clone https://github.com/PaaS-TA/service-deployment.git -b v5.1.2
```

### <div id="2.4"/> 2.4. Deployment 파일 수정

BOSH Deployment manifest는 Components 요소 및 배포의 속성을 정의한 YAML 파일이다.  
Deployment 파일에서 사용하는 network, vm_type, disk_type 등은 Cloud config를 활용하고, 활용 방법은 PaaS-TA AP 설치 가이드를 참고한다.  

- Cloud config 설정 내용을 확인한다.   

> $ bosh -e ${BOSH_ENVIRONMENT} cloud-config   

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

- Deployment YAML에서 사용하는 변수 파일을 서버 환경에 맞게 수정한다.

> $ vi ~/workspace/service-deployment/mysql/vars.yml	
```
# STEMCELL
stemcell_os: "ubuntu-bionic"                                     # stemcell os
stemcell_version: "1.34"                                       # stemcell version

# NETWORK
private_networks_name: "default"                                 # private network name

# MYSQL
mysql_azs: [z4]                                                  # mysql azs
mysql_instances: 1                                               # mysql instances (N)
mysql_vm_type: "small"                                           # mysql vm type
mysql_persistent_disk_type: "8GB"                                # mysql persistent disk type
mysql_port: 13306                                                # mysql port (e.g. 13306) -- Do Not Use "3306"
mysql_admin_password: "<MYSQL_ADMIN_PASSWORD>"                   # mysql admin password (e.g. "admin!Service")

# ARBITRATOR
arbitrator_azs: [z4]                                             # arbitrator azs 
arbitrator_instances: 1                                          # arbitrator instances (1)
arbitrator_vm_type: "small"                                      # arbitrator vm type

# PROXY
proxy_azs: [z4]                                                  # proxy azs
proxy_instances: 1                                               # proxy instances (1)
proxy_vm_type: "small"                                           # proxy vm type
proxy_mysql_port: 13307                                          # proxy mysql port (e.g. 13307) -- Do Not Use "3306"

# MYSQL_BROKER
mysql_broker_azs: [z4]                                           # mysql broker azs
mysql_broker_instances: 1                                        # mysql broker instances (1)
mysql_broker_vm_type: "small"                                    # mysql broker vm type
```

### <div id="2.5"/> 2.5. 서비스 설치

- 서버 환경에 맞추어 Deploy 스크립트 파일의 VARIABLES 설정을 수정하고, Option file을 추가할지 선택한다.  
     (선택) -o operations/cce.yml (CCE 조치를 적용하여 설치)


> $ vi ~/workspace/service-deployment/mysql/deploy.sh

```
#!/bin/bash

# VARIABLES
COMMON_VARS_PATH="<COMMON_VARS_FILE_PATH>"    # common_vars.yml File Path (e.g. ../../common/common_vars.yml)
BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"        # bosh director alias name (PaaS-TA에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

# DEPLOY
bosh -e ${BOSH_ENVIRONMENT} -n -d mysql deploy --no-redact mysql.yml \
    -o operations/cce.yml \
    -l ${COMMON_VARS_PATH} \
    -l vars.yml
```

- 서비스를 설치한다.  
```
$ cd ~/workspace/service-deployment/mysql  
$ sh ./deploy.sh  
```  


### <div id="2.6"/> 2.6. 서비스 설치 확인

설치 완료된 서비스를 확인한다.  

> $ bosh -e ${BOSH_ENVIRONMENT} -d mysql vms  

```
Using environment '10.0.1.6' as client 'admin'

Task 4525. Done

Deployment 'mysql'

Instance                                                       Process State  AZ  IPs            VM CID                                   VM Type  Active  
arbitrator/2e190b67-e2b7-4e2d-a72d-872c2019c963                running        z5  10.30.107.165  vm-214663a8-fcbc-4ae4-9aae-92027b9725a9  minimal  true  
mysql-broker/05c44b41-0fc1-41c0-b814-d79558850480              running        z5  10.30.107.167  vm-7c3edc00-3074-4e98-9c89-9e9ba83b47e4  minimal  true  
mysql/fe6943ed-c0c1-4a99-8f4c-d209e165898a                     running        z5  10.30.107.164  vm-81ecdc43-03d2-44f5-9b89-c6cdaa443d8b  minimal  true  
proxy/5b883a78-eb43-417f-98a2-d44c13c29ed4                     running        z5  10.30.107.168  vm-e447eb75-1119-451f-adc9-71b0a6ef1a6a  minimal  true  

5 vms

Succeeded
```	

## <div id='3'> 3. MySQL 연동 Sample Web App 설명  

본 Sample App은 MySQL의 서비스를 Provision한 상태에서 PaaS-TA에 배포하면 MySQL서비스와 bind되어 사용할 수 있다.  

### <div id='3.1'> 3.1. MySQL 서비스 브로커 등록  
Mysql 서비스팩 배포가 완료 되었으면 Application에서 서비스 팩을 사용하기 위해서 먼저 MySQL 서비스 브로커를 등록해 주어야 한다.  
서비스 브로커 등록시 PaaS-TA에서 서비스브로커를 등록할 수 있는 사용자로 로그인이 되어 있어야 한다.

- 서비스 브로커 목록을 확인한다.

> $ cf service-brokers  
```  
Getting service brokers as admin...

name   url
No service brokers found
```   

- MySQL 서비스 브로커를 등록한다.

>`$ cf create-service-broker {서비스팩 이름} {서비스팩 사용자ID} {서비스팩 사용자비밀번호} http://{서비스팩 URL(IP)}`

  서비스팩 이름 : 서비스 팩 관리를 위해 PaaS-TA에서 보여지는 명칭이다. 서비스 Marketplace에서는 각각의 API 서비스 명이 보여지니 여기서 명칭은 서비스팩 리스트의 명칭이다.  
  서비스팩 사용자ID / 비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID로, 서비스팩도 하나의 API 서버이기 때문에 아무나 접근을 허용할 수 없어 접근이 가능한 ID/비밀번호를 입력한다.  
  서비스팩 URL : 서비스팩이 제공하는 API를 사용할 수 있는 URL을 입력한다.  

>`$ cf create-service-broker mysql-service-broker admin cloudfoundry http://<mysql-broker_ip>:8080`
```  
$ cf create-service-broker mysql-service-broker admin cloudfoundry http://10.30.107.167:8080
Creating service broker mysql-service-broker as admin...
OK
```  

- 등록된 MySQL 서비스 브로커를 확인한다.

>`$ cf service-brokers`
```  
$ cf service-brokers
Getting service brokers as admin...

name                      url
mysql-service-broker      http://10.30.107.167:8080
```  

- 접근 가능한 서비스 목록을 확인한다.

>`$ cf service-access`
```  
$ cf service-access
Getting service access as admin...
broker: mysql-service-broker
   service    plan                 access   orgs
   Mysql-DB   Mysql-Plan1-10con    none
   Mysql-DB   Mysql-Plan2-100con   none
```  
서비스 브로커 생성시 디폴트로 접근을 허용하지 않는다.

- 특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)

> $ cf enable-service-access Mysql-DB  
```
Enabling access to all plans of service Mysql-DB for all orgs as admin...
OK
```
> $ cf service-access   
```
Getting service access as admin...
broker: mysql-service-broker
   service    plan                 access   orgs
   Mysql-DB   Mysql-Plan1-10con    all
   Mysql-DB   Mysql-Plan2-100con   all
```  

### <div id='3.2'> 3.2. Sample Web App 다운로드  

Sample App은 PaaS-TA에 App으로 배포되며 App구동시 Bind 된 MySQL 서비스 연결 정보로 접속하여 초기 데이터를 생성하게 된다.    
브라우져를 통해 App에 접속 후 "MYSQL 데이터 가져오기"를 통해 초기 생성된 데이터를 조회 할 수 있다.  

- Sample App 묶음 다운로드
```
$ wget https://nextcloud.paas-ta.org/index.php/s/8sCHaWcw4n36MiB/download --content-disposition  
$ unzip paasta-service-samples.zip  
$ cd paasta-service-samples/mysql  
```

### <div id='3.3'> 3.3. PaaS-TA에서 서비스 신청  
Sample App에서 MySQL 서비스를 사용하기 위해서는 서비스 신청(Provision)을 해야 한다.  

*참고: 서비스 신청시 PaaS-TA에서 서비스를 신청 할 수 있는 사용자로 로그인이 되어 있어야 한다.  

- 먼저 PaaS-TA Marketplace에서 서비스가 있는지 확인을 한다.  

> $ cf marketplace   
```  
Getting services from marketplace in org org system / space dev as admin...
OK

service      plans                                    description
Mysql-DB     Mysql-Plan1-10con, Mysql-Plan2-100con*   A simple mysql implementation

TIP:  Use 'cf marketplace -s SERVICE' to view descriptions of individual plans of a given service.
```  

- Marketplace에서 원하는 서비스가 있으면 서비스 신청(Provision)을 한다.  

> $ cf create-service {서비스명} {서비스플랜} {내서비스명}  

>서비스명 : Mysql-DB로 Marketplace에서 보여지는 서비스 명칭이다.  
>서비스플랜 : 서비스에 대한 정책으로 plans에 있는 정보 중 하나를 선택한다. MySQL 서비스는 10 connection, 100 connection 를 지원한다.  
>내 서비스명 : 내 서비스에서 보여지는 명칭이다. 이 명칭을 기준으로 환경설정정보를 가져온다.  

> $ cf create-service Mysql-DB Mysql-Plan2-100con mysql-service-instance   
```  
Creating service instance mysql-service-instance in org org system / space dev as admin...
OK

Attention: The plan `Mysql-Plan2-100con` of service `Mysql-DB` is not free.  The instance `mysql-service-instance` will incur a cost.  Contact your administrator if you think this is in error.
```  

- 생성된 MySQL 서비스 인스턴스를 확인한다.  

> $ cf services 
```  
Getting services in org system / space dev as admin...
OK

name                      service    plan                 bound apps            last operation
mysql-service-instance    Mysql-DB   Mysql-Plan2-100con                         create succeeded
```  

### <div id='3.4'> 3.4. Sample Web App 배포 및 MySQL바인드 확인   
서비스 신청이 완료되었으면 Sample Web App 에서는 생성된 서비스 인스턴스를 Bind 하여 App에서 MySQL 서비스를 이용한다.  
*참고: 서비스 Bind 신청시 PaaS-TA에서 서비스 Bind신청 할 수 있는 사용자로 로그인이 되어 있어야 한다.  

- manifest 파일을 확인한다.  

> $ vi manifest.yml   

```
---
applications:
- name: mysql-sample-app
  memory: 1024M
  instances: 1
  buildpack: java_buildpack
  path: mysql-sample-app.war
  env:
    mysql_datasource_driver-class-name: org.mariadb.jdbc.Driver
    mysql_datasource_jdbc-url: jdbc:\${vcap.services.mysql-service-instance.credentials.uri}
    mysql_datasource_username: \${vcap.services.mysql-service-instance.credentials.username}
    mysql_datasource_password: \${vcap.services.mysql-service-instance.credentials.password}

```

- --no-start 옵션으로 App을 배포한다.  
> $ cf push --no-start  
```  
Applying manifest file /home/ubuntu/workspace/samples/paasta-service-samples/mysql/manifest.yml...
Manifest applied
Packaging files to upload...
Uploading files...
 26.48 MiB / 26.48 MiB [================================================================================================================] 100.00% 1s

Waiting for API to complete processing files...

name:              mysql-sample-app
requested state:   stopped
routes:            mysql-sample-app.paasta.kr
last uploaded:     
stack:             
buildpacks:        

type:           web
sidecars:       
instances:      0/1
memory usage:   1024M
     state   since                  cpu    memory   disk     details
#0   down    2021-11-22T05:21:57Z   0.0%   0 of 0   0 of 0   
```  
	
- Sample Web App에서 생성한 서비스 인스턴스 바인드 신청을 한다.

> $ cf bind-service mysql-sample-app mysql-service-instance  

```
Binding service mysql-service-instance to app mysql-sample-app in org system / space dev as admin...
OK
```

App 구동 시 Service와의 통신을 위하여 보안 그룹을 추가한다.

- rule.json을 편집한다.  
> $ vi rule.json   
```
## mysql의 proxy IP를 destination에 설정
[
  {
    "protocol": "tcp",
    "destination": "<proxy_ip>",
    "ports": "13307"
  }
]
```
<br>

- 보안 그룹을 생성한다.  

> $ cf create-security-group mysql rule.json  

```
Creating security group mysql as admin...

OK		
```

<br>

- Mysql 서비스를 사용할수 있도록 생성한 보안 그룹을 적용한다.
> $ cf bind-running-security-group mysql  
```
Binding security group mysql to running as admin...
OK		
```
	
- App을 재기동 한다.  


> $ cf restart mysql-sample-app  
```	
Restarting app mysql-sample-app in org system / space dev as admin...

Staging app and tracing logs...
   Downloading java_buildpack...
   Downloaded java_buildpack
   Cell 4a88ce8b-1e72-485a-8f62-1fe0c6b9a7cd creating container for instance 678aa272-945b-41a9-8924-0782891d0cc4
   Cell 4a88ce8b-1e72-485a-8f62-1fe0c6b9a7cd successfully created container for instance 678aa272-945b-41a9-8924-0782891d0cc4
   Downloading app package...
   Downloaded app package (30.5M)

........
........
Instances starting...
Instances starting...

name:              mysql-sample-app
requested state:   started
routes:            mysql-sample-app.paasta.kr
last uploaded:     Mon 22 Nov 05:23:48 UTC 2021
stack:             cflinuxfs3
buildpacks:        
	name             version                                                             detect output   buildpack name
	java_buildpack   v4.37-https://github.com/cloudfoundry/java-buildpack.git#ab2b4512   java            java
```  

- App이 정상적으로 MySQL 서비스를 사용하는지 확인한다.  

![update_mysql_vsphere_34]  


[mysql_vsphere_1.3.01]:./images/mysql/mysql_vsphere_1.3.01.png
[mysql_vsphere_2.2.01]:./images/mysql/mysql_vsphere_2.2.01.png
[mysql_vsphere_2.2.02]:./images/mysql/mysql_vsphere_2.2.02.png
[mysql_vsphere_2.2.03]:./images/mysql/mysql_vsphere_2.2.03.png
[mysql_vsphere_2.2.04]:./images/mysql/mysql_vsphere_2.2.04.png
[mysql_vsphere_2.2.05]:./images/mysql/mysql_vsphere_2.2.05.png
[mysql_vsphere_2.2.06]:./images/mysql/mysql_vsphere_2.2.06.png
[mysql_vsphere_2.2.07]:./images/mysql/mysql_vsphere_2.2.07.png
[mysql_vsphere_2.2.08]:./images/mysql/mysql_vsphere_2.2.08.png
[mysql_vsphere_2.3.01]:./images/mysql/mysql_vsphere_2.3.01.png
[mysql_vsphere_2.3.02]:./images/mysql/mysql_vsphere_2.3.02.png
[mysql_vsphere_2.3.03]:./images/mysql/mysql_vsphere_2.3.03.png
[mysql_vsphere_2.3.04]:./images/mysql/mysql_vsphere_2.3.04.png
[mysql_vsphere_2.3.05]:./images/mysql/mysql_vsphere_2.3.05.png
[mysql_vsphere_2.3.06]:./images/mysql/mysql_vsphere_2.3.06.png
[mysql_vsphere_2.3.07]:./images/mysql/mysql_vsphere_2.3.07.png
[mysql_vsphere_2.4.01]:./images/mysql/mysql_vsphere_2.4.01.png
[mysql_vsphere_2.4.02]:./images/mysql/mysql_vsphere_2.4.02.png
[mysql_vsphere_2.4.03]:./images/mysql/mysql_vsphere_2.4.03.png
[mysql_vsphere_2.4.04]:./images/mysql/mysql_vsphere_2.4.04.png
[mysql_vsphere_2.4.05]:./images/mysql/mysql_vsphere_2.4.05.png
[mysql_vsphere_3.1.01]:./images/mysql/mysql_vsphere_3.1.01.png
[mysql_vsphere_3.2.01]:./images/mysql/mysql_vsphere_3.2.01.png
[mysql_vsphere_3.2.02]:./images/mysql/mysql_vsphere_3.2.02.png
[mysql_vsphere_3.2.03]:./images/mysql/mysql_vsphere_3.2.03.png
[mysql_vsphere_3.3.01]:./images/mysql/mysql_vsphere_3.3.01.png
[mysql_vsphere_3.3.02]:./images/mysql/mysql_vsphere_3.3.02.png
[mysql_vsphere_3.3.03]:./images/mysql/mysql_vsphere_3.3.03.png
[mysql_vsphere_3.3.04]:./images/mysql/mysql_vsphere_3.3.04.png
[mysql_vsphere_3.3.05]:./images/mysql/mysql_vsphere_3.3.05.png
[mysql_vsphere_3.3.06]:./images/mysql/mysql_vsphere_3.3.06.png
[mysql_vsphere_3.3.07]:./images/mysql/mysql_vsphere_3.3.07.png
[mysql_vsphere_3.3.08]:./images/mysql/mysql_vsphere_3.3.08.png
[mysql_vsphere_3.3.09]:./images/mysql/mysql_vsphere_3.3.09.png
[mysql_vsphere_4.1.01]:./images/mysql/mysql_vsphere_4.1.01.png
[mysql_vsphere_4.1.02]:./images/mysql/mysql_vsphere_4.1.02.png
[mysql_vsphere_4.1.03]:./images/mysql/mysql_vsphere_4.1.03.png
[mysql_vsphere_4.1.04]:./images/mysql/mysql_vsphere_4.1.04.png
[mysql_vsphere_4.1.05]:./images/mysql/mysql_vsphere_4.1.05.png
[mysql_vsphere_4.1.06]:./images/mysql/mysql_vsphere_4.1.06.png
[mysql_vsphere_4.1.07]:./images/mysql/mysql_vsphere_4.1.07.png
[mysql_vsphere_4.1.08]:./images/mysql/mysql_vsphere_4.1.08.png
[mysql_vsphere_4.1.09]:./images/mysql/mysql_vsphere_4.1.09.png
[mysql_vsphere_4.1.10]:./images/mysql/mysql_vsphere_4.1.10.png
[mysql_vsphere_4.1.11]:./images/mysql/mysql_vsphere_4.1.11.png
[mysql_vsphere_4.1.12]:./images/mysql/mysql_vsphere_4.1.12.png
[mysql_vsphere_4.1.13]:./images/mysql/mysql_vsphere_4.1.13.png
[mysql_vsphere_4.1.14]:./images/mysql/mysql_vsphere_4.1.14.png
[mysql_vsphere_4.1.15]:./images/mysql/mysql_vsphere_4.1.15.png
[mysql_vsphere_4.1.16]:./images/mysql/mysql_vsphere_4.1.16.png
[mysql_vsphere_4.1.17]:./images/mysql/mysql_vsphere_4.1.17.png



[update_mysql_vsphere_01]:./images/mysql/update_mysql_vsphere_01.png
[update_mysql_vsphere_02]:./images/mysql/update_mysql_vsphere_02.png
[update_mysql_vsphere_03]:./images/mysql/update_mysql_vsphere_03.png
[update_mysql_vsphere_04]:./images/mysql/update_mysql_vsphere_04.png
[update_mysql_vsphere_05]:./images/mysql/update_mysql_vsphere_05.png
[update_mysql_vsphere_06]:./images/mysql/update_mysql_vsphere_06.png
[update_mysql_vsphere_07]:./images/mysql/update_mysql_vsphere_07.png
[update_mysql_vsphere_08]:./images/mysql/update_mysql_vsphere_08.png
[update_mysql_vsphere_09]:./images/mysql/update_mysql_vsphere_09.png
[update_mysql_vsphere_10]:./images/mysql/update_mysql_vsphere_10.png
[update_mysql_vsphere_11]:./images/mysql/update_mysql_vsphere_11.png
[update_mysql_vsphere_12]:./images/mysql/update_mysql_vsphere_12.png
[update_mysql_vsphere_13]:./images/mysql/update_mysql_vsphere_13.png
[update_mysql_vsphere_14]:./images/mysql/update_mysql_vsphere_14.png
[update_mysql_vsphere_15]:./images/mysql/update_mysql_vsphere_15.png

[update_mysql_vsphere_25]:./images/mysql/update_mysql_vsphere_25.png
[update_mysql_vsphere_30]:./images/mysql/update_mysql_vsphere_30.png
[update_mysql_vsphere_31]:./images/mysql/update_mysql_vsphere_31.png
[update_mysql_vsphere_34]:./images/mysql/update_mysql_vsphere_34.png

[update_mysql_vsphere_35]:./images/mysql/update_mysql_vsphere_35.png
[update_mysql_vsphere_36]:./images/mysql/update_mysql_vsphere_36.png
[update_mysql_vsphere_37]:./images/mysql/update_mysql_vsphere_37.png
[update_mysql_vsphere_38]:./images/mysql/update_mysql_vsphere_38.png
[update_mysql_vsphere_39]:./images/mysql/update_mysql_vsphere_39.png
[update_mysql_vsphere_40]:./images/mysql/update_mysql_vsphere_40.png
[update_mysql_vsphere_41]:./images/mysql/update_mysql_vsphere_41.png
[update_mysql_vsphere_42]:./images/mysql/update_mysql_vsphere_42.png
[update_mysql_vsphere_43]:./images/mysql/update_mysql_vsphere_43.png
[update_mysql_vsphere_44]:./images/mysql/update_mysql_vsphere_44.png
[update_mysql_vsphere_45]:./images/mysql/update_mysql_vsphere_45.png
[update_mysql_vsphere_46]:./images/mysql/update_mysql_vsphere_46.png
[update_mysql_vsphere_47]:./images/mysql/update_mysql_vsphere_47.png
[update_mysql_vsphere_48]:./images/mysql/update_mysql_vsphere_48.png
[update_mysql_vsphere_49]:./images/mysql/update_mysql_vsphere_49.png
[update_mysql_vsphere_50]:./images/mysql/update_mysql_vsphere_50.png

### [Index](https://github.com/okpc579/paasta-guide-new/blob/main/README.md) > [AP Install](../README.md) > MySQL Service
