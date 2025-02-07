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

- 让我解释一下 P-tuning 相关的这两个配置：

```python
if pc.use_ptuning:
    config.pre_seq_len = pc.pre_seq_len        # 设置前缀序列长度
    config.prefix_projection = pc.prefix_projection  # 设置是否使用前缀投影
```

**1. pre_seq_len（前缀序列长度）**
- 这是 P-tuning v2 中的一个关键参数，表示要添加的可训练的连续提示（continuous prompt）的长度
- 比如设置 `pre_seq_len=128`，就意味着在输入序列前面添加 128 个可训练的虚拟词元（virtual tokens）
- 作用：
  - 这些虚拟词元会在训练过程中学习到任务相关的表示
  - 相比全参数微调，只需要训练这些前缀词元，大大减少了可训练参数的数量
  - 通常长度设置在 64-256 之间，长度越长效果可能越好，但参数量也会相应增加

**2. prefix_projection（前缀投影）**
- 这是一个布尔值参数，决定是否对前缀词元进行额外的投影变换
- 当设置为 True 时：
  - 会添加一个额外的 MLP 网络层来处理前缀词元
  - MLP 的结构通常是：`prefix_embedding -> dense layer -> dense layer -> final prefix`
- 作用：
  - 增加模型对前缀词元的表达能力
  - 可以学习到更复杂的特征表示
  - 但会略微增加参数量和计算开销

举个例子来说明工作原理：
```python
# 假设原始输入是：
input_text = "这是一个示例句子"

# 使用 P-tuning（pre_seq_len=4）后的实际处理过程：
virtual_tokens = "[V1][V2][V3][V4]"  # 4个可训练的虚拟词元
actual_input = virtual_tokens + input_text
# 实际输入变成：[V1][V2][V3][V4]这是一个示例句子

# 如果 prefix_projection=True：
# [V1][V2][V3][V4] 会先经过 MLP 处理
# virtual_tokens -> MLP -> transformed_virtual_tokens
# 最终输入：transformed_virtual_tokens + input_text
```

**优势：**
1. 参数效率高：
   - 相比全参数微调，P-tuning 只需要训练很小一部分参数
   - 例如：如果 pre_seq_len=128，可能只需要微调不到 1% 的参数

2. 效果好：
   - 在许多任务上，P-tuning 可以达到接近全参数微调的效果
   - 特别适合处理特定领域的任务适配

3. 内存友好：
   - 由于只需要存储和更新少量参数
   - 特别适合在显存受限的情况下使用

**使用建议：**
1. pre_seq_len 的选择：
   - 开始可以尝试设置为 128
   - 如果效果不够好，可以适当增加
   - 如果显存受限，可以适当减少

2. prefix_projection：
   - 如果计算资源允许，建议设置为 True
   - 如果需要极致的参数效率，可以设置为 False

