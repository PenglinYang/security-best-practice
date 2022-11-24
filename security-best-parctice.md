# Fate安全best practice guidance
当使用FATE系统执行联邦学习等任务时，需使用恰当的安全模型和安全协议以确保计算过程中不泄露隐私数据。本文稿尝试给出使用FATE架构的安全指导。
## FATE 安全现状
FATE安全协议有Paillier Encryption，RSA Encryption，Hash Factory，DH Key Exchange，SPDZ，OT和VSS。这些协议用于联邦学习中，为横向联邦、纵向联邦提供聚合阶段的安全保障。此外，一些联邦学习协议例如secureboost、Hetero Neural Network、Hetero Federated Transfer Learning等不需要第三方聚合服务器，也能保证参与方的数据隐私。因此从安全角度，FATE安全架构主要分为两种：有第三方arbiter和无第三方arbiter。

对于没有第三方arbiter的联邦学习安全假设为：各参与方为独立安全域，通过联邦学习协议进行数据传输，并且该协议能够保证各安全域的隐私数据不泄露。对于有第三方arbiter的联邦学习安全假设为：联邦学习各参与方互为独立的信任域，arbiter对于各参与方为半诚实安全域，即arbiter会遵守FATE协议执行，但是有可能会保留模型聚合阶段的中间数据。对于多个arbiter共同执行聚合、arbitration等操作时，要求arbiter之间不能共谋。采用安全多方计算等安全协议可以确保arbiter不会泄露各参与方的隐私数据。

基于此，使用FATE架构所带来的安全风险主要有：\
网络端口暴露、网络传输所带来的的安全风险\
半诚实arbiter带来的安全风险\
联邦模型带来的安全风险\
基于以上分析，本文稿给出了FATE平台的安全配置指导。


## 二. 节点认证
采用client_authentication和permission实现节点认证。

## 三．网络与传输密码安全
### Issue4：RSA、ECDH、SM2、HMAC、HASH等的基础密码要求；
在FATE涉及的网络传输过程中，例如基于HTTPS、Flask、gRPC协议等，采用SSL/TLS协议能够更好的保护数据传输的安全。尤其是在公网环境下进行网络传输时，SSL/TLS是必须选项。TLS密钥强度遵循RFC8446 section9.1
### 网络端口暴露
FATE平台通常暴露的网络端口为：
若采用防火墙保护fateflow网络安全，丢弃其余端口；

## 四．联邦学习安全协议
FATE目前支持的联邦学习安全协议为：paillier同态加密，仿射性同态加密，RSA，可交换加密，DH密钥交换，秘密共享(spdz)，OT，安全聚合；
### paillier同态加密


同态加密、秘密分享、差分隐私、不经意传输、匿踪查询的安全要求；
### Issue5：同态加密的半同态、全同态等参数要求
### Issue6：秘密分享的参数要求
### Issue7：差分隐私的参数要求
差分隐私用于防范差分攻击，通过对查询结果加入噪声，使得攻击者无法辨别某一样本是否在数据集中。
拉普拉斯噪声
高斯噪声
{中央差分隐私、本地差分隐私}->
Pr[M(x)∈S]≤exp(ε)Pr[M(y)∈S]+δ
这里的ε被称为隐私预算。小的ε可以提供更高程度的隐私保护。而δ允许了算法在两个相邻数据输出同一值的大的概率差异。
### Issue8：不经意传输的参数要求
### Issue9：匿踪查询的参数要求
## 五．联邦学习协议安全
### Issue10：横向联邦、纵向联邦、联邦迁移
CS或者p2p；梯度保护方法：同态加密、差分隐私、TEE

## 六．针对数据拥有者、数据消费者、计算引擎的合规checklist
### Issue11：个人信息保护法、gdpr


## 其它问题
### Issue12：纵向联邦中间结果泄露
### Issue13：交集内ID泄露问题
### Issue14:数据投毒，模型修改检测
### Issue15：模型结构保护










