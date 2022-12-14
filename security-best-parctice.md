# Fate安全best practice guidance
当使用FATE系统执行联邦学习等任务时，需使用恰当的安全模型和安全协议以确保计算过程中不泄露隐私数据。本文稿尝试给出使用FATE架构的安全指导。
## 一. FATE 安全现状
FATE安全协议有Paillier Encryption，RSA Encryption，Hash Factory，DH Key Exchange，SPDZ，OT和VSS。这些协议用于联邦学习中，为横向联邦、纵向联邦提供聚合阶段的安全保障。此外，一些联邦学习协议例如secureboost、Hetero Neural Network、Hetero Federated Transfer Learning等不需要第三方聚合服务器，也能保证参与方的数据隐私。因此从安全角度，FATE安全模型主要分为两种：有第三方arbiter和无第三方arbiter。

对于没有第三方arbiter的联邦学习安全假设为：各参与方为独立安全域，通过联邦学习协议进行数据传输，并且该协议能够保证各安全域的隐私数据不泄露。对于有第三方arbiter的联邦学习安全假设为：联邦学习各参与方互为独立的信任域，arbiter对于各参与方为半诚实安全域，即arbiter会遵守FATE协议执行，但是有可能会保留模型聚合阶段的中间数据。对于多个arbiter共同执行聚合、arbitration等操作时，要求arbiter之间不能共谋。采用安全多方计算等安全协议可以确保arbiter不会泄露各参与方的隐私数据。

基于此，使用FATE架构所带来的安全风险主要有：\
网络端口暴露、网络传输所带来的的安全风险\
半诚实arbiter带来的安全风险\
联邦模型带来的安全风险\
基于以上分析，本文稿给出了FATE平台的安全使用指导。


## 二. 网络安全建议
对于参与计算的FATE节点，各节点之间首先采用client_authentication和permission实现节点的身份认证。

在FATE涉及的网络传输过程中，例如基于HTTPS、Flask、gRPC协议等，采用SSL/TLS协议能够更好的保护数据传输的安全。尤其是在公网环境下进行网络传输时，SSL/TLS是必须选项。TLS密钥强度遵循RFC8446 section9.1

FATE平台通常暴露的网络端口为：
对于其余网络端口数据包，应丢弃。

## 三．安全协议配置建议
FATE目前支持的联邦学习安全协议为：paillier同态加密，RSA Encryption，Hash Factory，Diffne Hellman Key Exchange，SecretShare MPC Protocol，Oblivious Transfer，Feldman Verifiable secret sharing。现就相关安全协议给出安全配置建议,如下表所示。


| 协议名称        | 算法分类   | 安全配置建议 |
| :-------------:| :----------: | :------------: |
| Paillier Encryption|概率性非对称算法        | 参考RSA算法安全性要求，密钥长度（模长）最低1024，推荐2048          |
| RSA Encryption     |非对称加密算法        |密钥长度（模长）最低1024，推荐2048           |
| ECDH Encryption    | 非对称加密算法       |                                          |
| Hash Factory       |散列算法        |推荐使用sha256，sm3           |
| Diffie Hellman Key Exchange|密钥交换算法 |密钥长度（模长）最低1024，推荐2048          |
| SecretShare MPC Protocol (SPDZ) | 秘密分享算法  | 秘密分享servers之间不能共谋，并至少需要两个server；使用可信第三方生成乘法三元组或者使用paillier生成乘法三元组；|
|Feldman Verifiable Secret Share| 秘密分享算法 |      |
|Oblivious Transfer| 基于RSA非对称加密算法         | 密钥长度最低1024，推荐2048          |
|Differential Privacy|   |        | 




## 四．FederatedML Components安全建议

不同的联邦学习components具有不同的安全模型，有些允许参与方之间直接交换中间信息，有些需要arbiter进行调节，现列出FATE所支持的、涉及到不同party联邦学习components的安全模型以及相关的安全建议。其中，安全模型主要指是否需要第三方节点作为arbiter，以及该第三方节点的安全需求，是诚实模型或者半诚实模型。在FATE安全域假设的基础上，用户使用FATE引擎的安全威胁主要体现在两个方面：遵守协议的arbiter是否有可能泄露隐私数据、根据中间结果是否能反推数据。不同参与方之间是否能通过联邦学习的模型反推另一方数据。因此，安全建议主要关注：
 - 重构攻击（在模型训练阶段，计算方的目的是重构数据提供者的原始数据，或者学习关于数据的更多信息，而不是最终模型所提供的信息）；
 - 模型反演攻击和成员推理攻击（除非攻击者对模型有黑货或者白盒访问权限，该攻击不存在）；
 - 数据投毒和模型投毒：理论来说不可避免，需要非技术手段进行限制或者排除。
 - GANs，基于GANs的推理攻击：
 - 搭便车攻击的预防措施。
例如样本数要远大于特征数。Deep Leakage from Gradients

| Components名称 | 算法描述      |       安全模型  | 安全建议 |
| :-------------:| :----------: | :------------: |:-------:|
|  Intersect  |  计算两方的相交数据集，而不会泄漏任何差异数据集的信息。主要用于纵向任务。[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/intersect/)            |    参与方之间利用RSA、DH、ECDH加密算法保护数据隐私，不需要第三方arbiter            |  交集内ID信息会被共同参与方知晓       |
| Hetero Feature binning  |使用分箱的输入数据，计算每个列的iv和woe，并根据合并后的信息转换数据。[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/feature_binning/) |参与方之间利用paillier加性同态加密进行数据计算，不需要第三方arbiter | 数据提供方需正确遵守协议执行，若数据提供方故意错误执行Sum{Encry(yi)}，是否等于数据投毒？|
|Hetero Feature Selection| 提供多种类型的filter。每个filter都可以根据用户配置选择列。[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/feature_selection/)|不需要第三方|iv filter基于Hetero feature binning |
|Hetero LR|通过多方构建纵向逻辑回归模块。[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/logistic_regression/#heterogeneous-lr)|Phase1: Sample alignment. Phase2: Federated Training. |需要第三方arbiter，半诚实；梯度聚合可采用同态加密、秘密分享协议| 若采用同态加密，为避免arbiter获取梯度中间数据，需要附加掩码；若采用秘密分享协议，要求至少具有两个秘密分享server，且server之间不能共谋|
|Hetero SSHELR|heterogeneous logistic regression without arbiter role.[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/logistic_regression/#heterogeneous-sshe-logistic-regression) |不需要第三方arbiter，安全保证：采用Secure Matrix Multiplication Protocol which uses HE and Secret Sharing|重构攻击，模型反演，成员推理，特征推理，搭便车攻击|
|Homo LR|横向逻辑回归 [link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/logistic_regression/#homogeneous-lr)|需要第三方arbiter，半诚实；安全聚合可采用加性掩码、同态加密、秘密分享协议| |
|Hetero LinR|纵向线性回归[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/linear_regression/#heterogeneous-linr)|需要第三方arbiter，半诚实；安全聚合可采用加性掩码、同态加密、秘密分享协议| |
|Hetero Federated Poisson Regression |[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/poisson_regression/)|需要第三方arbiter，半诚实；安全聚合可采用加性掩码、同态加密、秘密分享协议||
|Homo NN||||
|Hetero Secure Boosting||||
|Hetero Fast Secure Boosting||||
|Hetero NN|[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/hetero_nn/)|不需要第三方arbiter，Party B works as dominating server，Party A is data provider without label y| |
|Homo Secure Boost|[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/ensemble/#homo-secureboost)|||
|Secure Information Retrieval|[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/sir/)|||
|Hetero FTL|[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/hetero_ftl/)|||
|PSI|[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/psi/)|||
|Hetero KMeans|[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/hetero_kmeans/)|||
|Feldman Verifiable Sum|[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/feldman_verifiable_sum/)|||


## 五. 合规性建议
### Issue11：个人信息保护法、gdpr









