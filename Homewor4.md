基于MedQA数据集微调InternLM-chat-7B模型

1.开始训练

![img]([https://app.yinxiang.com/FileSharing.action?hash=1/e9de4926570c339d59e76e0b45a72c99-117779](https://app.yinxiang.com/FileSharing.action?hash=1/e9de4926570c339d59e76e0b45a72c99-117779))

![img](https://app.yinxiang.com/FileSharing.action?hash=1/89c63d01989b2514855b3edd315a82bc-8248)

2.测试MedQA

测试微调后的模型

![img](https://app.yinxiang.com/FileSharing.action?hash=1/61c874e079debdef5379fc7454cc6b85-135597)

![img](https://app.yinxiang.com/FileSharing.action?hash=1/98d1fdd4203f6583027be408ec43d2a6-30551)

![img](https://app.yinxiang.com/FileSharing.action?hash=1/7033bb244fe7ff2925ddb96308474d2e-20347)

测试未微调的模型

![img](https://app.yinxiang.com/FileSharing.action?hash=1/b5fdfc90d1f127e81c149e440233e333-31300)

![img](https://app.yinxiang.com/FileSharing.action?hash=1/155f1507616acd6fa6271beca874c1e9-30669)

3.结论

从结果来看，微调前后回复其实相差不是很大，我用test data同时测了微调前后的模型，输出都有模有样。其实，可以看出InternLM-chat-7B模型已经在medical QA上作了微调，所以我这儿微调并没有什么明显提升。

4.微调自我认知LLM

1）使用1万条相同指令微调模型发现LLM还是收敛不充分，无法实现自我认知。

2）将训练数据扩充至3万条后发现模型微调2轮后就过拟合，出现灾难遗忘问题。

![img](https://app.yinxiang.com/FileSharing.action?hash=1/dd67ed8ca1f80ad9b43102c327c7b05a-31871)

