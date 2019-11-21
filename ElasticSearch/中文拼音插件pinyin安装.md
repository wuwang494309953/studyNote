### [elasticsearch-analysis-pinyin](<https://github.com/medcl/elasticsearch-analysis-pinyin>)

#### 安装方式

1. 类似ik安装费方式，使用**Elasticsearch**安装目录**bin**下边的工具**elasticsearch-plugin**

   > ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-pinyin/releases/download/v7.0.1/elasticsearch-analysis-pinyin-7.0.1.zip

2. 重启**Elasticsearch**



#### 测试使用

在**kibana**中测试，这里因为前边有配置过中文分词，所以这里就中文分词和拼音一起配置了

1. 创建index和mapping

   ```
   //创建一个index，并改变一些设置
   PUT /medcl/
   {
       "settings" : {
           "analysis" : {
               "analyzer" : {
                   "pinyin_analyzer" : {
                       "tokenizer" : "my_pinyin"
                       }
               },
               "tokenizer" : {
                   "my_pinyin" : {
                       "type" : "pinyin",
                       "keep_separate_first_letter" : false,
                       "keep_full_pinyin" : true,
                       "keep_original" : true,
                       "limit_first_letter_length" : 16,
                       "lowercase" : true,
                       "remove_duplicated_term" : true
                   }
               }
           }
       }
   }
   
   //创建mapping，这里前边和ik一样，后边fields是pinyin的设置
   PUT /medcl/_mapping
   {
     "properties": {
         "content": {
             "type": "text",
             "analyzer": "ik_max_word",
             "search_analyzer": "ik_smart",
             "fields": {
                 "pinyin": {
                     "type": "text",
                     "store": false,
                     "term_vector": "with_offsets",
                     "analyzer": "pinyin_analyzer",
                     "boost": 10
                 }
             }
         }
     }
   }
   ```

2. 添加几条测试数据

   ```
   POST /medcl/_create/1
   {"content":"美国留给伊拉克的是个烂摊子吗"}
   
   POST /medcl/_create/2
   {"content":"公安部：各地校车将享最高路权"}
   
   POST /medcl/_create/3
   {"content":"中韩渔警冲突调查：韩警平均每天扣1艘中国渔船"}
   
   POST /medcl/_create/4
   {"content":"中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首"}
   ```

3. 查询测试

   ```
   //指令
   GET /medcl/_search
   {
     "query": {
       "query_string": {
         "default_field": "content.pinyin",
         "query": "yuchuan"
       }
     }
   }
   //返回值
   {
     "took" : 2,
     "timed_out" : false,
     "_shards" : {
       "total" : 1,
       "successful" : 1,
       "skipped" : 0,
       "failed" : 0
     },
     "hits" : {
       "total" : {
         "value" : 1,
         "relation" : "eq"
       },
       "max_score" : 13.400328,
       "hits" : [
         {
           "_index" : "medcl",
           "_type" : "_doc",
           "_id" : "3",
           "_score" : 13.400328,
           "_source" : {
             "content" : "中韩渔警冲突调查：韩警平均每天扣1艘中国渔船"
           }
         }
       ]
     }
   }
   ```
   
   这里多次请求测试发现根据关键字**渔船**和全拼**yuchuan**是可以搜索到的，简拼**yc**就不行。
   
4. 一些补充指令

   ```
   //获取设置
   GET /medcl/_settings
   //返回值
   {
     "medcl" : {
       "settings" : {
         "index" : {
           "number_of_shards" : "1",
           "provided_name" : "medcl",
           "creation_date" : "1573815429511",
           "analysis" : {
             "analyzer" : {
               "pinyin_analyzer" : {
                 "tokenizer" : "my_pinyin"
               }
             },
             "tokenizer" : {
               "my_pinyin" : {
                 "lowercase" : "true",
                 "keep_original" : "true",
                 "remove_duplicated_term" : "true",
                 "keep_separate_first_letter" : "false",
                 "type" : "pinyin",
                 "limit_first_letter_length" : "16",
                 "keep_full_pinyin" : "true"
               }
             }
           },
           "number_of_replicas" : "1",
           "uuid" : "Y3ISVW3QRPupTuT1CPFGsA",
           "version" : {
             "created" : "7000199"
           }
         }
       }
     }
   }
   
   //pinyin分析
   GET /medcl/_analyze
   {
     "text": ["渔船"],
     "analyzer": "pinyin_analyzer"
   }
   //返回值
   {
     "tokens" : [
       {
         "token" : "yu",
         "start_offset" : 0,
         "end_offset" : 0,
         "type" : "word",
         "position" : 0
       },
       {
         "token" : "chuan",
         "start_offset" : 0,
         "end_offset" : 0,
         "type" : "word",
         "position" : 1
       },
       {
         "token" : "渔船",
         "start_offset" : 0,
         "end_offset" : 0,
         "type" : "word",
         "position" : 1
       },
       {
         "token" : "yc",
         "start_offset" : 0,
         "end_offset" : 0,
         "type" : "word",
         "position" : 1
       }
     ]
   }
   ```

   

