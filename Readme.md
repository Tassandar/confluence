这是一个confluence 6.3.4版本的破解版，已经完成了docker化

源地址

https://github.com/ealebed/confluence

作者是一个俄罗斯小哥

https://ealebed.github.io/


想要运行confluence 6.3.4版本的破解版
你需要做的是

1.安装并运行docker环境，例如 

```
   sudo yum install docker-io 
   
   sudo  service start docker
```

然后

2. 运行你的数据库环境，(记得改密码)


```
$ docker network create confluencenet
$ docker run --name postgres -d \
    --network confluencenet \
    -e 'POSTGRES_USER=confluencedb' \
    -e 'POSTGRES_PASSWORD=password' \
    -e 'POSTGRES_ENCODING=UTF8' \
    -e 'POSTGRES_COLLATE=C' \
    -e 'POSTGRES_COLLATE_TYPE=C' \
    -e 'POSTGRES_DB=confluencedb' \
    blacklabelops/postgres
```

3. 运行本破解版confluence


```
$ docker run -d --name confluence \
	  --network confluencenet \
	  -p 80:8090  ealebed/confluence
```


4. 打开浏览器，输入对应地址   http://confluence.example.com/

5.按照步骤进行安装，在数据库选择中选择  

   - 1.生产（production)
   
   - 2.选获取测试验证码（跳到jira注册一个帐号，然后自动会填写进去）
   
   - 3.JDBC模式与数据库链接
   
   - 4.按照如下填写表格
   
```
Driver Class Name: org.postgresql.Driver
Database URL: jdbc:postgresql://postgres:5432/confluencedb
User Name: confluencedb
Password: password
``` 

6.搞定，获得期限到2031年的confluence

_______________________________________________________________


以下是俄罗斯小哥的破解方法，我并没有试过


要 破解Confluence 6.3.4，需要对两个文件进行更改 ：

```
     atlassian-extras-decoder-v2-3.2.jar
     atlassian-universal-plugin-manager-plugin-2.22.5.jar
```

文件的位置是：
```
     /opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-3.2.jar
     /opt/atlassian/confluence/confluence/WEB-INF/atlassian-bundled-plugins/atlassian-universal-plugin-manager-plugin-2.22.5.jar
```

补丁文件atlassian-extras-decoder-v2-3.2.jar：

将文件 从目录/opt/atlassian/confluence/confluence/WEB-INF/lib/ 下载到本地计算机
atlassian-extras-common-3.2.jar, atlassian-extras-decoder-api-3.2.jar, atlassian-extras-api-3.2.jar , atlassian-extras-decoder-v2-3.2.jar

安装JD-GUI工具（http://jd.benow.ca/） ：

使用JD-GUI反编译器打开atlassian-extras-decoder-v2-3.2.jar
点击File - > Save All Sources或者Ctrl + Alt + S（保存atlassian-extras-decoder-v2-3.2.jar.src.zip）
使用归档程序解压缩生成的归档文件
在atlassian-extras-decoder-v2-3.2.jar.src 的  /com/atlassian/extras/decoder/v2/Version2LicenseDecoder.java 找到loadLicenseConfiguration（在看起来像）方法：

```
     /*     */   private Properties loadLicenseConfiguration(Reader text)
     /*     */   {
     /*     */     try
     /*     */     {
     /* 218 */       Properties props = new Properties();
     /* 219 */       new DefaultPropertiesPersister().load(props, text);
     /* 220 */       return props;
     /*     */     }
     /*     */     catch (IOException e)
     /*     */     {
     /* 224 */       throw new LicenseException("Could NOT load properties from reader", e);
     /*     */     }
     /*     */   }
```

并将有关许可证的信息添加到此方法中：

```
     /*     */   private Properties loadLicenseConfiguration(Reader text)
     /*     */   {
     /*     */     try
     /*     */     {
     /* 218 */       Properties props = new Properties();
     /* 219 */       new DefaultPropertiesPersister().load(props, text);

                     props.setProperty("LicenseExpiryDate", "2031-01-01");
                     props.setProperty("MaintenanceExpiryDate", "2031-01-01");
                     props.setProperty("Evaluation", "false");
                     props.setProperty("NumberOfUsers", "-1");
                     props.setProperty("Organisation", "TerriconMembers");
                     props.setProperty("PurchaseDate", "2017-01-01");
                     props.setProperty("SEN", "SEN-L10303078");

     /* 220 */       return props;
     /*     */     }
     /*     */     catch (IOException e)
     /*     */     {
     /* 224 */       throw new LicenseException("Could NOT load properties from reader", e);
     /*     */     }
     /*     */   }
```
保存文件

复制文件atlassian-extras-common-3.2.jar, atlassian-extras-decoder-api-3.2.jar, atlassian-extras-api-3.2.jar ， commons-codec-1.9.jar源目录（atlassian-extras-decoder-v2-3.2.jar.src）

转到源目录（atlassian-extras-3.2.jar.src）
并从我们运行的java文件编译该类：
```
javac -cp commons-codec-1.9.jar:atlassian-extras-common-3.2.jar:atlassian-extras-decoder-api-3.2.jar:atlassian-extras-api-3.2.jar -sourcepath ./ com/atlassian/extras/decoder/v2/Version2LicenseDecoder.java
```

可能会有错误（由源代码反编译的“曲率”引起），消除它们并重新编译该类

在com / atlassian / extras / decoder / v2 /目录下成功运行后，会出现Version2LicenseDecoder.class文件
与更换poluchennyey文件副本（相同的路径COM / Atlassian的/演员/解码器/ V2 /）到归档Atlassian的-额外解码器 - v2-3.2.jar（要做到这一点最简单的方法就是通过MC - 午夜指挥官）
在一般情况下，“新”的文件Atlassian的-额外解码器，v2-3.2.jar在目录/ opt / Atlassian的/汇合/合流的合流服务器需要把（带替换）/ WEB-INF / lib中/删除$目录中的内容{CONFLUENCE_HOME} /插件，OSGi的缓存/个变换插件/ *和$ {CONFLUENCE_HOME} /插件，OSGi的缓存/菲利克斯/ *然后重新启动汇合，在我们的情况下，有必要根据Dockerfile说明重建泊坞窗图像
同样，修补atlassian-universal-plugin-manager-plugin-2.22.5.jar文件：

使用JD-GUI反编译器打开atlassian-universal-plugin-manager-plugin-2.22.5.jar
点击文件 - >保存所有来源或按Ctrl + Alt + S（继续atlassian-universal-plugin-manager-plugin-2.22.5.jar.src.zip存档）
使用归档程序解压缩生成的归档文件
在Atlassian的万能 - 插件管理器，插件，2.22.5.jar.src / COM / Atlassian的/演员/解码器/ V2 / Version2LicenseDecoder.java找到loadLicenseConfiguration（在我的情况看起来像）方法：
```
       private Properties loadLicenseConfiguration(Reader text)
       {
         try
         {
           Properties props = new Properties();
           new DefaultPropertiesPersister().load(props, text);
           return props;
         }
         catch (IOException e)
         {
           throw new LicenseException("Could NOT load properties from reader", e);
         }
       }
```
并将有关许可证的信息添加到此方法中

```
       private Properties loadLicenseConfiguration(Reader text)
       {
         try
         {
           Properties props = new Properties();
           new DefaultPropertiesPersister().load(props, text);
           props.setProperty("LicenseExpiryDate", "2031-01-01");
           props.setProperty("MaintenanceExpiryDate", "2031-01-01");
           props.setProperty("Evaluation", "false");
           props.setProperty("NumberOfUsers", "-1");
           return props;
         }
         catch (IOException e)
         {
           throw new LicenseException("Could NOT load properties from reader", e);
         }
       }
```

保存文件

将commons-codec-1.9.jar复制到源目录（atlassian-universal-plugin-manager-plugin-2.22.5.jar.src）

我们去的源目录（Atlassian的万能 - 插件管理器，插件，2.22.5.jar.src）和编译类，我们有权对Java的文件：
```
     javac -cp commons-codec-1.9.jar -sourcepath ./ com/atlassian/extras/decoder/v2/Version2LicenseDecoder.java
```

可能会有错误（由源代码反编译的“曲率”引起），将其清除并重新编译。
 


 могут быть ошибки (вызваны “кривостью” декомпиляции исходников), устраняем их и еще раз компилируем класс. После успешного выполнения в каталоге com/atlassian/extras/decoder/v2/ появится файл Version2LicenseDecoder.class
полученныей файл копируем с заменой (по такому же пути com/atlassian/extras/decoder/v2/) в архив atlassian-universal-plugin-manager-plugin-2.22.5.jar (проще всего это сделать через mc - Midnight Commander)
в общем случае “новый” архив atlassian-universal-plugin-manager-plugin-2.22.5.jar необходимо положить (с заменой) на сервере с Jira в каталог /opt/atlassian/jira/atlassian-jira/WEB-INF/atlassian-bundled-plugins/, удалить каталоги ${JIRA_HOME}/plugins/.bundled-plugins и ${JIRA_HOME}/plugins/.bundled-plugins/.osgi-plugins и перезапустить жиру, в нашем случае нужно пересобрать docker-образ согласно инструкциям в Dockerfile
