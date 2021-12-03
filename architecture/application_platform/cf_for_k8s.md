

## 목적
본 문서는 cf-for-k8s의 Architecture를 제공한다

<br><br>

## 시스템 구성도

cf-for-k8s의 컴포넌트 구성은 다음과 같다.
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
> https://github.com/PaaS-TA/paas-ta-container-platform  
> https://kubespray.io  
> https://github.com/kubernetes-sigs/kubespray  
> https://github.com/cloudfoundry/cf-for-k8s  
> https://cf-for-k8s.io/docs/  

