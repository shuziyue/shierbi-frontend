# 智能BI平台

> 作者：[猫十二懿](https://github.com/kongshier)

## 项目介绍
本项目是基于React+Spring Boot+RabbitMQ+AIGC的智能BI数据分析平台。

访问地址：http://bi.kongshier.top

来自[编程导航](https://yupi.icu/)

> AIGC ：Artificial Intelligence Generation Content(AI 生成内容)

区别于传统的BI，数据分析者只需要导入最原始的数据集，输入想要进行分析的目标，就能利用AI自动生成一个符合要求的图表以及分析结论。此外，还会有图表管理、异步生成、AI对话等功能。只需输入分析目标、原始数据和原始问题，利用AI就能一键生成可视化图表、分析结论和问题解答，大幅降低人工数据分析成本。

**优势：** 让不会数据分析的用户也可以通过输入目标快速完成数据分析，大幅节约人力成本，将会用到 AI 接口生成分析结果

## 项目架构图
### 基础架构
基础架构：客户端输入分析诉求和原始数据，向业务后端发送请求。业务后端利用AI服务处理客户端数据，保持到数据库，并生成图表。处理后的数据由业务后端发送给AI服务，AI服务生成结果并返回给后端，最终将结果返回给客户端展示。

![](https://user-images.githubusercontent.com/94662685/248857523-deff2de3-c370-4a9a-9628-723ace5ab4b3.png)
### 优化项目架构-异步化处理
优化流程（异步化）：客户端输入分析诉求和原始数据，向业务后端发送请求。业务后端将请求事件放入消息队列，并为客户端生成取餐号，让要生成图表的客户端去排队，消息队列根据I服务负载情况，定期检查进度，如果AI服务还能处理更多的图表生成请求，就向任务处理模块发送消息。

任务处理模块调用AI服务处理客户端数据，AI 服务异步生成结果返回给后端并保存到数据库，当后端的AI工服务生成完毕后，可以通过向前端发送通知的方式，或者通过业务后端监控数据库中图表生成服务的状态，来确定生成结果是否可用。若生成结果可用，前端即可获取并处理相应的数据，最终将结果返回给客户端展示。在此期间，用户可以去做自己的事情。
![image](https://user-images.githubusercontent.com/94662685/248858431-6dbf41e0-adfe-40cf-94da-f3db6c73b69d.png)


## 项目运行
### Install `node_modules`:
```bash 
npm install 
```
OR
```bash 
yarn
```
### 启动项目
```bash 
npm start
```
### 构建项目
```bash 
npm run dev
```


## 项目技术栈
### 前端
1. React 18
2. Umi 4 前端框架
3. Ant Design Pro 5.x 脚手架
4. Ant Design 组件库 
5. OpenAPI 代码生成：自动生成后端调用代码（来自鱼聪明开发平台）
6. EChart 图表生成


### 后端
1. Java Spring Boot
2. MySQL数据库
3. Redis：Redisson限流控制
4. MyBatis-Plus 数据库访问结构 
5. IDEA插件 MyBatisX ： 根据数据库表自动生成
6. **RabbitMQ：消息队列**
7. AI SDK：AI接口开发
8. JDK 线程池及异步化
9. Easy Excel：表格数据处理
10. Swagger + Knife4j 项目文档
11. Hutool 、Apache Common Utils等工具库

## 项目功能
1. 用户登录
2. 智能分析（同步）。调用AI根据用户上传csv文件生成对应的 JSON 数据，并使用 ECharts图表 将分析结果可视化展示
3. 智能分析（异步）。使用了线程池异步生成图表，最后将线程池改造成使用 RabbitMQ消息队列 保证消息的可靠性，实现消息重试机制
4. 用户限流。本项目使用到令牌桶限流算法，使用Redisson实现简单且高效分布式限流，限制用户每秒只能调用一次数据分析接口，防止用户恶意占用系统资源
5. 调用AI进行数据分析，并控制AI的输出
6. 由于AIGC的输入 Token 限制，使用 Easy Excel 解析用户上传的 XLSX 表格数据文件并压缩为CSV，实测提高了20%的单次输入数据量、并节约了成本。
7. 后端自定义 Prompt 预设模板并封装用户输入的数据和分析诉求，通过对接 AIGC 接口生成可视化图表 JSON 配置和分析结论，返回给前端渲染。
### 新增功能
1. 新增用户注册功能
2. 新增用户调用次数表，用户注册同时完成插入次数表，当使用AI生成接口时分布式锁对使用次数锁定，次数扣减才释放锁
3. 新增删除图表/对话结果功能
4. 新增死信队列，将处理图表的队列绑定到死信队列上，保证消息可靠性，若分析失败，则进入死信，将补偿用户次数
5. 添加支付宝沙箱支付功能，下单充值AI使用次数完成，使用沙箱进行支付，完成支付功能，支付完成得到对应的使用次数
6. 新增延迟队列，当下单完成10分钟，还没有支付订单，则会标记为超时订单
7. 新增用户上传头像功能，使用阿里云对象存储OSS存储图片
8. 修改前端登录/注册界面
9. 优化前端显示，效果如下展示
10. 用户查看原始数据
11. 新增AI对话，用户提交问题，AI分析解答

## BI项目展示
### 用户登录注册
![用户登录注册](./images/README-1689391285070.png)
![注册](./images/README-1689391342301.png)

### 项目首页
![image](https://user-images.githubusercontent.com/94662685/248863020-67959893-4dce-4ea8-bcb1-8482b8e0c094.png)

### 同步分析数据生成图表
![image](https://user-images.githubusercontent.com/94662685/248860079-33574351-bbe4-4cb2-934a-3ffdcbb4cbf1.png)
![](https://user-images.githubusercontent.com/94662685/248860034-3141343f-8a4e-4c1d-8236-382da9abe8b4.png)

### 异步分析数据生成图表
![image](https://user-images.githubusercontent.com/94662685/248860258-2aa7c6ee-fe50-4cb1-982b-72fee4c1f526.png)

### 图表管理界面
![图表管理](./images/README-1689391411165.png)
### 查看图表原始数据
![图表数据](./images/README-1689391475075.png)

### AI 问答助手
![image](https://user-images.githubusercontent.com/94662685/248860583-e3aad1e7-ed4e-4506-aa69-e8b3f1d82046.png)

### AI 对话管理
![image](https://user-images.githubusercontent.com/94662685/248860609-8455b515-7f36-46cc-b512-707ad62d5f9a.png)

### 个人中心
个人信息，修改信息，点击头像修改头像
![个人信息](./images/README-1689391569451.png)

个人订单信息
![订单信息](./images/README-1689391631136.png)

付款订单
![付款订单](./images/README-1689391660775.png)

修改购买数量
![修改购买数量](./images/README-1689392309643.png)

订单交付结果信息查询
![订单交付结果信息查询](./images/README-1689391753285.png)

### 管理员功能
管理员介绍
![管理员介绍](./images/README-1689391825457.png)

用户管理
![用户管理](./images/README-1689399376185.png)

修改用户信息
![修改用户信息](./images/README-1689391925390.png)

删除用户
![删除用户](./images/README-1689392175260.png)

新增用户
![新增用户](./images/README-1689391878030.png)

图表管理
查看所有用户的图表信息，查看图表数据、删除图表
![管理员图表管理](./images/README-1689392222409.png)

订单管理、支付管理
![订单支付](./images/README-1689399466276.png)
![支付](./images/README-1689399487646.png)

查询交易信息
![交易信息](./images/README-1689399518474.png)
