## 安装与启动


## 备份与恢复
* 备份： mongodump -h <hostname><:port> -u username -p password -d dbname -o <path>
* 恢复： mongorestore -h <hostname><:port> -u username -p password -d dbname <path>
* windows恢复举例： ./mongorestore -h 127.0.0.1:27017 -d yapi C:\Users\path_to_folder




## python pymongo
```
import pymongo
myclient = pymongo.MongoClient('mongodb://localhost:27017/')
dblist = myclient.database_names()
yapidb = myclient["yapi"]
yapidb.authenticate('yapi-readonly','password')
mycol = yapidb["project"]

# 查询一条数据
x = mycol.find_one()
print(x)

# 查询集合中所有数据
for x in mycol.find():
  print(x)

# 查询指定字段的数据: 将要返回的字段对应值设置为 1 （cannot have a mix of inclusion and exclusion.）
for x in mycol.find({},{ "_id": 1}):
  print(x)

# 根据指定条件查询
myquery = {"name": "apollo.apollo-meta-manager"}
for x in mycol.find(myquery):
    print(x)

# 高级查询: 读取 name 字段中第一个字母 ASCII 值大于 "h" 的数据，大于的修饰符条件为 {"$gt": "h"} :
myquery = { "name": { "$gt": "h" } }
for x in mycol.find(myquery):
  print(x)

# 使用正则表达式查询: name 字段中第一个字母为 "R" 的数据，正则表达式修饰符条件为 {"$regex": "^R"} :
myquery = { "name": { "$regex": "^R" } } 
for x in mycol.find(myquery):
  print(x)

# 返回指定条数记录 
for x in mycol.find().limit(1):
  print(x)

```