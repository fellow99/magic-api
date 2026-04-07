# 005-jsr223 脚本引擎模块技术方案

**模块名称**: 脚本引擎 (Script Engine)  
**创建时间**: 2026-03-14  
**状态**: 已完成  
**版本**: 2.2.2

---

## 1. 架构设计

```
┌──────────────────────────────────────┐
│         ScriptEngine                  │
│      (JSR-223 Wrapper)                │
└──────────────────────────────────────┘
                │
                ▼
┌──────────────────────────────────────┐
│      MagicScriptEngine                │
│    (magic-script 实现)                 │
└──────────────────────────────────────┘
                │
        ┌───────┴───────┐
        ▼               ▼
┌──────────────┐  ┌──────────────┐
│ CompileCache │  │ScriptContext │
│(编译缓存)     │  │(上下文管理)   │
└──────────────┘  └──────────────┘
```

---

## 2. 核心实现

### 2.1 脚本引擎封装

```java
@Component
public class MagicScriptEngine {
    
    private final ScriptEngineManager manager = new ScriptEngineManager();
    private final Map<String, CompiledScript> cache = new ConcurrentHashMap<>();
    
    private final ScriptEngine engine;
    
    public MagicScriptEngine() {
        this.engine = manager.getEngineByName("magic-script");
    }
    
    public Object execute(String script, ScriptContext context) {
        // 检查缓存
        CompiledScript compiled = cache.get(script);
        if (compiled == null) {
            // 编译并缓存
            Compilable compilable = (Compilable) engine;
            try {
                compiled = compilable.compile(script);
                cache.put(script, compiled);
            } catch (ScriptException e) {
                throw new ScriptCompileException(e);
            }
        }
        
        // 执行脚本
        try {
            return compiled.eval(context.getBindings());
        } catch (ScriptException e) {
            throw new ScriptExecuteException(e);
        }
    }
    
    public void clearCache(String script) {
        cache.remove(script);
    }
}
```

### 2.2 上下文管理

```java
public class ScriptContextImpl implements ScriptContext {
    
    private final Bindings bindings = new SimpleBindings();
    private final RequestContext requestContext;
    
    public ScriptContextImpl(RequestContext requestContext) {
        this.requestContext = requestContext;
        // 绑定请求上下文变量
        bindings.put("request", requestContext.getRequest());
        bindings.put("response", requestContext.getResponse());
        bindings.put("user", requestContext.getUser());
    }
    
    @Override
    public void setAttribute(String name, Object value) {
        bindings.put(name, value);
    }
    
    @Override
    public Object getAttribute(String name) {
        return bindings.get(name);
    }
    
    public Bindings getBindings() {
        return bindings;
    }
}
```

---

## 3. 缓存管理

```java
@Component
public class ScriptCacheManager {
    
    @Value("${magic-api.script.cache-enabled:true}")
    private boolean cacheEnabled;
    
    @Value("${magic-api.script.cache-size:1000}")
    private int cacheSize;
    
    private final Cache<String, CompiledScript> cache;
    
    public ScriptCacheManager() {
        this.cache = CacheBuilder.newBuilder()
            .maximumSize(cacheSize)
            .expireAfterWrite(1, TimeUnit.HOURS)
            .build();
    }
    
    public CompiledScript get(String key) {
        return cache.getIfPresent(key);
    }
    
    public void put(String key, CompiledScript script) {
        if (cacheEnabled) {
            cache.put(key, script);
        }
    }
    
    public void invalidate(String key) {
        cache.invalidate(key);
    }
}
```

---

## 4. 超时控制

```java
public Object executeWithTimeout(String script, ScriptContext context, long timeout) {
    ExecutorService executor = Executors.newSingleThreadExecutor();
    Future<Object> future = executor.submit(() -> 
        scriptEngine.execute(script, context)
    );
    
    try {
        return future.get(timeout, TimeUnit.MILLISECONDS);
    } catch (TimeoutException e) {
        throw new ScriptTimeoutException("脚本执行超时：" + timeout + "ms");
    } catch (Exception e) {
        throw new ScriptExecuteException(e);
    } finally {
        executor.shutdownNow();
    }
}
```

---

## 5. 验收标准

- [ ] 支持 JSR-223 标准
- [ ] 支持脚本动态编译
- [ ] 支持编译缓存
- [ ] 支持超时控制

---

*文档生成时间：2026-03-14*
