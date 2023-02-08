# Fate security best practice guidance

When the FATE is used to perform tasks such as federated learning, security models and security protocols should be used to ensure that no privacy data is disclosed during the calculation process. This draft attempts to provide security instructions for using FATE.
## 1. State of Art of FATE

FATE includes multiple federal learning protocols, some of which are composed of participants and third-party arbiters, such as Homo LR. Other federated learning protocols do not require third-party arbiter to do parameter aggregation, such as secureboost, Hetero Neural Network, Hetero FTL, etc. Therefore, FATE security model can be divided into two types: with third-party arbiter and without third-party arbiter.

For the federated learning without third-party arbiter, its security assumption is: each participant in FATE is an honest, curious and independent security domain, and the data interaction between participants can ensure that the privacy data of each security domain is not leaked. For the federated learning with third-party arbiter, the security assumption is: the federal learning participants are honest, curious and independent security domains, and the arbiter is honest and curious for all participants. That is, the arbiter will comply with the FATE protocol's execution, but may retain or inference aggregation stage intermediate data. When multiple arbiters perform aggregation and arbitration operations, they must not collude with each other.

Security models of FATE need to use security protocols such as secure multi-party computing to protect data security during data interaction stage between different participants. FATE security protocols include Paillier Encryption, RSA Encryption, Hash Factory, DH Key Exchange, SPDZ, OT, etc.

## 2. Network Security Suggestions

For the participating nodes in FATE, client_authentication and permission components need to be used to realize identity authentication function between nodes.

In the network transmission process involved in FATE, such as HTTPS, Flask, gRPC, etc., SSL/TLS protocol is used to protect the security of data transmission. In particular, SSL/TLS is required for network transmission in a public network environment. TLS key strength complies with RFC8446 section9.1.

FATE usually expose one Network port: 9370，network data package of other ports need to be treated with caution.

## 3. Security Protocol Configuration Recommendations

The federated learning security protocol that FATE supports are: Paillier Homomorphic Encryption, Hash Factory, DH Key Exchange, SecretShare MPC Protocol, Oblivious Transfer. The table below shows the security configuration recommendation. 

| Protocol Name        | Algorithm Classification   | Security Configuration Recommendation |
| :-------------:| :----------: | :------------: |
| Paillier Encryption|asymmetric encryption algorithm   | Refer to RSA algorithm requirement, minimum key length (module length) is 1024 bits, 2048 bits is recommended |
| RSA Encryption     |asymmetric encryption algorithm   |minimum key length (module length) is 1024 bits, 2048 bits is recommended  |
| ECDH Encryption    | asymmetric encryption algorithm based on elliptic curve |        key length 256bits   |
| Diffie Hellman Key Exchange|key exchange algorithm| minimum key length 1024bits, 2048bits recommended      |
| Hash Factory       |hash algorithm        |sha256，sm3 are recommended            |
| SecretShare MPC Protocol (SPDZ) | secret share algorithm  | participants in secret share cannot collusion with each other, and two participants at least; use trusted third party or Paillier Encryption to generate multiply triples |
|Oblivious Transfer| based on RSA encryption algorithm    | minimum key length is 1024 bits, 2048bits is recommended    |

## 4. FederatedML Components Security Recommendations

When the security assumptions of FATE participating nodes are honest and curious, adversary cannot obtain valid plaintext information about the model and intermediate gradient data under the protection of security protocol described in Section 3. For example, model inference, reconstruction and GAN attacks require black or white box access to the model of participating nodes, or access to plaintext gradient data. When these data are protected by homomorphic encryption, secret sharing and other security protocols, the attack is not easy to implement in  FATE security assumption.

When a participant or arbiter is compromised by system attack, or the participant itself is a malicious node, it may expose local model, intermediate data and overall model. In view of this hypothesis, TEE can be adopted to protect the operational environment of FATE participants. Or using differential privacy to protect the confidentiality of participants' private data, to make sure inference, reconstruction, free-riding attacks can not obtain valid private data. Or using non-technical methods, like background check, signing agreement, etc., to ensure that all participants are at least semi-honest security model. According to this compromise hypothesis, data poisoning and model poisoning may also exist. These attacks only affect model availability, but does not reveal private data. When FATE participants find that machine learning model cannot converge or the classification accuracy decreases, the learning process should be terminated in time.

The following table shows the security models and related security protocol recommendations required by different federated learning components. Among them, security model mainly refers to whether the third party node is needed as arbiter, and the role of each participant. Existing security mechanisms or security recommendations refer to security mechanisms adopted by the federated learning algorithm or recommended security protocols and other security suggestions.

| Components | Algorithm Description      |       Security Model  | Exist Security Mechanism or Security Recommendations |
| :-------------:| :-------------: | :-------------: |:-------------:|
|  Intersect  |  This module helps two and more parties to find common entry ids without leaking non-overlapping ids. [link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/intersect/)            |    No third party arbiter is needed.           |  RSA, DH, ECDH encryption algorithms are used to protect data privacy between participants, and the ID information in the intersection will be known by participants.  
| Hetero Feature Binning  | Based on quantile binning and bucket binning methods, Guest (with label) could evaluate Host's data binning quality with WOE/IV/KS. [link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/feature_binning/) | No third party arbiter is needed | Participants use Paillier Encryption to protect label information|
|Hetero Feature Selection| If iv filter is used, Hetero Feature Binning will be invoked. [link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/feature_selection/)|No third party arbiter is needed. |Iv filter's security recommendations refer to Hetero Feature Binning. |
|Hetero LR|Hetero logistic regression between host and guest.[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/logistic_regression/#heterogeneous-lr)|Third party arbiter is needed, Party A holds the label. |Arbiter needs to use Paillier Encryption and mask to protect gradients.|
|Hetero SSHELR|heterogeneous logistic regression without arbiter role.[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/logistic_regression/#heterogeneous-sshe-logistic-regression) |No third parity arbiter is needed| Use Secure Matrix Multiplication Protocol, in which uses Paillier and Secret Sharing |
|Homo LR|Homogeneous logistic regression with arbiter role [link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/logistic_regression/#homogeneous-lr)|Participants are identical, and third party arbiter is needed| Secure aggregation uses Secret Sharing Protocol, and t>n/2 (t is the minimum pieces of rebuilding secret key, n is total number of secret sharing pieces). In order to prevent server from making up fake clients，PKI could be used to provide registration for clients. Use double-masking to prevent server from obtaining client gradient data when processing offline or network delay clients.|
|Hetero LinR|Heterogeneous liner regression 纵向线性回归[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/linear_regression/#heterogeneous-linr)|Party A为guest、Party B为Host，A方拥有label，需要第三方arbiter（Honest but curious）| 梯度聚合可采用paillier同态加密、秘密分享协议。若采用paillier同态加密，为避免arbiter获取梯度中间数据，需增加附加掩码；若采用秘密分享协议，要求至少具有两个秘密分享server，且server之间不能共谋|
|Hetero Federated Poisson Regression |纵向泊松回归 [link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/poisson_regression/)|Party A为guest、Party B为Host，A方拥有label，需要第三方arbiter（Honest but curious）|梯度聚合可采用paillier同态加密、秘密分享协议。若采用paillier同态加密，为避免arbiter获取梯度中间数据，需增加附加掩码；若采用秘密分享协议，要求至少具有两个秘密分享server，且server之间不能共谋|
|Homo Neural Network|横向神经网络[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/homo_nn/)|参与方身份对等，需要第三方arbiter（honest but curious）|安全聚合采用秘密分享协议，并且t>n/2（t是指恢复密码的最小份数，n为client总数？）；为防止server虚构client数量，推荐使用PKI为client提供注册信息；采用double-mask防止server在处理掉线client或者网络延迟情况下获得client梯度数据。|
|Hetero Secure Boosting|[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/ensemble/#hetero-secureboost )|Active Party (dominating server，label y), Passive Party|由Active Party生成Paillier Encryption密钥保护gi和hi|
|Hetero NN| [link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/hetero_nn/)|Party A：data matrix provider; Party B：data matrix and label y，dominating server|由Party B生成Paillier Encryption密钥保护activations，同时Party A、B对activations添加噪声防止推理。 |
|Homo Secure Boost|[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/ensemble/#homo-secureboost)|参与方身份对等，需要第三方arbiter（honest but curious）|Clients之间协商总和可抵消归零的随机数，附加到g和h中，使server无法知悉每个client的训练中间数据|
|Hetero FTL|[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/hetero_ftl/)|Host：label; Client;不需要第三方arbiter|Host和Client同时生成Paillier Encryption密钥和掩码保护训练中间数据|
|Hetero KMeans|[link](https://fate.readthedocs.io/en/latest/zh/federatedml_component/hetero_kmeans/)|需要第三方arbiter|Clients之间协商总和可抵消归零的随机数，附加到训练中间数据中，使server无法知悉每个client的训练中间数据|



## 五. 合规性建议
FATE本身是一个开源的联邦机器学习平台，各参与方通过安全协议等交换中间数据，确保自身的数据不出域，最终达到共同计算机器学习模型的目的。当FATE架构用于生产实践中时，一些与信息安全、网络安全、数据安全相关的法律法规有助于使用者从安全、合规角度更有效的使用FATE。以下列出了使用FATE过程中可能涉及的国内外法律法规，供参考。

### 国内相关法律法规
 - 中华人民共和国数据安全法
 - 中华人民共和国个人信息保护法
 - 中华人民共和国网络安全法
 - PIA GB∕T 39335-2020 信息安全技术 个人信息安全影响评估指南

### 国际法律法规

 - GDPR-General Data Protection Regulation
 - CCPA-California Consumer Privacy Act Regulation
 - HIPAA-Health Insurance Portability and Accountability Act of 1996
 - DPIA-Data Protection Impact Assessment








