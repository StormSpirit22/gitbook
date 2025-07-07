大家好，我是小开 coding ，去年收获了一个男宝，小名叫开开，新年新气象，于是决定把公众号名字改成小开coding。春节期间 DeepSeek 刷屏了，这里简单讲解一下 DeepSeek 以及本地部署教程。



## 一、什么是 DeepSeek-R1

2025.01.20 DeepSeek-R1 发布，DeepSeek R1 是 DeepSeek AI 开发的第一代推理模型，擅长复杂的推理任务，官方对标OpenAI o1正式版。适用于多种复杂任务，如数学推理、代码生成和逻辑推理等。

根据官方信息DeepSeek R1 可以看到提供多个版本，包括完整版（671B 参数）和蒸馏版（1.5B 到 70B 参数）。完整版性能强大，但需要极高的硬件配置；蒸馏版则更适合普通用户，硬件要求较低。

**蒸馏版与完整版的区别**

| **特性**     | **蒸馏版**                                            | **完整版**                            |
| :----------- | :---------------------------------------------------- | :------------------------------------ |
| **参数量**   | 参数量较少（如 1.5B、7B），性能接近完整版但略有下降。 | 参数量较大（如 32B、70B），性能最强。 |
| **硬件需求** | 显存和内存需求较低，适合低配硬件。                    | 显存和内存需求较高，需高端硬件支持。  |
| **适用场景** | 适合轻量级任务和资源有限的设备。                      | 适合高精度任务和专业场景。            |



## 1.1 如何使用

直接登录官网即可**免费**与 DeepSeek-R1 对话：https://www.deepseek.com/ ，还有手机 app 下载。



## 1.2 本地部署的好处

1. **数据隐私与安全性：**
    所有数据处理和存储均在本地完成，无需上传云端，有效保护个人或企业敏感信息。
2. **定制化与离线使用：**
    可根据自身需求对模型进行微调或定制知识库训练，且在无网络环境下也能使用。
3. **成本控制：**
    长期来看，本地部署避免了高额的云服务费用，适合频繁调用的场景。



## 二、总体架构与工具选择

## 2.1 硬件配置说明

下面整理了一个关于 Deepseek-R1 各版本（例如 1.5B、7B、8B、14B 等）在本地部署时，对于 Windows 与 macOS 平台的硬件推荐配置表。注意：

1. 表中配置仅供参考，实际需求会根据使用场景、上下文长度（num_ctx 参数）以及具体模型（例如蒸馏版与全量版）的差异有所变化；
2. macOS 部分分别给出了 Intel 芯片和 Apple Silicon（M1/M2 系列）两种设备的建议；
3. 对于 GPU 部分，若无独立显卡也可采用 CPU-only 模式，但响应速度可能较慢。

------

| Model Version | Platform                  | Recommended CPU                    | Recommended Memory     | Recommended GPU                                       | Recommended Storage |
| ------------- | ------------------------- | ---------------------------------- | ---------------------- | ----------------------------------------------------- | ------------------- |
| **1.5B**      | **Windows**               | Quad-core (如 Intel i5 或 Ryzen 5) | 8 ~ 16GB               | 可选：若有独立 GPU，建议 ≥4GB VRAM；无则使用 CPU 模式 | 50 ~ 100GB SSD      |
|               | **macOS (Intel)**         | Quad-core (如 Intel i5/i7)         | 8 ~ 16GB               | 可选：集成显卡可应付轻量任务；独立 GPU 效果更佳       | 50 ~ 100GB SSD      |
|               | **macOS (Apple Silicon)** | Apple M1/M2（或更新型号）          | 8GB 最低，建议 16GB    | Apple Silicon 集成 GPU（性能经过专门优化）            | 50 ~ 100GB SSD      |
| **7B**        | **Windows**               | 6 ~ 8 核 (如 Intel i7 或 Ryzen 7)  | 16 ~ 32GB              | 推荐独立 GPU，至少 ≥6GB VRAM（如 RTX 2060 及以上）    | ≥100GB SSD          |
|               | **macOS (Intel)**         | 6 ~ 8 核 (如 Intel i7)             | 16 ~ 32GB              | 建议采用独立 GPU                                      | ≥100GB SSD          |
|               | **macOS (Apple Silicon)** | Apple M1/M2 Pro/Max                | 16GB 最低，建议 32GB   | Apple Silicon 集成 GPU（表现优异）                    | ≥100GB SSD          |
| **8B**        | **Windows**               | 6 ~ 8 核 (如 Intel i7 或 Ryzen 7)  | 16 ~ 32GB              | 推荐独立 GPU，至少 ≥6~8GB VRAM                        | ≥100GB SSD          |
|               | **macOS (Intel)**         | 6 ~ 8 核 (如 Intel i7)             | 16 ~ 32GB              | 建议采用独立 GPU                                      | ≥100GB SSD          |
|               | **macOS (Apple Silicon)** | Apple M1/M2 Pro/Max                | 16GB 最低，建议 32GB   | Apple Silicon 集成 GPU                                | ≥100GB SSD          |
| **14B**       | **Windows**               | 8 核 (如 Intel i7/i9 或 Ryzen 9)   | ≥32GB                  | 高端独立 GPU，建议 ≥8GB VRAM（如 RTX 3070 及以上）    | ≥150GB SSD          |
|               | **macOS (Intel)**         | 8 核 (如 Intel i7/i9)              | ≥32GB                  | 建议采用独立 GPU                                      | ≥150GB SSD          |
|               | **macOS (Apple Silicon)** | Apple M1/M2 Max（或更新型号）      | 最低 32GB，理想为 64GB | Apple Silicon 集成 GPU（性能出色）                    | ≥150GB SSD          |

------

### 说明

- **CPU：** Windows 和 macOS (Intel) 设备建议采用多核处理器；Apple Silicon 设备则依托于其自研 CPU 和 GPU 高效协同。
- **内存：** 对于轻量版模型（1.5B）8GB 内存即可应付基础任务，但为获得更流畅体验，推荐 16GB 及以上；对于 7B 及 8B 版本建议 16–32GB，而较大版本（如 14B）则建议 32GB 以上。
- **GPU：** 如果有独立显卡，建议 Windows/macOS（Intel）用户选择具备 4GB 及以上（轻量版）或 6~8GB 以上（较大版本）的 GPU；Apple Silicon 设备则依赖系统集成 GPU，但高配置机型（如配备 M1/M2 Pro/Max）能提供良好性能。
- **存储：** 模型文件通常较大（根据版本从几十 GB 到上百 GB不等），建议预留足够的 SSD 存储空间，同时考虑到索引文件、缓存数据及临时文件的存储需求。



本教程采用以下工具和流程：

- **Ollama：** 开源大模型运行平台，支持模型下载、管理和 API 调用。
- **Deepseek-R1 模型：** 这里以 8B 版本为例，体积较小。
- **Open WebUI：** 提供基于 Web 的交互界面，让你无需命令行即可对话调试。



## 三、详细步骤

我的本地环境：
机器：MacBook Pro (16GB RAM+)
模型：deepseek-r1:8b

### 1. 安装 Ollama

1.1 **下载安装包：**

- 访问 [Ollama 官网](https://ollama.com/) ![image-20250206164637358](/Users/zonst/Library/Application Support/typora-user-images/image-20250206164637358.png)点击对应系统的下载链接（Windows 用户选择 Windows 版本)。
- 下载完成后安装

![image-20250206165001600](/Users/zonst/Library/Application Support/typora-user-images/image-20250206165001600.png)

- 控制台验证是否成功安装，这样就表示安装成功了

![image-20250206165928713](/Users/zonst/Library/Application Support/typora-user-images/image-20250206165928713.png)



### 2. 下载并部署 Deepseek-R1 模型

2.1 **通过 Ollama 拉取模型：**

- 在浏览器中访问 https://ollama.com/library/deepseek-r1 [Ollama 模型库中的 Deepseek-R1 页面](https://ollama.com/library/deepseek-r1)。

![image-20250206165134129](/Users/zonst/Library/Application Support/typora-user-images/image-20250206165134129.png)

- 根据自己电脑配置，建议选择参数较小的 8B 版本。

- 在 CMD 中执行以下命令自动下载并部署模型：

  ```bash
  ollama run deepseek-r1:8b
  ```

  执行后，系统会自动下载模型文件，下载完成后进入交互模式。

  ![image-20250206170004632](/Users/zonst/Library/Application Support/typora-user-images/image-20250206170004632.png)



### 3. 测试 Deepseek 模型交互

3.1 **命令行交互测试：**

- 在 CMD 窗口中执行：

  ```bash
  ollama run deepseek-r1:8b
  ```

- 你将进入一个交互界面，可以输入问题，例如：

  ![image-20250206170052749](/Users/zonst/Library/Application Support/typora-user-images/image-20250206170052749.png)

3.2 **注意：**

- 如果交互过程中响应较慢，可多按几次回车，耐心等待下载完成和加载模型。

------

### 4. 安装 Open WebUI（增强交互体验）

4.1 **安装 Open WebUI：**

Open-WebUI官方文档地址：https://docs.openwebui.com/getting-started/

![image-20250206170342830](/Users/zonst/Library/Application Support/typora-user-images/image-20250206170342830.png)

这里直接使用 docker 安装：

```shell
docker run -d -p 3000:8080 -v open-webui:/app/backend/data --name open-webui ghcr.io/open-webui/open-webui:main
```

如果没有安装过 docker，则从 https://www.docker.com/ [Docker 官方](https://www.docker.com/) 下载 [Docker 安装程序](https://www.docker.com/products/docker-desktop/)。依照提示完成安装即可。

4.2 **访问 Open WebUI：**

- 打开浏览器，访问 http://localhost:3000/，进入 WebUI 界面。

- 首次使用需要注册管理员账号，按提示填写信息完成注册

4.4 **在 Open WebUI 中配置 Deepseek 模型：**

- 登录后，进入模型管理页面，选择“添加本地模型”，并填写模型名称（例如：Deepseek-R1-8b）和运行命令。

![image-20250206171642721](/Users/zonst/Library/Application Support/typora-user-images/image-20250206171642721.png)

- 保存设置后，在主界面点击对应模型，即可开始与模型对话。

![image-20250206171628964](/Users/zonst/Library/Application Support/typora-user-images/image-20250206171628964.png)



## 四、常见问题与注意事项

1. **下载及部署过程缓慢？**
    检查网络环境，建议使用清华、阿里镜像加速 pip 下载。
2. **Ollama 模型下载失败？**
    可多次尝试或使用 VPN 解决网络问题。
3. **内存不足或响应过慢？**
    请检查系统资源，必要时关闭其他占用资源的软件。



## 五、总结

本地部署的蒸馏版对于回答的代码内容有些不尽人意，而且有时候思考时间会过长，特别吃机器性能。如果想在后续体验完整版的Deepseek，还没有高性能的硬件，那么直接使用deepseek官方的服务吧。

点击屏幕左下方的【关注】按钮，对话框里输入 **deepseek**，即可收到一份 29 页的 DeepSeek 使用教程 pdf，手慢无！