# 智学华文小达人 — 项目记录

## 项目简介
华文课堂互动学习系统，包含教师端和学员端，通过 Firebase Realtime Database 实时同步。

## 文件结构
```
huawen-class/
├── teacher.html   教师端（单元管理、课堂控制、成绩总览）
├── student.html   学员端（签到、朗读、词语、简报、心智图、Quiz、签出）
└── CLAUDE.md      本文件
```

## 部署
- **托管**：GitHub Pages（https://vwen123.github.io/huawen-class/）
- **数据库**：Firebase Realtime Database（vwenbcclass-default-rtdb）
- **Repo**：https://github.com/vwen123/huawen-class

## 技术架构

### Firebase 数据结构
```
/sessions/{sessionId}/
  students/{safeKey}/
    name, av, checkin, scores, lastPage, currentPage, done, doneTime
  classStatus    — 'waiting' | 'started'
  quizOpen       — boolean
/units/{unitId}/
  info/          — title, gradeId, gradeName, createdAt
  content/       — lessonTitle, lessonText, slidesUrl, fcMarkdown,
                   cpMarkdown, qzMarkdown, mmMarkdown,
                   flashcards[], compQs[], quizQs[], gameSettings
/grades/{gradeId}/ — name, createdAt
/meta/activeSessions/{code}/ — sessionId, unitId, code, classLabel
```

### 学员端流程（顺序解锁）
```
checkin → reading → flashcard → slides → mindmap → quiz → checkout
```
- `sectionDone` Set 控制解锁进度
- `markSectionDone(id)` 标记完成
- `quizOpen` Firebase flag 控制 Quiz 入口
- 登出后重新进入可续学（从 Firebase scores + lastPage 恢复）

### 分数权重
| 项目 | 权重 |
|------|------|
| 朗读 | 25% |
| 词语 | 20% |
| 心智图 | 10% |
| 理解 | 25% |
| Quiz | 20% |

## 主要功能

### 教师端
- 年级管理 + 单元列表（按年级分组，可手动分类）
- 内容编辑器（课文、词语、心智图、理解题、Quiz 题、签到题）
- 课堂工具（理解题大屏展示、词云、计时器、幸运抽签）
- 实时成绩看板（可添加/删除学生）
- Quiz 开放控制（教师按钮 → 全班同步进入）
- CSV / PDF 成绩下载

### 学员端
- 签到（心情 + 开放题，一次性，可续学）
- 朗读（段落录音 + 评分 + emoji 反馈）
- 词语闪卡（语音录入 + 游戏：match/blast/hangman/balloon，每次随机顺序）
- 简报（Canva → 打开按钮；Google Slides → iframe）
- 心智图（拖拽填空，随机顺序）
- Quiz（多选题，Fisher-Yates 随机）
- 签出（自评 + 总分报告）

## 关键函数索引

### student.html
| 函数 | 说明 |
|------|------|
| `enterByName()` | 学员登入主流程 |
| `nav(id)` | 页面跳转 + 解锁检查 |
| `canNavTo(id)` | 顺序锁 + quizOpen 检查 |
| `markSectionDone(id)` | 标记章节完成 |
| `listenClassStatus()` | 监听 quizOpen Firebase |
| `applyUnitContent(c)` | 加载单元内容到各组件 |
| `renderSlidesForStudent()` | 渲染简报页 |
| `renderSlidesForStudent()` | Canva→按钮，其他→iframe |
| `startFcGames()` | 词语游戏（随机顺序） |
| `fcGamePool()` | 随机词卡池 |
| `initMindMap()` | 心智图初始化（随机空格/选项）|
| `qgStart()` | Quiz 开始（Fisher-Yates shuffle）|

### teacher.html
| 函数 | 说明 |
|------|------|
| `loadUnitList()` | 按年级分组渲染单元列表 |
| `setUnitGrade(id,sel)` | 手动修改单元年级 |
| `filterGrade(gradeId)` | 年级筛选 |
| `selectUnit(id,title)` | 选择单元加载内容 |
| `saveAllContent()` | 保存所有内容到 Firebase |
| `compToolAutoLoad()` | 课堂工具自动加载当前班级理解题 |
| `compToolRender()` | 渲染理解题卡（同步大屏）|
| `compFsRender()` | 渲染大屏内容 |
| `deleteStudent(skey,name)` | 删除学生记录 |
| `promptAddStudent()` | 添加学生 |
| `toggleQuizOpen()` | 开放/关闭 Quiz |
| `renderDashboard()` | 班级概览看板 |
| `renderStudents()` | 全班成绩明细 |

## Firebase 安全规则
```json
{
  "rules": {
    "sessions": { ".read": true, ".write": true },
    "units":    { ".read": true, ".write": true },
    "grades":   { ".read": true, ".write": true },
    "meta":     { ".read": true, ".write": true }
  }
}
```
> Firebase Console → Realtime Database → Rules → Publish

## 已知注意事项
- Firebase web API key 可以公开在前端代码，安全由 Rules 控制
- Canva 不支持 iframe 嵌入，改用打开链接按钮
- `safeKey(name)` 把特殊字符替换为 `_`（Firebase key 限制）
- 学员端没有 Firebase Auth，只靠姓名+课堂码识别
