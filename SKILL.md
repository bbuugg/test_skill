---
name: event-landing-page
description: 'Create multilingual event landing pages and enrich them with modules (guests, rundown, sponsors, materials, live, custom content). Use when: building landing pages, adding multilingual support, enriching event pages with modules, configuring event display.'
argument-hint: 'event_id [language] [device]'
---

# Event Landing Page Builder

使用 MCP 工具创建多语言活动落地页并添加丰富的模块内容。

## 何时使用

- 为活动创建多语言落地页（中文、英文等）
- 丰富落地页内容：添加嘉宾、议程、赞助商、媒体素材、直播等模块
- 配置自定义模块（富文本、外部链接）
- 调整模块显示顺序和可见性

## 可用 MCP 工具

所有工具位于 `app/Mcp/Tools/Event/` 目录：

### 基础工具
- `get_event` - 获取活动详情（含房间、时区、嘉宾、标签）
- `get_events` - 获取活动列表（支持搜索和筛选）

### 落地页管理
- `get_landing_pages` - 获取活动的所有落地页
- `get_landing_page` - 获取单个落地页详情
- `create_landing_page` - 创建新的落地页（同时创建 PC 和移动端）
- `update_landing_page` - 更新落地页基础信息

### 模块管理
- `get_landing_page_modules` - 获取落地页的所有模块
- `add_custom_module` - 添加自定义模块（富文本或链接）
- `update_landing_page_module` - 更新模块配置
- `sort_landing_page_modules` - 调整模块排序
- `delete_custom_module` - 删除自定义模块

### 其他资源工具
- `get_guests` / `get_guest` - 获取嘉宾信息
- `get_rundowns` / `get_rundown` - 获取议程信息
- `get_rooms` / `get_room` - 获取直播间信息
- `get_teams` / `get_team_members` - 获取团队和成员信息

## 工作流程

### 1. 创建多语言落地页

**多语言架构说明**：
- 不同语言是独立的落地页记录（不是单个记录的多语言字段）
- 每个语言需要单独创建和配置
- 底层资源（嘉宾、议程等）可能在字段中使用 JSON 存储多语言内容

```workflow
步骤 1: 获取活动信息
- 使用 get_event 获取活动详情
- 检查活动是否存在

步骤 2: 创建落地页
- 使用 create_landing_page 为每种需要的语言创建落地页
- 参数：
  * event_id: 活动ID
  * device: 1=PC, 2=手机（会同时创建 PC 和移动端）
  * language: zh-CN（简体中文）、en-US（美式英语）、zh-TW（繁体中文）等
  * subject: 落地页标题（必填）
  * intro: 简介（可选）
  * location: 地区（可选）

步骤 3: 验证创建
- 使用 get_landing_pages 检查新创建的落地页
- 记录每个语言的 landing_page_id 供后续使用
```

### 2. 丰富落地页内容

#### 2.1 了解可用模块类型

落地页支持以下模块（slug）：

| Slug | 名称 | 说明 |
|------|------|------|
| `main` | 主体 | 活动主要信息 |
| `banner` | 横幅 | 顶部轮播图 |
| `register` | 报名 | 报名注册表单 |
| `organizer` | 主办方 | 主办方信息 |
| `rundown` | 议程 | 活动议程 |
| `guest` | 嘉宾 | 嘉宾列表 |
| `sponsor` | 赞助商 | 合作方/赞助商 |
| `material` | 媒体素材 | 图片/视频素材 |
| `live` | 直播 | 直播间配置 |
| `pc_view` | PC视图 | PC端查看方式 |
| `share` | 分享 | 分享功能 |
| `game` | 游戏 | 互动游戏 |
| `custom` | 自定义 | 富文本或外部链接 |

#### 2.2 获取现有模块

```workflow
- 使用 get_landing_page_modules 查看已有模块
- 参数：
  * event_id
  * landing_page_id
  * show: true/false（可选，筛选显示状态）
```

#### 2.3 配置内置模块

```workflow
使用 update_landing_page_module 更新模块：

通用配置：
- name: 模块名称
- name_show: 是否显示名称
- icon: 图标（URL或FontAwesome类名）
- icon_show: 是否显示图标
- icon_color: 图标颜色
- show: 是否显示模块
- verify: 是否需要报名才能查看
- verification: 是否需要验证
- nur: 小程序新用户需报名，老用户直接进入
- sort: 排序值
- flat: 是否平铺
- display_after_approve: 审核通过后显示

模块特定 value：
- guest: [guest_id1, guest_id2, ...] （嘉宾ID数组，按顺序显示）
- material: [{"id": material_id, "source": 1}, ...]
- live: 直播间配置对象
- custom: 见下方自定义模块说明
```

#### 2.4 添加自定义模块

```workflow
使用 add_custom_module 添加富文本或链接：

参数：
- event_id
- landing_page_id
- name: 模块名称（可选）
- value（可选，也可创建后再更新）：
  
  富文本类型：
  {
    "type": 1,
    "content": "<h2>标题</h2><p>HTML内容...</p>"
  }
  
  链接类型：
  {
    "type": 2,
    "url": "https://example.com"
  }
```

#### 2.5 调整模块顺序

```workflow
使用 sort_landing_page_modules 重新排序：

参数：
- event_id
- landing_page_id
- module_ids: [id1, id2, id3, ...] （按新顺序排列的模块ID数组）
```

### 3. 完整建议流程

**注意**：不支持批量操作，每个语言的落地页需要单独创建和配置。

```workflow
第1步：准备工作
1. 使用 get_event 获取活动详情
2. 使用 get_landing_pages 查看已有落地页

第2步：创建多语言落地页
1. 为每种语言分别调用 create_landing_page
   - zh-CN (简体中文)
   - en-US (英文)
   - zh-TW (繁体中文)
2. 记录每个落地页的 ID

第3步：丰富内容（针对每个语言的落地页重复以下步骤）
1. 使用 get_landing_page_modules 查看默认模块
2. 配置嘉宾模块：
   - 使用 get_guests 获取嘉宾列表
   - 使用 update_landing_page_module 设置 guest 模块的 value
3. 配置议程模块：
   - 使用 get_rundowns 获取议程
   - 更新 rundown 模块
4. 配置直播模块：
   - 使用 get_rooms 获取直播间
   - 更新 live 模块
5. 添加自定义内容（如需要）：
   - 根据语言添加对应的富文本内容
   - 添加外部链接（可以跨语言复用）

第4步：优化展示（每个语言单独配置）
1. 使用 sort_landing_page_modules 调整模块顺序
2. 使用 update_landing_page_module 配置各模块的 show 属性
3. 设置需要报名才能查看的模块（verify: true）

第5步：验证
1. 使用 get_landing_page 查看每个语言的最终配置
2. 使用 get_landing_page_modules 确认所有模块
```

## 最佳实践

### 多语言策略

**重要说明**：落地页采用分离式多语言管理，不同语言是独立的 landing_page 记录。

1. **独立创建**：每种语言（zh-CN、en-US、zh-TW）都需要独立创建落地页
2. **模块配置**：每个语言的落地页需要单独配置模块内容
3. **JSON 字段多语言**：某些字段内部采用 JSON 存储多语言数据，例如：
   ```json
   {
     "zh-CN": "中文内容",
     "en-US": "English Content",
     "zh-TW": "繁體內容"
   }
   ```
4. **资源复用**：嘉宾、议程、直播间等底层资源是跨语言共享的，但它们的字段可能也包含多语言 JSON
5. **一致性维护**：确保所有语言版本的模块结构保持一致，避免某个语言缺少重要模块

### 内容丰富度建议

**基础配置**（必需）：
- ✅ main - 活动主体信息
- ✅ banner - 视觉吸引力
- ✅ register - 报名入口

**进阶配置**（推荐）：
- ✅ guest - 展示嘉宾吸引参会者
- ✅ rundown - 清晰的议程安排
- ✅ organizer - 主办方信息增加可信度

**高级配置**（可选）：
- ✅ sponsor - 展示合作伙伴
- ✅ material - 往期精彩瞬间
- ✅ live - 直播回放功能
- ✅ custom - 常见问题、联系方式、特殊说明等

### 自定义模块使用

自定义模块支持两种类型：

1. **富文本类型** (`type: 1`)
   - 适用于：活动亮点、报名须知、联系方式、常见问题、特殊说明等
   - 可以使用完整的 HTML 格式化内容
   - 多语言需要为每个语言的落地页分别创建并配置不同的 HTML 内容

2. **链接类型** (`type: 2`)
   - 适用于：外部报名系统、官网跳转、第三方平台、相关资源等
   - URL 最大长度 255 字符
   - 建议给链接模块起清晰的名称以说明跳转目标

### 模块排序策略

推荐的模块顺序（从上到下）：

1. `banner` - 首屏吸引
2. `main` - 核心信息
3. `register` - 行动号召（报名）
4. `rundown` - 议程详情
5. `guest` - 嘉宾介绍
6. `custom` (亮点) - 特色展示
7. `live` - 直播/回放
8. `material` - 媒体素材
9. `sponsor` - 合作方
10. `organizer` - 主办方
11. `custom` (FAQ) - 常见问题
12. `share` - 分享功能

## 故障排查

### 问题 1: 语言已存在
**错误**: "该设备和语言的落地页已存在"
**解决**: 
- 使用 get_landing_pages 查看已有语言
- 如果是更新现有页面，使用 update_landing_page 而不是 create_landing_page

### 问题 2: 模块不显示
**原因**: 
- `show` 属性设置为 false
- `verify: true` 但用户未报名
- `display_after_approve: true` 但用户未审核通过
**解决**: 使用 update_landing_page_module 调整相关属性

### 问题 3: 模块顺序错乱
**解决**: 使用 sort_landing_page_modules 重新排序，传入正确顺序的 module_ids 数组

### 问题 4: 自定义模块内容不对
**检查**: 
- 富文本类型必须是 `{"type": 1, "content": "..."}`
- 链接类型必须是 `{"type": 2, "url": "..."}`
- URL 最大长度 255 字符
- value 必须是对象/数组，不能是 JSON 字符串

## 相关文件

- 控制器：`app/Http/Controllers/Admin/Api/EventLandingPageController.php`
- 模块控制器：`app/Http/Controllers/Admin/Api/EventLandingPageModuleController.php`
- 实体：`app/Entities/EventLandingPage.php`
- 模块实体：`app/Entities/EventLandingPageModule.php`
- 模块类型：`app/Models/Event/LandingPage/Modules/`
- MCP 工具：`app/Mcp/Tools/Event/`

## 示例对话

```
用户: 为活动 123 创建中英文落地页

助手: 
1. [调用 get_event(event_id=123)]
2. [调用 create_landing_page(event_id=123, device=1, language='zh-CN', subject='活动标题')]
3. [调用 create_landing_page(event_id=123, device=1, language='en-US', subject='Event Title')]
4. 已创建中英文落地页（PC和移动端同时创建）
   - 中文落地页 ID: xxx
   - 英文落地页 ID: yyy
```

```
用户: 为中文落地页添加嘉宾和议程

助手:
1. [调用 get_landing_page_modules(landing_page_id=xxx)] - 查看模块
2. [调用 get_guests(event_id=123)] - 获取嘉宾列表
3. [调用 update_landing_page_module(module_id=..., value=[1,2,3])] - 设置嘉宾顺序
4. [调用 get_rundowns(event_id=123)] - 获取议程
5. [调用 update_landing_page_module 配置议程模块]
6. 中文落地页内容已更新（注意：英文落地页需要单独配置）
```

```
用户: 添加常见问题自定义模块到英文落地页

助手:
[调用 add_custom_module(
  event_id=123, 
  landing_page_id=yyy,
  name='FAQ',
  value={
    "type": 1,
    "content": "<h3>Q: How to register?</h3><p>A: Click the register button...</p>"
  }
)]
已添加到英文落地页。如需在中文落地页添加，请单独创建。
```
