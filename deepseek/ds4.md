大家好，我是小开 coding。前几天的文章主要介绍了 DeepSeek 的本地部署和知识库搭建的方案。今天我们来介绍如何使用 DeepSeek 来一键生成小红书爆款内容，更好地调教 DeepSeek 为我们所用。



## STAR 原则

STAR 原则，即Situation（情景）、Task（任务）、Action（行动）和 Result（结果）四个英文单词的首字母组合。*STAR法则其实是一种关于在简历上进行结构化叙述的原则*，可以显著突出提升简历的亮点，增加面试官的认可度。在这里**我们可以使用 STAR 法则给 DeepSeek 提问**。



## 提问

> S:在竞争激烈的小红书平台，T:我需要一篇吸引用户的养生文案，A：请结合心理学中的稀缺效应和视觉优先原则，R：目标是让点击率提升30%。 将上述回答所有内容，以小红书卡片风格生成html代码，要求如下： 1. 每块文本对应一个卡片,每张卡片都提供下载为 png 的按钮，生成的图片中不要包括这个按钮。 2. 生成出来的图片排版合理，美观大气 3. 卡片背景采用点阵尾纹 4. 卡片增加纸张投影 5. 色系调整为小红书用户喜欢的色系 6. 你的小红书内容里需要添加具体的养生内容。所有内容帮我填充到html。

生成 html 的好处是直接生成可下载的 png 图片，下载后直接发小红书即可。DeepSeek 生成的 html 代码：

```html
<!DOCTYPE html>
<html>
<head>
    <title>养生文案卡片</title>
    <style>
        body {
            background: #fff3f5;
            padding: 30px;
            display: grid;
            gap: 25px;
            font-family: 'Microsoft YaHei', sans-serif;
        }

        .card-container {
            position: relative;
            width: 375px;
            min-height: 400px;
            padding: 25px;
            background: 
                linear-gradient(45deg, #ffeded 2%, transparent 2%),
                linear-gradient(-45deg, #ffeded 2%, transparent 2%);
            background-size: 20px 20px;
            background-color: white;
            border-radius: 15px;
            box-shadow: 
                0 4px 20px rgba(250, 114, 104, 0.1),
                0 8px 30px rgba(250, 114, 104, 0.15);
            color: #333;
        }

        .download-btn {
            position: absolute;
            bottom: 15px;
            right: 15px;
            background: #ff7c7c;
            color: white;
            border: none;
            padding: 8px 15px;
            border-radius: 20px;
            cursor: pointer;
            font-size: 12px;
            transition: 0.3s;
        }

        .download-btn:hover {
            background: #ff5a5a;
        }

        .content {
            margin: 15px;
            line-height: 1.6;
        }

        h2 {
            color: #ff5a5a;
            border-left: 4px solid #ffb6b6;
            padding-left: 10px;
            margin: 15px 0;
        }

        .highlight {
            color: #ff5a5a;
            font-weight: bold;
        }
    </style>
    <script src="https://html2canvas.hertzen.com/dist/html2canvas.min.js"></script>
</head>
<body>
    <!-- 卡片1 -->
    <div class="card-container" id="card1">
        <div class="content">
            <h2>🌸 限时养生秘籍 · 最后30份 🌸</h2>
            <p>抓住身体黄金修复期！<span class="highlight">独家中医配方</span>首次公开👉</p>
            <p>🔸 三招唤醒代谢力 🔸<br>
            🔸 熬夜补救方案 🔸<br>
            🔸 节气养生食谱 🔸</p>
            <p>⚠️ 仅限前100名领完整方案<br>
            ⏳ 倒计时3天失效</p>
        </div>
        <button class="download-btn" onclick="downloadCard('card1')">⬇️ 保存图片</button>
    </div>

    <!-- 卡片2 -->
    <div class="card-container" id="card2">
        <div class="content">
            <h2>💎 稀缺养生指南 · 手慢无 💎</h2>
            <p>中医世家的<span class="highlight">传女秘方</span>首曝光❗<br>
            👉 经期调理黄金公式<br>
            👉 气血双补食疗谱<br>
            👉 抗衰按摩图解</p>
            <p>🔥 前50名送定制养生日历<br>
            📌 私藏技巧首次公开<br>
        </div>
        <button class="download-btn" onclick="downloadCard('card2')">⬇️ 保存图片</button>
    </div>
    <!-- 卡片3：代谢调理 -->
    <div class="card-container" id="card1">
        <div class="content">
            <h2>🔥 代谢唤醒三部曲 · 限时公开</h2>
            <div class="tip-box">
                <p><span class="highlight">晨间拍打法：</span><br>
                ▶ 沿胆经敲打大腿外侧（风市穴→阳陵泉）<br>
                ▶ 每侧拍打100次，唤醒肝胆代谢</p>
                
                <p><span class="highlight">午间茶饮：</span><br>
                🍵 荷叶3g + 山楂5g + 陈皮2g<br>
                80℃热水焖泡15分钟</p>
                
                <p><span class="highlight">夜间泡脚方：</span><br>
                🌿 艾叶20g + 老姜10g + 花椒15粒<br>
                煮开后兑温水至40℃，泡20分钟</p>
            </div>
            ⏳ 配方保留至本周五</p>
        </div>
        <button class="download-btn" onclick="downloadCard('card1')">💾 保存方案</button>
    </div>

    <!-- 卡片4：气血调理 -->
    <div class="card-container" id="card2">
        <div class="content">
            <h2>🌺 气血双补秘籍 · 家传配方</h2>
            <div class="tip-box">
                <p><span class="highlight">五红升级粥：</span><br>
                🥣 红米+红皮花生+红豆各30g<br>
                ➕ 枸杞15g + 红枣5颗（去核）<br>
                ➕ 桂圆肉10g + 红糖1块</p>
                
                <p><span class="highlight">经期调理法：</span><br>
                ▶ 经前3天：三阴交艾灸15分钟/日<br>
                ▶ 经期第2天：喝红糖姜枣茶<br>
                ▶ 经后7天：四物汤（熟地12g+当归10g+白芍12g+川芎8g）</p>
            </div>
            <p>🎁 前50名送《节气养生日历》<br>
        </div>
        <button class="download-btn" onclick="downloadCard('card2')">💾 保存方案</button>
    </div>
    <script>
        function downloadCard(cardId) {
            const card = document.getElementById(cardId);
            const btn = card.querySelector('.download-btn');
            btn.style.display = 'none';
            
            html2canvas(card).then(canvas => {
                const link = document.createElement('a');
                link.download = '养生卡片.png';
                link.href = canvas.toDataURL();
                link.click();
                btn.style.display = 'block';
            });
        }
    </script>
</body>
</html>
```

把以上 html 代码复制，保存到文件，文件后缀为.html:![image-20250211225604160](/Users/zonst/Library/Application Support/typora-user-images/image-20250211225604160.png)

再双击，或者拖到浏览器打开，效果如下：

<img src="/Users/zonst/Library/Application Support/typora-user-images/image-20250211225859980.png" alt="image-20250211225859980" style="zoom:50%;" />

<img src="/Users/zonst/Library/Application Support/typora-user-images/image-20250211225911145.png" alt="image-20250211225911145" style="zoom:50%;" />

点击保存方案即可直接下载图片，上传到小红书啦。看看这样的文案能收获多少赞呢？

同样可以选择不同的赛道或者话题，按照以上的提问方式给 DeepSeek，让它生成代码下载，也可以让它同时可以生成多份文案～



## 网友恶搞

有网友提问 DeepSeek，你和 ChatGPT 谁更厉害，可能是喂了一个贴吧老哥给 DeepSeek，请看下图 DeepSeek 的回答，属实让人绷不住。

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-axegupay5k/9b0eab05d28c43a1abf42af0fec93084~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1739889105&x-signature=XMTUo6X77qf8HCuqJ0hq0gKFJeg%3D)



## 文末福利

关注公众号回复 "deepseek"，领取 DeepSeek 本地部署工具包以及使用教程（已更新）！

![image-20250211230327200](/Users/zonst/Library/Application Support/typora-user-images/image-20250211230327200.png)

