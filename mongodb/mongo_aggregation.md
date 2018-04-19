#### MongoDB Aggregation

最近在做产品的后台管理时，查询的数据量比较大时，MongoDB会报错,由于在查询时使用了排序功能，

```
Sort operation used more than the maximum 33554432 bytes of RAM.
Add an index, or specify a smaller limit..
```

根据官方文档的描述，`Sort`是在内存中的操作，使用的内存不能超过32MB，如果超过了32MB，就会报
错误。建议增加 `Index`索引，但是增加索引会增加存储量，也会影响写入的速度

可以使用 `Aggregation`聚合，来操作大量的数据的查询和排序

* Aggregation Pipeline操作的限制

  MongoDB2.6之后，Aggregation Pipeline操作的最大的内存限制是100MB，如果超过100MB也会报错，
  在使用 Aggregation时，如果要处理大量的数据时，请使用 `allowDiskUse` 参数，这个参数可以将
  要处理的数据先写入到临时文件中


#### Aggregation 操作

```
db.collection.aggregate( [ { <stage> }, ... ] )
```

```
$match
$sort
$limit
....
```

具体的stage相关的语句，请参考官方支持的表达式 [aggregate pipeline operation](https://docs.mongodb.com/manual/reference/operator/aggregation-pipeline/#aggregation-pipeline-operator-reference0)

#### Aggregation C# Driver

```
// connectionString="mongodb://127.0.0.1:27017"

var client = new MongoClient(connectionString);
IMongoDatabase _db = client.GetDatabase(dbName);
var collection = GetCollection<T>(collectionName)


IList<IPipelineStageDefinition> stages = new List<IPipelineStageDefinition>();

//查询
string f = "{ $or: [ { _id: { $eq:" + "\"" + query + "\"} }, { sn: { $eq: " + "\"" + query + "\" } } ] }";
string idMatch = "{ $match: " + f + "}";
PipelineStageDefinition<T, T> queryPipline = new JsonPipelineStageDefinition<T, T>(idMatch);

stages.Add(queryPipline);

//排序
string sortPipeline = "{$sort:{time:-1}}";
PipelineStageDefinition<T, T> sortPipline = new JsonPipelineStageDefinition<T, T>(sortPipeline);

stages.Add(sortPipline);

//分页
string skipPiple = "{$skip:" + page * 10 + "}";
string limitPiple = "{$limit:" + 10 + "}";
PipelineStageDefinition<T, T> skipPipline = new JsonPipelineStageDefinition<T, T>(skipPiple);
PipelineStageDefinition<T, T> limitPipline = new JsonPipelineStageDefinition<T, T>(limitPiple);

stages.Add(skipPipline);
stages.Add(limitPipline);

PipelineDefinition<T, T> pipe = new PipelineStagePipelineDefinition<T, T>(stages);


AggregateOptions options = new AggregateOptions();
options.AllowDiskUse = true;

collection.Aggregate<T>(pipeline, options).ToList();
```
