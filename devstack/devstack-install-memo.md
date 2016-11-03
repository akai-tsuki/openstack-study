# 参考ページ

Devstack入門
https://github.com/rafiror/openstack/wiki/Devstack%E5%85%A5%E9%96%80

DevStack-memo
https://github.com/ytooyama/devstack-memo

devstackを利用したOpenStackの構築 - CentOS7版
http://tzpst.hatenablog.com/entry/2015/05/31/225003


# 用意したマシン

- Virtualbox上に仮想マシンを作成
- CPU4コア
- メモリ：4096
- CentOS7をインストール

# 設定、事前準備

１）selinuxはdisabled

```
[root@devstack ~]# cat /etc/sysconfig/selinux
SELINUX=disabled
SELINUXTYPE=targeted
```

２）ネットワーク設定

```
[root@devstack ~]# cat /etc/sysconfig/network-scripts/ifcfg-enp0s3
TYPE=Ethernet
BOOTPROTO=none
DEFROUTE=yes
IPADDR=192.168.0.100
GATEWAY=192.168.0.1
PREFIX=24
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=enp0s3
UUID=aa9f7829-ad3c-4a92-a790-5f66cf370111
DEVICE=enp0s3
ONBOOT=yes
DNS1=8.8.8.8
```

```
[root@devstack ~]# cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
TYPE=Ethernet
BOOTPROTO=none
IPADDR=192.168.10.10
PREFIX=24
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=enp0s8
UUID=66f0e170-e885-4716-aee3-ac6d1b18e654
DEVICE=enp0s8
ONBOOT=yes
[root@devstack ~]#
```

３）hostname
```
[root@devstack ~]# hostnamectl set-hostname <名前>
```


４）gitをインストール

```
yum install git
```

※最初、DNS設定してなかったら以下のエラーが出た。  
（上記のネットワーク設定ではDNSを設定済み）

```
[root@devstack ~]# yum install git
Loaded plugins: fastestmirror
Could not retrieve mirrorlist http://mirrorlist.centos.org/?release=7&arch=x86_64&repo=os&infra=stock error was
14: curl#6 - "Could not resolve host: mirrorlist.centos.org; Unknown error"
```

```
[root@devstack ~]# git --version
git version 1.8.3.1
```

５）NetworkManagerはoff

```
# systemctl stop NetworkManager
# systemctl disable NetworkManager
```


６）stackユーザを追加

```
# adduser stack
# passwd stack
New password: ********
Retype new password: ********
```

７）sudoを設定

```
[root@devstack ~]# visudo

stack   ALL=(ALL)       NOPASSWD: ALL
を追加
```

８）epelをインストール

```
[root@devstack ~]# yum install epel-release
```

９）/etc/hosts編集(stackユーザで実施したのでsudo)

```
$ sudo vi /etc/hosts
$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.0.100 devstack
192.168.10.10 devstack
$
```


# devstackインストール

## devstack取得(stackユーザ？)

```
$ git clone -b stable/mitaka https://git.openstack.org/openstack-dev/devstack
$ ls -l
total 4
drwxrwxr-x 16 stack stack 4096 Oct 24 03:55 devstack
$ cd devstack/
$ git branch --all
* stable/mitaka
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/stable/kilo
  remotes/origin/stable/liberty
  remotes/origin/stable/mitaka
  remotes/origin/stable/newton
$ cd devstack/
$ ls
clean.sh      exercises    functions-common  lib              pkg           setup.py  tox.ini
data          exercise.sh  FUTURE.rst        LICENSE          README.md     stackrc   unstack.sh
doc           extras.d     gate              MAINTAINERS.rst  run_tests.sh  stack.sh
driver_certs  files        HACKING.rst       Makefile         samples       tests
exerciserc    functions    inc               openrc           setup.cfg     tools
$
```

## local.confを設置

設定ファイルは、local.confを参照。

## stack.shを実行

以下のエラーがでた。

```
2016-10-29 17:00:01.739 | Requirement already satisfied (use --upgrade to upgrade): pytz===2015.7 in /usr/lib/python2.7/site-packages (from -c /opt/stack/requirements/upper-constraints.txt (line 314))
2016-10-29 17:00:01.768 | Collecting rcssmin===1.0.6 (from -c /opt/stack/requirements/upper-constraints.txt (line 318))
2016-10-29 17:00:01.863 |   Using cached rcssmin-1.0.6.tar.gz
2016-10-29 17:00:02.457 |     Complete output from command python setup.py egg_info:
2016-10-29 17:00:02.457 |     running egg_info
2016-10-29 17:00:02.457 |     creating pip-egg-info/rcssmin.egg-info
2016-10-29 17:00:02.457 |     writing pip-egg-info/rcssmin.egg-info/PKG-INFO
2016-10-29 17:00:02.457 |     writing top-level names to pip-egg-info/rcssmin.egg-info/top_level.txt
2016-10-29 17:00:02.457 |     writing dependency_links to pip-egg-info/rcssmin.egg-info/dependency_links.txt
2016-10-29 17:00:02.457 |     writing manifest file 'pip-egg-info/rcssmin.egg-info/SOURCES.txt'
2016-10-29 17:00:02.457 |     warning: manifest_maker: standard file '-c' not found
2016-10-29 17:00:02.457 |
2016-10-29 17:00:02.457 |     Traceback (most recent call last):
2016-10-29 17:00:02.457 |       File "<string>", line 1, in <module>
2016-10-29 17:00:02.457 |       File "/tmp/pip-build-xH46KU/rcssmin/setup.py", line 42, in <module>
2016-10-29 17:00:02.457 |         setup()
2016-10-29 17:00:02.457 |       File "/tmp/pip-build-xH46KU/rcssmin/setup.py", line 33, in setup
2016-10-29 17:00:02.457 |         return run(script_args=args, ext=ext, manifest_only=_manifest)
2016-10-29 17:00:02.457 |       File "_setup/py2/setup.py", line 421, in run
2016-10-29 17:00:02.457 |         return _core.setup(**kwargs)
2016-10-29 17:00:02.457 |       File "/usr/lib64/python2.7/distutils/core.py", line 152, in setup
2016-10-29 17:00:02.457 |         dist.run_commands()
2016-10-29 17:00:02.457 |       File "/usr/lib64/python2.7/distutils/dist.py", line 953, in run_commands
2016-10-29 17:00:02.457 |         self.run_command(cmd)
2016-10-29 17:00:02.457 |       File "/usr/lib64/python2.7/distutils/dist.py", line 972, in run_command
2016-10-29 17:00:02.457 |         cmd_obj.run()
2016-10-29 17:00:02.457 |       File "/usr/lib/python2.7/site-packages/setuptools/command/egg_info.py", line 279, in run
2016-10-29 17:00:02.457 |         self.find_sources()
2016-10-29 17:00:02.457 |       File "/usr/lib/python2.7/site-packages/setuptools/command/egg_info.py", line 306, in find_sources
2016-10-29 17:00:02.457 |         mm.run()
2016-10-29 17:00:02.457 |       File "/usr/lib/python2.7/site-packages/setuptools/command/egg_info.py", line 533, in run
2016-10-29 17:00:02.457 |         self.add_defaults()
2016-10-29 17:00:02.457 |       File "/usr/lib/python2.7/site-packages/setuptools/command/egg_info.py", line 562, in add_defaults
2016-10-29 17:00:02.457 |         sdist.add_defaults(self)
2016-10-29 17:00:02.457 |       File "/usr/lib/python2.7/site-packages/setuptools/command/py36compat.py", line 35, in add_defaults
2016-10-29 17:00:02.457 |         self._add_defaults_data_files()
2016-10-29 17:00:02.457 |       File "/usr/lib/python2.7/site-packages/setuptools/command/py36compat.py", line 111, in _add_defaults_data_files
2016-10-29 17:00:02.457 |         dirname, filenames = item
2016-10-29 17:00:02.457 |     TypeError: 'Documentation' object is not iterable
2016-10-29 17:00:02.457 |
2016-10-29 17:00:02.457 |     ----------------------------------------
2016-10-29 17:00:02.608 | Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-xH46KU/rcssmin/
2016-10-29 17:00:02.658 | +inc/python:pip_install:1                  exit_trap
2016-10-29 17:00:02.667 | +./stack.sh:exit_trap:474                  local r=1
2016-10-29 17:00:02.670 | ++./stack.sh:exit_trap:475                  jobs -p
2016-10-29 17:00:02.674 | +./stack.sh:exit_trap:475                  jobs=
2016-10-29 17:00:02.679 | +./stack.sh:exit_trap:478                  [[ -n '' ]]
2016-10-29 17:00:02.683 | +./stack.sh:exit_trap:484                  kill_spinner
2016-10-29 17:00:02.687 | +./stack.sh:kill_spinner:370               '[' '!' -z '' ']'
2016-10-29 17:00:02.691 | +./stack.sh:exit_trap:486                  [[ 1 -ne 0 ]]
2016-10-29 17:00:02.695 | +./stack.sh:exit_trap:487                  echo 'Error on exit'
2016-10-29 17:00:02.695 | Error on exit
2016-10-29 17:00:02.699 | +./stack.sh:exit_trap:488                  generate-subunit 1477760151 251 fail
2016-10-29 17:00:02.881 | +./stack.sh:exit_trap:489                  [[ -z /opt/stack/logs ]]
2016-10-29 17:00:02.885 | +./stack.sh:exit_trap:492                  /home/stack/devstack/tools/worlddump.py -d /opt/stack/logs
2016-10-29 17:00:03.060 | +./stack.sh:exit_trap:498                  exit 1
```

## rcssminを手動でインストールしてみた

こちらのやり方はNG？
```
[stack@devstack rcssmin-1.0.6]$ pip install rcssmin
Collecting rcssmin
  Downloading rcssmin-1.0.6.tar.gz (582kB)
    100% |????????????????????????????????| 583kB 1.9MB/s
    Complete output from command python setup.py egg_info:
    running egg_info
    creating pip-egg-info/rcssmin.egg-info
    writing pip-egg-info/rcssmin.egg-info/PKG-INFO
    writing top-level names to pip-egg-info/rcssmin.egg-info/top_level.txt
    writing dependency_links to pip-egg-info/rcssmin.egg-info/dependency_links.txt
    writing manifest file 'pip-egg-info/rcssmin.egg-info/SOURCES.txt'
    warning: manifest_maker: standard file '-c' not found

    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-build-pw05wU/rcssmin/setup.py", line 42, in <module>
        setup()
      File "/tmp/pip-build-pw05wU/rcssmin/setup.py", line 33, in setup
        return run(script_args=args, ext=ext, manifest_only=_manifest)
      File "_setup/py2/setup.py", line 421, in run
        return _core.setup(**kwargs)
      File "/usr/lib64/python2.7/distutils/core.py", line 152, in setup
        dist.run_commands()
      File "/usr/lib64/python2.7/distutils/dist.py", line 953, in run_commands
        self.run_command(cmd)
      File "/usr/lib64/python2.7/distutils/dist.py", line 972, in run_command
        cmd_obj.run()
      File "/usr/lib/python2.7/site-packages/setuptools/command/egg_info.py", line 279, in run
        self.find_sources()
      File "/usr/lib/python2.7/site-packages/setuptools/command/egg_info.py", line 306, in find_sources
        mm.run()
      File "/usr/lib/python2.7/site-packages/setuptools/command/egg_info.py", line 533, in run
        self.add_defaults()
      File "/usr/lib/python2.7/site-packages/setuptools/command/egg_info.py", line 562, in add_defaults
        sdist.add_defaults(self)
      File "/usr/lib/python2.7/site-packages/setuptools/command/py36compat.py", line 35, in add_defaults
        self._add_defaults_data_files()
      File "/usr/lib/python2.7/site-packages/setuptools/command/py36compat.py", line 111, in _add_defaults_data_files
        dirname, filenames = item
    TypeError: 'Documentation' object is not iterable

    ----------------------------------------
Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-pw05wU/rcssmin/
[stack@devstack rcssmin-1.0.6]$
```


こっちの方法だとOK？
```
# wget http://storage.perlig.de/rcssmin/rcssmin-1.0.6.tar.gz
# tar xvfz rcssmin-1.0.6.tar.gz
# cd rcssmin-1.0.6
# python setup.py install
running install
running build
running build_py
running build_ext
running install_lib
copying build/lib.linux-x86_64-2.7/rcssmin.py -> /usr/lib64/python2.7/site-packages
copying build/lib.linux-x86_64-2.7/_rcssmin.so -> /usr/lib64/python2.7/site-packages
byte-compiling /usr/lib64/python2.7/site-packages/rcssmin.py to rcssmin.pyc
running install_data
creating /usr/share/doc/rcssmin
copying README.rst -> /usr/share/doc/rcssmin
copying docs/CHANGES -> /usr/share/doc/rcssmin
copying docs/BENCHMARKS -> /usr/share/doc/rcssmin
creating /usr/share/doc/rcssmin/apidoc
copying docs/apidoc/api-objects.txt -> /usr/share/doc/rcssmin/apidoc
copying docs/apidoc/crarr.png -> /usr/share/doc/rcssmin/apidoc
copying docs/apidoc/epydoc.css -> /usr/share/doc/rcssmin/apidoc
copying docs/apidoc/epydoc.js -> /usr/share/doc/rcssmin/apidoc
copying docs/apidoc/help.html -> /usr/share/doc/rcssmin/apidoc
copying docs/apidoc/identifier-index.html -> /usr/share/doc/rcssmin/apidoc
copying docs/apidoc/index.html -> /usr/share/doc/rcssmin/apidoc
copying docs/apidoc/module-tree.html -> /usr/share/doc/rcssmin/apidoc
copying docs/apidoc/rcssmin-module.html -> /usr/share/doc/rcssmin/apidoc
copying docs/apidoc/rcssmin-pysrc.html -> /usr/share/doc/rcssmin/apidoc
copying docs/apidoc/redirect.html -> /usr/share/doc/rcssmin/apidoc
running install_egg_info
Writing /usr/lib64/python2.7/site-packages/rcssmin-1.0.6-py2.7.egg-info
#
```

## 次に進んだけど、別のエラーが出た

```
2016-10-29 17:28:18.816 | Collecting rjsmin===1.0.12 (from -c /opt/stack/requirements/upper-constraints.txt (line 330))
2016-10-29 17:28:18.958 |   Downloading rjsmin-1.0.12.tar.gz (446kB)
2016-10-29 17:28:19.883 |     Complete output from command python setup.py egg_info:
2016-10-29 17:28:19.883 |     running egg_info
2016-10-29 17:28:19.883 |     creating pip-egg-info/rjsmin.egg-info
2016-10-29 17:28:19.883 |     writing pip-egg-info/rjsmin.egg-info/PKG-INFO
2016-10-29 17:28:19.883 |     writing top-level names to pip-egg-info/rjsmin.egg-info/top_level.txt
2016-10-29 17:28:19.883 |     writing dependency_links to pip-egg-info/rjsmin.egg-info/dependency_links.txt
2016-10-29 17:28:19.883 |     writing manifest file 'pip-egg-info/rjsmin.egg-info/SOURCES.txt'
2016-10-29 17:28:19.883 |     warning: manifest_maker: standard file '-c' not found
2016-10-29 17:28:19.883 |
2016-10-29 17:28:19.883 |     Traceback (most recent call last):
2016-10-29 17:28:19.883 |       File "<string>", line 1, in <module>
2016-10-29 17:28:19.883 |       File "/tmp/pip-build-eNBTYk/rjsmin/setup.py", line 42, in <module>
2016-10-29 17:28:19.883 |         setup()
2016-10-29 17:28:19.883 |       File "/tmp/pip-build-eNBTYk/rjsmin/setup.py", line 33, in setup
2016-10-29 17:28:19.883 |         return run(script_args=args, ext=ext, manifest_only=_manifest)
2016-10-29 17:28:19.883 |       File "_setup/py2/setup.py", line 421, in run
2016-10-29 17:28:19.883 |         return _core.setup(**kwargs)
2016-10-29 17:28:19.883 |       File "/usr/lib64/python2.7/distutils/core.py", line 152, in setup
2016-10-29 17:28:19.883 |         dist.run_commands()
2016-10-29 17:28:19.883 |       File "/usr/lib64/python2.7/distutils/dist.py", line 953, in run_commands
2016-10-29 17:28:19.883 |         self.run_command(cmd)
2016-10-29 17:28:19.883 |       File "/usr/lib64/python2.7/distutils/dist.py", line 972, in run_command
2016-10-29 17:28:19.883 |         cmd_obj.run()
2016-10-29 17:28:19.883 |       File "/usr/lib/python2.7/site-packages/setuptools/command/egg_info.py", line 279, in run
2016-10-29 17:28:19.883 |         self.find_sources()
2016-10-29 17:28:19.883 |       File "/usr/lib/python2.7/site-packages/setuptools/command/egg_info.py", line 306, in find_sources
2016-10-29 17:28:19.883 |         mm.run()
2016-10-29 17:28:19.883 |       File "/usr/lib/python2.7/site-packages/setuptools/command/egg_info.py", line 533, in run
2016-10-29 17:28:19.883 |         self.add_defaults()
2016-10-29 17:28:19.883 |       File "/usr/lib/python2.7/site-packages/setuptools/command/egg_info.py", line 562, in add_defaults
2016-10-29 17:28:19.883 |         sdist.add_defaults(self)
2016-10-29 17:28:19.883 |       File "/usr/lib/python2.7/site-packages/setuptools/command/py36compat.py", line 35, in add_defaults
2016-10-29 17:28:19.883 |         self._add_defaults_data_files()
2016-10-29 17:28:19.883 |       File "/usr/lib/python2.7/site-packages/setuptools/command/py36compat.py", line 111, in _add_defaults_data_files
2016-10-29 17:28:19.883 |         dirname, filenames = item
2016-10-29 17:28:19.883 |     TypeError: 'Documentation' object is not iterable
2016-10-29 17:28:19.883 |
2016-10-29 17:28:19.883 |     ----------------------------------------
2016-10-29 17:28:20.417 | Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-eNBTYk/rjsmin/
2016-10-29 17:28:20.489 | +inc/python:pip_install:1                  exit_trap
2016-10-29 17:28:20.493 | +./stack.sh:exit_trap:474                  local r=1
2016-10-29 17:28:20.497 | ++./stack.sh:exit_trap:475                  jobs -p
2016-10-29 17:28:20.505 | +./stack.sh:exit_trap:475                  jobs=
2016-10-29 17:28:20.510 | +./stack.sh:exit_trap:478                  [[ -n '' ]]
2016-10-29 17:28:20.515 | +./stack.sh:exit_trap:484                  kill_spinner
2016-10-29 17:28:20.520 | +./stack.sh:kill_spinner:370               '[' '!' -z '' ']'
2016-10-29 17:28:20.524 | +./stack.sh:exit_trap:486                  [[ 1 -ne 0 ]]
2016-10-29 17:28:20.536 | +./stack.sh:exit_trap:487                  echo 'Error on exit'
2016-10-29 17:28:20.536 | Error on exit
2016-10-29 17:28:20.541 | +./stack.sh:exit_trap:488                  generate-subunit 1477761847 253 fail
2016-10-29 17:28:20.746 | +./stack.sh:exit_trap:489                  [[ -z /opt/stack/logs ]]
2016-10-29 17:28:20.750 | +./stack.sh:exit_trap:492                  /home/stack/devstack/tools/worlddump.py -d /opt/stack/logs
2016-10-29 17:28:20.964 | +./stack.sh:exit_trap:498                  exit 1
```

## rjsminをインストールする

pip installはやっぱりだめ？
```
[root@devstack ~]# pip install rjsmin
Collecting rjsmin
  Using cached rjsmin-1.0.12.tar.gz
    Complete output from command python setup.py egg_info:
    running egg_info
    creating pip-egg-info/rjsmin.egg-info
    writing pip-egg-info/rjsmin.egg-info/PKG-INFO
    writing top-level names to pip-egg-info/rjsmin.egg-info/top_level.txt
    writing dependency_links to pip-egg-info/rjsmin.egg-info/dependency_links.txt
    writing manifest file 'pip-egg-info/rjsmin.egg-info/SOURCES.txt'
    warning: manifest_maker: standard file '-c' not found

    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-build-ozsTzu/rjsmin/setup.py", line 42, in <module>
        setup()
      File "/tmp/pip-build-ozsTzu/rjsmin/setup.py", line 33, in setup
        return run(script_args=args, ext=ext, manifest_only=_manifest)
      File "_setup/py2/setup.py", line 421, in run
        return _core.setup(**kwargs)
      File "/usr/lib64/python2.7/distutils/core.py", line 152, in setup
        dist.run_commands()
      File "/usr/lib64/python2.7/distutils/dist.py", line 953, in run_commands
        self.run_command(cmd)
      File "/usr/lib64/python2.7/distutils/dist.py", line 972, in run_command
        cmd_obj.run()
      File "/usr/lib/python2.7/site-packages/setuptools/command/egg_info.py", line 279, in run
        self.find_sources()
      File "/usr/lib/python2.7/site-packages/setuptools/command/egg_info.py", line 306, in find_sources
        mm.run()
      File "/usr/lib/python2.7/site-packages/setuptools/command/egg_info.py", line 533, in run
        self.add_defaults()
      File "/usr/lib/python2.7/site-packages/setuptools/command/egg_info.py", line 562, in add_defaults
        sdist.add_defaults(self)
      File "/usr/lib/python2.7/site-packages/setuptools/command/py36compat.py", line 35, in add_defaults
        self._add_defaults_data_files()
      File "/usr/lib/python2.7/site-packages/setuptools/command/py36compat.py", line 111, in _add_defaults_data_files
        dirname, filenames = item
    TypeError: 'Documentation' object is not iterable

    ----------------------------------------
Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-ozsTzu/rjsmin/
[root@devstack ~]#
```

手動インストールしてみた。
```
# wget <URL?>
# tar xvfz rjsmin-1.0.12.tar.gz
# cd rjsmin-1.0.12
# python setup.py install
```

# 完了

以下のようなメッセージが出て完了した。

```
$ ./stack.sh
（ログが出続ける）

========================
DevStack Components Timed
========================

run_process - 43 secs
pip_install - 98 secs
restart_apache_server - 8 secs
wait_for_service - 9 secs
yum_install - 22 secs



This is your host IP address: 192.168.0.100
This is your host IPv6 address: ::1
Horizon is now available at http://192.168.0.100/dashboard
Keystone is serving at http://192.168.0.100:5000/
The default users are: admin and demo
The password: nomoresecret
[stack@devstack devstack]$
```

# keystonerc_adminを用意

以下のような感じで準備した。

```
$ cat keystonerc_admin
export OS_USERNAME=admin
export OS_TENANT_NAME=admin
export OS_PASSWORD=nomoresecret
export OS_AUTH_URL=http://192.168.0.100:5000/v2.0/
export OS_REGION_NAME=RegionOne
export PS1='[\u@\h \W(keystone_admin)]\$ '
$
```
