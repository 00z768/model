# 环境配置

- 基础环境同上节课
- 安装环境依赖

```Bash
# 升级pip
python -m pip install --upgrade pip

pip install modelscope==1.9.5
pip install transformers==4.35.2
pip install streamlit==1.24.0
pip install sentencepiece==0.1.99
pip install accelerate==0.24.1
```

- 安装LangChain相关

```Bash
pip install langchain==0.0.292
pip install gradio==4.4.0
pip install chromadb==0.4.15
pip install sentence-transformers==2.2.2
pip install unstructured==0.10.30
pip install markdown==3.3.7
```

- 下载开源词向量模型 [Sentence Transformer](https://huggingface.co/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2)

```Bash
huggingface-cli download --resume-download sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2 --local-dir /root/data/model/sentence-transformer
```

- 使用开源词向量模型构建开源词向量的时候，需要用到第三方库 `nltk` 的一些资源

```Bash
cd /root
git clone https://gitee.com/yzy0612/nltk_data.git  --branch gh-pages
cd nltk_data
mv packages/*  ./
cd tokenizers
unzip punkt.zip
cd ../taggers
unzip averaged_perceptron_tagger.zip
```

# 搭建知识库

- 克隆需要使用的语料资源仓库

```Bash
# 进入到数据库盘
cd /root/data
# clone 上述开源仓库
git clone https://gitee.com/open-compass/opencompass.git
git clone https://gitee.com/InternLM/lmdeploy.git
git clone https://gitee.com/InternLM/xtuner.git
git clone https://gitee.com/InternLM/InternLM-XComposer.git
git clone https://gitee.com/InternLM/lagent.git
git clone https://gitee.com/InternLM/InternLM.git
```

- 使用语料库构建向量数据库

![img](https://pt35a5ibej.feishu.cn/space/api/box/stream/download/asynccode/?code=MjVlMDI4MDE4NGIwMjEwNjRhMDc5OWU0ZDZjNzI4ZjRfVlUzRHUyY0pwVUVDT1k3Mm5kSEhtR3FCY2FWbWdNVWlfVG9rZW46SUp5WmJFbEVub0NyZmJ4RWJYdGNRc0VlbjVqXzE3MDYyNjUyMzI6MTcwNjI2ODgzMl9WNA)

# 接入LangChain

![img](https://pt35a5ibej.feishu.cn/space/api/box/stream/download/asynccode/?code=YjVkMDc2ZTMxY2YzMGE3OTA2Mzg2ZjgzN2ViOWI0MDRfQjZJRVRzSTQyWUc1R2lHekR3T1I4S1FBcEhpVFE4bFpfVG9rZW46SlpGTGI4NWZEbzhMTlB4WUxxRGNscm1hbm1lXzE3MDYyNjUyMzI6MTcwNjI2ODgzMl9WNA)

![img](https://pt35a5ibej.feishu.cn/space/api/box/stream/download/asynccode/?code=ZDQyYzEyNDQxNjFhMTdhYmJiNjA4NWU2MzA5M2U4YTNfUHVIUEcyVEtqb2JmRTZGRTU0SzhocFFqWmR0bllVTG5fVG9rZW46WXlMc2IzRVF0b3hhWWZ4b21KT2NucDh0bmVjXzE3MDYyNjUyMzI6MTcwNjI2ODgzMl9WNA)

