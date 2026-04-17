---
title: learn Claude Code 1 - 工具分发（路由） 
date: 2026-04-17 16:31:00 +0800
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

# learn claude code 代码仓库

[learn-claude-code](https://github.com/shareAI-lab/learn-claude-code)

# Tools引入

在传统和web chat的AI工具进行协作的时候，更多是AI教你怎么去做，而没有办法让AI直接去做，如果我们给AI一双手去触及我们的项目或者环境，那么AI就能代替我们直接去进行创造和试错。

在s01中只用到了Bash工具进行命令行执行，但s01没有进行工具路由，直接解析LLM返回的工具执行参数，然后调用我们定义的回调函数`run_bash`执行该参数。如果我们存在多个工具，那么我们就需要对AI返回的调用的工具类型进行判断，然后调用对应的回调函数执行该参数。最直接的方法当然是在Agent Loop中写一个工具判断然后对应执行不同的回调函数。但这样的话，我们每次引入一个新的工具就需要对Loop的逻辑进行修改，最好的方法当然是将这两者的代码进行解耦，因为路由是一个固定的逻辑，所以可以做到配置化。

> "加一个工具, 只加一个 handler" -- 循环不用动, 新工具注册进 dispatch map 就行。

# Harness 层次
Tools系统的设计属于Harness的L2层，扩展模型能触达的边界。

![alt text](/assets/blog_res/2026-04-14-learn-claude-code-1.assets/harness.png)

# 实现

1. 定义每个工具的回调函数（处理函数）,`run_read`:

```python
def safe_path(p: str) -> Path:
    path = (WORKDIR / p).resolve()
    if not path.is_relative_to(WORKDIR):
        raise ValueError(f"Path escapes workspace: {p}")
    return path

def run_read(path: str, limit: int = None) -> str:
    text = safe_path(path).read_text()
    lines = text.splitlines()
    if limit and limit < len(lines):
        lines = lines[:limit]
    return "\n".join(lines)[:50000]
```
1. 定义一个`dispatch map`，将每个工具的回调函数通过匿名函数的方式注册到map中

```python
TOOL_HANDLERS = {
    "bash":       lambda **kw: run_bash(kw["command"]),
    "read_file":  lambda **kw: run_read(kw["path"], kw.get("limit")),
    "write_file": lambda **kw: run_write(kw["path"], kw["content"]),
    "edit_file":  lambda **kw: run_edit(kw["path"], kw["old_text"],
                                        kw["new_text"]),
}
```
2. 在 Agent Loop 中，循环中按名称查找处理函数。循环体本身与 s01 完全一致。
```python
for block in response.content:
    if block.type == "tool_use":
        handler = TOOL_HANDLERS.get(block.name)
        output = handler(**block.input) if handler \
            else f"Unknown tool: {block.name}"
        results.append({
            "type": "tool_result",
            "tool_use_id": block.id,
            "content": output,
        })
```

# 改进
这个项目的tools的处理都是基于Linux的，例如`run_bash`中，LLM返回的执行参数都是linux bash命令，在windows环境下会发生异常。为了在windows环境下正常运行，我们需要对tools的处理进行改进。

例如`run_bash`中，我们对于不同平台的差异在于LLM返回的命令，所以只需要在提示词中补充一下系统描述即可。这种环境信息最好要集成到上下文描述中，在可能会存在平台差异的地方需要LLM根据环境进行判断。

```python
TOOLS = [{
    "name": "bash",
    "description": f"Run a shell command in {os_name} shell.",
    "input_schema": {
        "type": "object",
        "properties": {"command": {"type": "string"}},
        "required": ["command"],
    },
}]

os_name = "Linux" if os.name == "posix" else "Windows" if os.name == "nt" else "Unknown"
```

