# ZH

![](../../eBPF_Documentation/static/imgs/LMP-logo.png)
# Linux microscope

LMP是一个基于BCC(BPF Compiler Collection)的Linux系统性能数据实时展示的web工具，它使用BPF(Berkeley Packet Filters)，也叫eBPF，目前LMP在ubuntu18.04上测试通过，内核版本4.15.0。

## server项目结构

```shell
├── api
│   └── v1
├── config
├── core
├── docs
├── global
├── initialize
│   └── internal
├── middleware
├── model
│   ├── request
│   └── response
├── packfile
├── resource
│   ├── excel
│   ├── page
│   └── template
├── router
├── service
├── source
└── utils
    ├── timer
    └── upload
```

| 文件夹       | 说明                    | 描述                        |
| ------------ | ----------------------- | --------------------------- |
| `api`        | api层                   | api层 |
| `--v1`       | v1版本接口              | v1版本接口                  |
| `config`     | 配置包                  | config.yaml对应的配置结构体 |
| `core`       | 核心文件                | 核心组件(zap, viper, server)的初始化 |
| `docs`       | swagger文档目录         | swagger文档目录 |
| `global`     | 全局对象                | 全局对象 |
| `initialize` | 初始化 | router,redis,gorm,validator, timer的初始化 |
| `--internal` | 初始化内部函数 | gorm 的 longger 自定义,在此文件夹的函数只能由 `initialize` 层进行调用 |
| `middleware` | 中间件层 | 用于存放 `gin` 中间件代码 |
| `model`      | 模型层                  | 模型对应数据表              |
| `--request`  | 入参结构体              | 接收前端发送到后端的数据。  |
| `--response` | 出参结构体              | 返回给前端的数据结构体      |
| `packfile`   | 静态文件打包            | 静态文件打包 |
| `resource`   | 静态资源文件夹          | 负责存放静态文件                |
| `--excel` | excel导入导出默认路径 | excel导入导出默认路径 |
| `--page` | 表单生成器 | 表单生成器 打包后的dist |
| `--template` | 模板 | 模板文件夹,存放的是代码生成器的模板 |
| `router`     | 路由层                  | 路由层 |
| `service`    | service层               | 存放业务逻辑问题 |
| `source` | source层 | 存放初始化数据的函数 |
| `utils`      | 工具包                  | 工具函数封装            |
| `--timer` | timer | 定时器接口封装 |
| `--upload`      | oss                  | oss接口封装        |


## 项目架构

![](../../eBPF_Documentation/static/imgs/LMP-arch4.png)
![](../../eBPF_Documentation/static/imgs/LMP-arch5.png)

## 界面截图

![homepage](../../eBPF_Documentation/static/imgs/homepage.png)

![homepage](../../eBPF_Documentation/static/imgs/grafana1.png)

![grafana4](../../eBPF_Documentation/static/imgs/grafana4.png)


# LMP web项目详细部署步骤

## 从源码构建lmp，需要的基本环境：

- golang：go1.12及以上；

- docker

  安装命令:

  ```bash
  curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
  ```



- docker:influxdb&grafana&influxdb；

  依赖安装命令:

  ```bash
  sudo docker pull grafana/grafana
  sudo docker pull influxdb
  ```



- bcc环境

- mysql：5.7.29测试通过

## 编译并安装

​	 下载项目源码，并进入后端文件夹:

```bash
git clone https://github.com/linuxkerneltravel/lmp.git   
cd ~/lmp/eBPF_Visualization/eBPF_server
```

​	打开当前目录下的config.yaml文件，可按需要修改mysql的用户和密码以及相关配置:

```bash
vim config.yaml
```

```yaml
mysql:
  path: 127.0.0.1
  port: "3306"
  config: charset=utf8mb4&parseTime=True&loc=Local
  db-name: gva
  username: root
  password: "123456"
  max-idle-conns: 0
  max-open-conns: 0
  log-mode: ""
  log-zap: false
```

自动安装go依赖库并编译安装：

```bash
go mod tidy
make
```

编译安装完成之后，在当前目录下会多出两个文件，一个是lmp-cli,另一个是lmp-server。lmp-cli是以命令行运行的方式收集ebpf程序的输出信息。lmp-server用于启动web项目的后端程序。

## 启动单机节点，运行后端程序

```bash
sudo ./lmp-server
```

能正常启动后端程序之后，安装nodejs,进入前端文件夹并安装前端运行的依赖，运行前端程序:

```bash
sudo apt-get install npm
cd ~/lmp/eBPF_Visualization/eBPF_web
npm install
npm run serve
```

正常启动后，可用浏览器访问http://localhost:8080/，实现本地部署。
## 单机节点，本地运行

```
# 项目的所有配置均位于config.yaml中，grafana的默认端口为3000端口，influxdb的默认端口为8086，修改配置信息的方式如下：
 vim lmp/config/config.yaml

#run grafana
 sudo docker run -d \
   -p 3000:3000 \
   --name=grafana \
   grafana/grafana

#run influxdb，按照如下命令启动influxdb之后，会自动带有database lmp，influxdb的用户名和密码位于config.yaml中
 sudo docker run -d \
    -p 8083:8083 \
    -p 8086:8086 \
    --name influxdb \
    -v ${YOUR_PROJECT_PATH}/lmp/test/influxdb_config/default.conf:/etc/influxdb/influxdb.conf \
    -v ${YOUR_PROJECT_PATH}/lmp/test/influxdb_config/data:/var/lib/influxdb/data \
    -v ${YOUR_PROJECT_PATH}/lmp/test/influxdb_config/meta:/var/lib/influxdb/meta \
    -v ${YOUR_PROJECT_PATH}/lmp/test/influxdb_config/wal:/var/lib/influxdb/wal influxdb1.8

#run lmp
 cd lmp/
 make
 sudo ./lmp -h
```

### 观测-步骤

在shell执行`sudo ./lmp`，即启动观测功能

在单机节点上部署完成lmp并启动之后，通过浏览器访问8080端口即可。如果是本地查看，则访问localhost:8080，如果远程访问，则访问remoteip:8080即可。

8080端口返回页面如下，该页面仅用于观测指标下发，输入栏输入的是观测时间，单位是分钟。可一次下发多个指标，但是注意实际环境使用bcc的开销问题，建议单个指标下发对比开销之后，再组合多个指标观测。

![homepage](../../eBPF_Documentation/static/imgs/homepage.png)

另外开启grafana页面，通过浏览器访问3000端口即可。如果是本地查看，则访问localhost:3000，如果远程访问，则访问remoteip:3000即可。

grafana用于指标数据观测，进入grafana之后，首先需要登录进入grafana，初始用户名和密码均为admin，之后需要配置grafana连接influxdb：

![grafana1](../../eBPF_Documentation/static/imgs/grafana1.png)

按照自己的ip地址配置完成以后，点击save&test按钮，测试influxdb是否连接成功，出现如下提示说明连接成功：

![grafana2](../../eBPF_Documentation/static/imgs/grafana2.png)

接下来导入/lmp/test/grafana-JSON下的lmp.json文件，即可自动创建grafana的dashboard：

![grafana3](../../eBPF_Documentation/static/imgs/grafana3.png)

在Dashboards中Manage中，点击Import，上传lmp.json文件即可观测：

![grafana4](../../eBPF_Documentation/static/imgs/grafana4.png)

统计时间到时后，lmp会自动关闭后台bcc插件，之后继续在8080端口页面下发指标即可。

### 扩展功能：例如机器学习
LMP通过命令行参数引入机器学习模型，你可以自己实现模型后对接到项目中，我们的想法是利用观测功能提取到数据后，利用这些数据来训练模型，目前仅可引入机器学习模型，使用示例如下：
```shell
⇒  ./lmp -h 
NAME:
   LMP - LMP is a web tool for real-time display of Linux system performance data based on BCC (BPF Compiler Collection). 
To get more info of how to use lmp:
  # lmp help


USAGE:
   lmp [global options] command [command options] [arguments...]

VERSION:
   v0.0.1

COMMANDS:
   cluster  Density peak clustering
   help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --help, -h     show help (default: false)
   --version, -v  print the version (default: false)
```
命令介绍中介绍了cluster命令，在命令行执行：
```shell
⇒  ./lmp cluster -h
NAME:
   lmp cluster - Density peak clustering

USAGE:
   lmp cluster [command options] [APP_NAME]

DESCRIPTION:
   Density peak clustering, Can be used for anomaly detection.
       example: ./lmp cluster --data /YOUR_PATH
       example: ./lmp cluster -d /YOUR_PATH

OPTIONS:
   --data value, -d value  specified the the dataset to run
   --help, -h              show help (default: false)

```
即可看到使用方法，关于如何增加一个模型见→[增加一个模型](http://lmp.kerneltravel.net/start/addmodel/)

### 如何增加插件
LMP目前支持BCC类型的插件程序，增加的方法见→[增加一个插件](http://lmp.kerneltravel.net/start/addplugin/)

### 如何调试项目
推荐使用 `postman` 下发web请求，具体调试方法见→[调试项目](http://lmp.kerneltravel.net/start/getstat/)

### 总结出的监控方案
[基于 eBPF 的 prometheus 监控方案](http://lmp.kerneltravel.net/study/ebpf_prometheus/)

[快速实现Linux系统性能数据提取、存储和可视化展示](http://lmp.kerneltravel.net/study/exporter_prometheuse_grafana/)

#

![](../../eBPF_Documentation/static/imgs/LMP-logo.png)

# Linux microscope

LMP is a web tool for real-time display of Linux system performance data based on BCC (BPF Compiler Collection), which uses BPF (Berkeley Packet Filter), also known as eBPF. Currently, LMP is tested on ubuntu18.04 and the kernel version is 4.15.0.

## Project architecture

![](../../eBPF_Documentation/static/imgs/LMP-arch4.png)

## Interface screenshot

![homepage2](../../eBPF_Documentation/static/imgs/homepage2.png)

![homepage](../../eBPF_Documentation/static/imgs/grafana.png)

![homepage](../../eBPF_Documentation/static/imgs/data.png)


## Project structure overview  

<details>
<summary>Expand to view</summary>
<pre><code>.
.
├── LICENSE
├── README.md
├── config  # 配置
├── docs    # 文档
├── pkg     # golang 服务代码
├── plugins # python ebpf 代码
├── static  # 网页代码
├── test    # 测试数据
└── vendor  # golang vendor 
</code></pre>
</details>


##  install lmp

###  Ubuntu-source

#### Build lmp from source，The basic environment required is as follows：

- golang
- docker
- bcc

###  Install dependent docker image

```
# For prometheus 
sudo docker pull prom/prometheus
# For grafana
sudo docker pull grafana/grafana
# For Influxdb
sudo docker pull influxdb
```

### Compile and install

```
 git clone https://github.com/linuxkerneltravel/lmp
 cd lmp
 make
 sudo make install
```

##  Single machine node, Run locally

```
# Modify configuration file
 vim lmp/config/config.yaml

#run grafana
 sudo docker run -d \
   -p 3000:3000 \
   --name=grafana \
   -v /opt/grafana-storage:/var/lib/grafana \
   grafana/grafana
   
#run influxdb
 sudo docker run -d \
    -p 8083:8083 \
    -p 8086:8086 \
    --name influxdb \
    -v ${YOUR_PROJECT_PATH}/lmp/test/influxdb_config/default.conf:/etc/influxdb/influxdb.conf \
    -v ${YOUR_PROJECT_PATH}/lmp/test/influxdb_config/data:/var/lib/influxdb/data \
    -v ${YOUR_PROJECT_PATH}/lmp/test/influxdb_config/meta:/var/lib/influxdb/meta \
    -v ${YOUR_PROJECT_PATH}/lmp/test/influxdb_config/wal:/var/lib/influxdb/wal influxdb
    
#run lmp
 cd lmp/
 make
 sudo ./lmp
```

### observation

http://localhost:8080/  After logging in to grafana, view it.

### Uninstall

```
make clean
```

## Thanks for the support of the following open source projects

- [Gin] - [https://gin-gonic.com/](https://gin-gonic.com/)
- [bcc] - [https://github.com/iovisor/bcc](https://github.com/iovisor/bcc)

# 