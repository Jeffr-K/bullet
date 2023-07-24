##

- [Service Discovery Pattern] https://www.msaschool.io/operation/design/design-six/
- 서버 쪽에서 디스커버리 로직을 구현한 방식 
- aws ELB, 구글 로드 밸런서 : Service discovery 
- kube-dns + etcd : Service registry 
- kubernetes proxy : Service loadbalancer


registration 방식
서비스들은 Service Registry 에 각 서비스를 등록/해제 해야합니다. 이렇게 서비스를 등록하는 방식은 두가지 방식이 있습니다.

1. Self-Registration

서비스 인스턴스는 service registry에 스스로 등록/해제할 책임이 있습니다.
필요하다면 등록정보가 만료되지 않게 하기위해 heartbeat를 service registry에 보내야합니다.
Netflix OSS Eureka client 가 대표적인 예 입니다.

Eureka client는 Eureka(service registry)에 서비스 등록/해제를 자동으로 처리합니다.
서비스의 Java Configuration 클래스에 @EnableEurekaClient 어노테이션만 달아주면 됩니다.

Self-Registration 장점
비교적 단순합니다.
별도의 시스템 컴포넌트가 추가될 필요가 없습니다.
Self-Registration 단점
서비스 인스턴스와 Service Registry간의 높은 결합도(coupling)이 생긴다.
서비스에서 등록/해제 관련된 코드를 구현해야됩니다.
2. 3rd party registration

서비스 인스턴스가 직접 service registry에 등록/해제에 대한 책임이 없습니다.
대신, 서비스 등록기(service registrar)가 service registry에 등록을 해준다.
서비스 등록기는 실행중인 서비스 인스턴스들에 polling을 하거나 이벤트를 구독하는 등의 작업을 통해서 서비스의 변경을 감지하고, service registry에 등록/해제를 합니다.
Netflix Prana 나 Registrator 등이 이에 해당합니다.
쿠버네티스에서는 명식적인 Service registry 가 없어서 infrastructure 의 built-in registry 를 이용합니다.

Netflix Prana : Non-JVM 시스템을 side-car 방식으로 같이 실행을 하여 Eureka 에 등록/해제 를 해줌.
Registrator: Docker containers 를 다양한 (etcd, Consul) Service registry 에 등록/해제 하여 준다.

Third-Party Registration 장점
서비스 인스턴스와 service registry가의 결합도를 끊을(decoupled) 수 있습니다.
서비스에 별도의 등록/해제 로직을 구현할 필요가 없습니다.
Third-Party Registration 단점
운영환경에 추가적인 시스템 컴포넌트가 필요합니다.
이 시스템은 설치 및 관리되어야 하고, 고가용성을 유지해야 합니다.