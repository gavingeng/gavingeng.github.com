---
layout: post
title: "mac安装nginx及mongo-gridfs"
date: 2012-12-15 19:08
comments: true
categories: gridfs mongo nginx
---

###1.  准备工作  
[nginx各版本下载链接页面](http://nginx.org/en/download.html "nginx download")  
    wget http://nginx.org/download/nginx-1.2.6.tar.gz  
[下载pcre](ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre)  
选取合适的版本下载，并解压  
下载nginx-gridfs以及mongo-c-driver  
    https://github.com/mdirolf/nginx-gridfs  
建议采取git clone的方式来下载  
    git clone https://github.com/mdirolf/nginx-gridfs.git  
    cd nginx-gridfs  
    git clone https://github.com/mongodb/mongo-c-driver.git  
###2.  解压pcre  
    tar -zxvf pcre-8.21.tar.gz  
    ln -s pcre-8.21 pcre  
####注:     
采用brew install pcre时，会发生在编译nginx时无法找到pcre，顾采用源码方式  
另外，openssl等可以通过brew来安装  
###3.  安装nginx  
    tar -zxvf nginx-1.2.6.tar.gz
    cd nginx-1.2.6/
    ./configure --prefix=/usr/local/lib/nginx --with-pcre=/Users/gavingeng/programs/pcre --with-http_ssl_module --add-module=/Users/gavingeng/projects/github/nginx-gridfs
    sudo make
    sudo make install
nginx的安装中，配置好pcre以及nginx-gridfs的模块路径即可.
###4. mongodb的安装请自行搜索  
略......
###5. 配置nginx-gridfs  
        location /pics/ {
            gridfs pics
            field=filename
            type=string;
            mongo 127.0.0.1:27017;
        }  
####参数释义
gridfs：nginx识别插件的关键字  
pics：db名  
[root_collection]: 选择collection，如root_collection=blog， mongod就会去找blog.files与blog.chunks两个块，默认是fs  
[field]：查询字段，保证mongdb里有这个字段名，支持_id, filename, 可省略, 默认是_id  
[type]：解释field的数据类型，支持objectid, int, string, 可省略, 默认是int  
[user]：用户名, 可省略  
[pass]：密码, 可省略  
mongo：mongodb url  
####[参考](https://github.com/mdirolf/nginx-gridfs)    https://github.com/mdirolf/nginx-gridfs  
###6. 上传图片  
    $MONGO_HOME/bin/mongofiles  --host 127.0.0.1 --port 27017 --db pics --local  $HOME/Pictures/firefox-os_b2g.png put firefox-os_b2g.png  --type png  
    $MONGO_HOME/bin/mongofiles  --host 127.0.0.1 --port 27017 --db pics --local  $HOME/Pictures/iphone_diushi.jpg put iphone_diushi.jpg  --type jpg  
登陆mongo查看图片  
    $MONGO_HOME/bin/mongo 127.0.0.1:27017/pics
    >db.fs.files.find()
    { "_id" : ObjectId("50cc4970778177bb5a959a8f"), "filename" : "firefox-os_b2g.png", "chunkSize" : 262144, "uploadDate" : ISODate("2012-12-15T09:57:04.099Z"), "md5" : "7a04fd004870e0b4c6c4c1c5b19ee076", "length" : 163386, "contentType" : "png" }
    { "_id" : ObjectId("50cc49b3e9c8a7cc617868e9"), "filename" : "iphone_diushi.jpg", "chunkSize" : 262144, "uploadDate" : ISODate("2012-12-15T09:58:11.419Z"), "md5" : "4de83971c5ae5b733da5273f6a361593", "length" : 470604, "contentType" : "jpg" }
查看更多选项mongofiles --help，其他同理  
###7. 启动nginx  
    sudo /usr/local/lib/nginx/sbin/nginx
OK,这时我们访问下我们的图片  
http://localhost/pics/firefox-os_b2g.png  
http://localhost/pics/iphone_diushi.jpg  
