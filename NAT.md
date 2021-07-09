





```
spring:
  profiles:
    active: dev
mybatis:
  mapper-locations: classpath*:com/itcrazy/alanmall/pay/dal/mapper/*Mapper.xml"
  type-aliases-package: com.itcrazy.alanmall.pay.dal.entity

#支付配置
payconfig:
  weixin:
    #应用ID
    wxappId: wx9f1fa58451efa9b2
    #商户ID号
    wxmchID: 1576040561
    # 默认微信商户证书base64密文
    wzappsecret: wzappsecret
    # 验签key
    wxmchsecret: signature
    #秘钥
    wxkey: QS8rrOISuu8LojP1OFd8xmswB7TQCfI1
    #默认回调地址
    wxnotifyUrl: http://mathyoung.eicp.top:38988/pay/wechatPayNotify

    #证书存储路径
    wxcertPath: D:/Projectforming/workspace/alanmall/pay-service/pay-provider/src/main/resources/apiclient_cert.p12


    #支付安全校验(验签)
  aes:
    #AES加密秘钥
    skey: ab2cc473d3334c39
    #验签盐
    salt: XPYQZb1kMES8HNaJWW8+TDu/4JdBK4owsU9eXCXZDOI=
```

![image-20210224195828396](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/image-20210224195828396.png)





![image-20210224200401387](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/image-20210224200401387.png)







A1B41F6D66D2832CEE6EB48ED74CCDB8

5B69A64E5C90EB47F4292CD8095CA373





0CB21735652705B4D0EEB487D282407C8926A60E1BF0FB61190FBB1DDD0445DE