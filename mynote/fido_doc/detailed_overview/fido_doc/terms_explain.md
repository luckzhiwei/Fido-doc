#  Fido 术语解释

  
 * [Authenticator Policy](#1)
 * [AuthenticatorIndex](#2)
 * [AuthenticationAlgorithm](#3)
 * [UserVerification](#4)
 * [Key_Proection](#5)
 * [AAID](#6)
 * [FacetID](#7)
 * [AppID](#8)
 * [UPV](#9)
 * [ASMVersion](#10)
 * [RequestType](#11)
 * [KeyHandleAccessToken](#12)
 * [CallerId](#13)
 * [PersonaId](#14)
 * [RawKeyHandle](#15)
 * [KeyHandle](#16)
 * [KeyId](#17)
 * [KeyRegistrationData (KRD)](#18)
 * [User Verification Token](#19)
 * [ASMToken](#20)
 * [PersonaId](#21)
 * [Server Challenge](#22)
 * [Sign Counter](#23)


  <h3 id="1">Authenticator Policy</h3>
   服务器的策略字段：一般由一段json字符串组成，规定了在本次操作中，可以使用哪些认证器，不可以使用哪些认证器。用于Fido Client进行认证器的筛选
   
  <h3 id="2">AuthenticatorIndex</h3>
  认证器的索引值，用于ASM寻找和定位认证器。有点类似于数组的索引值

  <h3 id="3">AuthenticationAlgorithm</h3> 认证器使用的加密算法类型：认证器在Fido流程中，会生成公钥私钥对，产生密钥对的算法就是由认证器自身所决定。
  
  <h3 id="4" >UserVerifications</h3>认证器的认证用户的方式：认证器认证用户有多种方式，有指纹，声音，脸型等各种各样的认证方式
 
  <h3 id="5">Key_Protection</h3>认证器用何种方式来安全地存储和保护私钥：比如 放入TE可信单元中。
 
  <h3 id="6">AAID</h3>一个认证器的身份证号，AAID由以下格式的数据组成:</br>V#M
   
   * V：代表认证器的供应商的信息
   * #:分割符  
   * M：代表认证器自身的模型信息
   
  <h3 id="7">FacetID</h3>由AppID获取的得到的可信任的列表信息，在android上为APK的签名数值

  <h3 id="8">AppId</h3>可以获取得到的FacetId的链接
 
  <h3 id="9">UPV</h3>代表UAF协议的最大版本号和最小版本号:在目前默认为1和0
  
  <h3 id="11">RequestType</h3>FidoClient请求ASM的操作类型:一般有获取信息，注册，认证，注销四种类型
  
     
  <h3 id="12">KeyHandleAccessToken（KHAccessToken）</h3>一个作为认证器信任ASM的依据的字段，代表认证器收到的来自ASM的信息是可信的。KeyHandleAccessToken 一般由AppId，ASMToken,CallerId,PersonId四个字段组成。
   
  <h3 id="13">CallerId</h3> 使用Fido Client的提供的API的第三方APP的Id：比如：对android系统而言：CallerId就是APP的apk的签名的。
  
  <h3 id="14">PersonaID</h3>PersonID根据Fido设备所在的外部系统环境所决定，一般而言，PersonalID代表正在使用这个操作系统的用户名

  <h3 id="15">RawKeyHandle</h3>未加密的KeyHandle：一般由:KeyHandleAccessToken,Pri_Key,UserName(一因子认证器需要加入)
  
  
  <h3 id="16">KeyHandle</h3>将RawKeyHandle用认证器的内部加密算法进行加密（官方回答为AES加密算法）后的KeyHandle。保证了密钥的安全性。

  <h3 id="17">KeyId</h3>KeyId，顾名思义就是KeyHandle的Id,其最关键的作用在于寻找KeyHandle（寻找KeyHandle的过程也就是寻找Pri_Key的过程）的关键字段。KeyId是用户在使用Fido UAF协议进行注册操作的时候由认证器生成的数据。  

  KeyId和AAID是在一个认证器上，组成了用户注册产生数据的唯一元组。而这一个数据元组，往往是Fido Server用来在认证器上关键数据查找的依据。

  <h3 id="18">KeyRegistrationData (KRD)</h3>KRD顾名思义就是在注册过程中认证器产生的关键信息，KRD说白了一群数据的集合，这些数据包括:AAID,公钥，认证器信息，挑战信息的签名，签名次数的计数器（int）等信息。这些信息都是注册过程中，认证器产生的关键信息，对之后的认证过程呢起了关键作用.

  <h3 id="19">User Verification Token</h3>认证器在识别用户身份成功后，会生成的字段信息，并且认证器会把这个信息交给ASM，用于后续操作流程。没有正确的User Verification Token，认证器是无法响应请求的。注：生成User Verification Token是由认证器自己决定的，不统一规定。


  <h3 id="20">ASMToken</h3>ASMToken顾名思义，就是ASM的身份认证数据，ASMToken其实就是一串固定的随机数，由ASM自己产生，并且由ASM自己保护。每个ASM都有属于自己的ASMToken，用来让认证器信任ASM。

  <h3 id="21">PersonaId</h3>PersonId主要指的是ASM所处的操作系统的用户帐号信息，比如Android就是使用当前系统的用户帐号。

  <h3 id="22">Server Challenge</h3>来自服务器的挑战信息。所谓的挑战信息，也就是有Fido Server在Fido UAF协议中发起的一串随机的数值，用于让客户端用私钥进行签名，之后返回服务器后，验证挑战的签名是否正确。

  <h3 id="23">Sign Counter</h3> 一个由认证器控制的自增数值，主要记录使用认证器使用私钥的次数。当然，这个数值也保存在Fido Server，用于Fido Server来检测认证器是否被克隆。