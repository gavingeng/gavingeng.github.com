---
layout: post
title: "基于mongodb的GridFS文件存储"
date: 2012-12-16 12:58
comments: true
categories: mongo 
---
在mongdb中以GridFS方式来存储文件有两种方式:  
     1. 命令行方式mongofiles  
     2. 客户端驱动编程【java python】 
##1. 命令行方式mongofiles  
在$MONGO_HOME/bin目录下，有mongofiles,即可完成命令行向moongodb中插入文件数据  
###启动mongod服务  
    $MONGO_HOME/bin/mongod --port 27021 --dbpath $MONGO_HOME/data/mongod --oplogSize 100 --logpath $MONGO_HOME/data/mongod.log --logappend --fork  

    $MONGO_HOME/bin/mongofiles -host 127.0.0.1:27017 -d files put $HOME/tmp/qr.txt   

    connected to: 127.0.0.1:27017  
    added file: { _id: ObjectId('50c7f0ec4d0d1a3453e61a4c'), filename: "/Users/gavingeng/tmp/qr.txt", chunkSize: 262144, uploadDate: new Date(1355280621912), md5: "76ef15777f1760888e71589f67f03ee6", length: 525 }
    done!
想数据库files中插入qr.txt文件，ip为本机，端口为27017， put为上传,具体可参考mongofiles --help  
在熟悉mongofiles的过程中，这个上面的存储会将文件名存储时带路径，所以修改为  
    $MONGO_HOME/bin/mongofiles -host 127.0.0.1:27017 -d files --local $HOME/tmp/qr.txt put qr.txt --type=txt
###查看文件  
进入files数据库，使用db.fs.files.find()来查看GridFS文件列表  
    $MONGO_HOME/bin/mongo 127.0.0.1:27017  
    MongoDB shell version: 2.0.7  
    connecting to: 127.0.0.1:27017/test
    > show dbs;
    files     0.203125GB
    gavin     0.203125GB
    local     (empty)
    test     0.203125GB
    > use files
    switched to db files
    > show tables;
    fs.chunks
    fs.files
    system.indexes
    > db.fs.files.find()
    { "_id" : ObjectId("50c7f0ec4d0d1a3453e61a4c"), "filename" : "/Users/gavingeng/tmp/qr.txt", "chunkSize" : 262144, "uploadDate" : ISODate("2012-12-12T02:50:21.912Z"), "md5" : "76ef15777f1760888e71589f67f03ee6", "length" : 525 }
##2. java存取文件  
    package org.gavingeng.mongodb;
    import java.io.File;
    import com.mongodb.DB;
    import com.mongodb.DBCursor;
    import com.mongodb.Mongo;
    import com.mongodb.gridfs.GridFS;
    import com.mongodb.gridfs.GridFSInputFile;

    public class MongoGridFSTest {

        public static void main(String[] args) {
            init();
        }
        public static void init(){
            try {
                Mongo mongo = new Mongo("127.0.0.1",27017);
                DB db = mongo.getDB("files");
                File file = new File("/Users/gavingeng/tmp/hello.go");
                GridFS fs = new GridFS(db);
                GridFSInputFile inputFile = fs.createFile(file);
                inputFile.save();
                DBCursor cursor = fs.getFileList();
                while(cursor.hasNext()){
                    System.out.println(cursor.next());
                }
                mongo.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
###打印结果如下:  
    { "_id" : { "$oid" : "50c7f0ec4d0d1a3453e61a4c"} , "chunkSize" : 262144 , "length" : 525 , "md5" : "76ef15777f1760888e71589f67f03ee6" , "filename" : "/Users/gavingeng/tmp/qr.txt" , "contentType" :  null  , "uploadDate" : { "$date" : "2012-12-12T02:50:21.912Z"} , "aliases" :  null }
    { "_id" : { "$oid" : "50c7f5b1e3185fa892a975b3"} , "chunkSize" : 262144 , "length" : 77 , "md5" : "3f2842a811c19ab3b87278bc7726b086" , "filename" : "hello.go" , "contentType" :  null  , "uploadDate" : { "$date" : "2012-12-12T03:10:41.047Z"} , "aliases" :  null }
此时，与在命令行中查询结果一致.  
##3. python 存取文件  
    import pymongo
    import gridfs

    def main():
        con = pymongo.Connection("127.0.0.1",27017)
        db = con.files
        #db.authenticate("gavin","gavin")
        fs = gridfs.GridFS(db)
        #get all
        dbf = db.fs.files
        results = dbf.find()
        for result in results:
            print result['filename'],result['md5'],result['_id'],result['uploadDate']#,result['contentType']

        #save one
        f = open("/Users/gavingeng/Pictures/zsh.png","rb")
        x=f.read()
        retCode = fs.put(x,filename="zsh",type="png",desp="pic info")
    if __name__=='__main__':
        main()
###link:  
[MONGO API](http://api.mongodb.org/)  
[JAVA API](http://api.mongodb.org/java/2.9.3/)  
[PYTHON API](http://api.mongodb.org/python/2.3/)  

