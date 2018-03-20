# Installation-of-Openstack-Pike

Installation of Openstack: Pike 

Previous problems

1.Setting Environment

-controller 노드에서, 다른 웹에는 모두 ping을 하는데 openstack.org에만 연결을 못하고 있는 문제. http포트, ssh 포트 모두 다른 웹과 다른 서버와는 통신 잘 되는 것 확인. => 보안 문제?
=> openstack.org에만 핑을 못하는 이유는 학교 네트워크 관련 문제인 것 같음.
다른 개인 우분투로는 ping이 굉장히 잘 되는 것 확인. 학교 서버들만 안됨! 
=>보안상 문제로 ping 막힌 것 확인.



-compute2 ssh 포트로 통신이 아예 안되고 있어서 이것저것 만지다가 뭔가 잘못 되었다. 포맷할까…?

2.Install openstack service
keystone
[HTTP 503]


#keystone-manage db_sync keystone 결과

memcached, messagequeue 정상작동 확인

/etc/nova/nova.conf 일부

-> 이부분 분석해서 해결해보기
https://ask.openstack.org/en/question/93695/openstack-compute-service-list-unknown-error-http-503-service-unavailable/
에러로그 확인하고

Keystone database 설정 문제
mysql 들어가서 use keystone; show tables; 입력 시 테이블은 생성되나 project 생성이 되지 않음
status; 입력으로 characterset 을 확인해보면 latin1, utf8 로 설정값이 모두 다르게 나옴

/etc/mysql/mariadb.conf.d 폴더에서 cnf 파일들의 characterset 의 utf8을 모두 latin1으로 바꿔야함
바꿨는데도 status 설정이 바뀌지 않음?
table은 정상으로 생기는데….
openstack project create --domain default \
  --description "Service Project" service
 실행 시 아래 화면 오류














Study

*Concepts*
Service
Project name
Description
Dashboard
Horizon
Provides a web-based self-service portal to interact with underlying OpenStack services, such as launching an instance, assigning IP addresses and configuring access controls.
Compute service
Nova
Manages the lifecycle of compute instances in an OpenStack environment. Responsibilities include spawning, scheduling and decommissioning of virtual machines on demand.
Networking service
Neutron
Enables Network-Connectivity-as-a-Service for other OpenStack services, such as OpenStack Compute. Provides an API for users to define networks and the attachments into them. Has a pluggable architecture that supports many popular networking vendors and technologies.
Object Storage service
Swift
Stores and retrieves arbitrary unstructured data objects via a RESTful, HTTP based API. It is highly fault tolerant with its data replication and scale-out architecture. Its implementation is not like a file server with mountable directories. In this case, it writes objects and files to multiple drives, ensuring the data is replicated across a server cluster.
Block Storage service
Cinder
Provides persistent block storage to running instances. Its pluggable driver architecture facilitates the creation and management of block storage devices.
Identity service
Keystone
Provides an authentication and authorization service for other OpenStack services. Provides a catalog of endpoints for all OpenStack services.
Image service
Glance
Stores and retrieves virtual machine disk images. OpenStack Compute makes use of this during instance provisioning.
Telemetry service
Ceilometer
Monitors and meters the OpenStack cloud for billing, benchmarking, scalability, and statistical purposes.
Orchestration service
Heat
Orchestrates multiple composite cloud applications by using either the native HOT template format or the AWS CloudFormation template format, through both an OpenStack-native REST API and a CloudFormation-compatible Query API.
Database service
Trove
Provides scalable and reliable Cloud Database-as-a-Service functionality for both relational and non-relational database engines.
Data Processing service
Sahara
Provides capabilities to provision and scale Hadoop clusters in OpenStack by specifying parameters like Hadoop version, cluster topology and nodes hardware details.

*Keystone*
The OpenStack Identity service provides a single point of integration for managing authentication, authorization, and a catalog of services.
The Identity service is typically the first service a user interacts with. Once authenticated, an end user can use their identity to access other OpenStack services. Likewise, other OpenStack services leverage the Identity service to ensure users are who they say they are and discover where other services are within the deployment. The Identity service can also integrate with some external user management systems (such as LDAP).
Users and services can locate other services by using the service catalog, which is managed by the Identity service. As the name implies, a service catalog is a collection of available services in an OpenStack deployment. Each service can have one or many endpoints and each endpoint can be one of three types: admin, internal, or public. In a production environment, different endpoint types might reside on separate networks exposed to different types of users for security reasons. For instance, the public API network might be visible from the Internet so customers can manage their clouds. The admin API network might be restricted to operators within the organization that manages cloud infrastructure. The internal API network might be restricted to the hosts that contain OpenStack services. Also, OpenStack supports multiple regions for scalability. For simplicity, this guide uses the management network for all endpoint types and the default RegionOne region. Together, regions, services, and endpoints created within the Identity service comprise the service catalog for a deployment. Each OpenStack service in your deployment needs a service entry with corresponding endpoints stored in the Identity service. This can all be done after the Identity service has been installed and configured.
The Identity service contains these components:
Server
A centralized server provides authentication and authorization services using a RESTful interface.
Drivers
Drivers or a service back end are integrated to the centralized server. They are used for accessing identity information in repositories external to OpenStack, and may already exist in the infrastructure where OpenStack is deployed (for example, SQL databases or LDAP servers).
Modules
Middleware modules run in the address space of the OpenStack component that is using the Identity service. These modules intercept service requests, extract user credentials, and send them to the centralized server for authorization. The integration between the middleware modules and OpenStack components uses the Python Web Server Gateway Interface.



































Procedure

*Setting*
1.Password
Password name
Description
Database password (no variable used)
Root password for the database
ADMIN_PASS
Password of user admin
CINDER_DBPASS
Database password for the Block Storage service
CINDER_PASS
Password of Block Storage service user cinder
DASH_DBPASS
Database password for the Dashboard
DEMO_PASS => 1234
Password of user demo
GLANCE_DBPASS
Database password for Image service
GLANCE_PASS
Password of Image service user glance
KEYSTONE_DBPASS
Database password of Identity service
METADATA_SECRET
Secret for the metadata proxy
NEUTRON_DBPASS
Database password for the Networking service
NEUTRON_PASS
Password of Networking service user neutron
NOVA_DBPASS
Database password for Compute service
NOVA_PASS
Password of Compute service user nova
PLACEMENT_PASS 
Password of the Placement service user placement
RABBIT_PASS
Password of RabbitMQ user openstack

+demo user password = 1234

2.Server Address

순서대로 compute2, controller, compute1.

*진행상황*

[날짜별]
1월 2일(화)
NTP 다시 설정
Install and Configure components 까지 완료
Configure the Apache HTTP Server 부터 

1월 4일(목)
Endpoint 구글링
Nova Compute node1: Finalize installaion까지 완료
Add the compute node to the cell database에서 에러

1월 5일(금)
진전 없음. glance까지 작동이 안됨
주말까지 해결해보고, 정 해결이 안되면 포맷한 후 139로 컨트롤러를 변경하여 진행.

1월 8일(월)
135 서버 포맷
controller에서 문제 되는 부분이 placement API 임을 찾음
문제점을 알았으니 해결할 수 있을거라는….희망을 가지고 진행 예정...ㅜㅜ

1월 9일(화)
controller의 log파일에서 찾아낸 메세지 큐 에러 해결(큐에 openstack user 추가)
compute1에서 admin-openrc 스크립트 추가하여 실행
그러나 여전히 에러. -> placement API에서 인증 실패 문제 

1월 11일(목)
Authentication ERROR 드디어 해결 (Sol. NTP/pipeline)
Compute 2 노드 세팅 후 Controller와 연결 완료
Neutron 설치 도중 필요한 Provider Network Name에 대한 정보 검색 중

1월 12일(금)
Neutron 설치 완료
Horizon 설치 후 HTTP 500 Internal Server Error.
에러 로그 분석 중

1월 14일(일)
Horizon 에러 해결 완료
대시보드 통해 admin/demo 모두 접속 되는 것까지 확인 완료
다음주부터 초기 설정 후 ceilometer 설치할 예정

1월 15일 ~ 17일 컴퓨터 시스템 소사이어티 컨퍼런스 참석

1월 18일(목)
Openstack: Pike Installation Guide 내용 정리
Ceilosca API Technical Report 작성 시작. 아래 URL로 넘김
https://docs.google.com/document/d/1kV8iZN84uKkBzAqjc2zKv7iwXUvoBrKIQE61fL45vZc/edit# 

2월 1일 ~ 2월 5일 호라이즌에서 selservice network 및 instance 생성 테스트
 	이 과정에서 발생한 트러블 슈팅 정리

2월 6일 (화) 
instance에서 ubuntu 설치하는 과정 중 openssh로 콘솔에 접근 불가능한 문제 발견 
selfservice 네트워크 환경이 아직 엉성함
[Section별]
1.Keystone 중반부
keystone User, Tenant, Role and Endpoint 생성
Service project
	for unique user for each service that i add to my environment.

Demo project
	for non-admin(regular) tasks. 

Demo user 생성
User role 생성
생성한 User role 을 demo project와 user 에 추가


As the admin user, request an authentication token:


As the demo user, 

 API port 5000 which only allows regular (non-admin) access to the Identity service API.


지금까지 openstack client를 통한 Identity service로 상호작용을 위한 옵션들과 변수를 설정했고,
이제 client file들의 효율성을 높이기 위해 OpenRC라 알려진 환경 스크립트를 설정한다. 
 
Note
The paths of the client environment scripts are unrestricted. For convenience, you can place the scripts in any location, however ensure that they are accessible and located in a secure place appropriate for your deployment, as they do contain sensitive credentials.

admin과 demo 의 프로젝트와 유저에 대한 스크립트들을 각각 생성. -> Guide 대로
=>결과



P. 설치가 잘 완료되었는 지 확인하기 위해 #keystone token-get 실행하였더니 다음과 같이 패키지 설치를 하라고 한다. 하지만 설치해도 계속 해당 패키지들을 못 찾는다.
S. 예전 버전의 command이기 때문에 알아 듣지 못한다. keystone이라는 명령어 자체가 사라지고 openstack으로 전부 변경된 것 같다. 이 명령어 대신 #openstack user list 를 사용하면 결과가 잘 나온다. 다음은 openstack --help에서 유용한 명령어만 선별하여 정리한 목록이다.

[Pike 버전에서 바뀐 keystone 명령어 정리 (잘 설치되었는 지 확인 위해서)]
domain list 
domain show
endpoint list/show
module list
policy list/show
role list/show
server list/show
service list/show
service provider list/show
user list/show

*Keystone 설치 확인 결과*


&d이거 앤드포인트 이해가 잘???? 이게 뭐지 구글링구글링 -->이따 갔다오면서



-현재 오픈스택 모듈 리스트



2.Glance

초반 부의 glance database 생성부분이 생략되어 아주 간단히 적혀있는데, 대충 넘어가다가 빠트리면 안된다. keystone에서 database 생성 부분을 참조하여 glance에 대한 database 생성과 권한 부여를 해주고 넘어가야 한다.

glance도 마찬가지로 명령어가 변경되었다. #glance image-list가 아닌 #openstack image list를 입력하면 정상 작동 여부를 확인 가능하다.

3. Nova
nova.conf 변경시키는 부분에서 시간이 많이 할애됨
[DEFAULT] : comment out 할 것 X. 바로 추가
[api] : 이미 존재. 
[keystone_authtoken]: comment out X
[glance]: 이미 존재. 내용 변경 필요
[oslo_concurrency]: 이미 존재. 내용 변경 필요
[placement]: os_region_name을 comment out하고 추가.
[vnc]: 이미 존재. 내용 변경.
***DEFAULT의 가장 윗부분에 있는 log_dir option을 제거해야함. -> 패키징 버그

-------------------------------------------------------------------------------------------------------------------

P. Compute node-Finalize installation까지 완료. service restart 한 후에 controller에서 compute 노드를 데이터 베이스에 추가시키는 부분에서 에러


P. openstack --debug image list 실행 시 결과


P. nova에서 placement API 가 문제인 것 확인!


Placement API?
Nova introduced the placement API service in the 14.0.0 Newton release. This is a separate REST API stack and data model used to track resource provider inventories and usages, along with different classes of resources. For example, a resource provider can be a compute node, a shared storage pool, or an IP allocation pool. The placement service tracks the inventory and usage of each provider. For example, an instance created on a compute node may be a consumer of resources such as RAM and CPU from a compute node resource provider, disk from an external shared storage pool resource provider and IP addresses from an external IP pool resource provider.
: Nova와 함께 설치하지만 원칙적으로는 별개의 API이며 다양한 리소스들을 추적하고 관리하는 역할을 한다. 예를 들어 컴퓨트 노드 위에 생성된 인스턴스는 컴퓨트 노드의 CPU나 RAM 등을 사용해야 하므로, 이 API가 해당 정보들을 저장하고 있다가 각각의 인스턴스에 적절한 IP를 할당해준다. 이 리소스들은 class 단위로 관리된다.

The FilterScheduler now requests allocation candidates from the Placement service during scheduling. The allocation candidates information was introduced in the Placement API 1.10 microversion, so you should upgrade the placement service before the Nova scheduler service so that the scheduler can take advantage of the allocation candidate information.

-------------------------------------------------------------------------------------------------------------------------

P. 기존의 발견한 문제점 외에 message error를 꽤 발견하여 message queue 설정 과정을 다시 진행하였음. 
S. #rabbitmqctl list_users 명령어를 실행하여 현재 유저를 확인 가능하니,  설치시 반드시 확인하고 넘어가기 바람.

이렇게 openstack [] 이라는 유저가 생성되어야 정상.

이로 인해 active 되지 않고 있던 nova_consoleauth 와 nova_scheduler 가 정상 활성화되었다.
nova 설치 후, #service <service name> status 명령어를 통해 5개의 서비스가 active 되었는 지 역시 반드시 확인하고 넘어갈 것.

이렇게 활성화 되어야 정상.

P. 다시 placement API가 인증 부분에서 비정상 작동하는 문제.

여기부터 다시

[Check]
nova-scheduler 정상작동
nova-consoleauth 정상작동
nova-conductor 정상
nova-api 정상
nova-novncproxy 정상

glance-api WARNING



---------------------------------------------------------------------------------------------------------------------------------
위와 같은 Authentication 문제가 발생하는 데, 각각의 서비스는 잘 동작하고 있다면 NTP 설정이 잘 되어 있는지 먼저 확인하고, 그래도 안된다면 /etc/keystone/keystone-paste.ini 의 pipeline 설정을 잘 확인해보기 바란다. NTP 설정을 변경하여 #systemctl restart chrony 명령어로 동기화를 시켜줌과 동시에 keystone-paste.ini의 pipeline 순서를 공식 사이트 예시와 비교하여 살짝 뒤바뀌어있는 순서를 제대로 고쳐주었더니 위의 문제가 한꺼번에 해결되었다!

아래는 keystone-paste.ini의 예시 코드이다. 
https://docs.openstack.org/keystone/latest/configuration/samples/keystone-paste-ini.html

***에러가 생겼을 때 공식사이트에서 제공하는 예시 코드를 보고 비교!

[Trouble Shooting - Results]


 
[Compute2 노드 추가한 결과]



4. Neutron
Option 1: provider network 로 설치

Q. Provider Network 주소를 config해야 하는데 무엇인 지 모르겠음. 구글링 
=> 각 서버마다 자기 이더넷 인터페이스 이름 ex.eno1, eth0 같은.

※이후에 Provider를 그대로 사용하는 것이 아닌, Selfservice network를 따로 구축하여 사용하려는 경우 반드시 Option 2로 설치해야만 함! 본인과 같은 경우, 학교망인 163.152.20.0 를 provider network로 사용하려 했기 때문에 192.168.0.0 이라는 내부망을 구축해야 했음. 따라서 option 1으로 설치했던 neutron을 다시 option 2로 설치했음. 자신의 상황을 고려하여 option 선택을 잘 하기바람

5. Horizon
[TroubleShooting]
1.var/lib/openstack-dashboard 에서 permission 설정하는 부분
P. controller/horizon에 접속 시 Internal Server Error가 발생 -> var/lib/apache2/error.log를 확인해보니 permission denied 문제가 발생하고 있었음
S. var/lib/openstack-dashboard에서 permission 설정을 해주어야 함. 처음 설치할 때 왜 잘못 되었는 지는 모르겠지만, 아래 url 참고한 후 아래 캡쳐처럼 권한 설정을 변경해주었더니 해결
https://stackoverflow.com/questions/42632130/cant-launch-openstack-horizon-dashboard-ioerror-errno-13-permission-denied


2.apache 에러 -> 디장고 파일 못 읽어오는 부분
 이런 식으로 Truncated or oversized 헤더 문제가 발생하는 경우,
S. GLOBAL 설정을 다음과 같이 변경해주어야 함. 아래 url 참고하여 /etc/apache2/conf-available/openstack-dashboard.conf 를 다음과 같이 변경함
https://ask.openstack.org/en/question/100625/solved-gateway-timeout-error-cannot-access-horizon-on-openstack-newton/


3.apache 에러 -> INVALID ALLOWED HOST: controller 추가

다음과 같이 django의 보안 문제 때문에 HTTP_HOST 헤더가 문제되는 경우.
S. 이 경우는 구글링을 아무리 해보아도 해결법이 딱히 나오지 않아, 에러 메세지가 하라는 대로 /etc/openstack-dashboard/local_settings.py 의 ALLOWED_HOSTS 옵션에 기존의 설치 가이드에서 제시한 example.com 두개 말고도 controller까지 새로 추가해줌으로써 해결하였다.


설치가 완료되었으면 controller/horizon에 접속하여 domain = default, user=admin 또는 demo 로 접속. 패스워드는 본인이 설정했던 패스워드로. (ADMIN_PASS, 1234)

6. Horizon에서 Selfservice network 및 Instance 생성하기
https://docs.openstack.org/neutron/pike/admin/deploy-lb-selfservice.html

[TroubleShooting]
P. Instance Build 중 Error 발생 -> “no tenant network is available for allocation”
linuxbridgeagent가 제대로 실행되지 않고, nova가 port binding을 실패하는 오류

linuxbridge_agent.log

호스트의 인터페이스에 local_ip가 바운드 되지 않고서는 tunneling이 불가능하단 얘기. local_ip 설정이 뭔가 잘못 된 것 같다. local_ip를 제대로 config 해달라….

nova-compute.log

네트워크 할당 부분이 제대로 동작 안되고...
무엇보다 포트를 바인딩 하는데에 실패함으로써 에러가 시작됨. 

S.
https://ask.openstack.org/en/question/103199/neutron-linux-bridge-cleanup-fails-on-host-startup/
The problem was caused by network configuration of the host.
Host was configured to get Ip from DHCP but server had not given IP yet when Linux bridge agent needed it.
Solution was to configure static IP.
=> local_ip 가 static IP로 config 되어야 한다는 말인 것 같은데..확실히 local_ip 설정이 잘못 된 것 같긴 함

***local_ip 는 각각 자신의 ip (예를 들어 163.152.20.141)로 config 해주어야 했음! 


이제 instance 생성 후 콘솔로 접근하여 자신이 설정한 flavor에 맞게 운영체제를 설치해주면. 되지 않는다.
