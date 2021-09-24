# get binary-storage debian package 
- 본 문서는 [PAAS-TA-PORTAL-API-RELEASE](https://github.com/PaaS-TA/PAAS-TA-PORTAL-API-RELEASE)의 [binary_storage](https://github.com/PaaS-TA/PAAS-TA-PORTAL-API-RELEASE/tree/master/jobs/binary_storage) job 에 대한 가이드를 제공합니다.
- 본 문서는 각 패키지의 세부설정은 제공하지 않으며 해당 내용은 아래 참고자료를 활용하시기 바랍니다.

### 00.binary-storage package
```
python-3.6
swift-all-in-one-2.23.2-PaaS-TA-v2
keystone-15.0.1
mariadb-10.5.8
```
<br/>

### 01.전제조건
> identity service를 설치 및 구성하기 전에 데이터베이스 생성
```console
$ mysql -uroot

MariaDB [(none)]> CREATE DATABASE keystone;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'KEYSTONE_DBPASS';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'KEYSTONE_DBPASS';
```
<br/>
> python3 alternatives 설정 (python3 를 python 명령으로 사용하기 위함)
```console
$ sudo apt-get update
$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.6 100 --skip-auto
$ sudo apt-get install -y python3-pip
$ sudo update-alternatives --install /usr/bin/pip pip /usr/bin/pip3 100 --skip-auto
```
<br/>

### 02.install binary-storage 
> install dependencies & components
```console
$ sudo apt-get update
$ sudo apt-get install -y curl gcc memcached rsync sqlite3 xfsprogs \
                        git-core libffi-dev python3-setuptools \
                        liberasurecode-dev libssl-dev
$ sudo apt-get install -y python3-coverage python3-dev python3-nose \
                        python3-xattr python3-eventlet \
                        python3-greenlet python3-pastedeploy \
                        python3-netifaces  python3-dnspython \
                        python3-mock
$ sudo apt-get install -y python3-swiftclient python3-pymysql python3-openstackclient
```
<br/>
> install keystone (13.0.4는 db conn 버그가 있으므로 stein 으로 변경하여 15.0.1을 설치)
```console
$ add-apt-repository cloud-archive:stein  
$ sudo apt-get install keystone
```
<br/>
> install swift-all-in-one (whl 파일을 다운로드 받기위해 사용)
```console
$ mkdir -p /var/vcap/packages/swift-all-in-one/
$ git clone https://github.com/openstack/swift.git -b <버전>
$ pip install -r /var/vcap/packages/swift-all-in-one/swift/requirements.txt
```
<br/>
> CCE 조치를 위한 whl 파일들은 직접 실행 및 다운로드 해야함
```console
$ pip install PyNaCl
$ pip install PyMySQL[ed25519]
```
<br/>

### 03.get files deb & whl
> apt-get 명령어로 설치한 파일 위치 검색 / pip install 명령어로 설치한 파일 위치 검색
```console
$ find / -name *.deb 
$ ll /var/cache/apt/archives/

$ find / -name *.whl 
$ ll /usr/share/python-wheels/
```

### 참고 : deb files 상호 의존 패키지 에러 없이 설치하기 
> deb_install_script.sh 를 생성하여 일괄 설치 파일 만들어 실행
<br/> 설치할 deb file들은 /var/vcap/packages/swift-all-in-one/ubuntu-bionic-files/deb/ 에 있다고 가정
```console
$ cd /var/vcap/packages/swift-all-in-one/ubuntu-bionic-files/deb/
$ echo 'dpkg -i \' >> deb_install_script.sh 
$ find /var/vcap/packages/swift-all-in-one/ubuntu-bionic-files/deb/ -name '*.deb' -exec basename {}' \' \; >> deb_install_script.sh
$ echo ";" >> deb_install_script.sh
$ source deb_install_script.sh
```

### 참고 자료
swift: [https://docs.openstack.org/swift](https://docs.openstack.org/swift/latest/development_saio.html)<br/>
keystone: [https://docs.openstack.org/keystone](https://docs.openstack.org/keystone/latest/install/keystone-install-ubuntu.html)<br/>