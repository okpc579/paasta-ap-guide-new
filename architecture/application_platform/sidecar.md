### [Index](https://github.com/okpc579/paasta-guide-new/blob/main/README.md) > [AP Architecture](../README.md) > PaaS-TA Sidecar

## 목적
본 문서는 PaaS-TA Sidecar의 Architecture를 제공한다
<br><br>

## 시스템 구성도
![cf-for-k8s-component](https://www.cloudfoundry.org/wp-content/uploads/cf4k8s-1024x576.png)
- [istio](https://github.com/istio/istio)
- [envoy](https://github.com/envoyproxy/envoy)
- [fluentd](https://www.fluentd.org/)
- [eirini](https://www.cloudfoundry.org/project-eirini/)
- [kpack](https://github.com/pivotal/kpack)
- [paketo buildpacks](https://paketo.io/)
- [CF API / CAPI-k8s-release](https://github.com/cloudfoundry/capi-k8s-release)  
- [cf-k8s-networking](https://github.com/cloudfoundry/cf-k8s-networking)  
- [cf-k8s-logging](https://github.com/cloudfoundry/cf-k8s-logging)  
- [UAA](https://github.com/cloudfoundry/uaa)  

<br>

## 참고자료
PaaS-TA 컨테이너 플랫폼 : [https://github.com/PaaS-TA/paas-ta-container-platform](https://github.com/PaaS-TA/paas-ta-container-platform)  
Kubespray : [https://kubespray.io](https://kubespray.io)  
Kubespray github : [https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)  
cf-for-k8s github : [https://github.com/cloudfoundry/cf-for-k8s](https://github.com/cloudfoundry/cf-for-k8s)  
cf-for-k8s Document : [https://cf-for-k8s.io/docs/](https://cf-for-k8s.io/docs/)  

### [Index](https://github.com/okpc579/paasta-guide-new/blob/main/README.md) > [AP Architecture](../README.md) > PaaS-TA Sidecar
