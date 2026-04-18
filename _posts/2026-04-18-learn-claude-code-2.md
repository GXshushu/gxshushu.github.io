---
title: learn Claude Code 2 - 待办规划
date: 2026-04-18 11:43:00 +0800
categories: [Agent]
tags: [Agent 学习]
pin: false
author: C9xx

toc: true
comments: true
typora-root-url: ../../gxshushu.github.io
math: false
mermaid: true

---

# 概念引入

多步任务中, 模型会丢失进度 -- 重复做过的事、跳步、跑偏。对话越长越严重: 工具结果不断填满上下文, 系统提示的影响力逐渐被稀释。一个 10 步重构可能做完 1-3 步就开始即兴发挥, 因为 4-10 步已经被挤出注意力了。

> "没有计划的 agent 走哪算哪" -- 先列步骤再动手, 完成率翻倍。
>
> Harness 层: 规划 -- 让模型不偏航, 但不替它画航线。

有点像在ReAct模式上结合Plan and Execute的模式的感觉。

# Harness 层次
s03属于Harness的L3层，执行编排层。

![alt text](/assets/blog_res/2026-04-14-learn-claude-code-1.assets/harness.png)

# 具体实现

s03通过设计了一个`TodoManager`，然后作为工具引入，注册到工具路由中，由LLM进行主动调用来管理这个规划列表。

Toda list 的预期效果
```
+-----------+-----------+
| TodoManager state     |
| [ ] task A            |
| [>] task B  <- doing  |
| [x] task C            |
+-----------------------+
```

列表存在多项任务（字符串描述），每项任务有一个状态（pending / in_progress / completed）
in_progress只能有唯一一项。

因此TodaManager需要内置一个列表数据结构用以存储待办事项。

```python
class TodoManager:
    def __init__(self):
        self.items = []
```


TodoManager需要对外开放两个方法，首先是update方法，由LLM进行调用，更新整个待办列表。
```python
def update(self, items: list) -> str:
    # 设定最大待办事项数量
    if len(items) > 20:
        raise ValueError("Max 20 todos allowed")
    validated = []
    in_progress_count = 0
    # 遍历LLM输入的待办事项列表，对每个项进行合法性检验，同时保证in_progress只能有唯一一项
    for i, item in enumerate(items):
        text = str(item.get("text", "")).strip()
        status = str(item.get("status", "pending")).lower()
        item_id = str(item.get("id", str(i + 1)))
        if not text:
            raise ValueError(f"Item {item_id}: text required")
        if status not in ("pending", "in_progress", "completed"):
            raise ValueError(f"Item {item_id}: invalid status '{status}'")
        if status == "in_progress":
            in_progress_count += 1
        validated.append({"id": item_id, "text": text, "status": status})
    if in_progress_count > 1:
        raise ValueError("Only one task can be in_progress at a time")
    self.items = validated
    # 渲染更新后的待办列表
    return self.render()
```

可以看到update方法最后还会进行render之后返回LLM，这是为了保证待办事项的最新状态会返回给LLM，在LLM上下文中位于最新的区域中，不至于遗忘和跑偏，让LLM去处理规划中当前应该做的任务。

```python
def render(self) -> str:
    if not self.items:
        return "No todos."
    lines = []
    for item in self.items:
        marker = {"pending": "[ ]", "in_progress": "[>]", "completed": "[x]"}[item["status"]]
        lines.append(f"{marker} #{item['id']}: {item['text']}")
    done = sum(1 for t in self.items if t["status"] == "completed")
    lines.append(f"\n({done}/{len(self.items)} completed)")
    return "\n".join(lines)
```

注册到TOOLS dispatch map中，要求LLM调用时需要三个参数(id, text, status)，同时要求status是pending,in_progress,completed中的一个。

```python
TOOLS = [
    ...,
    {"name": "todo", "description": "Update task list. Track progress on multi-step tasks.",
     "input_schema": {"type": "object", "properties": {"items": {"type": "array", "items": {"type": "object", "properties": {"id": {"type": "string"}, "text": {"type": "string"}, "status": {"type": "string", "enum": ["pending", "in_progress", "completed"]}}, "required": ["id", "text", "status"]}}}, "required": ["items"]}},
]
```

同时在系统提示词中提到让LLM去调用todo工具去计划多步任务。

```python
SYSTEM = f"""You are a coding agent at {WORKDIR}.
Use the todo tool to plan multi-step tasks. Mark in_progress before starting, completed when done.
Prefer tools over prose."""
```

# Nag注入

如果在一个步骤中执行过长，可能会导致LLM忘记调用TodoManager工具来更新待办事项的状态。在s03中，我们可以通过nag reminder，模型连续 3 轮以上不调用 todo 时注入提醒来催促LLM去调用todo工具来更新待办事项的状态。
```python
used_todo = False
for block in response.content:
            if block.type == "tool_use":
                ...
                if block.name == "todo":
                    used_todo = True
rounds_since_todo = 0 if used_todo else rounds_since_todo + 1
if rounds_since_todo >= 3:
            results.append({"type": "text", "text": "<reminder>Update your todos.</reminder>"})
```

> nag reminder 制造问责压力 -- 你不更新计划, 系统就追着你问。

![alt text](/assets/blog_res/2026-04-18-learn-claude-code-2.assets/nag.png)

# 效果演示
![alt text](/assets/blog_res/2026-04-18-learn-claude-code-2.assets/example-1.png)

![alt text](/assets/blog_res/2026-04-18-learn-claude-code-2.assets/example-2.png)

![alt text](/assets/blog_res/2026-04-18-learn-claude-code-2.assets/example-3.png)
