大家好，我是小开 coding。前两天的文章主要介绍了本地部署 DeepSeek 的流程。但是我们普通人的机器本地部署的都是 DeepSeek R1 的蒸馏小模型，显卡 4090 也最多就跑到 32b 的版本。DeepSeek R1原版模型的大小是671B，也就是有6710亿参数，基本没有任何电脑能跑得起来。再加上最近官网白天大概率是服务器繁忙，所以对于大众来说，更好的使用方案是使用第三方整合嵌入了DeepSeek R1的满**配版本**。



## 推荐方案

### 1. 秘塔搜索

官网： https://metaso.cn/

注册登录之后就可以直接使用满血版 DeepSeek R1 推理模型，有简洁、深入、研究三种模式可选：

![image-20250208143113564](/Users/zonst/Library/Application Support/typora-user-images/image-20250208143113564.png)

可以让你获得一个很长很完整的回答，特别适合用于搜集整理资料。秘塔搜索比较赞的是每个用户每天可以免费使用100次的R1推理。基本可以满足大部分需求。



### 2. 纳米 AI 搜索

![image-20250208150917936](/Users/zonst/Library/Application Support/typora-user-images/image-20250208150917936.png)

使用的时候选择慢思考模式。目前流量大了，优先使用360的智脑。



**但是这两种方式不会保存你之前的问题记录**，刷新一下就没了。更好的解决方案看下面。



### 3. 硅基流动

邀请注册链接： https://cloud.siliconflow.cn/i/6PDnXYiB

API的价格和 DeepSeek 官方优惠期价格保持一致。 **SiliconCloud 邀请别人注册，您与好友均可获赠2000万Tokens（14元平台配额）每次邀请家人朋友注册一下，你就有2000万Tokens，可以用很久很久**。

注册后按以下步骤进行操作：

- 点击 API 密钥，新建密钥。
- 点击密钥复制。

![image-20250208144111446](/Users/zonst/Library/Application Support/typora-user-images/image-20250208144111446.png)



![image-20250208144151406](/Users/zonst/Library/Application Support/typora-user-images/image-20250208144151406.png)



再打开 cherry studio（没有安装的朋友请看上一篇文章），点击设置，模型服务，选择硅基流动，在 API 密钥一栏粘贴刚刚复制的密钥，点击检查，选择 deepseek-ai/DeepSeek-R1，点击确定。

![image-20250208144505945](/Users/zonst/Library/Application Support/typora-user-images/image-20250208144505945.png)

最后回到对话框，选择硅基流动的 DeepSeek-R1，就可以开始对话啦。

![image-20250208145048285](/Users/zonst/Library/Application Support/typora-user-images/image-20250208145048285.png)

不过现在 DeepSeek 官方 API 很慢，这就导致用第三方调用也很慢，实测不是很好用💦。

最后，如果觉得 DeepSeek R1 的蒸馏小模型就可以满足需求，就可以用本地部署+知识库的方式来**流畅无卡顿**使用 DeepSeek～

## 另外

如果你想直接在客户端应用里体验 DeepSeek-R1 & V3 模型，可在本地安装以下产品，添加对应的密钥，即可体验 DeepSeek-R1 & V3。

- 大模型客户端应用：Cherry Studio、ChatBox、OneAPI、LobeChat、NextChat
- 代码生成应用：Cursor、Windsurf、Cline
- 大模型应用开发平台：Dify
- AI 知识库：Obsidian AI、FastGPT、anythingllm
- 翻译插件：沉浸式翻译、欧路词典

**文末福利**：关注公众号回复“deepseek”获取 cherry-studio 安装包+长达 29 页的使用教程！