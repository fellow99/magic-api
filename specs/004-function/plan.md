# 004-function 函数管理模块技术方案

**模块名称**: 函数管理 (Function Management)  
**创建时间**: 2026-03-14  
**状态**: 已完成  
**版本**: 2.2.2

---

## 1. 架构设计

```
┌──────────────────────────────────────┐
│       FunctionController              │
│         (REST API Layer)              │
└──────────────────────────────────────┘
                │
                ▼
┌──────────────────────────────────────┐
│       FunctionService                 │
│      (Function Registry)              │
└──────────────────────────────────────┘
                │
                ▼
┌──────────────────────────────────────┐
│       ScriptEngine                    │
│      (Function Execution)             │
└──────────────────────────────────────┘
```

---

## 2. 核心实现

### 2.1 函数注册

```java
@Service
public class FunctionService {
    
    private final Map<String, FunctionInfo> functions = new ConcurrentHashMap<>();
    
    @Autowired
    private ScriptEngine scriptEngine;
    
    public void register(FunctionInfo function) {
        functions.put(function.getName(), function);
        // 注册到脚本引擎
        scriptEngine.registerFunction(function.getName(), function.getScript());
    }
    
    public void unregister(String name) {
        functions.remove(name);
        scriptEngine.unregisterFunction(name);
    }
    
    public List<FunctionInfo> list() {
        return new ArrayList<>(functions.values());
    }
    
    public FunctionInfo get(String name) {
        return functions.get(name);
    }
}
```

### 2.2 函数调用

```java
// 在脚本中调用
// 示例：db.select("select * from user where id = ?", userId)

public class FunctionModule {
    
    @Autowired
    private FunctionService functionService;
    
    public Object call(String functionName, Object... args) {
        FunctionInfo function = functionService.get(functionName);
        if (function == null) {
            throw new FunctionNotFoundException(functionName);
        }
        // 执行函数脚本
        return scriptEngine.execute(function.getScript(), args);
    }
}
```

---

## 3. REST API 实现

```java
@RestController
@RequestMapping("/magic/api/function")
public class FunctionController {
    
    @Autowired
    private FunctionService functionService;
    
    @GetMapping("/list")
    public List<FunctionInfo> list() {
        return functionService.list();
    }
    
    @GetMapping("/info/{name}")
    public FunctionInfo info(@PathVariable String name) {
        return functionService.get(name);
    }
    
    @PostMapping("/save")
    public Result save(@RequestBody FunctionInfo function) {
        functionService.register(function);
        return Result.success();
    }
    
    @PostMapping("/delete/{name}")
    public Result delete(@PathVariable String name) {
        functionService.unregister(name);
        return Result.success();
    }
}
```

---

## 4. 验收标准

- [ ] 支持函数定义和注册
- [ ] 支持函数管理 API
- [ ] 支持脚本内调用

---

*文档生成时间：2026-03-14*
