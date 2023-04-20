# Docker简易操作


环境：
* 机器：mbp m1 pro
* python：Python 3.9.6
* pip: pip 21.2.4 from /Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.9/lib/python3.9/site-packages/pip (python 3.9)
  
注意：
本次安装使用代理并启动了终端代理

## 安装配置 git-lfs
```shell
# 安装
brew install git-lfs
# 验证安装
git lfs install
```

## 下载代码
```shell
# 下载代码
git clone https://github.com/THUDM/ChatGLM-6B.git

# 安装依赖
cd ChatGLM-6B && pip3 install -r requirements.txt
```

## 下载并验证模型
```shell
# 下载模型
git clone https://huggingface.co/THUDM/chatglm-6b # 此过程会耗时一段时间，你可以去接杯水
# 进入模型所在目录
cd chatglm-6b
# 验证模型
shasum -a 256 ice_text.model
# 如果输出“5e974d9a69c242ce014c88c2b26089270f6198f3c0b700a887666cd3e816f17e  ice_text.model” 则表示下载正确
```

## 修改部分代码
```python
# 修改模型路径，将原始的 THUDM/chatglm-6b 替换为模型所在目录的绝对路径， /Users/ares/workspace/python/projects/ChatGLM-6B/THUDM/chatglm-6b 为我的本地的路径

# 按照官方文档，Apple Silicon的 Mac 可以开启终端加速，即，将 half().cuda() 替换为 half().to('mps')
tokenizer = AutoTokenizer.from_pretrained("/Users/ares/workspace/python/projects/ChatGLM-6B/THUDM/chatglm-6b", trust_remote_code      =True)
model = AutoModel.from_pretrained("/Users/ares/workspace/python/projects/ChatGLM-6B/THUDM/chatglm-6b", trust_remote_code=True).half().cuda()
```

## 启动交互环境
```
python cli_demo.py
```

## Reference
[用 10 万条微信聊天记录和 280 篇博客文章，我克隆了一个数字版自己](https://sspai.com/post/79230)
[ChatGLM-6B README.md](https://github.com/THUDM/ChatGLM-6B/blob/main/README.md)
