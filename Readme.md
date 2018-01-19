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

   1.生产（production)
   2.选获取测试验证码（跳到jira注册一个帐号，然后自动会填写进去）
   3.JDBC模式与数据库链接
   4.填写 表格
   
```
Driver Class Name: org.postgresql.Driver
Database URL: jdbc:postgresql://postgres:5432/confluencedb
User Name: confluencedb
Password: password
``` 

6.done

_______________________________________________________________

Для "лицензирования" Confluence 6.3.4 необходимо внести изменения в два файла (приведены актуальные версии файлов на момент установки):
   
```
     atlassian-extras-decoder-v2-3.2.jar
     atlassian-universal-plugin-manager-plugin-2.22.5.jar
``` 

Месторасположение файлов соответственно:
   
```
     /opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-3.2.jar
     /opt/atlassian/confluence/confluence/WEB-INF/atlassian-bundled-plugins/atlassian-universal-plugin-manager-plugin-2.22.5.jar
```

Патч файла atlassian-extras-decoder-v2-3.2.jar:

- скачиваем на локальный компьютер файлы atlassian-extras-common-3.2.jar, atlassian-extras-decoder-api-3.2.jar, atlassian-extras-api-3.2.jar и atlassian-extras-decoder-v2-3.2.jar из каталога /opt/atlassian/confluence/confluence/WEB-INF/lib/
- устанавливаем инструмент JD-GUI (http://jd.benow.ca/). Далее:

   - открываем atlassian-extras-decoder-v2-3.2.jar с помощью декомпилятора JD-GUI
   - жмем File -> Save All Sources или Ctrl+Alt+S (сохранится архив atlassian-extras-decoder-v2-3.2.jar.src.zip)
   - распаковываем полученный архив с помощью архиватора
   - в atlassian-extras-decoder-v2-3.2.jar.src/com/atlassian/extras/decoder/v2/Version2LicenseDecoder.java находим метод loadLicenseConfiguration (в моем случае выглядел так):

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

   - и добавляем в данный метод информацию о лицензии:

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

   - сохраняем файл
   - копируем файлы atlassian-extras-common-3.2.jar, atlassian-extras-decoder-api-3.2.jar, atlassian-extras-api-3.2.jar и commons-codec-1.9.jar в директорию с исходниками (atlassian-extras-decoder-v2-3.2.jar.src)
   - переходим в каталог с исходниками (atlassian-extras-3.2.jar.src) и компилируем класс из java-файла, который мы правили:

```
     javac -cp commons-codec-1.9.jar:atlassian-extras-common-3.2.jar:atlassian-extras-decoder-api-3.2.jar:atlassian-extras-api-3.2.jar -sourcepath ./ com/atlassian/extras/decoder/v2/Version2LicenseDecoder.java
```
   - могут быть ошибки (вызваны “кривостью” декомпиляции исходников), устраняем их и еще раз компилируем класс
   - после успешного выполнения в каталоге com/atlassian/extras/decoder/v2/ появится файл Version2LicenseDecoder.class
   - полученныей файл копируем с заменой (по такому же пути com/atlassian/extras/decoder/v2/) в архив atlassian-extras-decoder-v2-3.2.jar (проще всего это сделать через mc - Midnight Commander)
   - в общем случае “новый” архив atlassian-extras-decoder-v2-3.2.jar необходимо положить (с заменой) на сервере с Confluence в каталог /opt/atlassian/confluence/confluence/WEB-INF/lib/, удалить содержимое каталогов ${CONFLUENCE_HOME}/plugins-osgi-cache/transformed-plugins/* и ${CONFLUENCE_HOME}/plugins-osgi-cache/felix/* после чего перезапустить confluence, в нашем случае нужно пересобрать docker-образ согласно инструкциям в Dockerfile
   
      
Аналогичным образом патчим файл atlassian-universal-plugin-manager-plugin-2.22.5.jar:

   - открываем atlassian-universal-plugin-manager-plugin-2.22.5.jar с помощью декомпилятора JD-GUI
   - жмем File -> Save All Sources или Ctrl+Alt+S (сохранится архив atlassian-universal-plugin-manager-plugin-2.22.5.jar.src.zip)
   - распаковываем полученный архив с помощью архиватора
   - в atlassian-universal-plugin-manager-plugin-2.22.5.jar.src/com/atlassian/extras/decoder/v2/Version2LicenseDecoder.java находим метод loadLicenseConfiguration (в моем случае выглядел так):
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
       
   - и добавляем в данный метод информацию о лицензии:
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
   - сохраняем файл
   - копируем commons-codec-1.9.jar в директорию с исходниками (atlassian-universal-plugin-manager-plugin-2.22.5.jar.src)

   - переходим в каталог с исходниками (atlassian-universal-plugin-manager-plugin-2.22.5.jar.src) и компилируем класс из java-файла, который мы правили:
```
     javac -cp commons-codec-1.9.jar -sourcepath ./ com/atlassian/extras/decoder/v2/Version2LicenseDecoder.java
```     
   - могут быть ошибки (вызваны “кривостью” декомпиляции исходников), устраняем их и еще раз компилируем класс. После успешного выполнения в каталоге com/atlassian/extras/decoder/v2/ появится файл Version2LicenseDecoder.class
   - полученныей файл копируем с заменой (по такому же пути com/atlassian/extras/decoder/v2/) в архив atlassian-universal-plugin-manager-plugin-2.22.5.jar (проще всего это сделать через mc - Midnight Commander)
   - в общем случае “новый” архив atlassian-universal-plugin-manager-plugin-2.22.5.jar необходимо положить (с заменой) на сервере с Jira в каталог /opt/atlassian/jira/atlassian-jira/WEB-INF/atlassian-bundled-plugins/, удалить каталоги ${JIRA_HOME}/plugins/.bundled-plugins и ${JIRA_HOME}/plugins/.bundled-plugins/.osgi-plugins и перезапустить жиру, в нашем случае нужно пересобрать docker-образ согласно инструкциям в Dockerfile
