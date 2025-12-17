# 让Claude Code用上Gemini：一个巧妙的AI协作方案

> 作者: 知识药丸 | 发布时间: 2025年12月15日 11:43

> 原文链接: https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649475289&idx=1&sn=569e60179ef940c2ea389cf7b51ccd93&chksm=8fddbc45473ec3116a3266753b44d4f4a9b1aa6df4746c6c5fd1e1fa1053d674eb161faea813&xtrack=1&req_id=1765785745806643&scene=90&subscene=93&sessionid=1765785806&flutter_pos=2&clicktime=1765785824&enterid=1765785824&finder_biz_enter_id=4&ranksessionid=1765785745&jumppath=1001_1765785759225%2C1156_1765785761755%2C1001_1765785764698%2C50094_1765785807028&jumppathdepth=4&ascene=56&fasttmpl_type=0&fasttmpl_fullversion=8041159-zh_CN-zip&fasttmpl_flag=0&realreporttime=1765785824281&devicetype=android-35&version=28004236&nettype=cmnet&lang=zh_CN&session_us=gh_690b43d4ba22&countrycode=CN&exportkey=n_ChQIAhIQS0%2FhqyDYzQsjm69d0NxxEBLuAQIE97dBBAEAAAAAAATpLYhQOgkAAAAOpnltbLcz9gKNyK89dVj05%2BVk%2FA%2BfteDTWg4bcrB%2BCuGiB8%2BoyHt1XDyS%2BiVWQq6CKjRd4qJM9hLH7UlgDStnNNIYzvyp90wLmJkiKdHnQlhIt13PJpYk74ASvEJWK7ad2J9Nqfz3XP%2BHkQZdtXxBzOo8saOWEfs65XmoqePbyGfC8lqUpuKCuguzbqYhT57rkXf2VsPJV%2Fuh%2B9oaYUiISQMXDxR75ejz8pAymTwvGTpr%2BPe3qhBP2BMcye8KMpU5KrSO5OFr2%2BXvvtUlKceYmUNfjeyryQc%3D&pass_ticket=9Sq%2F4pBHRjHWcrBvx8YsSQ%2FCzi8dlZKNUtBv9dmiqswUZTPxjRiu0TSzA7ZJJHh%2B&wx_header=3&poc_token=HDGQQGmjgwNANXX_UuQPu_rK9xp7CkKuafn3jxh6

---

👀 学到一个骚操作，Claude Code + Gemini CLI，双剑合璧，天下无敌
[《贾杰的AI编程秘籍》](https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649474437&idx=1&sn=4ae1516afc0ec71b7638dd79daaae2cb&scene=21#wechat_redirect)付费合集，共10篇，现已完结。30元交个朋友，学不到真东西找我退钱；）以及我的墨问合集《100个思维碎片》，1块钱100篇，与你探讨一些有意思的话题（文末有订阅方式
---

### 写在前面

最近在GitHub上发现了一个有趣的项目：gemini_cli_skill。

说实话，刚看到这个仓库名的时候我有点懵——Gemini CLI Skill for Claude Code？这是什么操作?

仔细研究了一遍源码之后，我发现这个项目的思路**太聪明了**：它把Google的Gemini CLI封装成了Claude Code的一个Skill，让Claude可以在需要的时候"召唤"Gemini来帮忙干活。

这就像是给Claude找了个**能打的小弟**，真正意义上的**取长补短**。

### 为什么要这么做？

你可能会问：Claude自己不是挺好用的吗，为什么还要折腾Gemini？

这就要说到AI模型各自的**长板**了。

#### Claude的优势

Claude擅长：

- • 复杂的推理和逻辑分析
- • 遵循复杂的指令
- • 代码生成和编辑
- • 上下文理解

#### Gemini的独门绝技

但Gemini有几个Claude暂时做不到的能力：

1. 1. **超大上下文窗口**：Gemini 3 Pro支持1M tokens的上下文，这意味着它可以一口气读完整个大型项目
2. 2. **Google Search集成**：通过`google_web_search`工具直接联网搜索，获取最新信息
3. 3. **codebase_investigator**：专门用于分析代码架构的工具，能够深度理解项目结构

所以这个项目的核心思路就是：**让两个AI模型扬长避短，互相配合**。

### 它是怎么工作的？

整个方案的架构其实很简单，但巧妙。

#### Skill机制

Claude Code有一个Skill系统（其实就是放在`~/.claude/skills/`目录下的一堆Markdown文档）。Claude会根据对话内容，自动判断要不要使用某个Skill。

这个项目就是把Gemini CLI的使用方法、最佳实践、常用模板全部整理成了Skill文档。

#### 核心文件结构

```
gemini-cli/
├── SKILL.md        # 主文档：什么时候用、怎么用
├── reference.md    # 命令行参数完整参考
├── templates.md    # 常用提示词模板
├── patterns.md     # 集成模式和工作流
└── tools.md        # Gemini内置工具文档
```

每个文件都有明确的职责，这种**模块化**的设计让Claude可以快速找到需要的信息。

### 典型使用场景

#### 场景1：第二视角代码审查

这是我最喜欢的一个用法。

当Claude写完代码后，你可以让它调用Gemini来做代码审查：

```
gemini "Review this code for security issues and bugs" -o text
```

为什么这样有用？因为**不同的AI模型有不同的"盲区"**。Claude可能忽略的问题，Gemini能发现；反之亦然。

这就像是：

- • Claude是项目负责人，写代码
- • Gemini是QA工程师，专门挑毛病

#### 场景2：获取最新信息

Claude的知识截止日期是2025年1月（虽然有web search，但Gemini的Google Search集成更强大）。

当你需要了解最新的技术动态时：

```
gemini "What's new in React 19? Use Google Search." -o text
```

Gemini会直接调用Google Search，给你最新的信息。

#### 场景3：大型项目架构分析

这是Gemini的**杀手锏**功能。

假设你接手了一个10万行代码的老项目，完全看不懂。你可以让Gemini用`codebase_investigator`工具来分析：

```
gemini "Use codebase_investigator to analyze this project" -o text
```

它会给你：

- • 整体架构概览
- • 关键文件的作用
- • 组件之间的依赖关系
- • 潜在的问题点

这就像是给项目做了个**全身CT扫描**。

### 深入细节：YOLO模式的陷阱

在研究源码的时候，我发现了一个有趣的细节。

文档里反复强调：**YOLO模式不是万能的**。

```
gemini "Create a todo app" --yolo -o text
```

你以为加了`--yolo`（自动批准所有工具调用）就万事大吉了？

**没那么简单**。

即使开启了YOLO模式，Gemini有时候还是会给你返回一个"计划"，然后问：Does this plan look good?

这是因为YOLO只是自动批准**工具调用**，但不影响Gemini本身的"思考习惯"。

#### 解决方案

文档给出了一个聪明的办法：**在提示词里用更强硬的语气**。

不要说：

```
"Create a todo app"
```

要说：

```
"Create a todo app. START BUILDING NOW. Apply changes immediately."
```

这种**祈使句+立即执行**的组合，能大大降低Gemini"犹豫不决"的概率。

### Generate-Review-Fix循环

这是文档里最精彩的一个Pattern。

```
# 第1步：生成代码
gemini "Create a user auth module" --yolo -o text

# 第2步：让Gemini审查自己的代码
gemini "Review auth.js for bugs and security issues" -o text

# 第3步：修复发现的问题
gemini "Fix these issues in auth.js: [问题列表]. Apply now." --yolo -o text
```

为什么这样有效？

因为**生成代码和审查代码是两种不同的"思维模式"**。

就像人写作文一样：

- • 第一遍是创作模式，思路奔放
- • 第二遍是检查模式，挑剔严苛

让同一个模型切换"人格"，往往能发现自己的问题。

### JSON输出的妙用

大部分人用Gemini CLI都是用`-o text`获取文本输出。

但其实`-o json`藏着很多**宝藏信息**：

```
{
  "response": "实际的回答内容",
  "stats": {
    "models": {
      "gemini-2.5-flash": {
        "tokens": {
          "prompt": 1500,
          "candidates": 500,
          "total": 2000,
          "cached": 800
        }
      }
    },
    "tools": {
      "totalCalls": 2,
      "byName": {
        "google_web_search": {
          "count": 1,
          "success": 1,
          "durationMs": 3000
        }
      }
    }
  }
}
```

这些统计数据可以用来：

- • 监控Token使用量（避免超支）
- • 追踪工具调用情况（哪个工具用得多）
- • 分析性能瓶颈（哪个步骤最慢）

如果你要做**自动化流程**，这些数据就是金矿。

### 速率限制的艺术

Gemini的免费额度是：

- • 每分钟60次请求
- • 每天1000次请求

这听起来很多，但实际用起来**很容易触发**。

#### 官方建议的应对策略

**策略1：让CLI自动重试**

Gemini CLI会自动处理速率限制，用指数退避重试。你只会看到：

```
quota will reset after 3s
```

然后它会自动等待并重试。

**策略2：换用更快的模型**

```
# 重要任务用Pro（默认）
gemini "complex analysis" -o text

# 简单任务用Flash（更快，额度独立）
gemini "format this JSON" -m gemini-2.5-flash -o text
```

Flash模型有**独立的速率限制**，这相当于给你了两条并行车道。

**策略3：批量操作**

与其：

```
gemini "Create file A" --yolo
gemini "Create file B" --yolo
gemini "Create file C" --yolo
```

不如：

```
gemini "Create files A, B, and C with [specs]. Create all now." --yolo
```

一次请求完成所有任务，省下2/3的配额。

### .geminiignore的重要性

Gemini虽然有1M tokens的上下文，但这不意味着你要把整个项目都塞给它。

创建`.geminiignore`文件：

```
node_modules/
dist/
*.log
.env
__pycache__/
```

这样做有两个好处：

1. 1. **节省配额**：不必要的文件不会被读取
2. 2. **提高准确度**：减少噪音，让模型专注于关键代码

就像是：你去图书馆找资料，不需要把整个图书馆都翻一遍，只需要看相关的那几本书。

### 我最喜欢的一个细节

在`templates.md`里，有一堆预制的提示词模板。

比如这个代码审查模板：

```
gemini "Review [file] and tell me:
1) What features it has
2) Any bugs or security issues
3) Suggestions for improvement
4) Code quality assessment" -o text
```

为什么我觉得这个好？

因为它把**审查任务结构化了**。不是简单地说"review this file"，而是明确告诉模型从哪几个维度去看。

这种**结构化提示**能显著提高输出质量。

就像是：你去餐厅点菜，说"给我来点好吃的"和说"我要一份麻辣小龙虾、中辣、加香菜、不要葱"，得到的结果肯定不一样。

### 这个方案的局限性

说了这么多优点，也要说说**不足之处**。

#### 1. 额外的开销

每次调用Gemini都要：

- • 启动命令行工具
- • 等待API响应
- • 解析输出

这比Claude直接操作要慢。

#### 2. 上下文分裂

Claude和Gemini是**两个独立的会话**。Gemini不知道你之前和Claude聊了什么，反之亦然。

这意味着你需要在提示词里**明确提供所有必要的上下文**。

#### 3. 需要手动验证

Gemini生成的代码不一定正确（所有AI都一样）。你需要：

- • 语法检查
- • 安全审查
- • 功能测试

不能无脑信任。

### 实际应用建议

基于源码的学习，我总结了几个**最佳实践**：

#### 1. 明确职责分工

- • **Claude**：主导开发，处理复杂逻辑
- • **Gemini**：辅助审查，获取外部信息

不要反过来，因为Claude的指令遵循能力更强。

#### 2. 充分利用模板

不要每次都手写提示词。从`templates.md`里找合适的模板，改改参数就能用。

这不仅省时间，还能**保证质量稳定性**。

#### 3. 善用JSON输出

如果你要做自动化（比如CI/CD集成），一定用`-o json`。

文本输出是给人看的，JSON是给程序看的。

#### 4. 建立验证流程

```
# 生成
gemini "Create utils.js" --yolo -o text

# 验证
node --check utils.js
eslint utils.js
npm test
```

自动化验证能快速发现问题。

### 总结

这个项目最巧妙的地方在于：**它没有发明什么新技术，只是把现有的工具组合得很好**。

核心思路就是：

1. 1. 认清每个工具的长处和短处
2. 2. 设计合理的协作流程
3. 3. 用文档把最佳实践固化下来

这种"**组合式创新**"往往比从零开始造轮子更实用。

如果你在用Claude Code，又经常遇到需要大上下文、联网搜索、或者代码架构分析的场景，不妨试试这个方案。

它可能会让你的开发效率**上一个台阶**。

### 参考资料

- • gemini_cli_skill GitHub仓库
- • Gemini CLI官方文档
- • Claude Code Skills文档

---

 坚持创作不易，求个一键三连，谢谢你～❤️![图片](images/image_1.jpeg)

以及「AI Coding技术交流群」，联系 ayqywx 我拉你进群，共同交流学习～