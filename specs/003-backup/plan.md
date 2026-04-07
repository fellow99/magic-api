# 003-backup 备份恢复模块技术方案

**模块名称**: 备份恢复 (Backup & Recovery)  
**创建时间**: 2026-03-14  
**状态**: 已完成  
**版本**: 2.2.2

---

## 1. 架构设计

```
┌──────────────────────────────────────┐
│       BackupController                │
│         (REST API Layer)              │
└──────────────────────────────────────┘
                │
                ▼
┌──────────────────────────────────────┐
│        BackupService                  │
│      (Backup Management)              │
└──────────────────────────────────────┘
                │
        ┌───────┴───────┐
        ▼               ▼
┌──────────────┐  ┌──────────────┐
│FileStorage   │  │DBStorage     │
│(文件存储)     │  │(数据库存储)   │
└──────────────┘  └──────────────┘
```

---

## 2. 核心实现

### 2.1 备份服务

```java
@Service
public class BackupService {
    
    @Autowired
    private BackupStorage storage;
    
    @Value("${magic-api.backup.keep-versions:10}")
    private int keepVersions;
    
    public void createBackup(String apiId, String content, String operator) {
        BackupInfo backup = new BackupInfo();
        backup.setId(UUID.randomUUID().toString());
        backup.setApiId(apiId);
        backup.setContent(content);
        backup.setVersion(generateVersion());
        backup.setBackupTime(System.currentTimeMillis());
        backup.setOperator(operator);
        
        storage.save(backup);
        
        // 清理旧版本
        cleanupOldVersions(apiId);
    }
    
    public List<BackupInfo> listBackups(String apiId) {
        return storage.listByApiId(apiId);
    }
    
    public BackupInfo getBackup(String backupId) {
        return storage.getById(backupId);
    }
    
    public void restoreBackup(String backupId) {
        BackupInfo backup = getBackup(backupId);
        // 恢复逻辑
        apiService.update(backup.getApiId(), backup.getContent());
    }
    
    private void cleanupOldVersions(String apiId) {
        List<BackupInfo> backups = storage.listByApiId(apiId);
        if (backups.size() > keepVersions) {
            for (int i = keepVersions; i < backups.size(); i++) {
                storage.delete(backups.get(i).getId());
            }
        }
    }
}
```

### 2.2 版本对比

```java
@Service
public class VersionCompareService {
    
    public VersionDiff compare(String version1, String version2) {
        DiffUtils diff = new DiffUtils();
        List<String> lines1 = Arrays.asList(version1.split("\n"));
        List<String> lines2 = Arrays.asList(version2.split("\n"));
        
        Patch<String> patch = diff.diff(lines1, lines2);
        
        VersionDiff result = new VersionDiff();
        result.setAdded(patch.getAdditions());
        result.setRemoved(patch.getDeletions());
        result.setModified(patch.getModified());
        
        return result;
    }
}
```

---

## 3. REST API 实现

```java
@RestController
@RequestMapping("/magic/api/backup")
public class BackupController {
    
    @Autowired
    private BackupService backupService;
    
    @GetMapping("/list")
    public List<BackupInfo> list(@RequestParam String apiId) {
        return backupService.listBackups(apiId);
    }
    
    @GetMapping("/detail/{id}")
    public BackupInfo detail(@PathVariable String id) {
        return backupService.getBackup(id);
    }
    
    @PostMapping("/restore/{id}")
    public Result restore(@PathVariable String id) {
        backupService.restoreBackup(id);
        return Result.success("恢复成功");
    }
    
    @PostMapping("/delete/{id}")
    public Result delete(@PathVariable String id) {
        backupService.deleteBackup(id);
        return Result.success();
    }
    
    @PostMapping("/compare")
    public VersionDiff compare(@RequestBody CompareRequest request) {
        BackupInfo v1 = backupService.getBackup(request.getVersion1());
        BackupInfo v2 = backupService.getBackup(request.getVersion2());
        return versionCompareService.compare(v1.getContent(), v2.getContent());
    }
}
```

---

## 4. 存储实现

### 4.1 文件存储

```java
@Component
@ConditionalOnProperty(name = "magic-api.backup.storage-type", havingValue = "file")
public class FileBackupStorage implements BackupStorage {
    
    @Value("${magic-api.backup.storage-path:/data/magic-api/backup}")
    private String storagePath;
    
    @Override
    public void save(BackupInfo backup) {
        String filePath = getFilePath(backup.getApiId(), backup.getId());
        FileUtils.writeStringToFile(new File(filePath), 
            JSON.toJSONString(backup), StandardCharsets.UTF_8);
    }
    
    @Override
    public BackupInfo getById(String id) {
        // 查找并读取文件
        File file = findFileById(id);
        String content = FileUtils.readFileToString(file, StandardCharsets.UTF_8);
        return JSON.parseObject(content, BackupInfo.class);
    }
}
```

---

## 5. 配置管理

```java
@Configuration
@ConfigurationProperties(prefix = "magic-api.backup")
public class BackupProperties {
    
    private boolean enabled = true;
    private int keepVersions = 10;
    private String storageType = "file";
    private String storagePath = "/data/magic-api/backup";
    
    // Getters and Setters
}
```

---

## 6. 验收标准

- [ ] 自动保存历史版本
- [ ] 支持版本列表和详情
- [ ] 支持版本对比
- [ ] 支持版本恢复
- [ ] 支持备份导出导入

---

*文档生成时间：2026-03-14*
