# Fido架构阐述

  1.  ##[1.总体架构](#1.1)
       1. [1.1 总体架构](#1.1)
       2. [1.2 总体流程阐述](#1.2)
  2.  ##2.fido sever
  
  3. ## 3.fido Client
      1.  3.1 fido client 概述
      1.  3.2 fido Client 同服务器交互
      2.  3.3 fido Client 同ASM 交互
             
  4.  ## 4.fido client ASM
     1. fido client ASM 概述   
     1. fido client ASM 同fido Client交互
     2. fido client ASM 同 fido 认证器交互
     
  5. ##5. 认证器
     1. fido client 认证器概述
     2. fido client 认证器同 fido client ASM 的交互
     3. 认证器返回的信息同服务器的关联
  
  </br></br>

  <h2>1总体架构</h2>
   <h3 id="1.1">1.1总体架构</h3>fido协议基于传统的client-sever结构，如下图所示从服务器从服务器端分为至上而下分为WebSever，FidoServer，FidometeDataSever，客户端至上而下分为Fido Client，ASM，Fido认证器。![](2.1.1.png)</br>接下来依次解释每一层的作用：
   
   * WebSever：一个传统的web服务器层，主要进行负责对监听client端的请求，并将数据交给fido Sever，同时会将fido Sever的数据返沪给client端，封装了fido sever，使之成为一个web服务器的形式
   * FidoServer：一个fido服务器，主要负责对客户端的产生的各种用户数据和安全信息做逻辑处理操作，并做数据存储；同时关联fidometaSever，做安全性认证。
   * FidoMetaDataSever：一个fido官方的安全性认证的服务器，比如验证fido认证器的类型；认证fido认证器包含的证书是否正确。
   * FidoClient:Fido 客户端，将整个fido包装成为一个可以调用的API，提供接口让给（浏览器/android/iOS）应用调用，同时处理服务器的发来的数据，和ASM进行数据传递.
   * ASM:同认证器协同交互的软件层，主要做统一各种各样的认证器在软件层次上的统一，同时将和认证器生成的数据做存储和查询。
   * 认证器：验证用户的身份，生成fido认证最关键的公钥和私钥，并且通过私钥来对服务器的挑战进行签名，服务器认证用户的身份，是fido协议最底层的一个层次，也是实现fido协议最关键的一个层次。
   
 <h3 id="1.2">1.2总体流程阐述</h3> 
1.  fido获取信息的过程：
   * ASM在注册，认证，注销的操作之前，都要获取和ASM关联的认证器的信息，认证器的信息大致包括:authenticatorIndex; isUserEnrolled;aaid;assertionScheme; authenticationAlgorithm;userVerification等信息，用于在之后的操作中使用。
    
1.  fido注册的过程：（这里只针对一因子的认证器）
   </br>fido注册的基本流程大致阐述如下：    
     * 首先，客户端发送注册信息给fido Sever
     * fido Sever收到客户端的请求之后，会在生成一系列的注册有关请求数据
     * fido client拿到服务器的认证数据后，形成最终的挑战信息，并且将数据再次往ASM传递
     * ASM收到信息之后，命令认证器认证用户身份，如果认证通过，根据认证器索引去查找对应的认证器，使用认证器的摘要算法来生成服务器挑战的摘要，并且生成KHAccessToken的数据
     * 认证器收到ASM的信息后，生成公私钥对，同时生成KeyHandle和KeyId，同时生成其他信息（比如证书信息），并用私钥进行对挑战签名，然后将KeyHandle（加密后）和KeyId，公钥信息返回给ASM
     * ASM拿到认证器的信息之后，存储KeyId和KeyHandle，然后将认证器返回的信息加以封装返回给fido client
     * fido client 再次为信息进行封装处理，加入Header信息等，然后将信息发送给fido服务器
     * fido服务器收到信息后，首先验证证书有效性，组织签名数据，使用对应的公钥做验签操作，验签成功后，告诉客户端注册成功。
2.  fido认证的过程：
      * 首先，客户端发送认证信息给fido Sever
     * fido Sever收到客户端的请求之后，会在生成一系列的认证有关请求数据（包括KeyID）
     * fido client拿到服务器的认证数据后，形成最终的挑战信息，并且将数据再次往ASM传递
     * ASM收到信息之后，命令认证器认证用户身份，如果认证通过，根据认证器索引去查找对应的认证器，并且用认证器的摘要算法生成挑战的摘要信息，然后根据服务器发送的KeyID找到对应的KeyHanlde，并且生成KHAccessToken的数据，将数据发送给认证器
     * 认证器收到ASM的信息后，通过KHAccessToken 查找可以信任的KeyHandle，然后解密KeyHandle，用KeyHandle中的私钥去对挑战信息的摘要进行签名，然后其他数据返回给ASM。
     * ASM拿到认证器的信息之后，再次做封装，交给Fido Client。
     * fido client 再次为信息进行封装处理，加入Header信息等，然后将信息发送给fido服务器。
     * fido服务器收到信息后，织签名数据，使用对应的公钥做验签操作，若验签成功，则告诉客户端验证过程成功。
3.  fido注销的过程：
   暂时没有看
   
     