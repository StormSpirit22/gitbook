大家好，我是小开 coding，昨天的文章是基于 mac 环境搭建的 DeepSeek 本地环境，有用户问 windows 怎么搭建。今天就来手把手教大家在 windows 环境本地部署 DeepSeek 以及搭建自己的知识库。全程无卡点，零基础上手 0 难度！

机器硬件配置请参考昨天的文章，这里就不赘述，这里模型还是选择 8b。

## 一、windows 本地部署

### 1. 安装 Ollama

- 访问 [Ollama 官网](https://ollama.com/) https://ollama.com/download ，下载 windows 版本，下载好后根据提示安装。

![image-20250207122715078](/Users/zonst/Library/Application Support/typora-user-images/image-20250207122715078.png)

### 2. 本地部署 DeepSeek-R1 8b

- 安装成功后，在 windows 搜索框里输入 **cmd**，点击命令提示符，在命令提示符里输入

```shell
ollama run deepseek-r1:8b
```

开始自动下载安装 deepseek-r1 8b 版本。

- 下载完成后即自动运行 deepseek，可以开始提问了，具体操作步骤如下图，可以看到我提问的 "小开coding是什么"，DeepSeek 并不知道，只能自己推理。

![301738887277_.pic](/Users/zonst/Downloads/301738887277_.pic.jpg)



## 二、搭建本地知识库

### 1. 下载 shaw/dmeta-embedding-zh

#### 1.1 dmeta-embedding-zh 是什么

> 在知识库应用中，文本数据通常是以大量文档的形式存在，传统的关键词搜索往往只能进行字面匹配，容易漏掉语义相近但表达不同的内容。通过使用 dmeta-embedding-zh 模型，你可以把知识库中所有文档转换成向量，同时将用户的查询也转换成向量，然后计算这些向量之间的相似度，从而实现基于语义的检索。这种方式大大提高了搜索的精准度和灵活性，使得知识库能够更智能地“理解”用户的意图，并返回更相关的信息。
>
> 这种模型在现代知识库搭建中非常关键，因为它为构建一个“语义知识库”提供了底层支持，让系统不仅仅停留在关键词匹配上，而是能真正理解内容的深层含义。

简单来说，这个模型就是用来整理你的文档，为 DeepSeek 模型更好的理解并运用你的知识库。

### 1.2 安装 shaw/dmeta-embedding-zh

命令提示符里输入

```shell
ollama pull shaw/dmeta-embedding-zh
```

等待下载完成：

![291738887246_.pic](/Users/zonst/Downloads/291738887246_.pic.jpg)



### 2. 搭建本地知识库

#### 2.1 下载安装 cherry studio

官网地址：https://cherry-ai.com/download ，如果打不开或者点击下载没反应，可以按照文末提示领取福利，里面有最新版的 windows 和 mac 的 cherry studio 的安装包。

#### 2.2 配置 ollama DeepSeek 模型

安装好 cherry studio 打开后，依次按以下步骤操作：

- 点击左下角设置，选择模型服务，选择 `Ollama`，点击管理。
- 选择 `deepseek-r1:8b` 和 `shaw/dmeta-embedding-zh:latest` 。

步骤图：

![image-20250207124519536](/Users/zonst/Library/Application Support/typora-user-images/image-20250207124519536.png)



- 注意：我这里**已经添加过了，是添加后的示意图**，所以显示的是 "-" 按钮，没添加过显示的是 "+" 按钮。点击一下 "+" 按钮即可。

![image-20250207124530387](/Users/zonst/Library/Application Support/typora-user-images/image-20250207124530387.png)



这样就添加好了模型。

验证模型：

- 点击左上角的对话按钮，选择 `deepseek-r1:8b|Ollama`，即可开始对话：

![image-20250207124825618](/Users/zonst/Library/Application Support/typora-user-images/image-20250207124825618.png)



#### 2.3 搭建知识库

依次按以下步骤操作：

- cherry studio 选择侧边栏知识库图标，点击添加，输入名称，嵌入模型选择之前添加过的 `shaw/dmeta-embedding-zh:latest`。
- 文件里添加你需要构建知识库用的文档，支持 pdf、docx、pptx、xlsx、txt、md 格式。也可以选择网址、网站。

我这里新建了一个 test.txt，里面就一句话，步骤图：

![image-20250207140008186](/Users/zonst/Library/Application Support/typora-user-images/image-20250207140008186.png)

![image-20250207140013918](/Users/zonst/Library/Application Support/typora-user-images/image-20250207140013918.png)

![image-20250207140019154](/Users/zonst/Library/Application Support/typora-user-images/image-20250207140019154.png)

添加好后回到对话框测试。

重新提问相同问题，可以看到它会引用知识库里的文档来回答问题了。

![image-20250207140051249](/Users/zonst/Library/Application Support/typora-user-images/image-20250207140051249.png)

### 三、结语

通过本地部署DeepSeek+知识库的好处：

- 隐私性很好，不用担心自己的资料外泄、离线可用。
- 在工作和学习过程中对自己整理的文档，能快速找到，并自动关联。
- 在代码开发上，能参考你的开发习惯，快速生成代码。

**文末福利**：关注公众号回复“deepseek”获取 cherry-studio 安装包+长达 29 页的使用教程！