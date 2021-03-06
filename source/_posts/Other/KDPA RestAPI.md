---
title: KDPA RestAPI
tags: [python]
categories: Python
description: 实验室提供API服务详解
date: 2019/3/10 10:00:00
---

<!--more-->

# KDPA RestAPI定义及说明

- [1 主机实例](#1主机实例)
    - [1-1 创建主机实例](#1-1创建主机实例)
    - [1-2 删除主机实例](#1-2删除主机实例)
    - [1-3 配置主机实例](#1-3配置主机实例)
    - [1-4 启动主机实例](#1-4启动主机实例)
    - [1-5 关闭主机实例](#1-5关闭主机实例)
    - [1-6 挂起主机实例](#1-6挂起主机实例)
    - [1-7 恢复主机实例](#1-7恢复主机实例)
    - [1-8 获取主机实例基本信息](#1-7获取主机实例基本信息)
    - [1-9 主机实例控制台](#1-9主机实例控制台)
    - [1-10 重命名主机实例](#1-10重命名主机实例)
    - [1-11 创建浮动IP](#1-11创建浮动IP)
    - [1-12 删除浮动IP](#1-12删除浮动IP)
    - [1-13 获取浮动IP信息](#1-13获取浮动IP信息)
    - [1-14 批量关闭实例组件](#1-14批量关闭实例组件)
    - [1-15 获取所有组件的状态](#1-15获取所有组件的状态)
- [2 网线](#2网线)
    - [2-1 创建网线](#2-1创建网线)
    - [2-2 删除网线](#2-2删除网线)
- [3 通用](#3通用)
    - [3-1 清除当前用户的实验室环境](#3-1清除当前用户的实验室环境)
    - [3-2 设置实训平台系统配置](#3-2设置实训平台系统配置)
    - [3-3 获取系统网卡信息及预留网段信息](#3-3获取系统网卡信息及预留网段信息)
    - [3-4 用户实验室环境未保存退出](#3-4用户实验室环境未保存退出)
    - [3-5 获取系统资源](#3-5获取系统资源)
    - [3-6 获取系统资源使用率](#3-6获取系统资源使用率)
    - [3-7 按日期获取系统资源使用率](#3-7按日期获取系统资源使用率)
- [4 系统](#4系统)
    - [4-1 创建镜像](#4-1创建镜像)
    - [4-2 删除镜像](#4-2删除镜像)
- [5 纵向](#5纵向)
    - [5-1 创建纵向](#5-1创建纵向)
    - [5-2 删除纵向](#5-2删除纵向)
- [6 串口线](#6串口线)
    - [6-1 创建串口线连接](#6-1创建串口线连接)
    - [6-2 删除串口线连接](#6-2删除串口线连接)
- [7 UKEY](#7UKEY)
    - [7-1 创建UKEY](#7-1创建UKEY)
    - [7-2 删除UKEY](#7-2删除UKEY)
    - [7-3 创建UKEY与纵向实例的连接](#7-3创建UKEY与纵向实例的连接)
    - [7-4 删除UKEY与纵向实例的连接](#7-4删除UKEY与纵向实例的连接)
- [8 隔离](#8隔离)
    - [8-1 创建隔离](#8-1创建隔离)
    - [8-2 删除隔离](#8-2删除隔离)
    - [8-3 启动隔离](#8-3启动隔离)
    - [8-4 关闭隔离](#8-4关闭隔离)
    - [8-5 挂起隔离](#8-5挂起隔离)
    - [8-6 恢复隔离](#8-6恢复隔离)
    - [8-7 隔离控制台](#8-7隔离控制台)
    - [8-8 隔离创建浮动IP](#8-8隔离创建浮动IP)
    - [8-9 隔离删除浮动IP](#8-9隔离删除浮动IP)
    - [8-10 创建隔离镜像](#8-10创建隔离镜像)
    - [8-11 删除隔离镜像](#8-11删除隔离镜像)
    - [6-2 删除串口线连接](#6-2删除串口线连接)
- [9 虚拟交换机](#9虚拟交换机)
    - [9-1 创建Untag虚拟交换机](#9-1创建Untag虚拟交换机)
    - [9-2 删除Untag虚拟交换机](#9-2删除Untag虚拟交换机)
    - [9-3 创建虚拟交换机](#9-3创建虚拟交换机)
    - [9-4 删除虚拟交换机](#9-4删除虚拟交换机)
    - [9-5 启动虚拟交换机](#9-5启动虚拟交换机)
    - [9-6 关闭虚拟交换机](#9-6关闭虚拟交换机)
    - [9-7 设置虚拟交换机端口](#9-7设置虚拟交换机端口)
    - [9-8 获取被占用vlan](#9-8获取被占用vlan)
- [10 虚拟路由器](#10虚拟路由器)
    - [10-1 创建虚拟路由器](#10-1创建虚拟路由器)
    - [10-2 删除虚拟路由器](#10-2删除虚拟路由器)
    - [10-3 查看虚拟路由器配置	](#10-3查看虚拟路由器配置)
    - [10-4 虚拟路由器上传镜像](#10-4虚拟路由器上传镜像)
    - [10-5 虚拟路由器删除镜像](#10-5虚拟路由器删除镜像)

> **Notice:**
> * 1.以下所有API的方法都为POST
> * 2.传参及返回值都为json格式，通用返回值格式为: {“code”: 状态码, “data”: 数据 }。
>  * 状态码为 0 代表操作成功，其他代表操作失败
> * 3.api地址中，controller为计算节点的IP地址（已配置在主机配置文件中。
8000为系统默认提供服务的端口号。
> * 4.*参数为必填参数，其余为非必填，非必填参数系统传参默认值。

> **限制:**
> * 1.所有组件:
>   * 所有组件在关机状态下不能连接网线，不能删除网线。
>   * 实验室环境在未点击保存时会被清理，如有需要，手动点击保存按钮。 
> * 2.主机:
>   * 处于有网线连接或者关机状态下的主机不能修改配置。
>   * 处于挂起状态下的主机只能执行恢复操作，不能执行其他操作。
> * 3.纵向:
>   * UKEY与纵向连接后，不能直接删除纵向，必须先拔出UKEY。
>   * 纵向与纵向之间通过网卡1连接，处理业务，如果纵向与纵向之间通过其他网卡连接，不予处理。
> * 4.串口线:
>   * 串口线目前仅支持主机、隔离。
> * 5.虚拟路由器:
>   * 虚拟路由器定义的配置在初始化后不能修改。
>   * 虚拟路由器与主机通过网线连接两端的网段必须相同。
> * 6.虚拟交换机:
>   * 虚拟交换机与虚拟交换机不能通过网线连接。
>   * 虚拟交换机VLAN接口只能连接虚拟组件，虚实接口只能连接实体组件。
>   * 不同虚拟交换机中的VLAN ID不能重复。
>   * 在接口有连接的情况下，不能修改虚拟交换机端口的类型（VLAN接口/虚实接口）。

> * 7.其他问题:
>   * 虚拟路由器与虚拟交换机网线连接后，目前虚拟路由器重启DHCP获取不到IP，导致虚拟路由器不可用。
>   * 虚拟路由器与主机组件之间有其他组件的情况下，需要手动为主机配置远端静态路由。
 

## 1主机实例

### 1-1创建主机实例
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/instance/create```
- Params:
    - **uuid**(str.)*: 主机UUID，唯一标识，必填，长度1~63
    - **name**(str.)*: 主机名称，必填，长度1~63
    - **imageName**(str.)*: 镜像名称，必填，长度1~63
    - **userId**(int.)*: 用户ID，必填
    - template(int.): 模板类型，非必填，
       - 1代表 1 vcpu, 1G ram, 10G disk
       - 2代表 2 vcpu, 2G ram, 20G disk
### 1-2删除主机实例
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/instance/delete```
- Params:
    - **uuid**(str.)*: 主机UUID，唯一标识，必填，长度1~63

### 1-3配置主机实例
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/instance/config```
- Params:
    - **uuid**(str.)*: 主机UUID，唯一标识，必填，长度1~63
    - **ipAddr**(str.)*: IPv4地址，必填，例如:192.168.1.100 
    - netmask(str.): 掩码地址，非必填，默认值为”255.255.255.0”
    - gateway(str.): 网关地址，非必填
    - number(int.): 网卡编号，非必填，默认值为1，表示主机目前都为1块网卡

### 1-4启动主机实例
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/instance/start```
- Params:
    - **uuid**(str.)*: 主机UUID，唯一标识，必填，长度1~63

### 1-5关闭主机实例
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/instance/stop```
- Params:
    - **uuid**(str.)*: 主机UUID，唯一标识，必填，长度1~63

### 1-6挂起主机实例
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/instance/suspend```
- Params:
    - **uuid**(str.)*: 主机UUID，唯一标识，必填，长度1~63
    
### 1-7恢复主机实例
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/instance/resume```
- Params:
    - **uuid**(str.)*: 主机UUID，唯一标识，必填，长度1~63

### 1-8获取主机实例基本信息
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/instance/show```
- Params:
    - **uuid**(str.)*: 主机UUID，唯一标识，必填，长度1~63

#### Response
- Body
```
{
    "code": 0, 
    "data": {
        "name": "HOST_1",
        "state": "Up",
        "interface": [{
            "macAddr": "fa:16:3e:62:7b:cb",   // MAC地址
            "ipAddr": "192.168.1.5",          //  IP地址
            "number": 0,                    // 网卡编号
            "netmask": "255.255.255.0",      // 掩码地址
            "cidr": "192.168.1.0/24",          // 网段地址
            "gateway": "192.168.1.1"         // 网关地址
        }]
    }
}
```

### 1-9主机实例控制台
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/instance/console```
- Params:
    - **uuid**(str.)*: 主机UUID，唯一标识，必填，长度1~63

#### Response
- Body
```
{
    "code": 0, 
    "data": {
        "console": {
            "url": "http://controller:6080/vnc_auto.html?token=aca31aec-fd05-46e4-9618-0e409c1e8b1e", 
            "type": "novnc"
        }
    }
}
```

### 1-10重命名主机实例
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/instance/rename```
- Params:
    - **uuid**(str.)*: 主机UUID，唯一标识，必填，长度1~63
    - **currentName**(str.)*: 修改后主机名称，必填，长度1~63 

### 1-11创建浮动IP
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/instance/createFIP```
- Params:
    - **uuid**(str.)*: 主机UUID，唯一标识，必填，长度1~63
    
### 1-12删除主机浮动IP
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/instance/deleteFIP```
- Params:
    - **uuid**(str.)*: 主机UUID，唯一标识，必填，长度1~63
    
### 1-13获取浮动IP信息
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/instance/showFIP```
- Params:
    - **uuid**(str.)*: 主机UUID，唯一标识，必填，长度1~63

#### Response
- Body
```
{
    "code": 0, 
    "data": {
        "ftpHost": "192.100.10.146", 
        "ftpPath": ftp://192.100.10.146/
        "username": "ftpuser", 
        "passwd": "ftpuser123", 
        "windowsPath": "c:\\ftpuser\\", 
        "linuxPath": "/home/ftpuser/", 
    }
}
```

### 1-14批量关闭实例组件
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/instance/batchStop```
- Params:
    - **userId**(int.)*: 用户ID，必填

### 1-15获取所有组件的状态
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/instance/showAllState```
- Params:
    - **userId**(int.)*: 用户ID，必填

#### Response
- Body
```
{
    "code": 0, 
    "data": {
        "lineList": [], 
        "instanceList": [{
            "state": "Up", 
            "uuid": "n-djsgs10124o",
            "ukeyid": "n-fds52326574vf"
        }]
    }
}
```
state: 包括 ”UP”: 开机，”Down”：关机，“Suspend”：挂起。

## 2网线

### 2-1创建网线
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/netline/create```
- Params:
    - **uuid**(str.)*: 源组件UUID，唯一标识，必填，长度1~63
    - **dstUuid**(str.)*: 网线对端组件UUID，必填，长度1~63
    - **netlineUuid**(int.)*: 网线UUID，必填，网线的唯一标识
    - **userId**(int.)*: 用户ID，必填
    - number(int.): 网卡编号，非必填，默认值为1，表示主机的第1块网卡
    - dstNumber(int.): 网线对端网设备卡编号，非必填，默认值为1
    - vlan(int.): 连接网线时所占用的vlan标签，非必填，默认值为0
      - 0: 默认值，表示当前连接的网线为常规网线，即虚拟组件与虚拟组件连接
      - 1~4094: 当vlan为1~4094之前的正整数时，表示此网线为虚实连线，vlan标签表示外部实体设备实际的vlan标签，此标签每个实体设备间不能相同。

### 2-2删除网线
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/netline/delete```
- Params:
    - **uuid**(str.)*: 网线UUID，唯一标识，必填，长度1~63
    - **userId**(int.)*: 用户ID，必填

## 3通用

### 3-1清除当前用户的实验室环境
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/env/clear```
- Params:
    - **userId**(int.)*: 用户ID，必填

### 3-2设置实训平台系统配置
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/system/config```
- Params:
    - **maxVmNumber**(str.)*: 用户最大创建虚拟机数量，默认值为10

### 3-3获取系统网卡信息及预留网段信息
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/system/info```
- Params:
    - 无

#### Response
- Body
```
{
    "code": 0,
    "data": {
        "networkParameters": [{
                "nodeType": "控制节点",
                "nodeName": "controller",
                "interfaceInfo": [{
                    "ethName": "enp2s0",
                    "ipAddr": "192.100.10.58"
                }, {
                    "ethName": "enp3s0",
                    "ipAddr": "10.0.0.11"
                }]
            },{
                "nodeType": "计算节点",
                "nodeName": "compute1",
                "interfaceInfo": [{
                    "ethName": "enp0s31f6",
                    "ipAddr": "192.100.10.160"
                }, {
                    "ethName": "enp0s20f0u8",
                    "ipAddr": "10.0.0.31"
                }]
        }] ,
        "reserveCidr": [ {
                "name": "浮动IP网段",
                "cidr": [{
                    "start": "192.100.10.140", "end": "192.100.10.141"
                }, {
                    "start": "192.100.10.161", "end": "192.100.10.169"
                }, {
                    "start": "192.100.10.144", "end": "192.100.10.149"
                }]
            },{
                "name": "Overlay网段",
                "cidr": [{
                    "start": "10.0.0.11", "end": "10.0.0.30"
                } ]
            },{
                "name": "内部网段",
                "cidr": [{
                    "start": "20.0.0.2", "end": "20.0.0.254"
                } ]
        } ]
    }
}
```

### 3-4用户实验室环境未保存退出
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/system/unsave```
- Params:
    - **instanceList**(list.)*: 实例组件uuid列表，必填
    - **netlineList**(list.)*: 网线组件uuid列表，必填
    - **userId**(int.)*: 用户ID，必填
    - **clickF5**(str)*: 是否点击F5进行刷新，默认为"False",非必填
    
### 3-5获取系统资源
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/system/resource```
- Params:
    - 无

#### Response
- Body
```
{
	"code": 0,
	"data": {
		"nodeNumber": 2,             # 节点数量
		"runningNodeNumber": 2,      # 正在运行的节点数量
		"instanceNumber": 2,          # 实例数量
		"vcpuUsed": 3,                # VCPU使用量        个数
		"vcpuTotal": 24,               # VCPU总量          个数
		"vcpu": "3/24",                # VCPU使用情况       个数
		"memoryUsed": 4.0,           # 内存使用量           GB
		"memoryTotal": 23.8,          # 内存总量             GB
		"memory": "4.0/23.8",          # 内存使用情况         GB
		"diskUsed": 30,               # 磁盘使用量           GB
		"diskTotal": 2198,             # 磁盘总量             GB
		"disk": "30/2198"              # 磁盘使用情况         GB
	}
}
```

### 3-6获取系统资源使用率
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/system/usage```
- Params:
    - 无
    
#### Response
- Body
```
{
	"code": 0,
	"data": {
	    "cpu": ["4.1", "4.1", "3.9", "4.6"],
	    "memory": ["67.6", "67.8", "67.8", "67.9"], 
	    "disk": ["5.7", "5.7", "5.7", "5.7"], 
	    "net_out": ["0", "653", "17702", "0"], 
	    "net_in": ["0", "2203", "18517", "0"], 
	    "time": [
	        "2019-06-14 07:38:01", 
	        "2019-06-14 07:38:10", 
	        "2019-06-14 07:38:20", 
	        "2019-06-14 07:38:30"] 
	}
}
```

### 3-7按日期获取系统资源使用率
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/system/dateUsage```
- Params:
    - **date**(str.)*: 日期，例："2019-06-17"

#### Response
- Body
```
{
    "code": 0, 
    "data": {
        "memory": ["71.8", "74.5", "74.5", "74.5", "74.1"], 
        "net_out": ["0", "772940", "130350", "167966", "172898"], 
        "net_in": ["0", "14704", "3530", "4194", "4442"], 
        "time": [
            "2019-06-17 02:00:02", 
            "2019-06-17 03:00:02", 
            "2019-06-17 04:00:02", 
            "2019-06-17 05:00:02", 
            "2019-06-17 06:00:02"
           ], 
        "disk": ["5.7", "5.7", "5.7", "5.7", "5.7"], 
        "cpu": ["4.6", "13.2", "13.8", "13.6", "13.5"]
    }
}
```

## 4系统

### 4-1创建镜像
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/image/create```
- Params:
    - **name**(str.)*: 镜像名称，必填
    - **url**(str.)*: 镜像路径，必填
    - defaultVCPU(int.): 默认虚拟CPU个数，非必填，默认值1
    - defaultRAM(int.): 默认内存大小，非必填，单位MB，默认值1024
    - defaultDISK (int.): 默认磁盘大小，非必填，单位GB，默认值10
    - advancedVCPU(int.): 高级虚拟CPU个数，非必填，默认值2
    - advancedRAM (int.): 高级内存大小，非必填，默认值2048
    - advancedDISK (int.): 高级磁盘大小，非必填，单位GB，默认值20
    
### 4-2删除镜像
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/image/delete```
- Params:
    - **name**(str.)*: 镜像名称，必填

## 5纵向

### 5-1创建纵向
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/pstunnel/create```
- Params:
    - **uuid**(str.)*: 纵向UUID，唯一标识，必填，长度1~63
    - **name**(str.)*: 纵向名称，必填，长度1~63
    - **userId**(int.)*: 用户ID，必填
    - template(int.): 模板类型，非必填，
        - 1代表 1 vcpu, 1G ram, 10G disk
        - 2代表 2 vcpu, 2G ram, 20G disk

### 5-2删除纵向
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/pstunnel/delete```
- Params:
    - **uuid**(str.)*: 纵向UUID，唯一标识，必填，长度1~63

## 6串口线

### 6-1创建串口线连接
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/serline/create```
- Params:
    - **uuid**(str.)*: 实例组件UUID，唯一标识，必填，长度1~63
    - **dstUuid**(str.)*: 目的实例组件UUID，唯一标识，必填，长度1~63
    - **seriallineUuid**(str.)*: 串口线UUID，唯一标识，必填，长度1~63
    - **userId**(int.)*: 用户ID，必填
    - number(int.): 默认值为0。对于主机，数值无意义。对于隔离，0==内隔离串口，1==外隔离串口
    - dstNumber(int.): 默认值为0。对于主机，数值无意义。对于隔离，0==内隔离串口，1==外隔离串口

### 6-2删除串口线连接
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/serline/delete```
- Params:
    - **uuid**(str.)*: 串口线UUID，唯一标识，必填，长度1~63
    - **userId**(int.)*: 用户ID，必填

## 7UKEY

### 7-1创建UKEY
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/ukey/create```
- Params:
    - **uuid**(str.)*: UKEY UUID，唯一标识，必填，长度1~63
    - **name**(str.)*: UKEY 名称，必填，长度1~63
    - **userId**(int.)*: 用户ID，必填

### 7-2删除UKEY
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/ukey/delete```
- Params:
    - **uuid**(str.)*: UKEY UUID，唯一标识，必填，长度1~63
    - **userId**(int.)*: 用户ID，必填

### 7-3创建UKEY与纵向实例的连接
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/ukey/connect```
- Params:
    - **uuid**(str.)*: UKEY UUID，唯一标识，必填，长度1~63
    - **psUuid**(str.)*: 纵向UUID，唯一标识，必填，长度1~63
    - **userId**(int.)*: 用户ID，必填

### 7-4删除UKEY与纵向实例的连接
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/ukey/disconnect```
- Params:
    - **uuid**(str.)*: UKEY UUID，唯一标识，必填，长度1~63
    - **psUuid**(str.)*: 纵向UUID，唯一标识，必填，长度1~63
    - **userId**(int.)*: 用户ID，必填

## 8隔离

### 8-1创建隔离
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/stonewall/create```
- Params:
    - **uuid**(str.)*: 隔离UUID，唯一标识，必填，长度1~63
    - **name**(str.)*: 隔离名称，必填，长度1~63
    - **imageName**(str.)*: 隔离镜像名称，必填，长度1~63
    - **userId**(int.)*: 用户ID，必填
    - template(int.): 模板类型，非必填，
        - 1代表 1 vcpu, 1G ram, 10G disk
        - 2代表 2 vcpu, 2G ram, 20G disk

### 8-2删除隔离
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/stonewall/delete```
- Params:
    - **uuid**(str.)*: 隔离UUID，唯一标识，必填，长度1~63
    - **userId**(int.)*: 用户ID，必填

### 8-3启动隔离
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/stonewall/start```
- Params:
    - **uuid**(str.)*: 隔离UUID，唯一标识，必填，长度1~63

### 8-4关闭隔离
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/stonewall/stop```
- Params:
    - **uuid**(str.)*: 隔离UUID，唯一标识，必填，长度1~63

### 8-5挂起隔离
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/stonewall/suspend```
- Params:
    - **uuid**(str.)*: 隔离UUID，唯一标识，必填，长度1~63

### 8-6恢复隔离
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/stonewall/resume```
- Params:
    - **uuid**(str.)*: 隔离UUID，唯一标识，必填，长度1~63

### 8-7隔离控制台
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/stonewall/console```
- Params:
    - **uuid**(str.)*: 隔离UUID，唯一标识，必填，长度1~63
    - number(int.): 标识内网/外网隔离，值范围：0/1，0代表内，1代表外，默认值为0

#### Response
- Body
```
{
    "code": 0, 
    "data": {
        "console": {
            "url": "http://controller:6080/vnc_auto.html?token=aca31aec-fd05-46e4-9618-0e409c1e8b1e", 
            "type": "novnc"
        }
    }
}
```

### 8-8隔离创建浮动IP
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/stonewall/createFIP```
- Params:
    - **uuid**(str.)*: 隔离UUID，唯一标识，必填，长度1~63
    - number(int.): 标识内网/外网隔离，值范围：0/1，0代表内，1代表外，默认值为0

### 8-9隔离删除浮动IP
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/stonewall/deleteFIP```
- Params:
    - **uuid**(str.)*: 隔离UUID，唯一标识，必填，长度1~63
    - number(int.): 标识内网/外网隔离，值范围：0/1，0代表内，1代表外，默认值为0

### 8-10创建隔离镜像
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/stonewall/createImg```
- Params:
    - **name**(str.)*: 镜像名称，必填
    - **urlInt**(str.)*: 内镜像路径，必填
    - **urlExt**(str.)*: 外镜像路径，必填
    - defaultVCPU(int.): 默认虚拟CPU个数，非必填，默认值1
    - defaultRAM(int.): 默认内存大小，非必填，单位MB，默认值1024
    - defaultDISK (int.): 默认磁盘大小，非必填，单位GB，默认值10
    - advancedVCPU(int.): 高级虚拟CPU个数，非必填，默认值2
    - advancedRAM (int.): 高级内存大小，非必填，默认值2048
    - advancedDISK (int.): 高级磁盘大小，非必填，单位GB，默认值20

### 8-11删除隔离镜像
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/stonewall/deleteImg```
- Params:
    - **name**(str.)*: 镜像名称，必填

## 9虚拟交换机
### 9-1创建Untag虚拟交换机
#### Request
- Method: **POST**
- API: `http://controller:8000/api/switch/createUntagSwitch`
- Params:
  - **uuid<str, 必填>**: 虚拟交换机UUID，唯一标识吗
  - **name<str, 必填>**: 虚拟交换机名称
  - **userId<str, 必填>**: 用户ID
  - *number<str, 非必填>*: 虚拟交换机网口数量，默认值8

#### Response
```
{
    "code": <int> API执行结果码，0-执行成功，其他正整数-执行失败的错误信息编码;
}
```
### 9-2删除Untag虚拟交换机
#### Request
- Method: **POST**
- API: `http://controller:8000/api/switch/deleteUntagSwitch`
- Params:
  - **uuid<str, 必填>**: 虚拟交换机UUID，唯一标识吗
#### Response
```
{
    "code": <int> API执行结果码，0-执行成功，其他正整数-执行失败的错误信息编码;
}
```
### 9-3创建虚拟交换机
#### Request
- Method: **POST**
- API: `http://controller:8000/api/switch/create`
- Params:
  - **uuid<str, 必填>**: 虚拟交换机UUID，唯一标识吗
  - **name<str, 必填>**: 虚拟交换机名称
  - **userId<str, 必填>**: 用户ID
  - *number<str, 非必填>*: 虚拟交换机网口数量，默认值8
#### Response
```
{
    "code": <int> API执行结果码，0-执行成功，其他正整数-执行失败的错误信息编码;
}
```
### 9-4删除虚拟交换机
#### Request
- Method: **POST**
- API: `http://controller:8000/api/switch/delete`
- Params:
  - **uuid<str, 必填>**: 虚拟交换机UUID，唯一标识吗
#### Response
```
{
    "code": <int> API执行结果码，0-执行成功，其他正整数-执行失败的错误信息编码;
}
```
### 9-5启动虚拟交换机
#### Request
- Method: **POST**
- API: `http://controller:8000/api/switch/start`
- Params:
  - **uuid<str, 必填>**: 虚拟交换机UUID，唯一标识吗
#### Response
```
{
    "code": <int> API执行结果码，0-执行成功，其他正整数-执行失败的错误信息编码;
}
```
### 9-6关闭虚拟交换机
#### Request
- Method: **POST**
- API: `http://controller:8000/api/switch/stop`
- Params:
  - **uuid<str, 必填>**: 虚拟交换机UUID，唯一标识吗
#### Response
```
{
    "code": <int> API执行结果码，0-执行成功，其他正整数-执行失败的错误信息编码;
}
```
### 9-7设置虚拟交换机端口
#### Request
- Method: **POST**
- API: `http://controller:8000/api/switch/configSwitchPort`
- Params:
  - **uuid<str, 必填>**: 虚拟交换机UUID，唯一标识吗
  - **number<str, 必填>**: 虚拟交换机端口序号，从0开始，最大值由交换机端口数量决定，必填
  - **vlan<str, 必填>**: 端口将要设置的具体vlan标签
    - *-1*：表示端口要设置成为【虚实口】
    - *0*： 表示端口默认状态，此状态下端口不可用，即不能连接网线
    - *1~4094*：表示端口设置为【vlan口】，vlan标签为1至4094之间的任意正整数
#### Response
```
{
    "code": <int> API执行结果码，0-执行成功，其他正整数-执行失败的错误信息编码;
}
```
### 9-8获取被占用vlan
#### Request
- Method: **POST**
- API: `http://controller:8000/api/switch/getUsedVlanList`
- Params:
  - **uuid<str, 必填>**: 虚拟交换机UUID，唯一标识吗
#### Response
```
{
    "code": <int> API执行结果码，0-执行成功，其他正整数-执行失败的错误信息编码;
    "data": [1, 23, 129, ...]
}
```

## 10虚拟路由器

### 10-1创建虚拟路由器
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/router/create```
- Params:
    - **uuid**(str.)*: 虚拟路由器UUID，唯一标识，必填，长度1~63
    - **name**(str.)*: 虚拟路由器名称，必填，长度1~63
    - **number**(int.)*: 虚拟路由器接口数量，默认值为2，必填
    - **cidrList**(list.)*: 虚拟路由器网段列表，必填
        - 例如：[“192.168.1.0/24”, “192.168.2.0/24”]
        - 网段的格式：网段地址/掩码位数， 网段不能重复。
    - **imageName**(str.)*: 镜像名称，必填，长度1~63
    - **userId**(int.)*: 用户ID，必填

### 10-2删除虚拟路由器
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/router/delete```
- Params:
    - **uuid**(str.)*: 虚拟路由器UUID，唯一标识，必填，长度1~63

### 10-3查看虚拟路由器配置	
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/router/show```
- Params:
    - **uuid**(str.)*: 虚拟路由器UUID，唯一标识，必填，长度1~63

#### Response
- Body
```
{
    "code": 0, 
    "data": [
        {
            "number": 0,
            "cidr": "192.168.1.0/24",
            "gateway": "192.168.1.1"
        },
        {
            "number": 1,
            "cidr": "192.168.2.0/24",
            "gateway": "192.168.2.1"
        }
    ]
}
```

### 10-4虚拟路由器上传镜像	
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/router/createImg```
- Params:
    - **name**(str.)*: 镜像名称，必填
    - **url**(str.)*: 镜像路径，必填
    
虚拟路由器镜像默认1VCPU，1G内存，10G磁盘

### 10-5虚拟路由器删除镜像	
#### Request
- Method: **POST**
- API: ```http://controller:8000/api/router/deleteImg```
- Params:
    - **name**(str.)*: 镜像名称，必填