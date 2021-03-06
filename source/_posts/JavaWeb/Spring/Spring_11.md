---
title: SpringInAction读书笔记(十一) 远程调用 
date: 2017-03-09 23:51:58
tags: [JavaWeb,Spring]
---

Spring 可以与RMI、Hessian、Burlap、HTTP invoker 等远程调用整合，但是都太麻烦了，在实际中应用很少，本文主要介绍如何使用Spring与WebService整合实现远程调用。
几个名词:

- SOAP:简单对象访问协议
- WSDL：WebService描述语言

<!--more-->

## JaxWs配置

```java
package data_persistent.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.remoting.jaxws.SimpleJaxWsServiceExporter;

/**
 * JAX-WS配置
 * @author NikoBelic
 * @create 27/01/2017 16:33
 */
@Configuration
public class RMIConfig
{
    @Bean
    public SimpleJaxWsServiceExporter jaxWsServiceExporter()
    {
        System.out.println("RMI配置文件初始化");
        SimpleJaxWsServiceExporter exporter = new SimpleJaxWsServiceExporter();
        exporter.setBaseAddress("http://localhost:8088/services/");
        return exporter;
    }
}

```


## WebService层


```java
package data_persistent.webservice;

import data_persistent.dao.UserDao;
import data_persistent.model.UserObj;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.jws.WebMethod;
import javax.jws.WebService;

/**
 * @author NikoBelic
 * @create 25/01/2017 22:44
 */
@Component
@WebService(serviceName = "userService")
public class UserWebService
{
    @Autowired
    private UserDao userDao;

    //@WebMethod
    public UserObj addUser(UserObj userObj)
    {
        return userDao.addUser(userObj);
    }
    //@WebMethod
    public UserObj findUserById(Integer id)
    {
        return userDao.findUserById(id);
    }
    //@WebMethod
    public void updateUserById(Integer id, UserObj userObj)
    {
        userDao.updateUserById(id, userObj);
    }
    //@WebMethod
    public void deleteUserById(Integer id)
    {
        userDao.deleteUserById(id);
    }
}

```

## 发布结果
访问`http://localhost:8088/services/userService?wsdl`

```xml
This XML file does not appear to have any style information associated with it. The document tree is shown below.
<!--
 Published by JAX-WS RI (http://jax-ws.java.net). RI's version is JAX-WS RI 2.2.9-b130926.1035 svn-revision#5f6196f2b90e9460065a4c2f4e30e065b245e51e. 
-->
<!--
 Generated by JAX-WS RI (http://jax-ws.java.net). RI's version is JAX-WS RI 2.2.9-b130926.1035 svn-revision#5f6196f2b90e9460065a4c2f4e30e065b245e51e. 
-->
<definitions xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd" xmlns:wsp="http://www.w3.org/ns/ws-policy" xmlns:wsp1_2="http://schemas.xmlsoap.org/ws/2004/09/policy" xmlns:wsam="http://www.w3.org/2007/05/addressing/metadata" xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/" xmlns:tns="http://webservice.data_persistent/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://schemas.xmlsoap.org/wsdl/" targetNamespace="http://webservice.data_persistent/" name="userService">
<types>
<xsd:schema>
<xsd:import namespace="http://webservice.data_persistent/" schemaLocation="http://localhost:8088/services/userService?xsd=1"/>
</xsd:schema>
</types>
<message name="addUser">
<part name="parameters" element="tns:addUser"/>
</message>
<message name="addUserResponse">
<part name="parameters" element="tns:addUserResponse"/>
</message>
<message name="findUserById">
<part name="parameters" element="tns:findUserById"/>
</message>
<message name="findUserByIdResponse">
<part name="parameters" element="tns:findUserByIdResponse"/>
</message>
<message name="updateUserById">
<part name="parameters" element="tns:updateUserById"/>
</message>
<message name="updateUserByIdResponse">
<part name="parameters" element="tns:updateUserByIdResponse"/>
</message>
<message name="deleteUserById">
<part name="parameters" element="tns:deleteUserById"/>
</message>
<message name="deleteUserByIdResponse">
<part name="parameters" element="tns:deleteUserByIdResponse"/>
</message>
<portType name="UserWebService">
<operation name="addUser">
<input wsam:Action="http://webservice.data_persistent/UserWebService/addUserRequest" message="tns:addUser"/
<output wsam:Action="http://webservice.data_persistent/UserWebService/addUserResponse" message="tns:addUserResponse"/>
</operation>
<operation name="findUserById">
<input wsam:Action="http://webservice.data_persistent/UserWebService/findUserByIdRequest" message="tns:findUserById"/>
<output wsam:Action="http://webservice.data_persistent/UserWebService/findUserByIdResponse" message="tns:findUserByIdResponse"/>
</operation>
<operation name="updateUserById">
<input wsam:Action="http://webservice.data_persistent/UserWebService/updateUserByIdRequest" message="tns:updateUserById"/>
<output wsam:Action="http://webservice.data_persistent/UserWebService/updateUserByIdResponse" message="tns:updateUserByIdResponse"/>
</operation>
<operation name="deleteUserById">
<input wsam:Action="http://webservice.data_persistent/UserWebService/deleteUserByIdRequest" message="tns:deleteUserById"/>
<output wsam:Action="http://webservice.data_persistent/UserWebService/deleteUserByIdResponse" message="tns:deleteUserByIdResponse"/>
</operation>
</portType>
<binding name="UserWebServicePortBinding" type="tns:UserWebService">
<soap:binding transport="http://schemas.xmlsoap.org/soap/http" style="document"/>
<operation name="addUser">
<soap:operation soapAction=""/>
<input>
<soap:body use="literal"/>
</input>
<output>
<soap:body use="literal"/>
</output>
</operation>
<operation name="findUserById">
<soap:operation soapAction=""/>
<input>
<soap:body use="literal"/>
</input>
<output>
<soap:body use="literal"/>
</output>
</operation>
<operation name="updateUserById">
<soap:operation soapAction=""/>
<input>
<soap:body use="literal"/>
</input>
<output>
<soap:body use="literal"/>
</output>
</operation>
<operation name="deleteUserById">
<soap:operation soapAction=""/>
<input>
<soap:body use="literal"/>
</input>
<output>
<soap:body use="literal"/>
</output>
</operation>
</binding>
<service name="userService">
<port name="UserWebServicePort" binding="tns:UserWebServicePortBinding">
<soap:address location="http://localhost:8088/services/userService"/>
</port>
</service>
</definitions>
```

