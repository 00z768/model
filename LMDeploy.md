# LMDeploy

N卡上大语言模型部署方案。支持多种接口、轻量化方法（4bit权重及8bitKV）、推理引擎（turbomind和pytorch）及gradio等服务。

## 轻量化方法

原因：自回归生产token中间结果（Key和Value）缓存导致内存开销大。推理受限于memory-bound的文件读取（小batch size）。 
Weight Only零量化（降低浮点精度来减少访存成本）：

1. AWQ算法：4bit化非重要权重再反量化回fp16进行推理）
2. KV CacheINT8 推理过程中的Key valueINT8话，推理时再反量化。精度基本不受影响,甚至提升（类似正则化，轻微噪声）。

## 推理引擎 TurboMind

1.持续批处理（Persistent线程）： 请求队列化为尽量填满地Batch Slots，每次slot推理完后将被释放，再拉取新请求。
2.推状态推理：在云端缓存token及KV Cache。

3. Blacked KV Cache： 未占用Free序列不储存、正在推理Active序列储存、推理完后迁移出去。
4. 高性能cuda算子：Flash Attension2等加速优化方法。

## 量化效果（没有输入情况下）

只用KV Cache：RAM节省0.2%
只用W4A16： RAM节省60.8%
KV Cache + W4A16： RAM节省61%

在实际使用时候还是得看效果能不能接受，并且可以考虑外推能力和批处理大小。
