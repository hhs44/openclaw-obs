# 三国2048项目 - Agent Team 完整方案

> 基于OpenClaw多Agent架构的正确实现

---

## 🏗️ 架构设计

### 1. Agent 资源池

```
OpenClaw Agent 资源池
├── C-3PO (dev) - 私人助理，项目协调
│   └── 状态：空闲
├── Engineer - 全栈工程师
│   └── 状态：空闲
├── Creator - 内容创作者
│   └── 状态：空闲
└── TeamLeader - 项目经理（待创建）
    └── 状态：空闲
```

### 2. Agent 状态管理

每个Agent维护一个状态文件：
```json
{
  "status": "idle" | "busy",
  "currentProject": null | "项目ID",
  "skills": ["技能1", "技能2"],
  "experience": [
    {
      "project": "三国2048",
      "role": "Team Leader",
      "startDate": "2026-02-23",
      "endDate": "2026-02-23",
      "achievements": ["完成项目协调"]
    }
  ]
}
```

---

## 📋 实施流程

### Phase 1: 准备阶段（用户发起）

**用户** → C-3PO: "创建三国2048游戏项目"

**C-3PO 执行**：
1. 检查可用Agent资源
2. 创建项目目录
3. 选择/创建Team Leader
4. 返回项目启动确认（不阻塞）

```javascript
// C-3PO 检查资源
const agents = [
  { id: "engineer", status: "idle", skills: ["全栈开发", "前端", "后端"] },
  { id: "creator", status: "idle", skills: ["UI设计", "动画", "内容创作"] },
  { id: "teamleader", status: "idle", skills: ["项目管理", "协调"] }
];

// 选择Team Leader（优先空闲且专业对口）
const teamLeader = agents.find(a => 
  a.id === "teamleader" && 
  a.status === "idle"
);

// 绑定Team Leader到项目
teamLeader.currentProject = "sanguo-2048";
teamLeader.status = "busy";
```

---

### Phase 2: 组建团队（Team Leader执行）

**C-3PO** → **Team Leader**: "组建三国2048项目团队"

**Team Leader 执行**：
1. 分析项目需求
2. 设计团队成员能力要求
3. 选择合适的Agent（优先空闲+专业对口）
4. 如无合适Agent则创建新Agent
5. 通过 `sessions_send` 分配任务

```javascript
// Team Leader 分析需求
const projectNeeds = [
  { role: "游戏设计师", skills: ["游戏设计", "数值策划"] },
  { role: "核心开发", skills: ["JavaScript", "游戏逻辑"] },
  { role: "前端开发", skills: ["HTML/CSS", "动画", "UI"] }
];

// 选择Agent
const team = [];

// 游戏设计 → Engineer（空闲+技术背景）
if (engineer.status === "idle") {
  team.push({ agent: engineer, role: "游戏设计师+核心开发" });
  engineer.status = "busy";
  engineer.currentProject = "sanguo-2048";
}

// 前端开发 → Creator（空闲+UI设计）
if (creator.status === "idle") {
  team.push({ agent: creator, role: "前端开发" });
  creator.status = "busy";
  creator.currentProject = "sanguo-2048";
}

// 通过sessions_send分配任务
sessions_send({
  sessionKey: "agent:engineer:main",
  message: `
你是三国2048项目的游戏设计师+核心开发。

任务：
1. 设计游戏系统和人物等级
2. 实现核心游戏逻辑
3. 完成0.1%跳级机制

完成后汇报给我。
  `
});

sessions_send({
  sessionKey: "agent:creator:main",
  message: `
你是三国2048项目的前端开发。

任务：
1. 等待Engineer完成核心逻辑
2. 实现游戏UI和动画
3. 完成收集系统界面

在Engineer完成后开始工作。
  `
});
```

---

### Phase 3: 任务协调（Team Leader持续监控）

**Team Leader 职责**：
1. 跟踪进展
2. 管理任务依赖
3. 解决阻塞
4. 促进Agent间协作

```javascript
// Team Leader 监控循环
setInterval(async () => {
  // 1. 检查进展
  const engineerStatus = await checkAgentProgress("engineer");
  const creatorStatus = await checkAgentProgress("creator");
  
  // 2. 处理依赖
  if (engineerStatus.completed && !creatorStatus.started) {
    // 通知Creator开始
    sessions_send({
      sessionKey: "agent:creator:main",
      message: "Engineer已完成核心逻辑，现在可以开始前端开发了"
    });
  }
  
  // 3. 解决阻塞
  if (engineerStatus.blocked) {
    sessions_send({
      sessionKey: "agent:engineer:main",
      message: "需要什么帮助？我来协调解决"
    });
  }
  
  // 4. Agent间协作
  // Engineer完成后通知Creator
}, 30000); // 每30秒检查
```

---

### Phase 4: Agent间协作

**示例：后端完成通知审计**

```javascript
// Engineer完成后
sessions_send({
  sessionKey: "agent:teamleader:main",
  message: "核心逻辑已完成，文件：core.js"
});

// Team Leader 通知 Creator
sessions_send({
  sessionKey: "agent:creator:main",
  message: "核心逻辑已完成，路径：/projects/sanguo-2048/src/core/core.js。请开始前端开发。"
});
```

---

### Phase 5: 项目完成（C-3PO确认）

**Team Leader** → **C-3PO**: "项目已完成"

**C-3PO 执行**：
1. 验收项目
2. 通知用户
3. 解除Agent绑定
4. 更新Agent能力简历

```javascript
// 解除绑定
engineer.status = "idle";
engineer.currentProject = null;

creator.status = "idle";
creator.currentProject = null;

teamLeader.status = "idle";
teamLeader.currentProject = null;

// 更新能力简历
engineer.experience.push({
  project: "三国2048",
  role: "游戏设计师+核心开发",
  achievements: ["完成游戏设计", "实现0.1%跳级机制"]
});

creator.experience.push({
  project: "三国2048",
  role: "前端开发",
  achievements: ["实现UI和动画", "收集系统界面"]
});

teamLeader.experience.push({
  project: "三国2048",
  role: "Team Leader",
  achievements: ["成功协调团队", "按时完成项目"]
});
```

---

## 🔧 技术实现

### 1. 创建Team Leader Agent

```bash
# 创建工作区
mkdir -p ~/.openclaw/workspace-teamleader

# 更新openclaw.json
jq '.agents.list += [{
  "id": "teamleader",
  "workspace": "/home/huang/.openclaw/workspace-teamleader",
  "identity": {
    "name": "Team Leader",
    "theme": "project manager",
    "emoji": "👔"
  }
}]' ~/.openclaw/openclaw.json
```

### 2. Agent状态文件

位置：`~/.openclaw/agents/<agentId>/status.json`

### 3. Agent通信

使用 `sessions_send` 工具：
- `sessionKey: "agent:<agentId>:main"`
- 支持同步等待（timeoutSeconds > 0）
- 支持异步（timeoutSeconds = 0）

### 4. 项目绑定

项目配置：`~/.openclaw/projects/<projectId>/team.json`
```json
{
  "projectId": "sanguo-2048",
  "teamLeader": "teamleader",
  "members": [
    { "agentId": "engineer", "role": "核心开发" },
    { "agentId": "creator", "role": "前端开发" }
  ],
  "status": "active"
}
```

---

## 📊 方案优势

| 特性 | 说明 |
|------|------|
| 独立Agent | 每个Agent独立运行，不依附 |
| 可复用 | Agent项目结束后可参与新项目 |
| 经验积累 | 能力简历记录成长 |
| 专业对口 | 优先选择专业匹配的Agent |
| 高效协作 | Agent间直接通信 |
| 依赖管理 | Team Leader管理任务顺序 |
| 阻塞解决 | Team Leader主动解决 |

---

## 📁 文件结构

```
~/.openclaw/
├── agents/
│   ├── dev/
│   │   └── status.json
│   ├── engineer/
│   │   └── status.json
│   ├── creator/
│   │   └── status.json
│   └── teamleader/
│       └── status.json
├── projects/
│   └── sanguo-2048/
│       ├── team.json
│       └── src/
└── workspace-teamleader/
    ├── IDENTITY.md
    ├── SOUL.md
    └── MEMORY.md
```

---

_方案设计：2026-02-23 20:25 GMT+8_
_等待用户确认后实施_
