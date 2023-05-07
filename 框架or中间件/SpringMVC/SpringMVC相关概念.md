## 1. MVC
MVC是一种软件架构的思想，将软件按照模型（model）、视图（view）、控制器（controller）来划分
- M：model，模型层，指工程中的JavaBean，为处理数据而存在，存在实体类Bean与业务处理类Bean
- V：view，视图层，指工程中的页面
- C：controller，控制层，指工程中的servlet，用于接收请求和回复响应

MVC工作流程：用户通过视图层发送请求到服务器，在服务器中请求被Controller接收，Controller调用相应Model层处理请求，处理完毕将结果返回Controller，Controller再根据请求处理的结果找到相应view视图，渲染数据最终将响应给浏览器

