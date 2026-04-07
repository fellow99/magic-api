# 007-plugins 插件技术方案

**模块名称**: 插件集合 (Plugins Collection)  
**创建时间**: 2026-03-14  
**状态**: 已完成  
**版本**: 2.2.2

---

## 1. 集群插件实现

```java
@Component
public class ClusterPlugin implements MagicPlugin {
    
    @Autowired
    private WebSocketSessionManager sessionManager;
    
    @EventListener
    public void onFileEvent(FileEvent event) {
        // 通知其他节点
        sessionManager.broadcast({
            "type": "file_change",
            "action": event.getAction(),
            "fileId": event.getFileId(),
            "content": event.getContent()
        });
    }
    
    @OnMessage
    public void onMessage(WebSocketSession session, String message) {
        // 处理集群消息
        ClusterMessage msg = JSON.parseObject(message, ClusterMessage.class);
        if ("file_change".equals(msg.getType())) {
            // 同步文件变更
            syncFile(msg.getFileId(), msg.getContent());
        }
    }
}
```

---

## 2. Redis 插件实现

```java
@MagicModule("redis")
public class RedisModule {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    public void set(String key, Object value) {
        redisTemplate.opsForValue().set(key, value);
    }
    
    public Object get(String key) {
        return redisTemplate.opsForValue().get(key);
    }
    
    public void del(String key) {
        redisTemplate.delete(key);
    }
    
    public void hset(String key, String field, Object value) {
        redisTemplate.opsForHash().put(key, field, value);
    }
    
    public Object hget(String key, String field) {
        return redisTemplate.opsForHash().get(key, field);
    }
    
    public void expire(String key, long seconds) {
        redisTemplate.expire(key, seconds, TimeUnit.SECONDS);
    }
}
```

---

## 3. MongoDB 插件实现

```java
@MagicModule("mongo")
public class MongoModule {
    
    @Autowired
    private MongoClient mongoClient;
    
    public List<Document> find(String collection, Document filter) {
        MongoDatabase db = mongoClient.getDatabase("test");
        MongoCollection<Document> col = db.getCollection(collection);
        return col.find(filter).into(new ArrayList<>());
    }
    
    public void insert(String collection, Document doc) {
        MongoDatabase db = mongoClient.getDatabase("test");
        db.getCollection(collection).insertOne(doc);
    }
    
    public void update(String collection, Document filter, Document update) {
        MongoDatabase db = mongoClient.getDatabase("test");
        db.getCollection(collection).updateOne(filter, update);
    }
    
    public void delete(String collection, Document filter) {
        MongoDatabase db = mongoClient.getDatabase("test");
        db.getCollection(collection).deleteOne(filter);
    }
}
```

---

## 4. Elasticsearch 插件实现

```java
@MagicModule("es")
public class ElasticsearchModule {
    
    @Autowired
    private RestHighLevelClient esClient;
    
    public SearchResponse search(String index, SearchSourceBuilder source) {
        SearchRequest request = new SearchRequest(index);
        request.source(source);
        return esClient.search(request, RequestOptions.DEFAULT);
    }
    
    public void index(String index, Object doc) {
        IndexRequest request = new IndexRequest(index);
        request.source(JSON.toJSONString(doc), XContentType.JSON);
        esClient.index(request, RequestOptions.DEFAULT);
    }
    
    public void delete(String index, String id) {
        DeleteRequest request = new DeleteRequest(index, id);
        esClient.delete(request, RequestOptions.DEFAULT);
    }
}
```

---

## 5. Nebula 插件实现

```java
@MagicModule("nebula")
public class NebulaModule {
    
    @Autowired
    private NebulaClient nebulaClient;
    
    public ResultSet query(String ngql) {
        return nebulaClient.execute(ngql);
    }
    
    public void insertVertex(String tag, Map<String, Object> props) {
        String ngql = buildInsertVertex(tag, props);
        nebulaClient.execute(ngql);
    }
    
    public void insertEdge(String edge, Map<String, Object> props) {
        String ngql = buildInsertEdge(edge, props);
        nebulaClient.execute(ngql);
    }
}
```

---

## 6. Swagger 插件实现

```java
@Component
public class SwaggerPlugin implements MagicPlugin {
    
    @Autowired
    private MagicDynamicRegistry registry;
    
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
            .select()
            .apis(RequestHandlerSelectors.any())
            .paths(PathSelectors.any())
            .build();
    }
    
    @EventListener
    public void onFileEvent(FileEvent event) {
        // 更新 Swagger 文档
        updateSwaggerDocs();
    }
}
```

---

## 7. SpringDoc 插件实现

```java
@Component
public class SpringDocPlugin implements MagicPlugin {
    
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("magic-api")
                .version("1.0.0"));
    }
}
```

---

## 8. Git 插件实现

```java
@MagicModule("git")
public class GitModule {
    
    @Autowired
    private Git git;
    
    public void commit(String message) throws Exception {
        git.commit().setMessage(message).call();
    }
    
    public List<RevCommit> log() throws Exception {
        List<RevCommit> commits = new ArrayList<>();
        for (RevCommit commit : git.log().call()) {
            commits.add(commit);
        }
        return commits;
    }
    
    public void push() throws Exception {
        git.push().call();
    }
}
```

---

## 9. 定时任务插件实现

```java
@MagicModule("task")
public class TaskModule {
    
    private final TaskScheduler scheduler = new ThreadPoolTaskScheduler();
    private final Map<String, ScheduledFuture<?>> tasks = new ConcurrentHashMap<>();
    
    public void create(String name, String cron, Runnable runnable) {
        ScheduledFuture<?> future = scheduler.schedule(runnable, 
            new CronTrigger(cron));
        tasks.put(name, future);
    }
    
    public void cancel(String name) {
        ScheduledFuture<?> future = tasks.remove(name);
        if (future != null) {
            future.cancel(false);
        }
    }
}
```

---

## 10. 组件插件实现

```java
@MagicModule("component")
public class ComponentModule {
    
    private final Map<String, Object> components = new ConcurrentHashMap<>();
    
    public void register(String name, Object component) {
        components.put(name, component);
    }
    
    public Object get(String name) {
        return components.get(name);
    }
}
```

---

## 11. 验收标准

- [ ] 集群插件：支持集群节点管理、接口变更同步
- [ ] Redis 插件：支持 Redis 基本操作、Key 过期管理
- [ ] MongoDB 插件：支持 MongoDB 基本操作、聚合查询
- [ ] Elasticsearch 插件：支持 ES 基本操作、全文搜索
- [ ] Nebula 插件：支持 Nebula 基本操作、nGQL 查询
- [ ] Swagger 插件：支持 Swagger 文档生成、Swagger UI 访问
- [ ] SpringDoc 插件：支持 OpenAPI 3.0 文档生成
- [ ] Git 插件：支持 Git 版本管理、代码提交
- [ ] 定时任务插件：支持定时任务创建、Cron 表达式、任务管理
- [ ] 组件插件：支持组件扩展

---

*文档生成时间：2026-03-14*
