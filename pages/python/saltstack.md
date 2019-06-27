## 序言
新工作需要对所有的运维自动化平台重新规划，这个季度的工作重点就是CD（持续交付平台），在产品设计中，将会建立制品库用于包文件管理（包括前端和后端）以及版本管理；我们就通过saltstack来打通系统之间的数据传输渠道；这样既可以解决数据加密问题还能利用其并发优势；

只是通过命令行来管理主机，并不能达到我们的要求；所以我们下一步将要实现基于saltstack的配置管理引擎，专门用于主机节点管理和部署；在通过salt-api实现接口调用，以此作为自动化的根基；

python3.0自从2009年第一个版本发布，到2019年（python3.7），经历了10年的演进，可以说，若不考历史原因，其性能和效率都已达到和超过2的最高水平；特别是对国内用户来说，编码问题始终让人头疼，python3对utf8的支持非常友好，毕竟未来还是python3的天下，所以最终选择在python3的环境下搭建使用saltstack。

说道，centos7和python3再加上saltstack的实现，坑不能说少，但绝对值得你去尝试一下。

## 系統环境
> OS （CentOS发行版）

<span style="color: blue">用于salt master节点</span>

操作系统 | 内核版本 | CPU | Memory
---|---|:---:|:---:
CentOS Linux release 7.6.1810 (Core) | 3.10.0-693.2.2.el7.x86_64 | 2 Core | 4GB

> Python

版本 | 资源地址
---|---
3.6.8 | https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tar.xz
get-pip.py | https://bootstrap.pypa.io/get-pip.py

> salstack

组件 | 版本 
---|---
salt-api | 2019.2.0-1.el7.noarch
salt-ssh | 2019.2.0-1.el7.noarch
salt-master | 2019.2.0-1.el7.noarch
salt-minion | 2019.2.0-1.el7.noarch
salt | 2019.2.0-1.el7.noarch
salt-syndic | 2019.2.0-1.el7.noarch
salt-cloud | 2019.2.0-1.el7.noarch

## CentOS7.6 Python2.7升级Python3.6
首先安装python3

可以不安装Python3, 默认安装py3版本的salt，它会自己安装一个python3.4版本，所有的salt操作都是在这个python3.4版本上运行的，只不过我自己的项目需要python3，所以自己安装了一个。特此说明

    wget https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tar.xz
    tar zxvf Python-3.6.8.tgz   
    cd  Python-3.6.8        
    ./configure --prefix=/usr/local/python/Python-3.6.8 --enable-optimizations
    make
    make install
    mv  /usr/bin/python /usr/bin/python2 # 如果是软连接，可以直接删除
    cd /usr/local/python/Python-3.6.8
    ln -s bin/python3.6 /usr/bin/python
    vim /usr/bin/yum   # 修改Yum,使yum依然有效，yum依靠老版本的python
    #!/usr/bin/python 修改为#!/usr/bin/python2
        
修改完/usr/bin/yum 依然还有问题，可以尝试修改/usr/libexec/urlgrabber-ext-down的文件python抬头

使用Python3直接启动salt，因为默认环境已经切换的python3, 所以直接启动即可

## 安装saltstack
```
# 更新yum
yum update

# Centos7 - Python3 - salt 安装源
yum install -y https://repo.saltstack.com/py3/redhat/salt-py3-repo-latest-2.el7.noarch.rpm 

# CentOS6 - Python2
yum install https://repo.saltstack.com/yum/redhat/salt-repo-latest.el6.noarch.rpm

yum clean expire-cache
# 安装必要软件（mariadb是mysql，用于存储salt命令执行结果和jobid，可不安装）
yum -y install mariadb mariadb-devel mariadb-server wget  python-devel gcc c++ make openssl openssl-devel passwd libffi libffi-devel
# 安装salt
yum install salt-master salt-minion salt-ssh salt-syndic salt-cloud salt-api

# Centos7/6  -Python2 安装源
yum install https://repo.saltstack.com/yum/redhat/salt-repo-latest-2.el7.noarch.rpm
yum install https://repo.saltstack.com/yum/redhat/salt-repo-latest-2.el6.noarch.rpm
```

```
# yum install salt-master salt-minion salt-ssh salt-syndic salt-cloud salt-api

===================
salt-api
salt-cloud
salt-master
salt-minion
salt-ssh
salt-syndic
-----------------------------------
libsodium 
libtomcrypt
libtommath
openpgm 
python34
python34-PyYAML
python34-backports_abc
python34-cherrypy
python34-crypto
python34-jinja2
python34-libcloud 
python34-libs
python34-markupsafe
python34-msgpack
python34-psutil
python34-pycurl
python34-setuptools 
python34-six
python34-tornado
python34-zmq
salt
zeromq 

yum 安装的依赖包
```

## 安装salt api
```
# 上一步已经安装了，写下单独安装的命令
# yum install salt-api  -y
 
# 创建证书
[root@centos7 ~]# cd /etc/pki/tls/certs/
# 生成自签名证书，用于ssl
[root@centos7 certs]# make testcert     
umask 77 ; \
/usr/bin/openssl genrsa -aes128 2048 > /etc/pki/tls/private/localhost.key
Generating RSA private key, 2048 bit long modulus
...................................................................+++
..+++
e is 65537 (0x10001)
Enter pass phrase:       # 输入加密密语，4到8191个字符
Verifying - Enter pass phrase:   # 确认加密密语
umask 77 ; \
/usr/bin/openssl req -utf8 -new -key /etc/pki/tls/private/localhost.key -x509 -days 365 -out /etc/pki/tls/certs/localhost.crt -set_serial 0
Enter pass phrase for /etc/pki/tls/private/localhost.key:     # 再次输入密语
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN      # 选填，可不填写直接回车
State or Province Name (full name) []:Shanghai  # 选填，可不填写直接回车
Locality Name (eg, city) [Default City]:Shanghai  # 选填，可不填写直接回车
Organization Name (eg, company) [Default Company Ltd]: # 选填，可不填写直接回车
Organizational Unit Name (eg, section) []: # 选填，可不填写直接回车
Common Name (eg, your name or your server's hostname) []: # 选填，可不填写直接回车
Email Address []: # 选填，可不填写直接回车
[root@centos7 certs]# cd ../private/
# 解密key文件，生成无密码的key文件, 过程中需要输入key密码，该密码为之前生成证书时设置的密码
[root@centos7 private]# openssl rsa -in localhost.key -out localhost_nopass.key
Enter pass phrase for localhost.key:
writing RSA key
[root@centos7 private]# ls
localhost.key  localhost_nopass.key
# 备注
    如果make testcert出现错误，则删除/etc/pki/tls/private/localhost.key文件，然后再make testcert
# 创建用户（用于salt-api认证）
useradd -M -s /sbin/nologin saltapi && echo "password"|/usr/bin/passwd saltapi --stdin
```

```
# 单独安装pip的方式
    wget https://bootstrap.pypa.io/get-pip.py
    python get-pip.py

# 升级下pip
    pip install --upgrade pip

# pip 安装salt-api所需软件，最新版本中默认yum已经安装，无需安装
    pip install pyOpenSSL   
    pip install cherrypy   

pip 安装与升级
```
## salt api相关配置说明
```
# 添加配置文件,可以把eauth.conf和api.conf合二为一为api.conf
[root@centos7 ~]# mkdir -p /etc/salt/master.d/        
# 这个目录默认不存在，需要手动创建，在/etc/salt/master主配置文件中有指定，类似include
[root@centos7 ~]# vim /etc/salt/master.d/eauth.conf   
# 处于安全因素，一般只给特定模块的使用权限，这里给saltapi用户所有模块的使用权限       
external_auth:
  pam:
    saltapi:
      - .*
      - '@wheel'
      - '@runner'
        
[root@centos7 ~]# vim /etc/salt/master.d/api.conf 
rest_cherrypy:
  port: 8000                       #  salt-api 监听端口
  ssl_crt: /etc/pki/tls/certs/localhost.crt          # ssl认证的证书
  ssl_key: /etc/pki/tls/private/localhost_nopass.key
 
# 备注：
    注意所有的缩进都是两个空格，要注意':'后面都有一个空格
　　

# salt-api 配置文件详解
port : 必须填写，salt-api启动的端口
host ：默认启动于0.0.0.0，可以不填写
debug : 默认为False，True开启后，会输出debug日志
log_access_file : HTTP访问日志的路径，在2016.11.0版本添加的
log_error_file : HTTP错误日志路径，在2016.11.0版本添加的
ssl_crt : SSL证书的绝对路径
ssl_key: SSK证书的私钥绝对路径
ssl_chain : 在使用PyOpenSSL时可选参数，将证书出递给' Context.load_verify_locations '
disable_ssl : 禁用SSL标识。认证证书将会被送进clear
webhook_disable_auth : False
webhook_url : /hook
thread_pool : 100
socket_queue_size : 30
expire_responses : True
max_request_body_size : 1048576
collect_stats : False
stats_disable_auth : False
更多详细参数请见：https://github.com/saltstack/salt/blob/develop/salt/netapi/rest_cherrypy/app.py
# 启动
systemctl start salt-master
systemctl start salt-minion
systemctl start salt-api

```
## salt api 使用说明
```
# salt-api 使用
# 登陆认证获取token
[root@aliyuntest ~]#  curl -sSk https://localhost:8000/login -H 'Accept: application/x-yaml' -d username=saltapi -d password=saltapi -d eauth=pam
return:
- eauth: pam
  expire: 1551279874.050427
  perms:
  - .*
  - '@wheel'
  - '@runner'
  start: 1551236674.0504262
  token: 0f6552dc94eec5e2ce671a75526f380e01cb2e0d
  user: saltapi

# 使用获取的token进行命令操作
[root@aliyuntest ~]# curl -sSk https://localhost:8000 -H 'Accept: application/x-yaml' -H 'X-Auth-Token: 0f6552dc94eec5e2ce671a75526f380e01cb2e0d' -d client=local  -d tgt='*' -d fun=test.ping
return:
- 10.81.160.163: true
 
参数解释：
client ： 模块，python处理salt-api的主要模块，‘client interfaces <netapi-clients>’
    local : 使用‘LocalClient <salt.client.LocalClient>’ 发送命令给受控主机，等价于saltstack命令行中的'salt'命令
    local_async : 和local不同之处在于，这个模块是用于异步操作的，即在master端执行命令后返回的是一个jobid，任务放在后台运行，通过产看jobid的结果来获取命令的执行结果。
    runner : 使用'RunnerClient<salt.runner.RunnerClient>' 调用salt-master上的runner模块，等价于saltstack命令行中的'salt-run'命令
    runner_async : 异步执行runner模块
    wheel : 使用'WheelClient<salt.wheel.WheelClient>', 调用salt-master上的wheel模块，wheel模块没有在命令行端等价的模块，但它通常管理主机资源，比如文件状态，pillar文件，salt配置文件，以及关键模块<salt.wheel.key>功能类似于命令行中的salt-key。
    wheel_async : 异步执行wheel模块
    备注：一般情况下local模块，需要tgt和arg(数组)，kwarg(字典)，因为这些值将被发送到minions并用于执行所请求的函数。而runner和wheel都是直接应用于master，不需要这些参数。
tgt : minions
fun : 函数
arg : 参数
expr_form : tgt的匹配规则
    'glob' - Bash glob completion - Default
    'pcre' - Perl style regular expression
    'list' - Python list of hosts
    'grain' - Match based on a grain comparison
    'grain_pcre' - Grain comparison with a regex
    'pillar' - Pillar data comparison
    'nodegroup' - Match on nodegroup
    'range' - Use a Range server for matching
    'compound' - Pass a compound match string
```

## salt api的实例
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# author : wangyongcun

import urllib,json
import urllib.request
import urllib.parse
import ssl
from SOPS import settings
ssl._create_default_https_context = ssl._create_unverified_context


class SaltAPI(object):
    __token_id = ''

    def __init__(self):
        self.__url = settings.SALT_API['url']
        self.__user = settings.SALT_API['user']
        self.__password = settings.SALT_API['password']

    def token_id(self):
        """
            用户登陆和获取token
        :return:
        """
        params = {'eauth': 'pam', 'username': self.__user, 'password': self.__password}
        encode = urllib.parse.urlencode(params)
        obj = urllib.parse.unquote(encode).encode('utf-8')
        content = self.postRequest(obj, prefix='/login')
        try:
            self.__token_id = content['return'][0]['token']
        except KeyError:
            raise KeyError

    def postRequest(self,obj,prefix='/'):
        url = self.__url + prefix
        headers = {'X-Auth-Token': self.__token_id}
        req = urllib.request.Request(url, obj, headers)
        opener = urllib.request.urlopen(req)
        content = json.loads(opener.read())
        return content

    def list_all_key(self):
        """
            获取包括认证、未认证salt主机
        """

        params = {'client': 'wheel', 'fun': 'key.list_all'}
        obj = urllib.parse.urlencode(params).encode('utf-8')
        self.token_id()
        content = self.postRequest(obj)
        minions = content['return'][0]['data']['return']['minions']
        minions_pre = content['return'][0]['data']['return']['minions_pre']
        return minions, minions_pre

    def delete_key(self, node_name):
        '''
            拒绝salt主机
        '''

        params = {'client': 'wheel', 'fun': 'key.delete', 'match': node_name}
        obj = urllib.parse.urlencode(params).encode('utf-8')
        self.token_id()
        content = self.postRequest(obj)
        ret = content['return'][0]['data']['success']
        return ret

    def accept_key(self,node_name):
        '''
            接受salt主机
        '''

        params = {'client': 'wheel', 'fun': 'key.accept', 'match': node_name}
        obj = urllib.parse.urlencode(params).encode('utf-8')
        self.token_id()
        content = self.postRequest(obj)
        ret = content['return'][0]['data']['success']
        return ret

    def salt_get_jid_ret(self,jid):
        """
            通过jid获取执行结果
        :param jid: jobid
        :return: 结果
        """
        params = {'client':'runner', 'fun':'jobs.lookup_jid', 'jid': jid}
        obj = urllib.parse.urlencode(params).encode('utf-8')
        self.token_id()
        content = self.postRequest(obj)
        ret = content['return'][0]
        return ret

    def salt_running_jobs(self):
        """
            获取运行中的任务
        :return: 任务结果
        """
        params = {'client':'runner', 'fun': 'jobs.active'}
        obj = urllib.parse.urlencode(params).encode('utf-8')
        self.token_id()
        content = self.postRequest(obj)
        ret = content['return'][0]
        return ret

    def remote_noarg_execution_sigle(self, tgt, fun):
        """
            单台minin执行命令没有参数
        :param tgt: 目标主机
        :param fun:  执行模块
        :return: 执行结果
        """
        params = {'client': 'local', 'tgt': tgt, 'fun': fun}
        obj = urllib.parse.urlencode(params).encode('utf-8')
        self.token_id()
        content = self.postRequest(obj)
        # print(content)
        # {'return': [{'salt-master': True}]}
        ret = content['return'][0]
        return ret

    def remote_execution_single(self, tgt, fun, arg):
        """
            单台minion远程执行，有参数
        :param tgt: minion
        :param fun: 模块
        :param arg: 参数
        :return: 执行结果
        """
        params = {'client': 'local', 'tgt': tgt, 'fun': fun, 'arg': arg}
        obj = urllib.parse.urlencode(params).encode('utf-8')
        self.token_id()
        content = self.postRequest(obj)
        # print(content)
        # {'return': [{'salt-master': 'root'}]}
        ret = content['return']
        return ret

    def remote_async_execution_module(self, tgt, fun, arg):
        """
            远程异步执行模块，有参数
        :param tgt: minion list
        :param fun: 模块
        :param arg: 参数
        :return: jobid
        """
        params = {'client': 'local_async', 'tgt': tgt, 'fun': fun, 'arg': arg, 'expr_form': 'list'}
        obj = urllib.parse.urlencode(params).encode('utf-8')
        self.token_id()
        content = self.postRequest(obj)
        # print(content)
        # {'return': [{'jid': '20180131173846594347', 'minions': ['salt-master', 'salt-minion']}]}
        jid = content['return'][0]['jid']
        return jid

    def remote_execution_module(self, tgt, fun, arg):
        """
            远程执行模块，有参数
        :param tgt: minion list
        :param fun: 模块
        :param arg: 参数
        :return: dict, {'minion1': 'ret', 'minion2': 'ret'}
        """
        params = {'client': 'local', 'tgt': tgt, 'fun': fun, 'arg': arg, 'expr_form': 'list'}
        obj = urllib.parse.urlencode(params).encode('utf-8')
        self.token_id()
        content = self.postRequest(obj)
        # print(content)
        # {'return': [{'salt-master': 'root', 'salt-minion': 'root'}]}
        ret = content['return'][0]
        return ret

    def salt_state(self, tgt, arg, expr_form):
        '''
        sls文件
        '''
        params = {'client': 'local', 'tgt': tgt, 'fun': 'state.sls', 'arg': arg, 'expr_form': expr_form}
        obj = urllib.parse.urlencode(params).encode('utf-8')
        self.token_id()
        content = self.postRequest(obj)
        ret = content['return'][0]
        return ret

    def salt_alive(self, tgt):
        '''
        salt主机存活检测
        '''
        params = {'client': 'local', 'tgt': tgt, 'fun': 'test.ping'}
        obj = urllib.parse.urlencode(params).encode('utf-8')
        self.token_id()
        content = self.postRequest(obj)
        ret = content['return'][0]
        return ret


if __name__ == '__main__':
        salt = SaltAPI()
        # minions, minions_pre = salt.list_all_key()
        # 说明如果'expr_form': 'list',表示minion是以主机列表形式执行时，需要把list拼接成字符串，如下所示
        minions = ['salt-master', 'salt-minion']
        hosts = map(str, minions)
        hosts = ",".join(hosts)
        ret = salt.remote_noarg_execution_sigle('salt-master', 'test.ping')
        print(ret)
        # print(type(ret))

Python3 版本salt-api class
```
上面的版本基本功能实现，但是未实现运行多参数命令的问题，原因未找到，如果有读者发现了，可以告诉我，感谢~！提供一个基于requests的版本，实现了多参数的执行。
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# author : wangyongcun

import requests
import copy

SALT_API = {
    "url": "https://192.168.11.12:8000",
    "user": "saltapi",
    "password": "password",
}


class SaltApi(object):

    def __init__(self):
        self.__user = SALT_API["user"]
        self.__passwd = SALT_API["password"]
        self.url = SALT_API["url"]
        self.headers = {
            'Content-Type': 'application/json',
            'Accept': 'application/json'
        }
        self.__base_data = dict(
            username=self.__user,
            password=self.__passwd,
            eauth='pam'
        )
        self.__token = self.get_token()

    def get_token(self):
        """  login salt-api and get token_id """
        params = copy.deepcopy(self.__base_data)
        requests.packages.urllib3.disable_warnings()  # close ssl warning, py3 really can do it!
        ret = requests.post(url=self.url + '/login', verify=False, headers=self.headers, json=params)
        ret_json = ret.json()
        token = ret_json["return"][0]["token"]
        return token

    def __post(self, **kwargs):
        """  custom post interface, headers contains X-Auth-Token """
        headers_token = {'X-Auth-Token': self.__token}
        headers_token.update(self.headers)
        requests.packages.urllib3.disable_warnings()
        ret = requests.post(url=self.url, verify=False, headers=headers_token, **kwargs)
        ret_code, ret_data = ret.status_code, ret.json()
        return (ret_code, ret_data)

    def list_all_keys(self):
        """  show all keys, minions have been certified, minion_pre not certification """
        params = {'client': 'wheel', 'fun': 'key.list_all'}
        r = self.__post(json=params)
        minions = r[1]['return'][0]['data']['return']['minions']
        minions_pre = r[1]['return'][0]['data']['return']['minions_pre']
        return minions, minions_pre

    def delete_key(self, tgt):
        """ delete a key """
        params = {'client': 'wheel', 'fun': 'key.delete', 'match': tgt}
        r = self.__post(json=params)
        return r[1]['return'][0]['data']['success']

    def accept_key(self, tgt):
        """  accept a key """
        params = {'client': 'wheel', 'fun': 'key.accept', 'match': tgt}
        r = self.__post(json=params)
        return r[1]['return'][0]['data']['success']

    def lookup_jid_ret(self, jid):
        """  depend on jobid to find result """
        params = {'client': 'runner', 'fun': 'jobs.lookup_jid', 'jid': jid}
        r = self.__post(json=params)
        return r[1]['return'][0]

    def salt_running_jobs(self):
        """ show all running jobs """
        params = {'client': 'runner', 'fun': 'jobs.active'}
        r = self.__post(json=params)
        return r[1]['return'][0]

    def run(self, params):
        """ remote common interface, you need custom data dict
            for example:
                params = {
                    'client': 'local',
                    'fun': 'grains.item',
                    'tgt': '*',
                    'arg': ('os', 'id', 'host' ),
                    'kwargs': {},
                    'expr_form': 'glob',
                    'timeout': 60
                }
         """
        r = self.__post(json=params)
        return r[1]['return'][0]

    def remote_execution(self, tgt, fun, arg, ex='glob'):
        """ remote execution, command will wait result
            arg must be a tuple, eg: arg = (a, b)
            expr_form : tgt m

        """
        params = {'client': 'local', 'tgt': tgt, 'fun': fun, 'arg': arg, 'expr_form': ex}
        r = self.__post(json=params)
        return r[1]['return'][0]

    def async_remote_execution(self, tgt, fun, arg, ex='glob'):
        """ async remote exection, it will return a jobid
            tgt model is list, but not python list, just like 'node1, node2, node3' as a string.
         """
        params = {'client': 'local_async', 'tgt': tgt, 'fun': fun, 'arg': arg, 'expr_form': ex}
        r = self.__post(json=params)
        return r[1]['return'][0]['jid']

    def salt_state(self, tgt, arg, ex='list'):
        """  salt state.sls """
        params = {'client': 'local', 'tgt': tgt, 'fun': 'state.sls', 'arg': arg, 'expr_form': ex}
        r = self.__post(json=params)
        return r[1]['return'][0]

    def salt_alive(self, tgt, ex='glob'):
        """ salt test.ping """
        params = {'client': 'local', 'tgt': tgt, 'fun': 'test.ping', 'expr_form': ex}
        r = self.__post(json=params)
        return r[1]['return'][0]


if __name__ == '__main__':
    data = {
        'client': 'local',
        'fun': 'grains.item',
        'tgt': '*',
        'arg': ('os', 'id', 'host' ),
        'kwargs': {},
        'expr_form': 'glob',
        'timeout': 60
    }
    obj = SaltApi()
    # ret = obj.list_all_keys()
    # ret = obj.accept_key('windows-test')
    # ret = obj.delete_key('windows-test')
    # ret = obj.lookup_jid_ret('20180612111505161780')
    # ret = obj.salt_running_jobs()
    # ret = obj.remote_execution('*', 'grains.item', ('os', 'id'))
    # ret = obj.async_remote_execution('*', 'grains.item', ('os', 'id'))
    # ret = obj.salt_alive('*', 'glob')
    ret = obj.run(data)
    print(ret)

Python 自定义salt-api 类（requests版本）
```
