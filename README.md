# 企业级部署之多群组多机构+新增节点

团队任务需要基于Weldentity做开发，狗子哥又刚好发了鸡腿任务。

把上次的单群组双机构双节点组网模式延伸下，再加上新增节点的操作。

[玩转WeIdentity之可视化部署](https://mp.weixin.qq.com/s/81c4C8kGuGeW5X5gNJhdNQ)
这里狗子哥通过build_chain高效完成了部署，如果只是为了学习Weldentity或者搭链测试还是建议使用这个或one_click_generator。

**生命不息，折腾不止**

---

这里依旧是使用真机环境。感谢队里师傅们的服务器👏

## 1. 下载安装

```bash
  git clone https://github.com/FISCO-BCOS/generator.git
  cd generator
  bash ./scripts/install.sh
  ./generator -h  #检查是否安装成功
  ./generator --download_fisco ./meta --cdn #通过cdn下载
  ./meta/fisco-bcos -v  #检查是否安装成功
```

这里有几个坑，

- 坑1：

  可以看下他的install源码

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004024753343.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW95dWUyMDE5,size_16,color_FFFFFF,t_70#pic_center)

  不支持mac os👀，狗子哥也在这里掉坑了。

- 坑2：

  **慢~🚲**

  下载缓慢可以增加参数 --cdn

  可以在generator中翻到,-h的帮助文档中也可以看到：

  ![](https://img-blog.csdnimg.cn/20201004024753349.png#pic_center)

## 2. 总览

官方教程中通过端口的不同来区别不同机器，从0加到了5，模拟3台机子。我们这里直接三台真机，其实这个就是对官方教程的实战。上次的[单群组双机构双节点组网模式](https://blog.csdn.net/xiaoyue2019/article/details/107500811)加上这次六节点三机构两群组组网模式，相信大家应该可以融会贯通搭建自定义的组网模式了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004024753411.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW95dWUyMDE5,size_16,color_FFFFFF,t_70#pic_center)

## 3. 使用CA生成链证书

```bash
./generator --generate_agency_certificate ./dir_agency_ca ./dir_chain_ca agencyA
ls dir_agency_ca/agencyA/
```

下图所示为生成成功：

[](https://img-blog.csdnimg.cn/20201004024753360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW95dWUyMDE5,size_16,color_FFFFFF,t_70#pic_center)

## 4. 初始化所有机构

> **官方教程中直接使用CA生成了机构证书和私钥，实践中可能机构会自己生成私钥，打包成证书请求文件后再由CA机构认证。这里我用机构A作为例子演示下实战中生成过程。**

```bash
#首先本地生成私钥agency.key
openssl genrsa -out agency.key 2048
#本地使用机构私钥agency.key生成证书请求文件agency.csr
openssl req -new -sha256 -subj "/CN=xiaoyue/O=fisco-bcos/OU=agency" -key ./agency.key -out ./agency.csr
#发送给ca进行认证
openssl x509 -req -days 3650 -sha256 -CA ./ca.crt -CAkey ./ca.key -CAcreateserial -in ../../generator-a/agency.csr -out ./agency.crt  -extensions v4_req
```

具体细节可参考[FISCO BCOS证书说明](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/articles/3_features/37_safety/certificate_description.html?highlight=agency.key)

个人感觉比较迷，下面还是和教程一样统一CA直接颁发机构证书。

```bash
./generator --generate_agency_certificate ./dir_agency_ca ./dir_chain_ca agencyA
./generator --generate_agency_certificate ./dir_agency_ca ./dir_chain_ca agencyB
./generator --generate_agency_certificate ./dir_agency_ca ./dir_chain_ca agencyC
```

## 5.配置所有机构

由于AB构建群组1、AC构建群组2

所以配置会有不一样的地方。

### 5.1 首先是AB配置、互发节点信息，并生成创世区块

1. 修改node_deployment.（generator会根据其生成相关节点证书、生成节点配置文件夹）

```java
[group]
group_id=1

[node0]
; host ip for the communication among peers.
; Please use your ssh login ip.
p2p_ip=波波的ip
; listen ip for the communication between sdk clients.
; This ip is the same as p2p_ip for physical host.
; But for virtual host e.g. vps servers, it is usually different from p2p_ip.
; You can check accessible addresses of your network card.
; Please see https://tecadmin.net/check-ip-address-ubuntu-18-04-desktop/
; for more instructions.
rpc_ip=0.0.0.0
channel_ip=0.0.0.0
p2p_listen_port=30300
channel_listen_port=20200
jsonrpc_listen_port=8545

[node1]
p2p_ip=波波的ip
rpc_ip=127.0.0.1
channel_ip=0.0.0.0
p2p_listen_port=30301
channel_listen_port=20201
jsonrpc_listen_port=8546
```

2. B的node_deployment，就改一下IP，相关配置在上篇文章中有比较详细的解释。

```java
[group]
group_id=1

[node0]
; host ip for the communication among peers.
; Please use your ssh login ip.
p2p_ip=牧子哥的ip
; listen ip for the communication between sdk clients.
; This ip is the same as p2p_ip for physical host.
; But for virtual host e.g. vps servers, it is usually different from p2p_ip.
; You can check accessible addresses of your network card.
; Please see https://tecadmin.net/check-ip-address-ubuntu-18-04-desktop/
; for more instructions.
rpc_ip=0.0.0.0
channel_ip=0.0.0.0
p2p_listen_port=30302
channel_listen_port=20202
jsonrpc_listen_port=8547

[node1]
p2p_ip=牧子哥的ip
rpc_ip=0.0.0.0
channel_ip=0.0.0.0
p2p_listen_port=30303
channel_listen_port=20203
jsonrpc_listen_port=8548
```

3. 其他的配置也相同，本机两个节点，如果向扩容就按照上面的格式再加一个node。

4. 生成全部节点证书和P2P连接信息文件。要保证拥有node_deployment、机构证书、私钥

```bash

./generator --generate_all_certificates ./agencyA_node_info
./generator --generate_all_certificates ./agencyB_node_info
./generator --generate_all_certificates ./agencyC_node_info
#记得是要在不同机构下跑哦！
```

5. 互发节点连接信息，因为机构A需要配置创世区块，所以机构还B需要把节点证书文件发送给A。

6. 修改group_genesis.ini，生成创世区块并分发群组1的创世区块到机构B

```java
# ./conf/group_genesis.ini
[group]
group_id=1

[nodes]
node0=波波ip:30300
node1=波波ip:30301
node2=牧子哥ip:30300
node3=牧子哥ip:30301
```

```bash
./generator --create_group_genesis ./group
```

7. 生成机构节点

```bash
./generator --build_install_package ./meta/peersB.txt ./nodeA
./generator --build_install_package ./meta/peersA.txt ./nodeB
#需要注意的是如果有多个连接文件需要合并，无格式的。
```

8. 启动两机构

```bash
bash ./nodeA/start_all.sh
bash ./nodeB/start_all.sh
```

9. 检查共识

```bash
tail -f ./node*/node*/log/log*  | grep +++
```

这个时候停止机构B

```bash
bash ./nodeB/stop_all.sh
````

不出我肖某人所料应该会timeout

### 5.2 接下来便是AC构建群组2

因为之前已经做好了机构C的相关配置，接下来就是纯构建群组的过程。

1. AC互发节点信息(俺是真机，需要文件转移都是直接xshell-ftp拖的✌。)

2. A需要发证书信息给C构建创世区块

3. 构建群组2创世区块并转移

```java
# ./conf/group_genesis.ini
[group]
group_id=2

[nodes]
node0=波波ip:30300
node1=波波ip:30301
node2=汤师傅ip:30304
node3=汤师傅ip:30305
```

```bash
./generator --create_group_genesis ./group
```

4. 机构A添加群组

```bash
./generator --add_group ./meta/group.2.genesis ./nodeA
# 增加节点C的信息
./generator --add_peers ./meta/peersC.txt ./nodeA
```

. 机构C生成节点

```bash
./generator --build_install_package ./meta/peersA.txt ./nodeC
```

5. 启动全部节点

```bash
# 重启A
bash ./nodeA/stop_all.sh
bash ./nodeA/start_all.sh
# 启动C
bash ./nodeC/start_all.sh
```

检查共识发现如下回显，那么恭喜你完成了我24小时的工作量！

![](https://img-blog.csdnimg.cn/20201004024753372.png#pic_center)

![](https://img-blog.csdnimg.cn/20201004024753375.png#pic_center)

**这里大部分原因是由于这个坑：**`not caused by omit empty block`

官方文档中说:`网络抖动、网络断连或配置出错(如同一个群组的创世块文件不一致)均有可能导致节点共识异常`

```
warning|2019-06-26 18:00:06.154102|[g:1][CONSENSUS][PBFT]ViewChangeWarning: not caused by omit empty block ,v=5,toV=6,curNum=715,hash=ed6e856d...,nodeIdx=3,myNode=e39000ea...
```

- 坑1

  conf文件夹下的两个配置文件中均有对groupid的配置。（千万记得要改，别只改个IP）

  ```java
  [group]
  group_id=2
  ```

- 坑2

  [端口配置](https://blog.csdn.net/xiaoyue2019/article/details/107401334)
  
  这个是真的坑，配置之前千万`sudo ufw status`看看端口是不是开着。（这种级别的配置当然不考虑关防火墙了）我是配好了之后才发现node2的端口根本没有对外数据传输，这卡了我一下午。

## 6. 新增节点

这里我将模拟机构A新增节点，并转移到一个真机当中（感谢客户的机🤣）

然后演示一下准入机制，在控制台将新增节点添加为共识节点。

1. 修改node_deployment

```java
[group]
group_id=1

[node0]
p2p_ip=客户ip
rpc_ip=客户ip
channel_ip=0.0.0.0
p2p_listen_port=30300
channel_listen_port=20201
jsonrpc_listen_port=8551
```

2. 生成证书与连接信息

```bash
./generator --generate_all_certificates ./agencyA_node_info_new
```

3. 生成节点

```bash
# 首先将机构中两个节点的ip信息合并
cat ./agencyA_node_info/peers.txt >> ./meta/peersB.txt
# 然后生成新的nodeA
./generator --build_install_package ./meta/peersB.txt ./nodeA_new
```

4. 转移节点。 ctrl+c，ctrl+v 放到别的机子上

5. 配置控制台，添加节点6进入group1

```bash
./generator --download_console ./ --cdn
```

节点6的conf文件夹下有个node.nodeid，查看一下

```
degca519148cafc3e92c8d9a8572b41ea2f62d0d19e99273ee18cccd34ab50079b4ec82fe5f4ae51bd95dd711811c97153ece8c05eac7a5ae34c96454c4d3123
```

**然后启动控制台**

```bash
# 这里后面没有groupid默认进入1
cd ~/generator-A/console && bash ./start.sh
```

**添加共识节点**

```bash
addSealer nodeid
```

![](https://img-blog.csdnimg.cn/20201004024753369.png#pic_center)

查看共识，发现红色加加。

**恭喜，你完成了多群组多机构多节点组网并且准入了一个新的节点。**

接下来可以尝试完成构建国密区块链
研读generator详细配置文档，尝试写脚本改进构建过程


---

来自上海对外经贸大学区块链与应用研究中心-肖越

*website：[cnmf.net.cn](cnmf.net.cn)*

*csdn: [https://blog.csdn.net/xiaoyue2019](https://blog.csdn.net/xiaoyue2019)*

*github: [https://github.com/xiaoyue2019](https://github.com/xiaoyue2019)*

---

*>https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/enterprise_tools/tutorial_detail_operation.html#id34*

*>https://fisco-bcos-documentation.readthedocs.io/zh_CN/release-2/docs/design/security_control/node_management.html?highlight=%E8%A7%82%E5%AF%9F%E8%8A%82%E7%82%B9#id6*
