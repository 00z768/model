# xturner微调工具箱--微调大模型

## 环境配置

### 1、源码安装xturner：

```
conda create --name xtuner0.1.9 python=3.10 -y
# 激活环境
conda activate xtuner0.1.9
# 进入家目录 （~的意思是 “当前用户的home路径”） root用户下和cd /root一样
cd ~
# 创建版本文件夹并进入，以跟随本教程
mkdir xtuner019 && cd xtuner019


# 拉取 0.1.9 的版本源码
git clone -b v0.1.9  https://kkgithub.com/InternLM/xtuner
# 无法访问github的用户请从 gitee 拉取:
# git clone -b v0.1.9 https://gitee.com/Internlm/xtuner

# 进入源码目录
cd xtuner

# 从源码安装 XTuner
pip install -e '.[all]'

```

### 2.创建并进入工作目录

```
# 创建一个微调 oasst1 数据集的工作路径，进入
mkdir ~/ft-oasst1 && cd ~/ft-oasst1
```

### 3.配置xtuner

使用xtuner内置的开箱即用的配置文件

其中，`internlm_chat_7b_qlora_oasst1_e3`表示用internlm_chat_7b模型和qlora算法微调oasst1数据集并且跑3次（e3 =epoch 3）

#### 查看内置配置文件

```
# 列出所有内置配置
xtuner list-cfg
```

拷贝需要的配置文件道工作目录

```
cd ~/ft-oasst1
xtuner copy-cfg internlm_chat_7b_qlora_oasst1_e3 .
# 最后有个英文句号，代表复制到当前路径
```



## 下载数据集和模型

选择微调数据集: oasst1 (子集[timdettmers/openassistant-guanaco](https://hf-mirror.com/datasets/timdettmers/openassistant-guanaco))

选择模型: internlm-chat-7b

下载好的路径：

```
|-- internlm-chat-7b
|   |-- README.md
|   |-- config.json
|   |-- configuration.json
|   |-- configuration_internlm.py
|   |-- generation_config.json
|   |-- modeling_internlm.py
|   |-- pytorch_model-00001-of-00008.bin
|   |-- pytorch_model-00002-of-00008.bin
|   |-- pytorch_model-00003-of-00008.bin
|   |-- pytorch_model-00004-of-00008.bin
|   |-- pytorch_model-00005-of-00008.bin
|   |-- pytorch_model-00006-of-00008.bin
|   |-- pytorch_model-00007-of-00008.bin
|   |-- pytorch_model-00008-of-00008.bin
|   |-- pytorch_model.bin.index.json
|   |-- special_tokens_map.json
|   |-- tokenization_internlm.py
|   |-- tokenizer.model
|   `-- tokenizer_config.json
|-- internlm_chat_7b_qlora_oasst1_e3_copy.py
`-- openassistant-guanaco
    |-- openassistant_best_replies_eval.jsonl
    `-- openassistant_best_replies_train.jsonl
```



## 修改配置文件

修改internlm_chat_7b_qlora_oasst1_e3_copy.py模型和数据集路径

```
# 修改模型为本地路径
- pretrained_model_name_or_path = 'internlm/internlm-chat-7b'
+ pretrained_model_name_or_path = './internlm-chat-7b'

# 修改训练数据集为本地路径
- data_path = 'timdettmers/openassistant-guanaco'
+ data_path = './openassistant-guanaco'
```

常用超参

| 参数名              | 解释                                                   |
| ------------------- | ------------------------------------------------------ |
| data_path           | 数据路径或 HuggingFace 仓库名                          |
| max_length          | 单条数据最大 Token 数，超过则截断                      |
| pack_to_max_length  | 是否将多条短数据拼接到 max_length，提高 GPU 利用率     |
| accumulative_counts | 梯度累积，每多少次 backward 更新一次参数               |
| evaluation_inputs   | 训练过程中，会根据给定的问题进行推理，便于观测训练状态 |
| evaluation_freq     | Evaluation 的评测间隔 iter 数                          |

如果想把显卡的现存吃满，充分利用显卡资源，可以将 max_length 和 batch_size 这两个参数调大。



## 开始微调

在工作目录下运行train命令

```
# 单卡
## 用刚才改好的config文件训练
xtuner train ./internlm_chat_7b_qlora_oasst1_e3_copy.py

# 多卡
NPROC_PER_NODE=${GPU_NUM} xtuner train ./internlm_chat_7b_qlora_oasst1_e3_copy.py

# 若要开启 deepspeed 加速，增加 --deepspeed deepspeed_zero2 即可
```

训练之后的路径

```
|-- internlm-chat-7b
|-- internlm_chat_7b_qlora_oasst1_e3_copy.py
|-- openassistant-guanaco
|   |-- openassistant_best_replies_eval.jsonl
|   `-- openassistant_best_replies_train.jsonl
`-- work_dirs
    `-- internlm_chat_7b_qlora_oasst1_e3_copy
        |-- 20231101_152923
        |   |-- 20231101_152923.log
        |   `-- vis_data
        |       |-- 20231101_152923.json
        |       |-- config.py
        |       `-- scalars.json
        |-- epoch_1.pth
        |-- epoch_2.pth
        |-- epoch_3.pth
        |-- internlm_chat_7b_qlora_oasst1_e3_copy.py
        `-- last_checkpoint
```



## 转换模型并合并

### 转换为HuggingFace模型，生成Adapter文件夹

```
mkdir hf
# 设置环境变量
export MKL_SERVICE_FORCE_INTEL=1
# 转换命令
xtuner convert pth_to_hf ./internlm_chat_7b_qlora_oasst1_e3_copy.py ./work_dirs/internlm_chat_7b_qlora_oasst1_e3_copy/epoch_3.pth ./hf
# xtuner convert pth_to_hf \
#     ${CONFIG_NAME_OR_PATH} \
#     ${PTH_file_dir} \
#     ${SAVE_PATH}

```

转换后的路径：

```
|-- internlm-chat-7b
|-- internlm_chat_7b_qlora_oasst1_e3_copy.py
|-- openassistant-guanaco
|   |-- openassistant_best_replies_eval.jsonl
|   `-- openassistant_best_replies_train.jsonl
|-- hf
|   |-- README.md
|   |-- adapter_config.json
|   |-- adapter_model.bin
|   `-- xtuner_config.py
`-- work_dirs
    `-- internlm_chat_7b_qlora_oasst1_e3_copy
        |-- 20231101_152923
        |   |-- 20231101_152923.log
        |   `-- vis_data
        |       |-- 20231101_152923.json
        |       |-- config.py
        |       `-- scalars.json
        |-- epoch_1.pth
        |-- epoch_2.pth
        |-- epoch_3.pth
        |-- internlm_chat_7b_qlora_oasst1_e3_copy.py
        `-- last_checkpoint
```

hf 文件夹即为我们平时所理解的所谓 “LoRA 模型文件”

​		可以简单理解：LoRA 模型文件 = Adapter

### 将HuggingFace adapter合并到大语言模型

```
# 合并命令
xtuner convert merge ./internlm-chat-7b ./hf ./merged --max-shard-size 2GB
# xtuner convert merge \
#     ${NAME_OR_PATH_TO_LLM} \
#     ${NAME_OR_PATH_TO_ADAPTER} \
#     ${SAVE_PATH} \
#     --max-shard-size 2GB
```



## 对话测试

```
# 加载 Adapter 模型对话（Float 16）
xtuner chat ./merged --prompt-template internlm_chat

# 4 bit 量化加载
# xtuner chat ./merged --bits 4 --prompt-template internlm_chat
```

xtuner chat启动参数

| 参数名                    | 作用                                                         |
| ------------------------- | ------------------------------------------------------------ |
| ******--prompt-template** | 指定对话模板                                                 |
| --system                  | 指定SYSTEM文本                                               |
| --system-template         | 指定SYSTEM模板                                               |
| -******-bits**            | LLM位数                                                      |
| --bot-name                | bot名称                                                      |
| --with-plugins            | 指定要使用的插件                                             |
| ******--no-streamer**     | 是否启用流式传输                                             |
| ******--lagent**          | 是否使用lagent                                               |
| --command-stop-word       | 命令停止词                                                   |
| --answer-stop-word        | 回答停止词                                                   |
| --offload-folder          | 存放模型权重的文件夹（或者已经卸载模型权重的文件夹）         |
| --max-new-tokens          | 生成文本中允许的最大 `token` 数量                            |
| ******--temperature**     | 温度值                                                       |
| --top-k                   | 保留用于顶k筛选的最高概率词汇标记数                          |
| --top-p                   | 如果设置为小于1的浮点数，仅保留概率相加高于 `top_p` 的最小一组最有可能的标记 |
| --seed                    | 用于可重现文本生成的随机种子                                 |

或创建cli_demo.py

​	model_name_or_path = "/root/model/Shanghai_AI_Laboratory/internlm-chat-7b"

​	model_name_or_path = "merged"

```
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM


model_name_or_path = "/root/model/Shanghai_AI_Laboratory/internlm-chat-7b"  # 修改为模型文件夹路径，这里是merged

tokenizer = AutoTokenizer.from_pretrained(model_name_or_path, trust_remote_code=True)
model = AutoModelForCausalLM.from_pretrained(model_name_or_path, trust_remote_code=True, torch_dtype=torch.bfloat16, device_map='auto')
model = model.eval()

system_prompt = """You are an AI assistant whose name is InternLM (书生·浦语).
- InternLM (书生·浦语) is a conversational language model that is developed by Shanghai AI Laboratory (上海人工智能实验室). It is designed to be helpful, honest, and harmless.
- InternLM (书生·浦语) can understand and communicate fluently in the language chosen by the user such as English and 中文.
"""

messages = [(system_prompt, '')]

print("=============Welcome to InternLM chatbot, type 'exit' to exit.=============")

while True:
    input_text = input("User  >>> ")
    input_text.replace(' ', '')
    if input_text == "exit":
        break
    response, history = model.chat(tokenizer, input_text, history=messages)
    messages.append((input_text, response))
    print(f"robot >>> {response}")
```

