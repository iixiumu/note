在行业内，目前对于 Serverless 有几种解读方法：

在某些场景可以解读为一种软件系统架构方法，通常称为 Serverless 架构
而在另一些情况下，又可以代表一种产品形态，称为 Serverless 产品

在说起 Serverless 架构时，Serverless 代表的是利用 Serverless 形态的产品实现的应用架构，这种架构完全依托于云厂商或云平台提供产品完成系统的组织及构建。在这种架构中，用户无需关注支撑应用服务运行的主机，而将关注点投入在系统架构，业务开发，业务支撑运维上。

而说起 Serverless 产品时，代表的是无需理解、管理服务器，按需使用，按使用付费的产品。Serverless 产品中，其实也可以包含存储、计算等多种类型的产品。而典型的计算产品，就是云函数这种形态。

云函数，或者称为函数即服务 (Function as a Service)，它和后端即服务 (Backend as a Service) 一起，都可以称为 Serverless 产品。通过组合使用这些产品，开发者可以构建自身的业务 Serverless 架构。

Serverless应用可以细分为BaaS和FaaS两类，

BaaS: Backend as a Service，这里的Backend可以指代任何第三方提供的应用和服务，比如提供云数据库服务的Firebase和Parse，提供统一用户身份验证服务的Auth0和Amazon Cognito等。
FaaS: Functions as a Service，应用以函数的形式存在，并由第三方云平台托管运行，比如之前提到的AWS Lambda，Google Cloud Functions等。