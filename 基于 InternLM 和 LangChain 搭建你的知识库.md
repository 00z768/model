

# 基于 InternLM 和 LangChain 搭建你的知识库

## 环境配置

### InterLM的环境

平台使用A100(1/4) 和 cuda11.7 conda环境

下载pytorch2.0.1环境

下载internlm-chat-7b模型

### LangChain相关环境配置

安装相关依赖 Langchain gradio chromadb sentence-transformers unstrctured markdown

需要使用到开源词向量模型 [Sentence Transformer](https://huggingface.co/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2)来进行 Embedding

### 下载NLTK相关资源

在使用开源词向量模型构建开源词向量的时候，需要用到第三方库 `nltk` 的一些资源，使用以下命令安装：

```
cd /root
git clone https://gitee.com/yzy0612/nltk_data.git  --branch gh-pages
cd nltk_data
mv packages/*  ./
cd tokenizers
unzip punkt.zip
cd ../taggers
unzip averaged_perceptron_tagger.zip
```

### 从 GitHub clone本项目代码

```
cd /root/data
git clone https://github.com/InternLM/tutorial
```



## 知识库搭建

### 数据收集

选择由上海人工智能实验室开源的一系列大模型工具开源仓库作为语料库来源，

这里将上述仓库中所有的markdown、txt文件作为示例语料库。如果希望将代码文件加入到知识库中，最好对代码基于模块进行分割再加入向量数据库。

克隆以上仓库

递归指定文件夹路径，返回其中所有满足条件（即后缀名为 .md 或者 .txt 的文件）的文件路径（os.walk）

### 加载数据

得到所有目标文件路径之后，使用 `LangChain` 提供的`document_loaders` 对象来加载目标文件，得到由目标文件解析出的纯文本内容。

由于不同类型的文件需要对应不同的 `document_loaders`，对于 'md' 结尾的数据，使用`UnstructuredMarkdownLoader`。

- 对于 'txt' 结尾的文本，使用 `UnstructuredFileLoader` 方法来得到加载之后的纯文本对象。

### 构建向量数据库

使用LangChain中的字符串递归分割器`RecursiveCharacterTextSplitter`进行文本切分，并选择分块大小为 500，块重叠长度为 150

使用开源词向量模型 [Sentence Transformer](https://huggingface.co/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2) 来进行文本向量化

选择 Chroma 作为向量数据库，基于上文分块后的文档以及加载的开源向量化模型，将语料加载到指定路径下的向量数据库

## 接入LangChain

继承 LangChain 的 LLM 类自定义一个 InternLM_LLM 子类

重写构造函数与 `_call` 函数 ，调用初始化的一开始加载本地部署的 InternLM 模型

调用已实例化模型的 chat 方法，从而实现对模型的调用并返回调用结果



## 构建检索问答系统

### 加载向量数据库

将上文构建的向量数据库导入进来，我们可以直接通过 Chroma 以及上文定义的词向量模型来加载已构建的数据库

构建的`vectordb`对象可以针对用户的 `query` 进行语义向量检索，得到与用户提问相关的知识片段

### 实例化自定义的LLM和Prompt Template

实例化一个LLM对象

构建检索问答链，和Prompt Template，这个Template会讲检索到的文档片段填入Template中，实现带有知识的Prompt

### 构建检索问答链

调用 LangChain 提供的检索问答链`RetrievalQA`构造函数，基于自定义 LLM、Prompt Template 和向量知识库来构建一个基于 InternLM 的检索问答链



## 部署Web Demo

将上文的代码内容封装为一个返回构建的检索问答链对象的函数，并在

启动 Gradio 调用该函数得到检索问答链对象，直接使用该对象进行问答对话，从而避免重复加载模型