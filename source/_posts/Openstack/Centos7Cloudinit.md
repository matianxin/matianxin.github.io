---
title: Centos7 CloudInit 安装使用及配置说明
tags: [openstack]
categories: Openstack
description: Centos7 CloudInit 安装使用及配置说明
date: 2020/10/21 10:40:00
---

1.使用本地YUM源安装cloud-init及其依赖，cloud-init版本18.2
```python
[root@localhost ~] yum -y install cloud-init

Loaded plugins: fastestmirror
Determining fastest mirrors
Resolving Dependencies
--> Running transaction check
---> Package cloud-init.x86_64 0:18.2-1.el7.centos will be installed
--> Processing Dependency: python-six for package: cloud-init-18.2-1.el7.centos.x86_64
--> Processing Dependency: python-requests for package: cloud-init-18.2-1.el7.centos.x86_64
--> Processing Dependency: python-prettytable for package: cloud-init-18.2-1.el7.centos.x86_64
--> Processing Dependency: python-jsonpatch for package: cloud-init-18.2-1.el7.centos.x86_64
--> Processing Dependency: python-jinja2 for package: cloud-init-18.2-1.el7.centos.x86_64
--> Processing Dependency: pyserial for package: cloud-init-18.2-1.el7.centos.x86_64
--> Processing Dependency: policycoreutils-python for package: cloud-init-18.2-1.el7.centos.x86_64
--> Processing Dependency: PyYAML for package: cloud-init-18.2-1.el7.centos.x86_64
--> Running transaction check
---> Package PyYAML.x86_64 0:3.10-11.el7 will be installed
--> Processing Dependency: libyaml-0.so.2()(64bit) for package: PyYAML-3.10-11.el7.x86_64
---> Package policycoreutils-python.x86_64 0:2.5-29.el7 will be installed
--> Processing Dependency: policycoreutils = 2.5-29.el7 for package: policycoreutils-python-2.5-29.el7.x86_64
--> Processing Dependency: setools-libs >= 3.3.8-4 for package: policycoreutils-python-2.5-29.el7.x86_64
--> Processing Dependency: libsemanage-python >= 2.5-14 for package: policycoreutils-python-2.5-29.el7.x86_64
--> Processing Dependency: audit-libs-python >= 2.1.3-4 for package: policycoreutils-python-2.5-29.el7.x86_64
--> Processing Dependency: python-IPy for package: policycoreutils-python-2.5-29.el7.x86_64
--> Processing Dependency: libqpol.so.1(VERS_1.4)(64bit) for package: policycoreutils-python-2.5-29.el7.x86_64
--> Processing Dependency: libqpol.so.1(VERS_1.2)(64bit) for package: policycoreutils-python-2.5-29.el7.x86_64
--> Processing Dependency: libcgroup for package: policycoreutils-python-2.5-29.el7.x86_64
--> Processing Dependency: libapol.so.4(VERS_4.0)(64bit) for package: policycoreutils-python-2.5-29.el7.x86_64
--> Processing Dependency: checkpolicy for package: policycoreutils-python-2.5-29.el7.x86_64
--> Processing Dependency: libqpol.so.1()(64bit) for package: policycoreutils-python-2.5-29.el7.x86_64
--> Processing Dependency: libapol.so.4()(64bit) for package: policycoreutils-python-2.5-29.el7.x86_64
---> Package pyserial.noarch 0:2.6-6.el7 will be installed
---> Package python-prettytable.noarch 0:0.7.2-3.el7 will be installed
---> Package python2-jinja2.noarch 0:2.10-2.el7 will be installed
--> Processing Dependency: python2-babel >= 0.8 for package: python2-jinja2-2.10-2.el7.noarch
--> Processing Dependency: python2-markupsafe for package: python2-jinja2-2.10-2.el7.noarch
---> Package python2-jsonpatch.noarch 0:1.21-1.el7 will be installed
--> Processing Dependency: python2-jsonpointer for package: python2-jsonpatch-1.21-1.el7.noarch
---> Package python2-requests.noarch 0:2.14.2-1.el7 will be installed
--> Processing Dependency: python2-urllib3 = 1.21.1 for package: python2-requests-2.14.2-1.el7.noarch
--> Processing Dependency: python-idna for package: python2-requests-2.14.2-1.el7.noarch
--> Processing Dependency: python-chardet for package: python2-requests-2.14.2-1.el7.noarch
---> Package python2-six.noarch 0:1.10.0-9.el7 will be installed
--> Running transaction check
---> Package audit-libs-python.x86_64 0:2.8.4-4.el7 will be installed
--> Processing Dependency: audit-libs(x86-64) = 2.8.4-4.el7 for package: audit-libs-python-2.8.4-4.el7.x86_64
---> Package checkpolicy.x86_64 0:2.5-8.el7 will be installed
---> Package libcgroup.x86_64 0:0.41-20.el7 will be installed
---> Package libsemanage-python.x86_64 0:2.5-14.el7 will be installed
--> Processing Dependency: libsemanage = 2.5-14.el7 for package: libsemanage-python-2.5-14.el7.x86_64
---> Package libyaml.x86_64 0:0.1.4-11.el7_0 will be installed
---> Package policycoreutils.x86_64 0:2.5-22.el7 will be updated
---> Package policycoreutils.x86_64 0:2.5-29.el7 will be an update
--> Processing Dependency: libsepol >= 2.5-10 for package: policycoreutils-2.5-29.el7.x86_64
--> Processing Dependency: libselinux-utils >= 2.5-14 for package: policycoreutils-2.5-29.el7.x86_64
---> Package python-IPy.noarch 0:0.75-6.el7 will be installed
---> Package python-chardet.noarch 0:2.2.1-1.el7_1 will be installed
---> Package python2-babel.noarch 0:2.3.4-1.el7 will be installed
--> Processing Dependency: pytz for package: python2-babel-2.3.4-1.el7.noarch
---> Package python2-idna.noarch 0:2.5-1.el7 will be installed
---> Package python2-jsonpointer.noarch 0:1.10-4.el7 will be installed
---> Package python2-markupsafe.x86_64 0:0.23-16.el7 will be installed
---> Package python2-urllib3.noarch 0:1.21.1-1.el7 will be installed
--> Processing Dependency: python2-pyOpenSSL for package: python2-urllib3-1.21.1-1.el7.noarch
--> Processing Dependency: python-pysocks for package: python2-urllib3-1.21.1-1.el7.noarch
--> Processing Dependency: python-cryptography for package: python2-urllib3-1.21.1-1.el7.noarch
---> Package setools-libs.x86_64 0:3.3.8-4.el7 will be installed
--> Processing Dependency: libselinux >= 2.5-14.1 for package: setools-libs-3.3.8-4.el7.x86_64
--> Running transaction check
---> Package audit-libs.x86_64 0:2.8.1-3.el7 will be updated
--> Processing Dependency: audit-libs(x86-64) = 2.8.1-3.el7 for package: audit-2.8.1-3.el7.x86_64
---> Package audit-libs.x86_64 0:2.8.4-4.el7 will be an update
---> Package libselinux.x86_64 0:2.5-12.el7 will be updated
--> Processing Dependency: libselinux(x86-64) = 2.5-12.el7 for package: libselinux-python-2.5-12.el7.x86_64
---> Package libselinux.x86_64 0:2.5-14.1.el7 will be an update
---> Package libselinux-utils.x86_64 0:2.5-12.el7 will be updated
---> Package libselinux-utils.x86_64 0:2.5-14.1.el7 will be an update
---> Package libsemanage.x86_64 0:2.5-11.el7 will be updated
---> Package libsemanage.x86_64 0:2.5-14.el7 will be an update
---> Package libsepol.x86_64 0:2.5-8.1.el7 will be updated
---> Package libsepol.x86_64 0:2.5-10.el7 will be an update
---> Package python2-cryptography.x86_64 0:2.1.4-2.el7 will be installed
--> Processing Dependency: python2-cffi >= 1.7 for package: python2-cryptography-2.1.4-2.el7.x86_64
--> Processing Dependency: python2-asn1crypto >= 0.21 for package: python2-cryptography-2.1.4-2.el7.x86_64
--> Processing Dependency: python-enum34 for package: python2-cryptography-2.1.4-2.el7.x86_64
---> Package python2-pyOpenSSL.noarch 0:17.3.0-3.el7 will be installed
---> Package python2-pysocks.noarch 0:1.5.6-3.el7 will be installed
---> Package pytz.noarch 0:2016.10-2.el7 will be installed
--> Running transaction check
---> Package audit.x86_64 0:2.8.1-3.el7 will be updated
---> Package audit.x86_64 0:2.8.4-4.el7 will be an update
---> Package libselinux-python.x86_64 0:2.5-12.el7 will be updated
---> Package libselinux-python.x86_64 0:2.5-14.1.el7 will be an update
---> Package python-enum34.noarch 0:1.0.4-1.el7 will be installed
---> Package python2-asn1crypto.noarch 0:0.23.0-2.el7 will be installed
---> Package python2-cffi.x86_64 0:1.11.2-1.el7 will be installed
--> Processing Dependency: python-pycparser for package: python2-cffi-1.11.2-1.el7.x86_64
--> Running transaction check
---> Package python-pycparser.noarch 0:2.14-1.el7 will be installed
--> Processing Dependency: python-ply for package: python-pycparser-2.14-1.el7.noarch
--> Running transaction check
---> Package python-ply.noarch 0:3.4-11.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                     Arch        Version                Repository
                                                                           Size
================================================================================
Installing:
 cloud-init                  x86_64      18.2-1.el7.centos      kdpa      776 k
Installing for dependencies:
 PyYAML                      x86_64      3.10-11.el7            kdpa      153 k
 audit-libs-python           x86_64      2.8.4-4.el7            kdpa       76 k
 checkpolicy                 x86_64      2.5-8.el7              kdpa      295 k
 libcgroup                   x86_64      0.41-20.el7            kdpa       66 k
 libsemanage-python          x86_64      2.5-14.el7             kdpa      113 k
 libyaml                     x86_64      0.1.4-11.el7_0         kdpa       55 k
 policycoreutils-python      x86_64      2.5-29.el7             kdpa      456 k
 pyserial                    noarch      2.6-6.el7              kdpa      124 k
 python-IPy                  noarch      0.75-6.el7             kdpa       32 k
 python-chardet              noarch      2.2.1-1.el7_1          kdpa      227 k
 python-enum34               noarch      1.0.4-1.el7            kdpa       52 k
 python-ply                  noarch      3.4-11.el7             kdpa      123 k
 python-prettytable          noarch      0.7.2-3.el7            kdpa       37 k
 python-pycparser            noarch      2.14-1.el7             kdpa      104 k
 python2-asn1crypto          noarch      0.23.0-2.el7           kdpa      172 k
 python2-babel               noarch      2.3.4-1.el7            kdpa      4.8 M
 python2-cffi                x86_64      1.11.2-1.el7           kdpa      229 k
 python2-cryptography        x86_64      2.1.4-2.el7            kdpa      510 k
 python2-idna                noarch      2.5-1.el7              kdpa       94 k
 python2-jinja2              noarch      2.10-2.el7             kdpa      527 k
 python2-jsonpatch           noarch      1.21-1.el7             kdpa       21 k
 python2-jsonpointer         noarch      1.10-4.el7             kdpa       14 k
 python2-markupsafe          x86_64      0.23-16.el7            kdpa       32 k
 python2-pyOpenSSL           noarch      17.3.0-3.el7           kdpa       92 k
 python2-pysocks             noarch      1.5.6-3.el7            kdpa       20 k
 python2-requests            noarch      2.14.2-1.el7           kdpa      116 k
 python2-six                 noarch      1.10.0-9.el7           kdpa       31 k
 python2-urllib3             noarch      1.21.1-1.el7           kdpa      173 k
 pytz                        noarch      2016.10-2.el7          kdpa       46 k
 setools-libs                x86_64      3.3.8-4.el7            kdpa      620 k
Updating for dependencies:
 audit                       x86_64      2.8.4-4.el7            kdpa      250 k
 audit-libs                  x86_64      2.8.4-4.el7            kdpa      100 k
 libselinux                  x86_64      2.5-14.1.el7           kdpa      162 k
 libselinux-python           x86_64      2.5-14.1.el7           kdpa      235 k
 libselinux-utils            x86_64      2.5-14.1.el7           kdpa      151 k
 libsemanage                 x86_64      2.5-14.el7             kdpa      151 k
 libsepol                    x86_64      2.5-10.el7             kdpa      297 k
 policycoreutils             x86_64      2.5-29.el7             kdpa      916 k

Transaction Summary
================================================================================
Install  1 Package  (+30 Dependent packages)
Upgrade             (  8 Dependent packages)

Total download size: 12 M
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
--------------------------------------------------------------------------------
Total                                               16 MB/s |  12 MB  00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : libsepol-2.5-10.el7.x86_64                                  1/47 
  Updating   : libselinux-2.5-14.1.el7.x86_64                              2/47 
  Updating   : audit-libs-2.8.4-4.el7.x86_64                               3/47 
  Installing : python2-idna-2.5-1.el7.noarch                               4/47 
  Installing : python2-six-1.10.0-9.el7.noarch                             5/47 
  Updating   : libsemanage-2.5-14.el7.x86_64                               6/47 
  Updating   : libselinux-python-2.5-14.1.el7.x86_64                       7/47 
  Installing : libsemanage-python-2.5-14.el7.x86_64                        8/47 
  Installing : audit-libs-python-2.8.4-4.el7.x86_64                        9/47 
  Installing : setools-libs-3.3.8-4.el7.x86_64                            10/47 
  Updating   : libselinux-utils-2.5-14.1.el7.x86_64                       11/47 
  Updating   : policycoreutils-2.5-29.el7.x86_64                          12/47 
  Installing : python-chardet-2.2.1-1.el7_1.noarch                        13/47 
  Installing : python-prettytable-0.7.2-3.el7.noarch                      14/47 
  Installing : libyaml-0.1.4-11.el7_0.x86_64                              15/47 
  Installing : PyYAML-3.10-11.el7.x86_64                                  16/47 
  Installing : python-ply-3.4-11.el7.noarch                               17/47 
  Installing : python-pycparser-2.14-1.el7.noarch                         18/47 
  Installing : python2-cffi-1.11.2-1.el7.x86_64                           19/47 
  Installing : checkpolicy-2.5-8.el7.x86_64                               20/47 
  Installing : python2-asn1crypto-0.23.0-2.el7.noarch                     21/47 
  Installing : python2-markupsafe-0.23-16.el7.x86_64                      22/47 
  Installing : python2-jsonpointer-1.10-4.el7.noarch                      23/47 
  Installing : python2-jsonpatch-1.21-1.el7.noarch                        24/47 
  Installing : python-IPy-0.75-6.el7.noarch                               25/47 
  Installing : python2-pysocks-1.5.6-3.el7.noarch                         26/47 
  Installing : pytz-2016.10-2.el7.noarch                                  27/47 
  Installing : python2-babel-2.3.4-1.el7.noarch                           28/47 
  Installing : python2-jinja2-2.10-2.el7.noarch                           29/47 
  Installing : pyserial-2.6-6.el7.noarch                                  30/47 
  Installing : python-enum34-1.0.4-1.el7.noarch                           31/47 
  Installing : python2-cryptography-2.1.4-2.el7.x86_64                    32/47 
  Installing : python2-pyOpenSSL-17.3.0-3.el7.noarch                      33/47 
  Installing : python2-urllib3-1.21.1-1.el7.noarch                        34/47 
  Installing : python2-requests-2.14.2-1.el7.noarch                       35/47 
  Installing : libcgroup-0.41-20.el7.x86_64                               36/47 
  Installing : policycoreutils-python-2.5-29.el7.x86_64                   37/47 
  Installing : cloud-init-18.2-1.el7.centos.x86_64                        38/47 
  Updating   : audit-2.8.4-4.el7.x86_64                                   39/47 
  Cleanup    : policycoreutils-2.5-22.el7.x86_64                          40/47 
  Cleanup    : libsemanage-2.5-11.el7.x86_64                              41/47 
  Cleanup    : libselinux-utils-2.5-12.el7.x86_64                         42/47 
  Cleanup    : libselinux-python-2.5-12.el7.x86_64                        43/47 
  Cleanup    : libselinux-2.5-12.el7.x86_64                               44/47 
  Cleanup    : audit-2.8.1-3.el7.x86_64                                   45/47 
  Cleanup    : audit-libs-2.8.1-3.el7.x86_64                              46/47 
  Cleanup    : libsepol-2.5-8.1.el7.x86_64                                47/47 
  Verifying  : libcgroup-0.41-20.el7.x86_64                                1/47 
  Verifying  : python-enum34-1.0.4-1.el7.noarch                            2/47 
  Verifying  : pyserial-2.6-6.el7.noarch                                   3/47 
  Verifying  : cloud-init-18.2-1.el7.centos.x86_64                         4/47 
  Verifying  : audit-libs-2.8.4-4.el7.x86_64                               5/47 
  Verifying  : audit-2.8.4-4.el7.x86_64                                    6/47 
  Verifying  : pytz-2016.10-2.el7.noarch                                   7/47 
  Verifying  : python2-pysocks-1.5.6-3.el7.noarch                          8/47 
  Verifying  : python-IPy-0.75-6.el7.noarch                                9/47 
  Verifying  : python2-cffi-1.11.2-1.el7.x86_64                           10/47 
  Verifying  : python2-requests-2.14.2-1.el7.noarch                       11/47 
  Verifying  : python2-jsonpatch-1.21-1.el7.noarch                        12/47 
  Verifying  : python2-jsonpointer-1.10-4.el7.noarch                      13/47 
  Verifying  : setools-libs-3.3.8-4.el7.x86_64                            14/47 
  Verifying  : policycoreutils-2.5-29.el7.x86_64                          15/47 
  Verifying  : python2-markupsafe-0.23-16.el7.x86_64                      16/47 
  Verifying  : python2-urllib3-1.21.1-1.el7.noarch                        17/47 
  Verifying  : libsemanage-python-2.5-14.el7.x86_64                       18/47 
  Verifying  : python2-asn1crypto-0.23.0-2.el7.noarch                     19/47 
  Verifying  : python2-babel-2.3.4-1.el7.noarch                           20/47 
  Verifying  : libsemanage-2.5-14.el7.x86_64                              21/47 
  Verifying  : libsepol-2.5-10.el7.x86_64                                 22/47 
  Verifying  : python2-cryptography-2.1.4-2.el7.x86_64                    23/47 
  Verifying  : checkpolicy-2.5-8.el7.x86_64                               24/47 
  Verifying  : python-ply-3.4-11.el7.noarch                               25/47 
  Verifying  : python2-six-1.10.0-9.el7.noarch                            26/47 
  Verifying  : libselinux-python-2.5-14.1.el7.x86_64                      27/47 
  Verifying  : audit-libs-python-2.8.4-4.el7.x86_64                       28/47 
  Verifying  : python-pycparser-2.14-1.el7.noarch                         29/47 
  Verifying  : libyaml-0.1.4-11.el7_0.x86_64                              30/47 
  Verifying  : python-prettytable-0.7.2-3.el7.noarch                      31/47 
  Verifying  : libselinux-utils-2.5-14.1.el7.x86_64                       32/47 
  Verifying  : policycoreutils-python-2.5-29.el7.x86_64                   33/47 
  Verifying  : python2-idna-2.5-1.el7.noarch                              34/47 
  Verifying  : python-chardet-2.2.1-1.el7_1.noarch                        35/47 
  Verifying  : PyYAML-3.10-11.el7.x86_64                                  36/47 
  Verifying  : libselinux-2.5-14.1.el7.x86_64                             37/47 
  Verifying  : python2-jinja2-2.10-2.el7.noarch                           38/47 
  Verifying  : python2-pyOpenSSL-17.3.0-3.el7.noarch                      39/47 
  Verifying  : libsemanage-2.5-11.el7.x86_64                              40/47 
  Verifying  : libselinux-python-2.5-12.el7.x86_64                        41/47 
  Verifying  : audit-libs-2.8.1-3.el7.x86_64                              42/47 
  Verifying  : libselinux-utils-2.5-12.el7.x86_64                         43/47 
  Verifying  : policycoreutils-2.5-22.el7.x86_64                          44/47 
  Verifying  : audit-2.8.1-3.el7.x86_64                                   45/47 
  Verifying  : libsepol-2.5-8.1.el7.x86_64                                46/47 
  Verifying  : libselinux-2.5-12.el7.x86_64                               47/47 

Installed:
  cloud-init.x86_64 0:18.2-1.el7.centos                                         

Dependency Installed:
  PyYAML.x86_64 0:3.10-11.el7                                                   
  audit-libs-python.x86_64 0:2.8.4-4.el7                                        
  checkpolicy.x86_64 0:2.5-8.el7                                                
  libcgroup.x86_64 0:0.41-20.el7                                                
  libsemanage-python.x86_64 0:2.5-14.el7                                        
  libyaml.x86_64 0:0.1.4-11.el7_0                                               
  policycoreutils-python.x86_64 0:2.5-29.el7                                    
  pyserial.noarch 0:2.6-6.el7                                                   
  python-IPy.noarch 0:0.75-6.el7                                                
  python-chardet.noarch 0:2.2.1-1.el7_1                                         
  python-enum34.noarch 0:1.0.4-1.el7                                            
  python-ply.noarch 0:3.4-11.el7                                                
  python-prettytable.noarch 0:0.7.2-3.el7                                       
  python-pycparser.noarch 0:2.14-1.el7                                          
  python2-asn1crypto.noarch 0:0.23.0-2.el7                                      
  python2-babel.noarch 0:2.3.4-1.el7                                            
  python2-cffi.x86_64 0:1.11.2-1.el7                                            
  python2-cryptography.x86_64 0:2.1.4-2.el7                                     
  python2-idna.noarch 0:2.5-1.el7                                               
  python2-jinja2.noarch 0:2.10-2.el7                                            
  python2-jsonpatch.noarch 0:1.21-1.el7                                         
  python2-jsonpointer.noarch 0:1.10-4.el7                                       
  python2-markupsafe.x86_64 0:0.23-16.el7                                       
  python2-pyOpenSSL.noarch 0:17.3.0-3.el7                                       
  python2-pysocks.noarch 0:1.5.6-3.el7                                          
  python2-requests.noarch 0:2.14.2-1.el7                                        
  python2-six.noarch 0:1.10.0-9.el7                                             
  python2-urllib3.noarch 0:1.21.1-1.el7                                         
  pytz.noarch 0:2016.10-2.el7                                                   
  setools-libs.x86_64 0:3.3.8-4.el7                                             

Dependency Updated:
  audit.x86_64 0:2.8.4-4.el7                                                    
  audit-libs.x86_64 0:2.8.4-4.el7                                               
  libselinux.x86_64 0:2.5-14.1.el7                                              
  libselinux-python.x86_64 0:2.5-14.1.el7                                       
  libselinux-utils.x86_64 0:2.5-14.1.el7                                        
  libsemanage.x86_64 0:2.5-14.el7                                               
  libsepol.x86_64 0:2.5-10.el7                                                  
  policycoreutils.x86_64 0:2.5-29.el7                                           

Complete!

```

2.查看cloud-init是否安装成功。
```log
[root@localhost ~] cloud-init -v
/bin/cloud-init 18.2
```

3.cloud-init默认日志路径：/var/log/cloud-init.log