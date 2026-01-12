\[ English / [简体中文](readme-zh.md) \]

# Brollo Original and Open-Source Testing System (Boots)

**!Attention: You should check another repo called Boots for more details. Check it [HERE](https://github.com/BoogonAgora/Boots)**

A simple project for running AI VTuber like my Brollo.

## Deployment Guide

**This section contains critical information for proper project operation. Please follow the instructions carefully.**

Before running this project, please ensure:

1. You have sufficient computational resources to run your intended models

2. You possess the large language model files you plan to use in GGUF format, or have means to convert them to GGUF

3. You have adequate Python knowledge to make reasonable, purposeful code adjustments based on comments and related information

First, clone this repository:

```bash
git clone https://github.com/BoogonClothman/OldBoots.git
cd OldBoots
```

Next, install the required third-party libraries as specified in `requirements.txt`. Using a virtual environment is recommended. Python versions between `3.9` and `3.12` are suggested - do not use `3.13` as many libraries don't yet support it. Development used `3.11`. 

```bash
pip install -i requirements.txt
```

Some installation recommendations:

1. The project uses **CUDA** for accelerated computing, so the code is **CUDA**-oriented. If you need other platforms, consider deep refactoring based on later explanations.
2. For installation issues like `CMake` compilation failures, follow the author's approach of finding precompiled Wheel files from official repositories.
3. This project isn't complete and isn't registered on PYPI, so you can't `pip install boots` directly. If you have better ideas or alternative library choices, feel free to extend the project or submit issues.

After installation, verify the directory structure matches:
```
Boots/
  ├─ Core/               # Core library
  │  ├─ Context/         # 1. Context manager
  │  │  └─ context.py
  │  ├─ Filter/          # 2. Filter
  │  │  └─ filter.py
  │  ├─ LLM/             # 3. Generator
  │  │  └─ langmodel.py
  │  ├─ Memory/          # 4. Memory management
  │  │  └─ memory.py
  │  ├─ TTS/             # 5. Speech synthesis interface
  │  │  └─ tts.py
  │  ├─ __init__.py
  │  └─ core.py          # Core library interface (requires adjustments)
  ├─ documents/          # Reference documents
  ├─ Framework/          # Framework library
  │  ├─ Frontend/        # 1. Frontend
  │  │  ├─ .static/      #   Static resources
  │  │  │  └─ ...
  │  │  ├─ frontend.py   #   Frontend server
  │  │  └─ index.html    #   Frontend rendering page (requires adjustments)
  │  ├─ InputManager/    # 2. Input manager
  │  │  ├─ inputmgr.py
  │  │  └─ textbench.py
  │  └─ __init__.py
  ├─ output/
  │  └─ .tts/            # TTS output directory
  ├─ resources/          # Resource directory
  │  ├─ .bad/            # Banned words
  │  ├─ .llm/            # GGUF model files
  │  └─ .rag/            # Vector embedding models
  │     ├─ models/
  │     ├─ index.faiss   # Embedding index
  │     └─ texts.pkl     # Content reference
  ├─ main.py             # Main program
  └─ requirements.txt
  ```

  If directories are missing, create them before running (or modify the structure and adjust path references in code). Missing files like `index.faiss` and `texts.pkl` will be generated during runtime, while resource files (LLM models, embedding models, banned word lists, reference documents, Live2D models) must be added manually.

  **This project is not plug-and-play** - it depends on your specific choices. Below are the key adjustment points:

1. `OldBoots/Core/core,py`
```py
# ...
        # Memory
        self.rag = MemoryManager(
            ragmodel_path="GanymedeNil/text2vec-large-chinese",  # Default embedding model
            ...
        )
        self.rag.create_index()

# ...
        # LLM
        self.llm = LangModel(
            model_path="./resources/.llm/<your-llm-gguf>",  # MUST point to your GGUF model!
            ...
        )

# ...
        # TTS
        self.ttse = TTSEngine(
            "zh-CN-XiaoyiNeural",  # Modify voice parameter (using Edge-TTS)
            ...
        )
```

2. `OldBoots/Framework/Frontend/.static/`

   Place your Live2D model files here, including but not limited to `***.model3.json`, `***.moc3` and texture files.

3. `OldBoots/Framework/Frontend/index.html`
```html
<script>
        /******************** Live2D模型 ********************/
        /* Update path to your Live2D model */
        const model = await PIXI.live2d.Live2DModel.from("./static/<your-model>.model3.json", {autoInteract: false});
</script>
```

4. `OldBoots/main.py`
```py
# ...
    def __init__(self):
        self.core = BootsCore()
        self.core.set_sysprompt("")  # Modify system prompt here
        self.inputmgr = InputManager("text", on_commit_callback=self.handle_input)
        self.frontend = Frontend("localhost", 5000)  # Modify IP for LAN access
# ...
```
After modifications, run `main.py` to launch. Access the Live2D interface at `http://localhost:5000/`. You should see a tkinter input window with AI responses rendered as subtitles in the web view.

## Hardware Requirements

Developed on a laptop with:

* CPU: Intel Core i9-13900HX

* GPU: NVIDIA GeForce RTX 4060 (Laptop)

* RAM: 64GB

Testing used Smegmma-9B Q3 quantized model (~6.4GB VRAM usage, <1s response latency excluding Edge-TTS RTT).

## Project Overview

Workflow:

1. **Input manager** receives user input

2. **Memory management** retrieves relevant information

3. Input, references, and conversation history from **context manager** feed into the **generator**

4. **Generator** output passes through the **filter** for refinement

5. **TTS engine** produces voice output while **frontend** renders subtitles

6. **Frontend** synchronizes Live2D model animations
