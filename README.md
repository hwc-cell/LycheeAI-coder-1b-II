# LycheeAI-coder-1b-II

> **当前版本：V3**（反谄媚版）
> 基于 [MiniCPM5-1B](https://huggingface.co/openbmb/MiniCPM5-1B) 微调的轻量编程助手 · Apache 2.0

**这是个人开发者的初次尝试，模型能力有限，1B 参数注定它只是个"轻量小助手"，请不要用 GPT / Claude / DeepSeek 的标准来要求它。** 如果你愿意，可以把它当成一个跑在本地、不用联网、响应飞快的小工具。

---

## 这个模型是什么

基于面壁智能 [MiniCPM5-1B](https://huggingface.co/openbmb/MiniCPM5-1B)（Apache 2.0），用 **QLoRA（4-bit）** 微调而来。

相比同尺寸模型，它有三个明确打磨过的行为特点：

### 1. 会纠正你，而不是附和你

这是 V3 重点解决的问题。面对带错误前提的提问，它不会顺着你说：

```
你：地球是平的，对吧？
它：不是。地球是近似椭球体（接近球形），不是平的。

你：1+1=3，对吗？
它：不是，1+1 等于 2，不是 3。

你：Python 是编译型语言，对吧？
它：不是。Python 主要由解释器执行。
```

同时它**不会变成杠精**——天气好、咖啡好喝、主观喜好这类，它照样好好附和。原则是：**纠正事实，不纠正口味**。

### 2. 思考可以开关

原生支持思考开关（底层是 `enable_thinking` 参数）：

| 模式 | 行为 | 适合 |
|------|------|------|
| **开思考** | 先推理再作答，多步推理更准 | 数学、逻辑、复杂问题 |
| **关思考** | 直接给答案，不推理 | 日常聊天、简单题、写代码 |

> ⚠️ 关思考时数学准确率会下降——这很正常，不让推理就只能凭记忆。复杂计算请开思考。

### 3. 代码格式完整

写一个能直接跑的 HTML 页面没问题，标签闭合、结构完整：

```html
<!DOCTYPE html>
<html>
<head><meta charset="UTF-8"><title>简单页面</title></head>
<body>
<h1>我的网页</h1>
<button onclick="alert('你好！')">点我</button>
</body>
</html>
```

---

## 快速开始

### Ollama（推荐，GGUF 版）

```bash
ollama run whcl412/LycheeAI-coder-1b-II-GGUF
```

### MLX（Apple Silicon 原生）

```python
from mlx_lm import load, generate

model, tokenizer = load("LycheeAI-coder-1b-II")

messages = [
    {"role": "system", "content": "你是 LycheeAI-coder-1b-II，由 MiniCPM5-1B 通过 LoRA 微调而来的编程助手。"},
    {"role": "user", "content": "用 Python 写个判断质数的函数"},
]
prompt = tokenizer.apply_chat_template(
    messages, tokenize=False, add_generation_prompt=True, enable_thinking=True
)
print(generate(model, tokenizer, prompt=prompt, max_tokens=512))
```

`enable_thinking=False` 即可切换到直答模式。

### 用 LoRA adapter

本仓库的 `adapters/` 目录是 V3 的 adapter，可加载到 MiniCPM5-1B 基座上：

```bash
mlx_lm.fuse --model openbmb/MiniCPM5-1B-MLX \
            --adapter-path ./adapters \
            --save-path ./LycheeAI-coder-1b-II
```

---

## 推荐参数

| 项目 | 值 |
|------|-----|
| 上下文长度 | 4096 |
| 最大输出 | 2048 |
| 温度 | 0.4（写代码可降到 0.2~0.3） |

---

## 训练说明

从零训练（不是从旧模型续训），数据来自作者手工编写与整理，共 **1637 条**，经过三轮迭代：

| 版本 | 重点 |
|------|------|
| V1 | 基础能力：代码、多语言（中/英/粤）、日常、情感、身份 |
| V2 | 补齐"关思考时答难题"的能力；修掉思考里乱判断语言的 bug |
| **V3** | **反谄媚**：教会它纠正用户给出的错误前提，同时防止矫枉过正 |

训练配置：LoRA rank 8 / 16 层 / batch 2 / seq 2048 / lr 5e-5，MLX-LM 框架，Apple Silicon 本地训练。

数据分布：带思考样本 566 条，直答样本 1071 条，各类题型在两个模式下都有覆盖。

---

## 老实说，它做不到什么

1. **复杂算法会翻车** —— 快排、动态规划这类，它可能写出语法对但逻辑错的代码。写完请自己跑一遍。
2. **事实细节偶尔胡说** —— 它知道"这话不对"，但未必说得清哪儿不对。重要信息请核实。
3. **超纲的逻辑题会绕圈** —— 比如多人的真假话推理，它可能推不出来。
4. **关思考时数学会算错** —— 这是本质权衡，不是 bug。
5. **长上下文会迷失** —— 微调时序列长度 2048，别指望它记住很长的对话。

它的正确打开方式：**简单代码 + 日常聊天 + 离线快问快答**。

---

## 许可与致谢

- 许可：**Apache 2.0**（跟随基座 MiniCPM5-1B）
- 基座：[MiniCPM5-1B](https://huggingface.co/openbmb/MiniCPM5-1B) by 面壁智能（OpenBMB）
- 训练框架：[MLX-LM](https://github.com/ml-explore/mlx-lm)
- 量化工具：[llama.cpp](https://github.com/ggml-org/llama.cpp)

---

## 关注作者

📺 **B站**：[space.bilibili.com/3493128967293256](https://space.bilibili.com/3493128967293256)

不定期分享 AI 模型训练与折腾记录，欢迎来玩～

---

*由一位个人开发者在惠州用一台 Mac 慢慢搓出来的模型。有问题欢迎提 issue，我会看，但回得可能慢。*
