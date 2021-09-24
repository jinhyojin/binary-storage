# get migration binary-storage
- 본 문서는 [PAAS-TA-PORTAL-API-RELEASE](https://github.com/PaaS-TA/PAAS-TA-PORTAL-API-RELEASE)의 [binary_storage](https://github.com/PaaS-TA/PAAS-TA-PORTAL-API-RELEASE/tree/master/jobs/binary_storage) job 에 대한 가이드를 제공합니다.
- 본 문서는 PaaS-TA Service 를 마이그레이션 하기 위한 용도이며 다른 서비스의 경우 가이드에서 제공하는 구성과 다를 수 있습니다. .

### 01.binary-storage 구성 및 명령어 변경 사항 (xenial -> bionic)
> package
```diff
- python-2.7
+ python-3.6

- swift-all-in-one-2.23.2-PaaS-TA
+ swift-all-in-one-2.23.2-PaaS-TA-v2   #ubuntu-bionic 전용 deb / whl 파일 추가 

- keystone-9.3.0
+ keystone-15.0.1

- wsgi (python : port 5000, 35357)
+ wsgi (apache2 : port 5000)

- deb files install : foreach ....
+ deb files install : make sh file (deb_install_script.sh)

```
<br>

> service
```
- systemctl stop/start keystone
+ systemctl stop/start apache2
```
<br>

> keystone cli
```diff
- bootstrap-admin-url http://localhost:35357/v3/
+ bootstrap-admin-url http://localhost:5000/v3/  #bootstrap-public / admin 통합 (default port 5000) 
```
<br>

> keystone.conf (신규 keystone.conf 파일로 교체하는것을 추천)
```diff
  [database]
- connection = mysql://<id>:<password>@<ip>:<port>/keystone
+ connection = mysql+pymysql://<id>:<password>@<ip>:<port>/keystone

  [token]
- #provider = uuid
+ provider = fernet
```
<br>

> keystone client properties
```diff
- swift.authMethod: 'keystone'
+ swift.authMethod: 'keystone_v3'

- swift.authUrl: http://<ip>:<port>/v2.0/tokens
+ swift.authUrl: http://<ip>:<port>/v3/auth/tokens
```
<br>


### 02.binary-storage JOB 세부 변경 사항 ([branche:bionic](https://github.com/PaaS-TA/PAAS-TA-PORTAL-API-RELEASE/commits/bionic))
- [update to use ubuntu-bionic stemcells](https://github.com/PaaS-TA/PAAS-TA-PORTAL-API-RELEASE/commit/26724b88a676917d8c6465e5b0844eed19787a16)

