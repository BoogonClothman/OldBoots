\[ [English](README.md) / 简体中文 \]

# 布洛洛原创开源测试系统（Boots）

**！注意：你应该检查另外一个叫做Boots的仓库。请在[这里](https://github.com/BoogonAgora/Boots)查看**

一个用于运行像布洛洛一样的AI虚拟主播的简单项目。

## 如何部署

**这部分内容是您能否正常运行本项目的关键，请您务必按照指示进行操作。**

运行本项目之前，请先确认以下内容：
1. 您拥有足够的算力水平，可以运行您计划使用的模型
2. 您拥有您计划使用的大语言模型文件，其格式为GGUF，或有方法将其转变为GGUF格式
3. 您对Python足够了解，能够根据注释及相关信息对代码进行合理的、有目的的调整

首先克隆本仓库：
```bash
git clone https://github.com/BoogonClothman/OldBoots.git
cd OldBoots
```

接下来按照`requirements.txt`的内容，安装指定版本的第三方库。建议使用虚拟环境，Python版本建议在`3.9`到`3.12`之间，不要使用`3.13`版本，因为很多库还没有支持这个版本。开发过程中使用了`3.11`版本。

```bash
pip install -i requirements.txt
```

有关安装，在这里提供几点建议：

1. 项目中使用了**CUDA**加速计算，所以代码更加偏向**CUDA**，如果你需要使用其他平台，建议结合后文的解析对项目进行深度重构
2. 如果出现`CMake`工具编译失败等安装问题，建议像作者一样前往对应的官方仓库寻找预编译Wheel文件进行安装
3. 本项目并不完善，也没有在PYPI仓库注册，不能`pip install boots`直接安装本项目。如果您有更好的想法或更合适的第三方库选择，欢迎进行扩展和提出issue

安装完毕后，请检查仓库目录结构是否如下：
```
Boots/
  ├─ Core/               # 核心库
  │  ├─ Context/         # 1. 上下文管理器
  │  │  └─ context.py
  │  ├─ Filter/          # 2. 过滤器
  │  │  └─ filter.py
  │  ├─ LLM/             # 3. 生成器
  │  │  └─ langmodel.py
  │  ├─ Memory/          # 4. 记忆管理
  │  │  └─ memory.py
  │  ├─ TTS/             # 5. 语音合成接口
  │  │  └─ tts.py
  │  ├─ __init__.py
  │  └─ core.py          # 核心库接口，需调整内容（见后续内容）
  ├─ documents/          # 参考文档目录
  ├─ Framework/          # 框架库
  │  ├─ Frontend/        # 1. 前端
  │  │  ├─ .static/      #   存放静态资源
  │  │  │  └─ ...
  │  │  ├─ frontend.py   #   前端服务器程序
  │  │  └─ index.html    #   前端渲染页面，需调整内容（见后续内容）
  │  ├─ InputManager/    # 2. 输入管理器
  │  │  ├─ inputmgr.py
  │  │  └─ textbench.py
  │  └─ __init__.py
  ├─ output/
  │  └─ .tts/            # TTS输出目录
  ├─ resources/          # 资源目录
  │  ├─ .bad/            # 禁用词文件目录
  │  ├─ .llm/            # GGUF模型文件目录
  │  └─ .rag/            # 向量化嵌入模型目录
  │     ├─ models/
  │     ├─ index.faiss   # 嵌入索引文件
  │     └─ texts.pkl     # 内容查询文件
  ├─ main.py             # 主程序
  └─ requirements.txt
```
如果有目录缺失，请在运行前首先新建文件夹，并暂时先保持与上述结构一致（您也可以调整结构，并对代码中的预设路径进行修改）；如果有文件缺失，例如`index.faiss`和`texts.pkl`等中间文件，这些文件会在运行时生成，其他资源性的文件需要自行添加，例如大语言模型文件、嵌入模型文件、禁用词文件、参考文档、Live2D模型文件等。

应该指出，**本项目并不是开箱即用的**，受限于您自己的各方面选择。这里将从上述目录自上而下进行调整。

1. `OldBoots/Core/core,py`
```py
# ...
        # Memory
        self.rag = MemoryManager(
            ragmodel_path="GanymedeNil/text2vec-large-chinese",  # 默认使用的嵌入模型，您可选择其他嵌入模型
            ...
        )
        self.rag.create_index()

# ...
        # LLM
        self.llm = LangModel(
            model_path="./resources/.llm/<your-llm-gguf>",  # 请务必将这里调整为您的LLM模型的GGUF文件路径！
            ...
        )

# ...
        # TTS
        self.ttse = TTSEngine(
            "zh-CN-XiaoyiNeural",  # 您可以修改这里的音色参数，使之更符合您的模型形象。这里使用Edge-TTS作为TTS引擎
            ...
        )
```

2. `OldBoots/Framework/Frontend/.static/`

   请在这里放入您计划使用的Live2D模型文件，其中包括但不限于`***.model3.json`、`***.moc3`以及纹理层文件。

3. `OldBoots/Framework/Frontend/index.html`
```html
<script>
        /******************** Live2D模型 ********************/
        /* 请将这里的路径改为您的Live2D模型路径 */
        const model = await PIXI.live2d.Live2DModel.from("./static/<your-model>.model3.json", {autoInteract: false});
</script>
```

4. `OldBoots/main.py`
```py
# ...
    def __init__(self):
        self.core = BootsCore()
        self.core.set_sysprompt("")  # 在这里修改系统提示词
        self.inputmgr = InputManager("text", on_commit_callback=self.handle_input)
        self.frontend = Frontend("localhost", 5000)  # 这里可以修改IP，在局域网内可访问
# ...
```

修改完毕后，运行`main.py`即可启动项目，在`http://localhost:5000/`可以看到Live2D模型的界面。您应该可以看见一个基于tkinter的输入界面，并在网页中看到AI回答内容渲染出的字幕。

## 硬件需求

项目开发过程中使用的是笔记本电脑：
* CPU: Intel Core i9-13900HX
* GPU: NVIDIA GeForce RTX 4060 (Laptop)
* 内存: 64G

开发过程中，我选择了Smegmma-9B的Q3量化模型作为测试，实际显存占用6.4G左右，回答延迟在1s以内（不考虑Edge-TTS的RTT影响下）

## 项目简析

1. **输入管理器**中获取到用户输入
2. 传递给**记忆管理**模块检索已有的参考信息
3. 输入和参考信息以及**上下文管理器**中存放的历史对话一并传入**生成器**的输入接口
4. **生成器**传出模型回答结果，结果经过**过滤器**过滤得到合适的输出
5. 通过**TTS引擎**接口获得声音并播放，同时向**前端**发送字幕数据在**前端**实时渲染
6. **前端**同步进行Live2D模型的动画。
