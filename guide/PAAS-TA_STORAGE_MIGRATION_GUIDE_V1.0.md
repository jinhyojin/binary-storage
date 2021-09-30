# migration binary-storage
- 본 문서는 [PAAS-TA-PORTAL-API-RELEASE](https://github.com/PaaS-TA/PAAS-TA-PORTAL-API-RELEASE)의 [binary_storage](https://github.com/PaaS-TA/PAAS-TA-PORTAL-API-RELEASE/tree/master/jobs/binary_storage) job 에 대한 가이드를 제공합니다.
- 본 문서는 PaaS-TA Service 를 마이그레이션 하기 위한 용도이며 다른 서비스의 경우 가이드에서 제공하는 구성과 다를 수 있습니다. .

### 01.binary-storage 구성 및 명령어 변경 사항 (xenial -> bionic)
> package ([portal-api-release-src download](https://nextcloud.paas-ta.org/index.php/s/Yp7JEeDax9gy4yk/download))
```diff
- python-2.7.8
+ python-3.6.9

- swift-all-in-one-2.23.2-PaaS-TA
+ swift-2.23.2
+ swift-2.23.2-bionic-dependencies

- keystone-9.3.0
+ keystone-15.0.1

- wsgi (python : port 5000, 35357)
+ wsgi (apache2 : port 5000)
```
<br/>

> service
```diff
- systemctl stop/start keystone
+ systemctl stop/start apache2
```
<br/>

> keystone cli
```diff
- bootstrap-admin-url http://localhost:35357/v3/
+ bootstrap-admin-url http://localhost:5000/v3/  #bootstrap-public / admin 통합 (default port 5000) 
```
<br/>

> keystone.conf (신규 [keystone.conf](https://github.com/PaaS-TA/PAAS-TA-PORTAL-API-RELEASE/blob/3b4b7a89b35c4494d65bf137b931b08738955bb7/jobs/binary_storage/templates/config/keystone/etc/keystone.conf.sample.erb) 파일로 교체하는것을 추천) 
```diff
  [database]
- connection = mysql://<id>:<password>@<ip>:<port>/keystone
+ connection = mysql+pymysql://<id>:<password>@<ip>:<port>/keystone

  [token]
- #provider = uuid
+ provider = fernet
```
<br/>

> keystone client properties
```diff
- swift.authMethod: 'keystone'
+ swift.authMethod: 'keystone_v3'

- swift.authUrl: http://<ip>:<port>/v2.0/tokens
+ swift.authUrl: http://<ip>:<port>/v3/auth/tokens
```
<br/>


### 02.binary-storage jobs & packages 세부 변경 사항 ([branche:bionic](https://github.com/PaaS-TA/PAAS-TA-PORTAL-API-RELEASE/commits/bionic))
- [update to use ubuntu-bionic stemcells](https://github.com/PaaS-TA/PAAS-TA-PORTAL-API-RELEASE/commit/3b4b7a89b35c4494d65bf137b931b08738955bb7)
<br/>

### 03.binary-storage trouble shooting 
> get token 
```bash
$ curl -X POST http://<binary_storage.ip>:<binary_storage.auth_port>/v3/auth/tokens -d '{
    "auth": {
        "identity": {
            "methods": [
                "password"
            ],
            "password": {
                "user": {
                    "name": "<binary_storage.username>",
                    "domain": {
                        "name": "default"
                    },
                    "password": "<binary_storage.password>"
                }
            }
        }
    }
  }' -H "Content-Type: application/json"

# ===== response token =====
{"token": {"methods": ["password"], "user": {"domain": {"id": "default", "name": "Default"}, "id": "989206e4c53d4274affe03026ca953d3", "name": "paasta-portal", "password_expires_at": null}, "audit_ids": ["8QvHU5aWTlqa7mynRBUjKQ"], "expires_at": "2021-09-30T08:53:58.000000Z", "issued_at": "2021-09-30T07:53:58.000000Z", "project": {"domain": {"id": "default", "name": "Default"}, "id": "77b13b045f374370bd878119005ec49d", "name": "paasta-portal"}, "is_domain": false, "roles": [{"id": "bb5feb04d7d5468ea2025f89bb24598e", "name": "member"}, {"id": "c67b350ddad64aa385df029b7a5695fd", "name": "reader"}, {"id": "947b4559f09d4be4ba8bf5818ef654b4", "name": "admin"}], "catalog": [{"endpoints": [{"id": "157e76f7fa9c4f5ab3fba904beff1088", "interface": "admin", "region_id": "paasta", "url": "http://localhost:10008/v1", "region": "paasta"}, {"id": "c7e7a14defc542108c0f6fb6d3c9633d", "interface": "public", "region_id": "paasta", "url": "http://100.0.0.68:10008/v1/AUTH_77b13b045f374370bd878119005ec49d", "region": "paasta"}, {"id": "f53c07dc2f114b179c3de1d5e43c54ea", "interface": "internal", "region_id": "paasta", "url": "http://localhost:10008/v1/AUTH_77b13b045f374370bd878119005ec49d", "region": "paasta"}], "id": "ee7ae021011547efb5008d39df9b5f4e", "type": "object-store", "name": "swift"}, {"endpoints": [{"id": "7e47d2b7ddbe4edb9a41cbe23057d886", "interface": "public", "region_id": "paasta", "url": "http://100.0.0.68:15001/v3/", "region": "paasta"}, {"id": "a706b2d126a046ae87c08049c9015e79", "interface": "internal", "region_id": "paasta", "url": "http://localhost:15001/v3/", "region": "paasta"}, {"id": "b6a2c5c7e6f548078c2d70baa364df37", "interface": "admin", "region_id": "paasta", "url": "http://localhost:15001/v3/", "region": "paasta"}], "id": "a08fa818eab14a2ab32c9003eb9e497b", "type": "identity", "name": "keystone"}, {"endpoints": [], "id": "9704b6cbb0a74286b43ee4cb5580ef94", "type": "identity", "name": "keystone"}]}}

```
<br/>

> if there is no catalog content in the response token (keystone_v3 login failed)
```bash
# ===== response token =====
{"token": {"methods": ["password"], "user": {"domain": {"id": "default", "name": "Default"}, "id": "989206e4c53d4274affe03026ca953d3", "name": "paasta-portal", "password_expires_at": null}, "audit_ids": ["8QvHU5aWTlqa7mynRBUjKQ"], "expires_at": "2021-09-30T08:53:58.000000Z", "issued_at": "2021-09-30T07:53:58.000000Z", "project": {"domain": {"id": "default", "name": "Default"}, "id": "77b13b045f374370bd878119005ec49d", "name": "paasta-portal"}, "is_domain": false, "roles": [{"id": "bb5feb04d7d5468ea2025f89bb24598e", "name": "member"}, {"id": "c67b350ddad64aa385df029b7a5695fd", "name": "reader"}, {"id": "947b4559f09d4be4ba8bf5818ef654b4", "name": "admin"}]}}

$ export OS_USERNAME=admin
$ export OS_PASSWORD=<binary_storage.admin_password>
$ export OS_PROJECT_NAME=admin
$ export OS_USER_DOMAIN_NAME=Default
$ export OS_PROJECT_DOMAIN_NAME=Default
$ export OS_AUTH_URL=http://localhost:<binary_storage.auth_port>/v3
$ export OS_IDENTITY_API_VERSION=3
$ openstack role add --project <binary_storage.projectname> --user <binary_storage.username> admin
$ curl -X POST http://<binary_storage.ip>:<binary_storage.auth_port>/v3/auth/tokens -d '{
    ....
  }' -H "Content-Type: application/json"
```
<br/>

> swift file upload / download test
```bash
$ swift --version
python-swiftclient 3.7.0

$ openstack container create <container_name>
$ swift upload <container_name> <file_name>
$ openstack object list <container_name>
$ swift download <container_name> <file_name>
```
<br/>

### 참고 자료
swift: [https://docs.openstack.org/swift](https://docs.openstack.org/swift/latest/development_saio.html)<br/>
keystone: [https://docs.openstack.org/keystone](https://docs.openstack.org/keystone/latest/install/keystone-install-ubuntu.html)<br/>