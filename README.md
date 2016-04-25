### disconf 的搭建
1. 环境配置
安装mysql, redis, zookeeper, nginx, tomcat, 网上很多方法

2. clone disconf
git clone https://github.com/knightliao/disconf.git

3. 修改环境变量(对bash, 修改.bash_profile)
ONLINE_CONFIG_PATH=[自定义目录]/disconf/resource
WAR_ROOT_PATH=[自定义目录]/disconf/war
export ONLINE_CONFIG_PATH
export WAR_ROOT_PATH
注意:这个目录下必须有写得权限

4. 修改配置
将配置文件移到上面配置好了的[ONLINE_CONFIG_PATH]目录下
配置文件包括:
- jdbc-mysql.properties (数据库配置)
- redis-config.properties (redis配置)
- zoo.properties (zookeeper配置)
- application.properties (应用配置）
注意:记得执行将application-demo.properties重命名成application.properties, 并将这些配置修改成自己本地的配置

5. 生成war包
cd到clone下来项目的disconf-web,然后
执行sh deploy/deploy.sh指令,如下:
cd disconf/disconf-web
sh deploy/deploy.sh
然后,会发现[WAR_ROOT_PATH]目录下会生成相应的war文件

6. 初始化数据库
按照disconf-master/disconf-web/sql/readme.txt说明按照顺序导入两个sql语句

7. 配置tomcat
安装tomcat配置server.xml配置war目录
<Context path="" docBase="WAR_ROOT_PATH目录"></Context>

8. 配置nginx
找到nginx.conf配置文件(如果是brew install nginx, 则配置文件目录在/usr/local/etc/nginx目录下)
在http这个标记对里面加上如下配置:
upstream disconf {
    server 127.0.0.1:8080;
}  
  
server {
    listen   8088;
    server_name localhost;
    access_log logs/disconf_access.log;
    error_log logs/disconf_error.log;
  
    location / {  
        root /home/disconf/disconf/war/html;  
        if ($query_string) {  
            expires max;  
        }  
    }  
  
    location ~ ^/(api|export) {  
        proxy_pass_header Server;  
        proxy_set_header Host $http_host;  
        proxy_redirect off;  
        proxy_set_header X-Real-IP $remote_addr;  
        proxy_set_header X-Scheme $scheme;  
        proxy_pass http://disconf;  
    }  
} 
注意:
1. 修改上面log记录文件,为自己定义的新建的目录文件
2. 修改root节点文件,为之前环境变量中war文件夹下的html
3. 可以修改其中的端口
4. upstream disconf 配置中的端口必须与tomcat启动的端口一致,server中listen是启动该disconf前端的端口
disconf使用前后端分离的部署方式，前端配置在nginx是那个，后端api部署在tomcat中

9. 启动tomcat,nginx
./startup.sh --- 在安装tomcat的bin目录下
./nginx ---  安装nginx的bin目录下;如果是brew安装,则目录为/usr/local/bin

通过http://localhost:8088/即可访问  默认用户:admin/admin

附录:
config      配置（配置文件或配置项）
    config_id       唯一的ID（没有啥意义，主键，自增长而已）
    type            配置文件/配置项
    name            配置文件名/配置项KeY名
    value           配置文件：文件的内容，配置项：配置值
    app_id          appid
    version         版本
    env_id          envid
    create_time     生成时间
    update_time     修改时间

app
    app_id          APPID（主键，自增长）
    name            APP名(一般是产品线+服务名)
    description     介绍
    create_time     生成时间
    update_time     修改时间
    emails          邮箱列表逗号分隔

env  (rd/qa/local可以自定义，默认为 DEFAULT_ENV)
    env_id          环境ID（主键，自增长）
    name            环境名字

user  
    user_id         用户ID（主键，自增长）
    name            姓名
    password        密码
    token           token
    ownapps         能操作的APPID,逗号分隔
    role_id         角色ID

role  
    role_id         ID（主键，自增长）
    role_name       角色名
    create_time     生成时间
    create_by       创建人
    update_time     修改时间
    update_by       更新人

role_resource  
    role_res_id     role-resource id（主键，自增长）
    role_id         用户角色id
    url_pattern     controller_requestMapping_value + method_requestMapping_value
    url_description url功能描述
    method_mask     GET, PUT, POST, DELETE, 1: accessible
    update_time     更新时间            
上面是一些表的说明,可以在env表中添加自定义的环境.

参考: 
https://github.com/knightliao/disconf/tree/master/disconf-web