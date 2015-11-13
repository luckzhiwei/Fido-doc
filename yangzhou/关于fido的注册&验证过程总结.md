#关于FIDO的注册过程总结
###一、FIDO Server 向 FIDO Client 发送的UAF Registration Request
UAF Registration Request 的结构如下：

![Struct Register Request](http://i.imgur.com/8FOYnXK.png?1)

官方的协议文档给了一个例子如下所示：

	[{
		"header": {
			"upv": {
				"major": 1,
				"minor": 0
			},
			"op": "Auth",
			"appID": "https://uaf-test-1.noknoktest.com:8443/SampleApp/uaf/facets"
			"serverData": "5s7n8-7_LDAtRIKKYqbAtTTOezVKCjl2mPorYzbpxRrZ-_3wWroMXsF_pLYjNVm_l7bplAx4bkEwK6ibil9EHGfdfKOQ1q0tyEkNJFOgqdjVmLioroxgThlj8Istpt7q"
		},
		"challenge": "HQ1VkTUQC1NJDOo6OOWdxewrb9i5WthjfKIehFxpeuU",
		"policy": {
			"accepted": [
				[
					{
						"userVerification": 512,
						"keyProtection": 1,
						"tcDisplay": 1,
						"authenticationAlgorithms": [
							1
						],
						"assertionSchemes": [
							"UAFV1TLV"
						]
					}
				],
				[
					{
						"userVerification": 4,
						"keyProtection": 1,
						"tcDisplay": 1,
						"authenticationAlgorithms": [
							1
						],
						"assertionSchemes": [
							"UAFV1TLV"
						]
					}
				],
				[
					{
						"userVerification": 4,
						"keyProtection": 1,
						"tcDisplay": 1,
						"authenticationAlgorithms": [
							2
						]
					}
				],
				[
					{
						"userVerification": 2,
						"keyProtection": 4,
						"tcDisplay": 1,
						"authenticationAlgorithms": [
							2
						]
					}
				],
				[
					{
						"userVerification": 4,
						"keyProtection": 2,
						"tcDisplay": 1,
						"authenticationAlgorithms": [
							1,
							3
						]
					}
				],
				[
					{
						"userVerification": 2,
						"keyProtection": 2,
						"authenticationAlgorithms": [
							2
						]
					}
				],
				[
					{
						"userVerification": 32,
						"keyProtection": 2,
						"assertionSchemes": [
							"UAFV1TLV"
						]
					},
					{
						"userVerification": 2,
						"authenticationAlgorithms": [
							1,
							3
						],
						"assertionSchemes": [
							"UAFV1TLV"
						]
					},
					{
						"userVerification": 2,
						"authenticationAlgorithms": [
							1,
							3
						],
						"assertionSchemes": [
							"UAFV1TLV"
						]
					},
					{
						"userVerification": 4,
						"keyProtection": 1,
						"authenticationAlgorithms": [
							1,
							3
						],
						"assertionSchemes": [
							"UAFV1TLV"
						]
					}
				]
			],
			"disallowed": [
				{
					"userVerification": 512,
					"keyProtection": 16,
					"assertionSchemes": [
						"UAFV1TLV"
					]
				},
				{
					"userVerification": 256,
					"keyProtection": 16
				}
			]
		}
	 }]

上面报文中header的结构固定，详细情况见【名词解释】

Policy中的元素见【名词解释】

###二、FIDO Client解析注册请求并生成ASM Request
![Imgur](http://i.imgur.com/ZwXELAn.png?1)

下面仅对一些必要步骤做出说明：

* 1.选择版本号为1.0的报文m
* 4.根据请求报文中的策略过滤可用的认证器，特别是根据KeyIDs过滤已经注册过的认证器* 5.关于获取FacetID的算法和设置AppID的方法，详见【名词解释】中的AppID。* 6.如果可用的话，获取TLS data。这里TLS data就是FinalChall中的channelBinding（祥见【名词解释】）。ebay源码实现中将其设置为””* 7.生成FinalChallengeParams（fcp结构），并设置fcp.appID, fcp.challenge, fcp.facetID和fcp.channelBinding。
	* 计算方法：FinalChallenge = base64url(serialize(utf8encode(fcp)))
* 8.生成ASM Request报文：
	* 首先生成RegisterIn Object：
	
	![Imgur](http://i.imgur.com/kJ7oZ9w.png?1)
	* 然后再生成ASM Request：
	
	![Imgur](http://i.imgur.com/UxqDXjQ.png?1)
		* requestType包括：GetInfo, Register, Authenticate, Deregister, GetRegistrations, OpenSettings		* authenticatorIndex由GetInfo方法获得。		* 在注册过程中，args就是RegisterIn Object###三、ASM向Authenticator发送Register Command
![Imgur](http://i.imgur.com/04KspHo.png?1)

KHAccessToken: 【名词解释】VerificationToken: 【名词解释】
其余都可以从 ASM Request中获取###四、Authenticator收到Register Command之后做以下操作
![Imgur](http://i.imgur.com/beVIi6p.png?1)

第5步，生成公私钥对，其算法应该是由认证器自己metadata中的publicKeyAlgAndEncoding来指明的。
第6步，生成RawKeyHandle，见【名词解释】
第7步，Wrap RawKeyHandle，关于Key Wrap的算法，见【名词解释】
第8步，生成KRD结构：如果是second-factor roaming authenticator，则将key handle放入TAG_KEYID；否则随机生成KeyID放入TAG_KEYID（__这是我们关心的策略__）；

---
KRD是包含在整个authenticator返回的assertion中的，assertion的结构如下（KRD结构应该在签名数据前都是）
![Imgur](http://i.imgur.com/Ml1O8ZL.png?1)
![Imgur](http://i.imgur.com/0aWOWv3.png?1)
![Imgur](http://i.imgur.com/yhtBzST.png?1)
因此，这个KRD包含：AAID，ASSERTION_INFO(AuthenticatorVersion, AuthenticationMode, SignatureAlgAndEncoding, PublicKeyAlgAndEncoding)，FINAL_CHALLENGE，KEYID，COUNTERS, PUB_KEY注意,上面的TAG_ATTESTATION_CERT可能会出现多次，它是一个证书链，后出现的证书是前一个证书的颁发者。第一个一定是attestation certification，最后一个一定是attestationRootCertificates（这是一个trust anchor证书数组，见【名词解释】）中的一个。
---第9步，根据AttestationType去生成下面的签名，证书链之类。
第10步，生成TAG_AUTHENTICATOR_ASSERTION:
1. 如果是一个first-factor roaming authenticator - add KeyID and key handle into internal storage.2. 如果是一个bound authenticator - return key handle inside TAG_KEYHANDLE(在TAG_UAFV1_REGISTER_CMD_RESPONSE - 这是个authenticator最终返回给ASM的报文)
第11步，生成authenticator最终返回給ASM的报文TAG_UAFV1_REGISTER_CMD_RESPONSE：![Imgur](http://i.imgur.com/OYyMqed.png?1)注意最后关于KeyID有这么一句：
>If the KeyID is generated randomly(instead of, for example, being derived from a key handle) - it should be stored inside RawKeyHandle so that it can be accessed by the authenticator while processing the Sign command.__这里是这么解释__，RawKeyHandle的结构一般只包括：KHAccessToken, UAuth.priv, Username Size, Username，但是实际上，RawKeyHandle的结构是跟authenticator的具体实现有关，因此是可以放入KeyID的。（__这里随机生成KeyID并放入KeyID也是我们所关心的策略，将在之后的验证过程中用到KeyID__）
###五、生成UAF Client的Response报文
![Imgur](http://i.imgur.com/Y756hC4.png?1)
注意，其中只是将authenticator的返回报文中的ASSERTION部分copy了出来，因此UAF Client的Response报文并没有包含TAG_KEYHANDLE，因此KEYHANDLE也并没有被传递到服务器，而是被保存在了ASM中。
一个例子如下：

![Imgur](http://i.imgur.com/KIpumqW.png?1)

###六、FIDO Server解析UAF Client发回来的response
![Imgur](http://i.imgur.com/bhHEN8D.png?1)
![Imgur](http://i.imgur.com/jLxDtTl.png?1)
![Imgur](http://i.imgur.com/xYEc1ZG.png?1)
前面的步骤就是一堆验证，重点在于对attestation certificate的验证。最后Server存储PublicKey, KeyID, SignCounter, AuthenticatorVersion, AAID和tcDisplayPNGCharacteristic到一个record，这个record用AAID和KeyID标识。#关于FIDO的验证过程总结###一、 FIDO Server向FIDO Client发送的UAF Authentication Request除了op变成"Auth"，其余跟Registration一样。###二、 FIDO Client给ASM生成的ASMRequest![Imgur](http://i.imgur.com/EfTRQjJ.png?1)除了第8步，其余跟Registration差不多。关于这个Authenticator Interface Version是什么还没搞清。###三、 ASM向authenticator发送的Sign Command在ASM API文档中：![](http://i.imgur.com/fKqkbjf.png)![](http://i.imgur.com/ok3bEKx.png)注意，在第7步中，ASM根据AuthenticateIn.keyID和AuthenticateIn.appID来筛选了KeyHandles，填充到了下面的TAG_UAFV1_SIGN_CMD中。之后，认证器根据AccessToken过滤后如果还是有多个KeyHandle，则会到这里的第10步，让用户去选择一个。![Imgur](http://i.imgur.com/rOSUYDl.png?1)ASM怎么确定`TAG_AUTHENTICATOR_INDEX`的值？  其中的`TAG_KEYHANDLE`被允许多次出现，这是为什么？###四、 authenticator生成TAG_UAFV1_SIGN_CMD_RESPONSE发给ASM`TAG_UAF_AUTH_ASSERTION`的结构如下：![](http://i.imgur.com/4Ulr2eP.png)![](http://i.imgur.com/4VPJNHZ.png)![Imgur](http://i.imgur.com/SMz3xBl.png?1)![Imgur](http://i.imgur.com/3Z2NpFc.png?1)前面跟registration差不多。  第7步，如果通过`KHAccessToken`筛选后还有多个`RawKeyHandle`，那么就copy所有剩下的`RawKeyHandle`的`{Command.KeyHandle, RawKeyHandle.username}`到`TAG_USERNAME_AND_KEYHANDLE`中，然后copy这个`TAG_USERNAME_AND_KEYHANDLE`到`TAG_UAFV1_SIGN_CMD_RESPONSE`并返回。  第8步，如果`RawKeyHandle`只剩1个，创建`TAG_UAFV1_SIGNED_DATA`并且设置`TAG_UAFV1_SIGNED_DATA.AuthenticationMode`为`0x01`创建`TAG_UAFV1_AUTH_ASSERTION`，