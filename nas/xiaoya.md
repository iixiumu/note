# 安装

https://xiaoyaliu.notion.site/xiaoya-docker-69404af849504fa5bcf9f2dd5ecaa75f

一键安装和更新容器，标准模式，打开端口 5678<br>
bash -c "$(curl http://docker.xiaoya.pro/update_new.sh)"<br><br>

一键安装和更新容器，host模式（推荐，软路由和NAS上更少网络故障，打开端口 5678）<br>
bash -c "$(curl http://docker.xiaoya.pro/update_new.sh)" -s host<br><br>

webdav 账号密码<br>
用户: guest 密码: guest_Api789<br><br>

重启就会自动更新数据库及搜索索引文件<br>
docker restart xiaoya<br><br>

## emby全家桶

https://www.xazz.top/archives/xiaoya

# 常用命令

## 网络

docker exec -it xiaoya wget -q -O - http://docker.xiaoya.pro/version.txt

## 更新

docker restart xiaoya

## 清理

bash -c "$(curl -s https://xiaoyahelper.ddsrem.com/aliyun_clear.sh | tail -n +2)" -s 1 -tg

## 检查token

docker exec xiaoya /checktoken   xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

xxxxxxxxxxxxxxxxxxxxxxxxxx   是你的 token

## 检查opentoken

docker exec xiaoya /checkopentoken   xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

xxxxxxxxxxxxxxxxxxxxxxxxxx   是你的 open token

## 签到

docker exec xiaoya /ali_auto_checkin.sh   xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

xxxxxxxxxxxxxxxxxxxxxxxxxx   是你的 token


