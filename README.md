# 大模型学习笔记

## 学习笔记链接
- [Day 01 笔记](https://997240.github.io/my-anki-reviews/LLM-Notes/day01/)
- [Day 02 笔记](https://997240.github.io/my-anki-reviews/LLM-Notes/day02/)
- [Day 03 笔记](https://997240.github.io/my-anki-reviews/LLM-Notes/day03/)
- [Day 04 笔记](https://997240.github.io/my-anki-reviews/LLM-Notes/day04/)
- [Day 05 - LangChain 笔记](https://997240.github.io/my-anki-reviews/LLM-Notes/day05/LangChain笔记)
- [Day 05 - 物流行业智能问答 RAG 系统笔记](https://997240.github.io/my-anki-reviews/LLM-Notes/day05/物流行业智能问答RAG系统笔记)
- [Day 06 - 物流行业智能问答 RAG 系统笔记](https://997240.github.io/my-anki-reviews/LLM-Notes/day06/物流行业智能问答RAG系统笔记)

 






好的，我来用一个具体的例子来说明整个数据处理流程：

1. **原始输入数据** (BIO格式的文本文件):
```
扬 B-LOC
州 I-LOC
四 O
吨 O
炒 O
饭 O

北 B-LOC
京 I-LOC
的 O
天 O
气 O
```

2. **load_data函数处理后**:
```python
sentences = [
    ['扬', '州', '四', '吨', '炒', '饭'],
    ['北', '京', '的', '天', '气']
]

labels = [
    ['B-LOC', 'I-LOC', 'O', 'O', 'O', 'O'],
    ['B-LOC', 'I-LOC', 'O', 'O', 'O']
]
```

3. **NERDataset初始化后** (`_preprocess_data`处理):
```python
# 转换为连续文本
texts = [
    '扬州四吨炒饭',
    '北京的天气'
]

# batch_encode_plus处理后的encodings
encodings = {
    'input_ids': [
        [101, 2207, 4545, 1744, 3209, 3696, 6121, 102],  # [CLS] 扬州四吨炒饭 [SEP]
        [101, 1266, 3221, 4638, 1921, 3698, 102]         # [CLS] 北京的天气 [SEP]
    ],
    'attention_mask': [
        [1, 1, 1, 1, 1, 1, 1, 1],
        [1, 1, 1, 1, 1, 1, 1]
    ]
}

# 标签转换为ID
label_ids = [
    [1, 2, 0, 0, 0, 0],  # B-LOC=1, I-LOC=2, O=0
    [1, 2, 0, 0, 0]
]
```

4. **__getitem__返回单个样本**:
```python
# dataset[0] 返回
{
    'input_ids': [101, 2207, 4545, 1744, 3209, 3696, 6121, 102],
    'attention_mask': [1, 1, 1, 1, 1, 1, 1, 1],
    'labels': [1, 2, 0, 0, 0, 0],
    'length': 6
}
```

5. **collate_fn处理batch** (假设batch_size=2):
```python
# 输入是2个样本的列表
batch = [
    {
        'input_ids': [101, 2207, 4545, 1744, 3209, 3696, 6121, 102],
        'attention_mask': [1, 1, 1, 1, 1, 1, 1, 1],
        'labels': [1, 2, 0, 0, 0, 0],
        'length': 6
    },
    {
        'input_ids': [101, 1266, 3221, 4638, 1921, 3698, 102],
        'attention_mask': [1, 1, 1, 1, 1, 1, 1],
        'labels': [1, 2, 0, 0, 0],
        'length': 5
    }
]

# collate_fn处理后（padding到最大长度8）
{
    'input_ids': tensor([
        [101, 2207, 4545, 1744, 3209, 3696, 6121, 102],
        [101, 1266, 3221, 4638, 1921, 3698, 102, 0]
    ]),
    'attention_mask': tensor([
        [1, 1, 1, 1, 1, 1, 1, 1],
        [1, 1, 1, 1, 1, 1, 1, 0]
    ]),
    'labels': tensor([
        [1, 2, 0, 0, 0, 0, 7, 7],  # 7是PAD标签的ID
        [1, 2, 0, 0, 0, 7, 7, 7]
    ]),
    'lengths': tensor([6, 5])
}
```

6. **最终DataLoader输出**:
```python
for batch in train_loader:
    print("批次大小:", batch['input_ids'].shape[0])      # 2
    print("序列最大长度:", batch['input_ids'].shape[1])  # 8
    print("输入形状:", batch['input_ids'].shape)         # torch.Size([2, 8])
    print("注意力掩码形状:", batch['attention_mask'].shape)  # torch.Size([2, 8])
    print("标签形状:", batch['labels'].shape)            # torch.Size([2, 8])
    print("序列实际长度:", batch['lengths'])             # tensor([6, 5])
    break
```

这个例子展示了数据是如何一步步被处理的：
1. 从原始BIO文本
2. 到分割后的句子和标签
3. 到tokenize后的ID序列
4. 最后到batch级别的张量

这样的处理确保了：
1. 所有序列都被正确编码
2. 标签与输入对齐
3. batch内的序列长度统一（通过padding）
4. 保留了原始序列长度信息

您觉得这样的解释清晰吗？







好的，让我解释这两个参数的设置原因：

1. `padding=False` 的原因：
```python
# 不好的做法：在batch_encode_plus时就padding
tokenizer.batch_encode_plus(
    texts,
    padding=True,  # 会padding到整个数据集的最大长度
    ...
)

# 好的做法：在collate_fn中进行padding
tokenizer.batch_encode_plus(
    texts,
    padding=False,  # 先不padding
    ...
)
```

原因是：
- 如果在 `batch_encode_plus` 时就padding，会padding到整个数据集中最长序列的长度
- 这样会造成大量不必要的padding，浪费内存
- 而在 `collate_fn` 中padding只需要考虑当前batch中的最大长度，更高效

2. `return_tensors=None` 的原因：
```python
# 不好的做法：直接返回tensor
tokenizer.batch_encode_plus(
    texts,
    return_tensors='pt',  # 直接返回PyTorch tensor
    ...
)

# 好的做法：先返回列表，后续再转tensor
tokenizer.batch_encode_plus(
    texts,
    return_tensors=None,  # 返回Python列表
    ...
)
```

原因是：
- 如果直接返回tensor，在进行padding和截断时操作不够灵活
- 在 `collate_fn` 中，我们需要先处理长度，再进行padding
- 处理Python列表比处理tensor更方便
- 最后统一在 `collate_fn` 中转换为tensor更高效

完整的处理流程：
```python
# 1. batch_encode_plus返回列表
encodings = tokenizer.batch_encode_plus(
    texts,
    padding=False,
    return_tensors=None
)

# 2. 在collate_fn中处理
def collate_fn(batch):
    # ... 前面的代码 ...
    
    # 找到当前batch的最大长度
    max_len = max([len(ids) for ids in input_ids])
    
    # 只padding到当前batch的最大长度
    input_ids_padded = [ids + [0] * (max_len - len(ids)) for ids in input_ids]
    
    # 最后统一转换为tensor
    input_ids_padded = torch.tensor(input_ids_padded)
```

这样的设计优点：
1. 内存使用更高效（只padding到必要的长度）
2. 处理更灵活（可以根据batch动态调整）
3. 计算更高效（避免不必要的padding计算）
4. 代码逻辑更清晰（数据处理步骤分明）

您觉得这样解释清楚了吗？

