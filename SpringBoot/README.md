11.21 AKS에 Spring Boot 애플리케이션 배포

** 사전 요구 사항
- Install Docker Desktop
- Install JDK
- Install Maven
- Intall Azure CLI
- Create AKS Cluster
- Create ACR


[ https://docs.microsoft.com/ko-kr/azure/developer/java/spring-framework/deploy-spring-boot-java-app-on-kubernetes]


# 디렉토리 만들기 및 이동
 mkdir C:\SpringBoot
 cd C:\SpringBoot

② Spring Boot on Docker 샘플 복제
 git clone https://github.com/spring-guides/gs-spring-boot-docker.git

③ 이동
 cd gs-spring-boot-docker
 cd complete

④ Maven을 사용하여 샘플 앱을 빌드하고 실행
 mvn package spring-boot:run

⑤ 웹앱 테스트
 curl http://localhost:8080

⑥ pom.xml 수정

⑦ 이미지 빌드 및 레지스트리 푸시
 az acr login && mvn compile jib:build

⑧ AKS 로그인
 az aks get-credentials --resource-group=<resource_group_name> --name=<aks_cluster_name>

⑨ AKS 클러스터에서 ACR 이미지를 받아 컨테이너 실행
 kubectl run gs-spring-boot-docker --image=sjleeacr.azurecr.io/spring-boot-docker-complete:latest

⑩ 서비스 이름, 앱에 액세스하는 데 사용되는 공용 TCP 포트 및 앱이 수신 대기하는 내부 대상 포트를 지정
 kubectl expose pod gs-spring-boot-docker --type=ClusterIP --port=80 --target-port=8080

⑪ 인그레스 작업 ?
 kubectl apply -f spring.yaml

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend-ingress
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
    appgw.ingress.kubernetes.io/backend-path-prefix: "/"
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          service:
            name: gs-spring-boot-docker
            port:
              number: 80
        pathType: Exact

⑫ 앱게이트웨이 주소 연결 및 확인
