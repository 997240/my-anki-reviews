# 大模型学习笔记

## 学习笔记链接
- [Day 01 笔记](https://997240.github.io/my-anki-reviews/LLM-Notes/day01/)
- [Day 02 笔记](https://997240.github.io/my-anki-reviews/LLM-Notes/day02/)
- [Day 03 笔记](https://997240.github.io/my-anki-reviews/LLM-Notes/day03/)
- [Day 04 笔记](https://997240.github.io/my-anki-reviews/LLM-Notes/day04/)
- [Day 05 - LangChain 笔记](https://997240.github.io/my-anki-reviews/LLM-Notes/day05/LangChain笔记)
- [Day 05 - 物流行业智能问答 RAG 系统笔记](https://997240.github.io/my-anki-reviews/LLM-Notes/day05/物流行业智能问答RAG系统笔记)
- [Day 06 - 物流行业智能问答 RAG 系统笔记](https://997240.github.io/my-anki-reviews/LLM-Notes/day06/物流行业智能问答RAG系统笔记)
- [面试算法题](https://997240.github.io/my-anki-reviews/leetcode)

  


 

我来为您解释 `model2train` 函数的每一行代码：

```python
def model2train():
    # 1. 加载tokenizer和模型配置
    tokenizer = AutoTokenizer.from_pretrained(pc.pre_model, trust_remote_code=True)
    config = AutoConfig.from_pretrained(pc.pre_model, trust_remote_code=True)
    
    # 2. 如果使用p-tuning，设置相关配置
    if pc.use_ptuning:
        config.pre_seq_len = pc.pre_seq_len
        config.prefix_projection = pc.prefix_projection
    
    # 3. 加载预训练模型
    model = AutoModel.from_pretrained(pc.pre_model, config=config, trust_remote_code=True)
    model = model.half().to(pc.device)  # 转换为半精度并移到指定设备
```

让我逐行解释主要功能：

1. **模型初始化部分**:
- 创建tokenizer用于文本处理
- 加载模型配置
- 如果启用p-tuning，设置前缀长度和投影配置
- 加载预训练模型并转换为半精度(FP16)格式

```python
    # 4. 模型训练设置
    model.gradient_checkpointing_enable()  # 启用梯度检查点
    model.enable_input_require_grads()     # 启用输入需要梯度
    model.config.use_cache = False         # 禁用缓存以节省显存
    
    # 5. P-tuning相关设置
    if pc.use_ptuning:
        model.transformer.prefix_encoder.float()  # prefix_encoder使用全精度
        
    # 6. LoRA相关设置
    if pc.use_lora:
        model.lm_head = CastOutputToFloat(model.lm_head)
        peft_config = peft.LoraConfig(
            task_type=peft.TaskType.CAUSAL_LM,  # 因果语言模型
            inference_mode=False,   # 训练模式
            r=pc.lora_rank,        # LoRA秩
            lora_alpha=32,         # 缩放因子
            lora_dropout=0.1       # Dropout率
        )
        model = peft.get_peft_model(model, peft_config)
```

2. **优化器设置部分**:
```python
    # 7. 设置优化器参数组
    no_decay = ['bias', 'LayerNorm.weight']
    optimizer_grouped_parameters = [
        {
            'params': [p for n, p in model.named_parameters() 
                      if not any(nd in n for nd in no_decay)],
            'weight_decay': pc.weight_decay
        },
        {
            'params': [p for n, p in model.named_parameters() 
                      if any(nd in n for nd in no_decay)],
            'weight_decay': 0.0
        }
    ]
    optimizer = torch.optim.AdamW(optimizer_grouped_parameters, lr=pc.learning_rate)
```

3. **数据加载和学习率调度**:
```python
    # 8. 加载数据
    train_dataloader, dev_dataloader = get_data()
    
    # 9. 设置学习率调度
    num_update_steps_per_epoch = len(train_dataloader)
    max_train_steps = pc.epochs * num_update_steps_per_epoch
    warm_steps = int(pc.warmup_ratio * max_train_steps)
    lr_scheduler = get_scheduler(
        name='linear',
        optimizer=optimizer,
        num_warmup_steps=warm_steps,
        num_training_steps=max_train_steps
    )
```

4. **训练循环**:
```python
    # 10. 初始化训练变量
    loss_list = []
    tic_train = time.time()
    global_step, best_eval_loss = 0, float('inf')
    
    # 11. 训练循环
    for epoch in range(1, pc.epochs + 1):
        for batch in train_dataloader:
            # 前向传播
            if pc.use_lora:
                with autocast():
                    loss = model(
                        input_ids=batch['input_ids'].to(dtype=torch.long, device=pc.device),
                        labels=batch['labels'].to(dtype=torch.long, device=pc.device)
                    ).loss
            else:
                loss = model(
                    input_ids=batch['input_ids'].to(dtype=torch.long, device=pc.device),
                    labels=batch['labels'].to(dtype=torch.long, device=pc.device)
                ).loss
                
            # 反向传播
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()
            lr_scheduler.step()
            
            # 记录损失
            loss_list.append(float(loss.cpu().detach()))
            global_step += 1
```

5. **日志记录和模型保存**:
```python
            # 12. 打印训练日志
            if global_step % pc.logging_steps == 0:
                # 计算并打印训练统计信息
                
            # 13. 定期保存和评估
            if global_step % pc.save_freq == 0:
                # 评估模型
                eval_loss = evaluate_model(model, dev_dataloader)
                # 如果评估损失更好，保存最佳模型
```

这个函数实现了一个完整的模型训练流程，包括：
- 模型初始化和配置
- 优化器设置
- 学习率调度
- 训练循环
- 定期评估和模型保存
- 支持P-tuning和LoRA等微调方法
