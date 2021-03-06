# 项目介绍
# 一、平台简介
​    工业互联网场景下，数据采集回传存储分析管理和网页显示的系统。目的是一举统一所有子项目的接口规范，提高系统的可扩展性，尽量用松耦合的方式组织各部分功能，细化人员分工各司其职，用合理的方式保护数据的安全。
# 二、初步构架图
![image](https://github.com/sunwu51/mesh/blob/master/img/1.png)

# 三、五块功能划分
![image](https://github.com/sunwu51/mesh/blob/master/img/2.png)

# 四、流程简介

## 采集与传输部分：
设备采集数据，采集方式有很多种，取决于硬件设备。之后将数据传到网关，传输方式也有多种比如以太网、WIFI、PLC、900M工业频段网络等，这一部分的格式规定由相关人员商讨定制。网关拿到数据后通过Mqtt将数据传到服务器。此部分数据流到此结束，因为数据接入Mqtt服务器，与接收程序互不影响，即松耦合。

## 序列化与存储部分：
NodeRed订阅MqttServer的数据，获得网关上传的原始数据。然后进行序列化（字节转json字符串）。之后将序列化后的数据重新发布到Mqtt，同时将数据存到pipelinedb以及周期性存入hdfs。该部分需要Node-Red暴露Rest接口和WebSocket接口，分别用于反向配置网关、设备 和 实时传输设备数据到管理系统。管理系统通过绿3访问该Rest接口，通过紫1访问WebSocket接口。以持久化存储和Rest接口形式实现耦合。

## 数据分析部分：
pipelinedb进行流式数据存储和分析并提供Rest接口供管理系统提交参数和获取分析结果（绿1）。而Hbase Hive Kylin Spark和机器学习部分在HDFS的基础上展开，也提供Rest接口供管理系统调用。同样是Rest接口的方式耦合。

## 管理系统部分：
关联各部分的枢纽，访问前面两块获取资源，同时向网页展示数据。同时需要有用户管理，设备管理，用户权限管理等内容。面向前端以Rest接口的形式进行开发，前后分离式。

## 展示网页部分：
通过Rest和WebSocket的方式只能访问管理系统获取相应的资源，并进行展示。

# 五、数据安全
云平台只需要暴露管理系统和Mqtt的端口，其他本分全都是内部完成。Mqtt通过配置Mqtts的方式进行传输加密，配置账号密码的方式进行入网认证。而管理系统配置Https方式进行传输加密，JsonWebToken或HttpBasic方式进行权限认证。

