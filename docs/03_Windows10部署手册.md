# Windows 10 本地部署手册

> 极客时间《DeepResearch前沿智能体实战课》Windows 10 超详细部署指南

---

## 目录

1. [环境依赖清单](#1-环境依赖清单)
2. [Step-by-Step 安装指南](#2-step-by-step-安装指南)
3. [各模块运行说明](#3-各模块运行说明)
4. [常见错误排查方案](#4-常见错误排查方案)
5. [验证部署成功的测试步骤](#5-验证部署成功的测试步骤)

---

## 1. 环境依赖清单

### 1.1 系统要求

| 组件 | 版本要求 | 说明 |
|------|---------|------|
| **操作系统** | Windows 10 (1903+) 或 Windows 11 | 需要支持 WSL2（可选但推荐）|
| **内存** | 最低 8GB，推荐 16GB | LLM 工作流会占用较多内存 |
| **磁盘空间** | 至少 10GB 可用空间 | Python 环境 + 依赖包 + 输出文件 |
| **网络** | 需要访问 DeepSeek API（国内可用）| Web 搜索功能需要 SearXNG 服务或 BochaSearch API |

### 1.2 必装软件

| 软件 | 版本 | 下载地址 | 备注 |
|------|------|---------|------|
| **Python** | **3.11.x**（必须是 3.11！）| https://www.python.org/downloads/release/python-3119/ | ⚠️ 不要使用 3.12 或 3.13 |
| **Git** | 2.x | https://git-scm.com/download/win | 用于克隆仓库 |
| **pip** | 23.x+（随 Python 自带）| 随 Python 安装自动获取 | 通常无需单独安装 |

### 1.3 API Key 清单

| API | 必要性 | 申请地址 | Key 格式示例 |
|-----|--------|---------|------------|
| **DeepSeek API Key** | **必须** | https://platform.deepseek.com | `sk-xxxxxxxxxxxxxxxxxxxxxxxx` |
| **通义千问 API Key** | **必须**（多个模块使用）| https://dashscope.aliyun.com | `sk-xxxxxxxxxxxxxxxx` |
| **BochaSearch API Key** | 用于 11.websearch 模块 | BochaAI 官网 | 按官方格式 |

> ⚠️ **注意：** 请务必在开始之前先申请 DeepSeek API Key，申请通常需要手机号验证，几分钟内即可完成。

---

## 2. Step-by-Step 安装指南

### 步骤 1：安装 Python 3.11

> ⚠️ **重要：** 本项目必须使用 Python 3.11。Python 3.12 和 3.13 可能存在兼容性问题，请勿使用。

**1.1 下载 Python 3.11.9**

访问：https://www.python.org/downloads/release/python-3119/

下载页面滚动到底部，选择：
- **Windows installer (64-bit)**（适合大多数现代电脑）
- 文件名：`python-3.11.9-amd64.exe`

**1.2 安装时的关键选项**

双击下载的安装包，**在第一个安装界面务必勾选**：

```
☑ Add Python 3.11 to PATH    ← ⚠️ 这个必须勾选！
☑ Install launcher for all users
```

然后点击 **"Install Now"**（推荐使用默认路径）。

> ⚠️ **注意：** 如果忘记勾选 "Add Python to PATH"，后续命令行中会找不到 python 命令。解决方法：卸载重装，或手动添加 PATH（控制面板 → 系统 → 高级系统设置 → 环境变量）。

**1.3 验证安装**

安装完成后，打开 **命令提示符（CMD）**（按 `Win+R`，输入 `cmd`，回车）：

```cmd
python --version
```

**预期输出：**
```
Python 3.11.9
```

如果看到 `Python 3.11.x`，则安装成功。

```cmd
pip --version
```

**预期输出：**
```
pip 23.x.x from C:\Users\用户名\AppData\...\pip (python 3.11)
```

---

### 步骤 2：安装 Git

**2.1 下载 Git**

访问：https://git-scm.com/download/win

点击 **"Click here to download"** 下载最新版本（如 `Git-2.47.0-64-bit.exe`）。

**2.2 安装 Git（使用默认选项）**

双击安装包，全程使用默认选项点击 **"Next"** 即可。关键选项说明：

- "Choosing the default editor"：推荐选择 **Notepad** 或 **Visual Studio Code**（如已安装）
- "Adjusting your PATH environment"：选择 **"Git from the command line and also from 3rd-party software"**（默认）

**2.3 验证安装**

```cmd
git --version
```

**预期输出：**
```
git version 2.47.0.windows.1
```

---

### 步骤 3：克隆仓库

打开 **命令提示符**，进入你想存放项目的目录，然后执行：

```cmd
cd C:\Users\你的用户名\Documents

git clone https://github.com/igeng/Geek03.git

cd Geek03
```

**预期输出：**
```
Cloning into 'Geek03'...
remote: Enumerating objects: xxx, done.
remote: Total xxx (delta 0), reused 0 (delta 0)
Receiving objects: 100% (xxx/xxx), xxx KiB | xxx MiB/s, done.
```

> 💡 **初学者提示：** 如果 `git clone` 速度很慢，可以尝试使用 Gitee 镜像或配置 Git 代理。

---

### 步骤 4：创建虚拟环境

> 💡 **为什么需要虚拟环境？** 不同模块可能依赖相同包的不同版本。使用虚拟环境可以将每个模块的依赖隔离，避免冲突。

本项目有多个模块，建议**按需为每个模块单独创建虚拟环境**。下面以 `18.deepresearch` 模块为例：

```cmd
cd C:\Users\你的用户名\Documents\Geek03\18.deepresearch

python -m venv venv
```

**激活虚拟环境：**

```cmd
venv\Scripts\activate
```

**预期输出（命令提示符前缀变化）：**
```
(venv) C:\Users\你的用户名\Documents\Geek03\18.deepresearch>
```

看到 `(venv)` 前缀说明虚拟环境已成功激活。

**退出虚拟环境（可选）：**

```cmd
deactivate
```

---

### 步骤 5：安装依赖

> ⚠️ **注意：** 确保虚拟环境已激活（看到 `(venv)` 前缀）再执行以下命令。

**5.1 安装核心依赖（所有模块通用）**

```cmd
pip install langchain langchain-openai langchain-deepseek langgraph python-dotenv openai
```

**预期输出：**
```
Successfully installed langchain-0.3.x langchain-openai-0.2.x ...
```

**5.2 针对 DeepResearch 模块（18.deepresearch）**

```cmd
pip install streamlit langchain-community langchain-mcp-adapters
```

> 💡 **初学者提示：** DeepResearch 模块使用 SearXNG 作为搜索引擎（通过 `langchain-community` 中的 `SearxSearchWrapper`），需要可用的 SearXNG 服务地址。不需要 Tavily API Key。

**5.3 针对金融研报模块（21~26.financial）**

```cmd
cd C:\Users\你的用户名\Documents\Geek03\21~26.financial
python -m venv venv
venv\Scripts\activate

pip install langchain langchain-openai langchain-deepseek langgraph python-dotenv
pip install akshare pandas python-docx pyyaml
pip install langchain-community langchain-mcp-adapters
```

**5.4 针对记忆模块（19~20.mem0）**

```cmd
pip install mem0ai
```

**5.5 针对浏览器模块（14.browser_use）**

```cmd
pip install browser-use
playwright install chromium
```

**5.6 升级 pip（如果安装速度慢或报错）**

```cmd
python -m pip install --upgrade pip

# 切换为国内镜像源（加速下载）
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

---

### 步骤 6：配置环境变量（.env 文件）

每个模块目录下都需要一个 `.env` 文件来存储 API Key。

**6.1 在模块目录下创建 .env 文件**

以 `18.deepresearch` 为例：

```cmd
cd C:\Users\你的用户名\Documents\Geek03\18.deepresearch
notepad .env
```

**6.2 填写以下内容**

```
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxx
TONGYI_API_KEY=sk-xxxxxxxxxxxxxxxx
```

> ⚠️ **注意事项：**
> - 等号前后**不要有空格**（`KEY=value`，不是 `KEY = value`）
> - Key 值**不要加引号**（`sk-xxx`，不是 `"sk-xxx"`）
> - 每行一个键值对
> - 保存文件时确保编码为 **UTF-8**（记事本另存为时可以选择）
> - `18.deepresearch` 需要 `DEEPSEEK_API_KEY` 和 `TONGYI_API_KEY`

**6.3 金融模块的 .env 配置（21~26.financial）**

```
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxx
TONGYI_API_KEY=sk-xxxxxxxxxxxxxxxx
```

**6.4 Web 搜索模块（11.websearch）额外的 .env 配置**

```
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxx
TONGYI_API_KEY=sk-xxxxxxxxxxxxxxxx
BOCHA_API_KEY=你的BochaSearch_API_Key
```

**6.4 验证 .env 配置**

```cmd
python -c "from dotenv import load_dotenv; import os; load_dotenv(); print('DeepSeek Key:', os.environ.get('DEEPSEEK_API_KEY', '未找到')[:10] + '...')"
```

**预期输出：**
```
DeepSeek Key: sk-xxxxxxxx...
```

---

### 步骤 7：运行各模块

#### 7.1 运行 DeepResearch Web UI（18.deepresearch）

```cmd
cd C:\Users\你的用户名\Documents\Geek03\18.deepresearch
venv\Scripts\activate
streamlit run app.py
```

**预期输出：**
```
  You can now view your Streamlit app in your browser.

  Local URL: http://localhost:8501
  Network URL: http://192.168.x.x:8501
```

然后浏览器会自动打开 `http://localhost:8501`，显示深度研究助手界面。

> 💡 **初学者提示：** 如果浏览器没有自动打开，手动在浏览器地址栏输入 `http://localhost:8501` 即可。

#### 7.2 运行 ReAct 示例（06.ReAct）

```cmd
cd C:\Users\你的用户名\Documents\Geek03\06.ReAct
python -m venv venv
venv\Scripts\activate
pip install openai python-dotenv

# 确保 .env 文件已配置（需要 TONGYI_API_KEY）
python react\agent.py
```

> 💡 **初学者提示：** `06.ReAct/react/` 使用通义千问 (`qwen-max`)，需要在 `.env` 中配置 `TONGYI_API_KEY`。而 `06.ReAct/functioncalling/` 使用 DeepSeek，需要 `DEEPSEEK_API_KEY`。

#### 7.3 运行金融研报 Agent（21~26.financial）

> ⚠️ **注意：** 金融研报模块运行时间较长（可能需要 30-60 分钟），请确保网络稳定，DeepSeek API 余额充足。

```cmd
cd C:\Users\你的用户名\Documents\Geek03\21~26.financial
venv\Scripts\activate
python workflow.py
```

**预期输出（开始阶段）：**
```
🖼️ 处理图片路径...
📋 生成报告大纲...
✍️ 开始分段生成深度研报...
  正在生成：第一章 公司概况...
```

---

## 3. 各模块运行说明

### 3.1 模块目录与运行命令汇总

| 模块 | 进入目录 | 主要运行命令 |
|------|---------|------------|
| `06.ReAct` | `cd 06.ReAct` | `python react/agent.py` |
| `07.CodeAct` | `cd 07.CodeAct` | `python codeact/graph.py` |
| `08.planmode` | `cd 08.planmode` | `python planmode-sample/graph.py` |
| `09.reflection` | `cd 09.reflection` | `python reflection/graph.py` |
| `10.human` | `cd 10.human` | `python human/graph.py` |
| `11.websearch` | `cd 11.websearch` | `python bochasearch/agent.py` |
| `12.websearch` | `cd 12.websearch` | `python searXNG/agent.py` |
| `14.browser_use` | `cd 14.browser_use` | `python browseruse/agent.py` |
| `17.deep-thinking` | `cd 17.deep-thinking` | `python agent.py` |
| `18.deepresearch` | `cd 18.deepresearch` | `streamlit run app.py` |
| `19~20.mem0` | `cd 19~20.mem0` | `python mem0/agent.py` |
| `21~26.financial` | `cd 21~26.financial` | `python workflow.py` |
| `直播二.smolagents` | `cd 直播二.smolagents` | `python test1/agent.py` |
| `直播三.context7` | `cd 直播三.context7` | `python agent.py` |

### 3.2 各模块额外依赖

| 模块 | 额外需要安装的包 |
|------|--------------|
| `06.ReAct` | `pip install openai`（使用原生 OpenAI SDK）|
| `11.websearch` | 需要 `BOCHA_API_KEY`（BochaSearch API）|
| `12.websearch` | `pip install langchain-community`（SearXNG 搜索引擎）|
| `14.browser_use` | `pip install browser-use && playwright install chromium` |
| `17.deep-thinking` | `pip install langchain-community`（SearXNG 搜索）|
| `18.deepresearch` | `pip install streamlit langchain-community langchain-mcp-adapters` |
| `19~20.mem0` | `pip install mem0ai` |
| `21~26.financial` | `pip install akshare pandas python-docx pyyaml langchain-community` |
| `直播二.smolagents` | `pip install smolagents` |

---

## 4. 常见错误排查方案

### 4.1 Python 相关错误

**错误：`python: command not found` 或 `'python' 不是内部或外部命令`**

```
原因：Python 未正确添加到 PATH 环境变量。

解决方案：
方案一（推荐）：卸载 Python，重新安装，确保勾选 "Add Python to PATH"

方案二（手动修复）：
1. 找到 Python 安装路径（通常是 C:\Users\用户名\AppData\Local\Programs\Python\Python311\）
2. 按 Win+R，输入 sysdm.cpl，打开系统属性
3. 点击 "高级" → "环境变量"
4. 在 "系统变量" 中找到 Path，双击编辑
5. 点击 "新建"，添加 Python 路径和 Scripts 路径：
   C:\Users\用户名\AppData\Local\Programs\Python\Python311\
   C:\Users\用户名\AppData\Local\Programs\Python\Python311\Scripts\
6. 关闭所有 CMD 窗口并重新打开
```

**错误：`Python was not found; run without arguments to install from the Microsoft Store`**

```
原因：系统有 Python 别名指向 Microsoft Store。

解决方案：
1. 打开 "设置" → "应用" → "应用执行别名"
2. 找到 "python.exe" 和 "python3.exe"，将其关闭（设为"关闭"）
3. 重新打开 CMD，再次尝试
```

---

### 4.2 pip 安装错误

**错误：`ERROR: Could not install packages due to an OSError: [WinError 5] Access is denied`**

```
原因：权限不足。

解决方案：
方案一：以管理员身份运行 CMD（右键 CMD → "以管理员身份运行"）
方案二：使用用户级安装（推荐）
pip install --user 包名
```

**错误：`ERROR: pip's dependency resolver does not currently take into account all the packages that are installed.`**

```
原因：包版本冲突（通常是警告，不是错误）

解决方案：忽略该警告，或强制重装特定包
pip install 包名 --force-reinstall
```

**错误：`ReadTimeoutError` 或 `Connection aborted` 安装超时**

```
原因：网络问题，连接 PyPI 服务器超时。

解决方案：切换为国内镜像
pip install 包名 -i https://pypi.tuna.tsinghua.edu.cn/simple

或者永久设置镜像：
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

---

### 4.3 运行时错误

**错误：`ModuleNotFoundError: No module named 'langchain_deepseek'`**

```cmd
pip install langchain-deepseek
```

**错误：`ModuleNotFoundError: No module named 'langchain_mcp_adapters'`**

```cmd
pip install langchain-mcp-adapters
```

**错误：`AuthenticationError: API key is invalid` 或 `401 Unauthorized`**

```
原因：API Key 无效或 .env 文件配置有误。

排查步骤：
1. 检查 .env 文件是否在当前运行目录（not 父目录）
2. 确认格式：DEEPSEEK_API_KEY=sk-xxx（无空格无引号）
3. 确认 Key 未过期且账户余额充足
4. 测试命令：
   python -c "from dotenv import load_dotenv; import os; load_dotenv(); print(os.environ.get('DEEPSEEK_API_KEY'))"
```

**错误：`KeyError: 'messages'` 或 `KeyError: 'stock_code'`**

```
原因：LangGraph State 中缺少必要字段的初始化值。

解决方案：在调用 workflow.invoke() 时提供所有必要字段：
workflow.invoke({
    "stock_code": "600519",
    "stock_name": "贵州茅台",
    "market": "A股",
    "year": ["2022", "2023", "2024"],
    "messages": [],
    "company_report": {},
    "compare_company_report": {},
    "formatted_output": []
})
```

**错误：`streamlit: command not found` 或 `'streamlit' 不是内部或外部命令`**

```cmd
# 方案一：确保虚拟环境已激活
venv\Scripts\activate
streamlit run app.py

# 方案二：使用 Python 模块方式运行
python -m streamlit run app.py
```

**错误：网络连接 DeepSeek API 超时**

```
原因：网络问题或 DeepSeek 服务器压力大。

解决方案：
方案一：重试（API 偶发超时属正常现象）
方案二：在代码中设置重试参数
  llm = ChatDeepSeek(
      model="deepseek-chat",
      max_retries=3,
      timeout=120,
  )
方案三：检查是否有代理设置影响了 HTTPS 请求
  在 .env 中添加代理设置（如有）：
  HTTP_PROXY=http://proxy.example.com:8080
  HTTPS_PROXY=http://proxy.example.com:8080
```

**错误：`SyntaxError` 或 `match` 语法相关错误（Python 版本不兼容）**

```
原因：使用了 Python 3.12+ 废弃的某些语法，或使用了 3.10+ 的 match 语句但 Python 版本 < 3.10。

解决方案：确认 Python 版本为 3.11.x
python --version

如果版本不对，需要切换到正确版本。可以使用 py launcher 指定版本：
py -3.11 script.py
```

---

### 4.4 AKShare 相关错误

**错误：`ConnectionError: HTTPSConnectionPool` 或数据获取失败**

```
原因：网络问题或 AKShare 数据源变动。

解决方案：
方案一：更新 AKShare 到最新版本
  pip install akshare --upgrade

方案二：检查网络连接（AKShare 访问的是境内数据源）

方案三：单独测试 AKShare
  python -c "import akshare as ak; print(ak.__version__)"
  python -c "import akshare as ak; df = ak.stock_info_a_code_name(); print(df.head(3))"
```

---

## 5. 验证部署成功的测试步骤

### 5.1 测试 Python 和基础包安装

```cmd
python -c "import langchain; import langgraph; import langchain_deepseek; print('基础包安装成功')"
```

**预期输出：**
```
基础包安装成功
```

### 5.2 测试 DeepSeek API 连接

创建测试文件 `test_api.py`（在任意模块目录下，确保 `.env` 已配置）：

```python
from dotenv import load_dotenv
import os
from langchain_deepseek import ChatDeepSeek

load_dotenv()

def test_deepseek():
    api_key = os.environ.get("DEEPSEEK_API_KEY")
    if not api_key:
        print("❌ 错误：DEEPSEEK_API_KEY 未配置")
        return False
    
    try:
        llm = ChatDeepSeek(
            model="deepseek-chat",
            api_key=api_key,
        )
        response = llm.invoke("请用一句话介绍你自己。")
        print(f"✅ DeepSeek API 连接成功！")
        print(f"   模型回复：{response.content[:50]}...")
        return True
    except Exception as e:
        print(f"❌ DeepSeek API 连接失败：{e}")
        return False

if __name__ == "__main__":
    test_deepseek()
```

```cmd
python test_api.py
```

**预期输出：**
```
✅ DeepSeek API 连接成功！
   模型回复：我是 DeepSeek，一个由深度求索公司开发的 AI 助手...
```

### 5.3 测试 DeepResearch 模块

```cmd
cd C:\Users\你的用户名\Documents\Geek03\18.deepresearch
venv\Scripts\activate

# 确保安装了所有依赖
pip install langchain langchain-openai langchain-deepseek langgraph streamlit python-dotenv langchain-community langchain-mcp-adapters

# 启动 Streamlit
streamlit run app.py
```

在浏览器中：
1. 访问 `http://localhost:8501`
2. 在侧边栏选择"快速模式"
3. 在对话框输入：`人工智能行业最新动态`
4. 观察执行步骤，等待结果

**验证成功标准：**
- 页面正常加载，无报错
- 能看到执行步骤（`generate_query` → `web_research` → `reflection` → `finalize_answer`）
- 最终生成了一段研究报告

### 5.4 测试金融数据获取（AKShare）

创建测试文件 `test_akshare.py`（在 `21~26.financial` 目录下）：

```python
def test_akshare():
    try:
        import akshare as ak
        print(f"✅ AKShare 版本：{ak.__version__}")
        
        # 测试获取股票基本信息
        df = ak.stock_individual_info_em(symbol="600519")
        print(f"✅ 获取贵州茅台信息成功，共 {len(df)} 行数据")
        print(df.head(3))
        return True
    except Exception as e:
        print(f"❌ AKShare 测试失败：{e}")
        return False

if __name__ == "__main__":
    test_akshare()
```

```cmd
cd C:\Users\你的用户名\Documents\Geek03\21~26.financial
venv\Scripts\activate
python test_akshare.py
```

**预期输出：**
```
✅ AKShare 版本：1.14.x
✅ 获取贵州茅台信息成功，共 x 行数据
   item         value
0  股票代码        600519
1  股票简称        贵州茅台
2  ...
```

### 5.5 完整运行一次金融研报生成

> ⚠️ **警告：** 此步骤需要大量 API 调用，请确保 DeepSeek API 余额充足（建议充值 50 元以上），运行时间约 30-60 分钟。

```cmd
cd C:\Users\你的用户名\Documents\Geek03\21~26.financial
venv\Scripts\activate
python workflow.py
```

**观察输出，成功标志：**

```
🖼️ 处理图片路径...
📋 生成报告大纲...

✍️ 开始分段生成深度研报...
  正在生成：公司概况...
  ✅ 已完成：公司概况
  正在生成：财务分析...
  ✅ 已完成：财务分析
  ...

🎨 格式化报告...
📄 转换为Word文档...
✅ 第二阶段完成！深度研报已保存到: 深度财务研报分析_20241201_120000.md
```

**最终输出文件位置：**
```
21~26.financial/
  final_output/
    深度财务研报分析_时间戳.md    ← Markdown 版本
    深度财务研报分析_时间戳.docx  ← Word 版本
```

---

## 附录：快速问题排查清单

遇到问题时，按以下顺序排查：

```
□ 1. Python 版本是否为 3.11.x？（python --version）
□ 2. 虚拟环境是否已激活？（看到 (venv) 前缀）
□ 3. .env 文件是否在当前目录？（dir .env）
□ 4. .env 格式是否正确？（无空格、无引号）
□ 5. 所有必要的包是否都已安装？（pip list）
□ 6. API Key 是否有效且余额充足？（到对应官网查看）
□ 7. 网络是否正常？（能否访问 platform.deepseek.com）
□ 8. AKShare 是否为最新版本？（pip install akshare --upgrade）
```
