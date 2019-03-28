#  

## 准备工作

- 创建一个javaWeb项目

  1. new------> Dynamic Web Project
  2. Generate web.xml deployment descriptor    勾选
  3. 将解压好的的jar 包  复制到Web_INF  下的lib中
  4. 进行web.xml 文件的配置
  5. 依赖Web App Libraries

- 分包

  - src

    - cache

      ​

    - controller

    - dao

    - model

    - service

    - utils

    - webservice

  - config

    - mysql.properties
    - servlet-context.xml
    - velocity-toolbox.xml