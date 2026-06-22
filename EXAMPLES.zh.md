# 示例

展示四个原则的真实代码示例。每个示例都展示了 LLM 常见错误以及如何修复。

---

## 1. 编码前思考

### 示例 1：隐藏的假设

**用户请求：** "添加导出用户数据的功能"

**❌ LLM 的做法（错误的假设）**

```python
def export_users(format='json'):
    """将所有用户导出到文件。"""
    users = User.query.all()
    
    if format == 'json':
        with open('users.json', 'w') as f:
            json.dump([u.to_dict() for u in users], f)
    elif format == 'csv':
        with open('users.csv', 'w') as f:
            writer = csv.DictWriter(f, fieldnames=['id', 'email', 'name'])
            writer.writeheader()
            writer.writerows([u.to_dict() for u in users])
    
    return f"已导出 {len(users)} 位用户"
```

**问题：**
- 假设要导出所有用户（分页呢？隐私呢？）
- 未询问就假设了文件位置
- 假设了要包含哪些字段
- 未检查实际数据结构就假设了 CSV 字段名

**✅ 正确的做法（呈现假设）**

```
在实现之前，我需要澄清：

1. **范围**：导出所有用户还是筛选后的子集？（隐私影响）
2. **格式**：你说的"导出"是指：
   - 在浏览器中下载文件？
   - 后台作业并通过邮件通知？
   - 返回数据的 API 端点？
3. **字段**：哪些用户字段？（有些可能涉及敏感信息）
4. **数据量**：通常有多少用户？（会影响方案选择）

最简单的方法：添加一个返回分页 JSON 的 API 端点。
如果需要基于文件的导出，需要更多信息。

你倾向于哪种方式？
```

### 示例 2：多种解释

**用户请求：** "让搜索更快"

**❌ LLM 的做法（默默选择）**

```python
# 在未询问"更快"具体指什么的情况下
# 添加了缓存、数据库索引和异步处理
@lru_cache(maxsize=1000)
async def search(query: str) -> List[Result]:
    # ... 200 行优化代码
```

**✅ 正确的做法**

```
"让搜索更快"可能指：

1. **更快的响应时间**（从 500ms 降到 100ms 以下）
   - 添加数据库索引
   - 缓存频繁查询
   - 预估工作量：2 小时

2. **处理更多并发搜索**（吞吐量）
   - 添加 async/await
   - 连接池
   - 预估工作量：4 小时

3. **更快的感知速度**（用户体验）
   - 立即显示部分结果
   - 渐进式加载
   - 预估工作量：3 小时

当前搜索对典型查询大约需要 500ms。
对你的用例来说，哪个方面最重要？
```

---

## 2. 简洁优先

### 示例 1：过度抽象

**用户请求：** "添加一个计算折扣的函数"

**❌ LLM 的做法（过度设计）**

```python
from abc import ABC, abstractmethod
from enum import Enum
from typing import Protocol, Union
from dataclasses import dataclass

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, amount: float) -> float:
        pass

class PercentageDiscount(DiscountStrategy):
    def __init__(self, percentage: float):
        self.percentage = percentage
    
    def calculate(self, amount: float) -> float:
        return amount * (self.percentage / 100)

class FixedDiscount(DiscountStrategy):
    def __init__(self, fixed_amount: float):
        self.fixed_amount = fixed_amount
    
    def calculate(self, amount: float) -> float:
        return min(self.fixed_amount, amount)

@dataclass
class DiscountConfig:
    strategy: DiscountStrategy
    min_purchase: float = 0.0
    max_discount: float = float('inf')

class DiscountCalculator:
    def __init__(self, config: DiscountConfig):
        self.config = config
    
    def apply_discount(self, amount: float) -> float:
        if amount < self.config.min_purchase:
            return 0
        discount = self.config.strategy.calculate(amount)
        return min(discount, self.config.max_discount)

# 使用需要 30 多行的设置代码来计算一个简单的折扣
```

**✅ 正确的做法（简洁）**

```python
def calculate_discount(amount: float, percent: float) -> float:
    """计算折扣金额。percent 应为 0-100。"""
    return amount * (percent / 100)

# 使用
discount = calculate_discount(100.0, 10.0)  # 减 10 美元
```

**何时增加复杂性：** 只有当你确实需要多种折扣类型时才这样做。如果这个需求后来才出现，那时再重构。

### 示例 2：推测性功能

**用户请求：** "将用户偏好保存到数据库"

**❌ LLM 的做法（不必要的功能）**

```python
class PreferenceManager:
    def __init__(self, db, cache=None, validator=None):
        self.db = db
        self.cache = cache or InMemoryCache()
        self.validator = validator or DefaultValidator()
    
    def save(self, user_id: int, prefs: dict, 
             merge: bool = True, 
             validate: bool = True,
             notify: bool = False) -> bool:
        """保存偏好，支持可选合并、验证和通知。"""
        
        if validate:
            errors = self.validator.validate(prefs)
            if errors:
                raise ValidationError(errors)
        
        existing = self.get(user_id) if merge else {}
        
        if merge:
            existing.update(prefs)
            prefs = existing
        
        self.db.execute(
            "UPDATE users SET preferences = ? WHERE id = ?",
            (json.dumps(prefs), user_id)
        )
        
        if self.cache:
            self.cache.set(f"prefs:{user_id}", prefs)
        
        if notify:
            self.notify_preference_change(user_id, prefs)
        
        return True
    
    def notify_preference_change(self, user_id: int, prefs: dict):
        # 又是 30 行没人要求的功能代码
        pass
```

**✅ 正确的做法（只做要求的）**

```python
def save_preferences(db, user_id: int, preferences: dict):
    """将用户偏好保存到数据库。"""
    db.execute(
        "UPDATE users SET preferences = ? WHERE id = ?",
        (json.dumps(preferences), user_id)
    )
```

**以后需要时再添加：** 缓存（当性能成为问题时）、验证（当出现错误数据时）、合并（当需求出现时）。

---

## 3. 精准修改

### 示例 1：顺手重构

**用户请求：** "修复空邮箱导致验证器崩溃的 bug"

**❌ LLM 的做法（改得太多）**

```diff
   def validate_user(user_data):
-      # 检查邮箱格式
-      if not user_data.get('email'):
+      """验证用户数据。"""
+      email = user_data.get('email', '').strip()
+      
+      # 验证邮箱
+      if not email:
           raise ValueError("邮箱必填")
-      
-      # 基本邮箱验证
-      if '@' not in user_data['email']:
+      if '@' not in email or '.' not in email.split('@')[1]:
           raise ValueError("邮箱无效")
       
-      # 检查用户名
-      if not user_data.get('username'):
+      # 验证用户名
+      username = user_data.get('username', '').strip()
+      if not username:
           raise ValueError("用户名必填")
+      if len(username) < 3:
+          raise ValueError("用户名太短")
+      if not username.isalnum():
+          raise ValueError("用户名必须为字母数字")
       
       return True
```

**问题：**
- "改进"了超出 bug 修复范围的邮箱验证
- 添加了没人要求的用户名验证
- 修改了注释
- 添加了文档字符串

**✅ 正确的做法（精准修改）**

```diff
   def validate_user(user_data):
       # 检查邮箱格式
-      if not user_data.get('email'):
+      email = user_data.get('email', '')
+      if not email or not email.strip():
           raise ValueError("邮箱必填")
       
       # 基本邮箱验证
-      if '@' not in user_data['email']:
+      if '@' not in email:
           raise ValueError("邮箱无效")
       
       # 检查用户名
       if not user_data.get('username'):
           raise ValueError("用户名必填")
       
       return True
```

**只修改了：** 修复空邮箱处理的具体行。

### 示例 2：风格漂移

**用户请求：** "给上传函数添加日志"

**❌ LLM 的做法（什么都改）**

```diff
- def upload_file(file_path, destination):
+ def upload_file(file_path: str, destination: str) -> bool:
+     """带日志上传文件到目标位置。"""
+     logger.info(f"正在上传 {file_path} 到 {destination}")
+     
      try:
-          with open(file_path, 'rb') as f:
-              data = f.read()
-          
-          response = requests.post(destination, files={'file': data})
-          
-          if response.status_code == 200:
-              return True
-          else:
-              return False
+          with open(file_path, "rb") as f:
+              data = f.read()
+          response = requests.post(destination, files={"file": data})
+          success = response.status_code == 200
+          
+          if success:
+              logger.info(f"上传成功：{file_path}")
+          else:
+              logger.error(f"上传失败：{file_path}, 状态码={response.status_code}")
+          
+          return success
      except Exception as e:
-          print(f"错误：{e}")
+          logger.exception(f"上传错误 {file_path}：{e}")
          return False
```

**问题：**
- 修改了引号风格（' 改为 "）
- 添加了没人要求的类型提示
- 添加了文档字符串
- 重新格式化了空白
- 修改了布尔返回逻辑

**✅ 正确的做法（匹配现有风格）**

```diff
+ import logging
+ 
+ logger = logging.getLogger(__name__)
+ 
   def upload_file(file_path, destination):
+      logger.info(f'开始上传：{file_path}')
      try:
          with open(file_path, 'rb') as f:
              data = f.read()
          
          response = requests.post(destination, files={'file': data})
          
          if response.status_code == 200:
+              logger.info(f'上传成功：{file_path}')
              return True
          else:
+              logger.error(f'上传失败：{file_path}, 状态码={response.status_code}')
              return False
      except Exception as e:
-          print(f"错误：{e}")
+          logger.exception(f'上传错误：{file_path}')
          return False
```

**保持了：** 单引号、无类型提示、原有的布尔模式、间距风格。

---

## 4. 目标驱动执行

### 示例 1：模糊 vs. 可验证

**用户请求：** "修复认证系统"

**❌ LLM 的做法（模糊的方式）**

```
我将通过以下步骤修复认证系统：
1. 审查代码
2. 识别问题
3. 进行改进
4. 测试更改

[在没有明确成功标准的情况下开始修改]
```

**✅ 正确的做法（可验证的目标）**

```
要"修复认证"，我需要定义成功标准。
我们具体要解决什么问题？

如果问题是"密码更改后用户仍处于登录状态"：

计划：
1. 编写测试：修改密码 → 验证旧会话失效
   验证：测试失败（重现 bug）

2. 实现：在密码更改时使会话失效
   验证：测试通过

3. 检查边界情况：多个活跃会话、并发更改
   验证：额外测试通过

4. 验证无回归：现有认证测试仍通过
   验证：全部测试套件通过

当前认证的测试覆盖率：[显示覆盖率]
你遇到的具体认证问题是什么？
```

### 示例 2：带验证的多步骤

**用户请求：** "给 API 添加限流"

**❌ LLM 的做法（一次性全部完成）**

```python
# 在一个 300 行的提交中实现了包含 Redis、多种策略、
# 配置系统和监控的完整限流功能
# 没有清晰的验证步骤
```

**✅ 正确的做法（增量执行 + 验证）**

```
限流计划：

1. 添加基本的内存限流（单个端点）
   验证：
   - 测试：100 次请求 → 前 10 次成功，其余返回 429
   - 手动：curl 端点 11 次，看到限流错误

2. 提取为中间件（应用于所有端点）
   验证：
   - 测试：限流适用于 /users 和 /posts
   - 现有端点测试仍通过

3. 添加 Redis 后端（用于多服务器）
   验证：
   - 测试：应用重启后限流状态保持
   - 测试：两个应用实例共享限流计数器

4. 添加配置（按端点设置速率）
   验证：
   - 测试：/search 允许 10次/分钟，/users 允许 100次/分钟
   - 配置文件解析正确

每个步骤都可独立验证和部署。
先从步骤 1 开始？
```

### 示例 3：测试优先验证

**用户请求：** "当有重复分数时排序会出错"

**❌ LLM 的做法（不重现就直接修复）**

```python
# 在未确认 bug 的情况下立即修改排序逻辑
def sort_scores(scores):
    return sorted(scores, key=lambda x: (-x['score'], x['name']))
```

**✅ 正确的做法（先重现）**

```python
# 1. 首先，编写重现问题的测试
def test_sort_with_duplicate_scores():
    """测试当多个项目分数相同时的排序。"""
    scores = [
        {'name': 'Alice', 'score': 100},
        {'name': 'Bob', 'score': 100},
        {'name': 'Charlie', 'score': 90},
    ]
    
    result = sort_scores(scores)
    
    # bug：重复分数的顺序是不确定的
    # 多次运行此测试，结果应该一致
    assert result[0]['score'] == 100
    assert result[1]['score'] == 100
    assert result[2]['score'] == 90

# 验证：运行测试 10 次 → 因排序不一致而失败

# 2. 现在用稳定排序修复
def sort_scores(scores):
    """按分数降序排序，分数相同时按名称升序排序。"""
    return sorted(scores, key=lambda x: (-x['score'], x['name']))

# 验证：测试稳定通过
```

---

## 反模式总结

| 原则 | 反模式 | 修正 |
|-----------|-------------|-----|
| 编码前思考 | 默默假设文件格式、字段、范围 | 明确列出假设，请求澄清 |
| 简洁优先 | 为单个折扣计算使用策略模式 | 在真正需要复杂性之前，只用一个函数 |
| 精准修改 | 修复 bug 时顺手改引号、加类型提示 | 只修改修复报告问题所需的行 |
| 目标驱动 | "我会审查并改进代码" | "为 bug X 编写测试 → 让它通过 → 验证无回归" |

## 核心洞察

"过度复杂"的示例并非明显错误——它们遵循设计模式和最佳实践。问题在于**时机**：它们在需要之前就引入了复杂性，这会导致：

- 代码更难理解
- 引入更多 bug
- 实现时间更长
- 更难测试

"简洁"版本的优势：
- 更容易理解
- 实现更快
- 更容易测试
- 当复杂性确实需要时，可以稍后重构

**好代码是简洁地解决今天的问题的代码，而不是过早地解决明天的问题。**
