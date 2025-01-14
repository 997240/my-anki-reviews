# Day04 笔记

## 1 金融行业动态方向评估任务介绍

- 背景介绍：

  ```properties
  当前金融领域数据大量激增, 如何从繁杂的数据中获取有效的信息, 进而帮助投资者或者研究者减少决策失误带来的损失，成为金融数据分析方法研究的热门话题。
  随着科技的进步，人工智能技术在各行业中的应用越来越广泛，而金融领域也不例外。人工智能技术的应用可以为金融企业提供更高效、精准的服务，也可以帮助投资者更好的地进行投资决策。
  ```

- 大模型应用三种场景：
  - 金融文本分类
  - 金融信息抽取
  - 金融文本匹配
- 任务目的：基于金融领域数据，实现LLM的直接应用，重点是掌握Prompt的书写方式

------

## 2 金融文本分类

- 目的

  ```properties
  举例说明：
  "公司资产负债表显示，公司偿债能力强劲，现金流充足，为未来投资和扩张提供了坚实的财务基础。"
  判断上述这句话描述的是['新闻报道',  '公司公告',  '财务公告', '分析师报告']中哪一种类型的报告。
  ```

- prompt设计

  ```properties
  要点: 向模型解释什么叫作「文本分类任务」; 需要让模型按照我们指定的格式输出
  ```

  > 为了让模型知道什么叫做「文本分类」，我们借用 Incontext Learning 的方式，先给模型展示几个正确的例子
  >
  > User: "今日，股市经历了一轮震荡，受到宏观经济数据和全球贸易紧张局势的影响。投资者密切关注美联储可能的政策调整，以适应市场的不确定性。" 是['新闻报道', '公司公告', '财务公告 '分析师报告']里的什么类别？
  > Bot: 新闻报道
  > User: "本公司年度财务报告显示，去年公司实现了稳步增长的盈利，同时资产负债表呈现强劲的状况。经济环境的稳定和管理层的有效战略执行为公司的健康发展奠定了基础。"是['新闻报道', '公司公告', '财务公告 '分析师报告']里的什么类别？
  > Bot: 财务报告 

- 代码实现

  ```python
  # —*-coding:utf-8-*-
  """
  利用 LLM 进行文本分类任务。
  """
  # 注意：pip install rich
  from rich import print
  from rich.console import Console
  import ollama
  
  
  # 提供所有类别以及每个类别下的样例
  class_examples = {
      '新闻报道': '今日，股市经历了一轮震荡，受到宏观经济数据和全球贸易紧张局势的影响。投资者密切关注美联储可能的政策调整，以适应市场的不确定性。',
      '财务报告': '本公司年度财务报告显示，去年公司实现了稳步增长的盈利，同时资产负债表呈现强劲的状况。经济环境的稳定和管理层的有效战略执行为公司的健康发展奠定了基础。',
      '公司公告': '本公司高兴地宣布成功完成最新一轮并购交易，收购了一家在人工智能领域领先的公司。这一战略举措将有助于扩大我们的业务领域，提高市场竞争力',
      '分析师报告': '最新的行业分析报告指出，科技公司的创新将成为未来增长的主要推动力。云计算、人工智能和数字化转型被认为是引领行业发展的关键因素，投资者应关注这些趋势'}
  
  def init_prompts():
      """
      初始化前置prompt，便于模型做 incontext learning。
      """
      class_list = list(class_examples.keys())
      pre_history = [{"role": "system", "content": f"现在你是一个文本分类器，你需要按照要求将我给你的句子分类到：{class_list}类别中。"}, ]
  
      for _type, exmpale in class_examples.items():
          pre_history.append({"role": "user", "content": f'“{exmpale}”是 {class_list} 里的什么类别？'})
          pre_history.append({"role": "assistant", "content":  _type})
  
      return {'class_list': class_list, 'pre_history': pre_history}
  
  
  def inference(
          sentences: list,
          custom_settings: dict
  ):
      """
      推理函数。
  
      Args:
          sentences (List[str]): 待推理的句子。
          custom_settings (dict): 初始设定，包含人为给定的 few-shot example。
      """
      for sentence in sentences:
          with console.status("[bold bright_green] Model Inference..."):
              sentence_with_prompt = f"“{sentence}”是 {custom_settings['class_list']} 里的什么类别？"
              response = ollama.chat(model='qwen2.5:7b',
                                     messages=[*custom_settings['pre_history'],
                                               {"role": 'user', "content": sentence_with_prompt}])
              response = response["message"]["content"]
          print(f'>>> [bold bright_red]sentence: {sentence}')
          print(f'>>> [bold bright_green]inference answer: {response}')
          # print(history)
  
  
  if __name__ == '__main__':
      console = Console()
  
      sentences = [
          "今日，央行发布公告宣布降低利率，以刺激经济增长。这一降息举措将影响贷款利率，并在未来几个季度内对金融市场产生影响。",
          "本公司宣布成功收购一家在创新科技领域领先的公司，这一战略性收购将有助于公司拓展技术能力和加速产品研发。",
          "公司资产负债表显示，公司偿债能力强劲，现金流充足，为未来投资和扩张提供了坚实的财务基础。",
          "最新的分析报告指出，可再生能源行业预计将在未来几年经历持续增长，投资者应该关注这一领域的投资机会",
          ]
  
      custom_settings = init_prompts()
      print(custom_settings)
  
      inference(
          sentences,
          custom_settings
      )
  
  ```
## 3 金融文本信息抽取

- 目的

  ```properties
  举例说明：
  "2023-02-15，寓意吉祥的节日，股票佰笃[BD]美股开盘价10美元，虽然经历了波动，但最终以13美元收盘，成交量微幅增加至460,000，投资者情绪较为平稳。"
  抽取上述这句话中的 ['日期', '股票名称', '开盘价', '收盘价', '成交量']关键信息。
  ```

- prompt设计

  ```properties
  要点: 向模型解释什么叫作「信息抽取任务」; 让模型按照我们指定的格式（json）输出
  ```

  > 为了让模型知道什么叫做「信息抽取」，我们借用 Incontext Learning 的方式，先给模型展示几个正确的例子
  >
  > User: '2023-01-10，股市震荡。股票古哥-D[EOOE]美股今日开盘价100美元，一度飙升至105美元，随后回落至98美元，最终以102美元收盘，成交量达到520000。'。提取上述句子中“金融”('日期', '股票名称', '开盘价', '收盘价', '成交量')类型的实体，并按照JSON格式输出，上述句子中没有的信息用['原文中未提及']来表示，多个值之间用','分隔。
  > Bot: {'日期': ['2023-01-10'], '股票名称': ['古哥-D[EOOE]美股'], '开盘价': ['100美元'],  '收盘价': ['102美元'], 成交量': ['520000']}  

- 代码实现

  ```python
  import json
  import ollama
  import re
  
  # 定义不同实体下的具备属性
  schema = {
      '金融': ['日期', '股票名称', '开盘价', '收盘价', '成交量'],
  }
  
  IE_PATTERN = "{}\n\n提取上述句子中{}的实体，并按照JSON格式输出，上述句子中不存在的信息用['原文中未提及']来表示，多个值之间用','分隔。"
  
  
  # 提供一些例子供模型参考
  ie_examples = {
      '金融': [
          {
              'content': '2023-01-10，股市震荡。股票古哥-D[EOOE]美股今日开盘价100美元，一度飙升至105美元，随后回落至98美元，最终以102美元收盘，成交量达到520000。',
              'answers': {
                  '日期': ['2023-01-10'],
                  '股票名称': ['古哥-D[EOOE]美股'],
                  '开盘价': ['100美元'],
                  '收盘价': ['102美元'],
                  '成交量': ['520000'],
              }
          }
      ]
  }
  
  
  def init_prompts():
      """
      初始化前置prompt，便于模型做 incontext learning。
      """
  
      ie_pre_history = [{"role": "system", "content": "你是一个信息抽取助手。"},]
  
      for _type, example_list in ie_examples.items():
          for example in example_list:
              sentence = example['content']
              properties_str = ', '.join(schema[_type])
              schema_str_list = f'“{_type}”({properties_str})'
  
              sentence_with_prompt = IE_PATTERN.format(sentence, schema_str_list)
  
              ie_pre_history.append({"role": "user", "content": f'{sentence_with_prompt}'})
              ie_pre_history.append({"role": "assistant", "content": f"{json.dumps(example['answers'], ensure_ascii=False)}"})
  
      return {'ie_pre_history': ie_pre_history}
  
  
  def clean_response(response: str):
      """
      后处理模型输出。
  
      Args:
          response (str): _description_
      """
      # response1='```json["name":lucy]```abc```json["name":lucy]```'
      if '```json' in response:
          res = re.findall(r'```json(.*?)```', response, re.DOTALL)
          if len(res) and res[0]:
              response = res[0]
          response.replace('、', ',')
      try:
          return json.loads(response)
      except:
          return response
  
  
  def inference(
          sentences: list,
          custom_settings: dict
      ):
      """
      推理函数。
  
      Args:
          sentences (List[str]): 待抽取的句子。
          custom_settings (dict): 初始设定，包含人为给定的 few-shot example。
      """
      for sentence in sentences:
          cls_res = "金融"
          if cls_res not in schema:
              print(f'The type model inferenced {cls_res} which is not in schema dict, exited.')
              exit()
          properties_str = ', '.join(schema[cls_res])
          schema_str_list = f'“{cls_res}”({properties_str})'
          sentence_with_ie_prompt = IE_PATTERN.format(sentence, schema_str_list)
          # print(f'sentence_with_ie_prompt-->{sentence_with_ie_prompt}')
          # 使用 Ollama 调用 Qwen2.5:7b 模型
  
          response = ollama.chat(
              model="qwen2.5:7b",
              messages=[
                  *custom_settings['ie_pre_history'],
                  {"role": "user", "content": sentence_with_ie_prompt}
              ]
          )
          res_content = response["message"]["content"]
          ie_res = clean_response(res_content)
          print(f'>>> [bold bright_red]sentence: {sentence}')
          print(f'>>> [bold bright_green]inference answer: {ie_res}')
  
  
  if __name__ == '__main__':
  
      # 初始化句子和自定义设置
      sentences = [
          '2023-02-15，寓意吉祥的节日，股票佰笃[BD]美股开盘价10美元，虽然经历了波动，但最终以13美元收盘，成交量微幅增加至460,000，投资者情绪较为平稳。',
          '2023-04-05，市场迎来轻松氛围，股票盘古(0021)开盘价23元，尽管经历了波动，但最终以26美元收盘，成交量缩小至310,000，投资者保持观望态度。',
      ]
  
      # 初始化自定义设置
      custom_settings = init_prompts()
  
      # 开始推理
      inference(
          sentences,
          custom_settings
      )
  
  ```
## 4 金融文本匹配

- 目的

  ```properties
  举例说明：
  "股票市场今日大涨，投资者乐观。', '持续上涨的市场让投资者感到满意。"
  判断上述这两句话属于['相似', '不相似', '相似']中哪一种类型。
  ```

- prompt设计

  ```properties
  要点: 向模型解释什么叫作「文本匹配任务」; 需要让模型按照我们指定的格式输出
  ```

  > 为了让模型知道什么叫做「文本匹配」，我们借用 Incontext Learning 的方式，先给模型展示几个正确的例子
  >
  > User: 句子一: 公司ABC发布了季度财报，显示盈利增长。\n句子二: 财报披露，公司ABC利润上升
  > Bot: 是
  > User: 句子一: 黄金价格下跌，投资者抛售。\n句子二: 外汇市场交易额创下新高
  > Bot: 不是

- 代码实现

  ```python
  # !/usr/bin/env python3
  """
  利用 LLM 进行文本匹配任务。
  """
  from rich import print
  import ollama
  
  
  # 提供相似，不相似的语义匹配例子
  examples = {
      '是': [
          ('公司ABC发布了季度财报，显示盈利增长。', '财报披露，公司ABC利润上升。'),
      ],
      '不是': [
          ('黄金价格下跌，投资者抛售。', '外汇市场交易额创下新高。'),
          ('央行降息，刺激经济增长。', '新能源技术的创新。')
      ]
  }
  
  
  
  def init_prompts():
      """
      初始化前置prompt，便于模型做 incontext learning。
      """
      pre_history = [{"role": "system", "content": "现在你需要帮助我完成文本匹配任务，当我给你两个句子时，你需要回答我这两句话语义是否相似。只需要回答是否相似，不要做多余的回答。"}, ]
      for key, sentence_pairs in examples.items():
          for sentence_pair in sentence_pairs:
              sentence1, sentence2 = sentence_pair
              pre_history.append({"role": "user", "content":  f'句子一: {sentence1}\n句子二: {sentence2}\n上面两句话是相似的语义吗？'})
              pre_history.append({"role": "assistant", "content":  key})
  
      return {'pre_history': pre_history}
  
  
  def inference(
          sentence_pairs: list,
          custom_settings: dict
      ):
      """
      推理函数。
  
      Args:
          model (transformers.AutoModel): Language Model 模型。
          sentence_pairs (List[str]): 待推理的句子对。
          custom_settings (dict): 初始设定，包含人为给定的 few-shot example。
      """
      for sentence_pair in sentence_pairs:
          sentence1, sentence2 = sentence_pair
          sentence_with_prompt = f'句子一: {sentence1}\n句子二: {sentence2}\n上面两句话是相似的语义吗？'
          response = ollama.chat(model="qwen2.5:7b",
                                 messages=[*custom_settings["pre_history"],
                                           {"role":'user', "content":sentence_with_prompt}])
          response = response["message"]["content"]
          print(f'>>> [bold bright_red]sentence: {sentence_pair}')
          print(f'>>> [bold bright_green]inference answer: {response}')
  
  
  if __name__ == '__main__':
  
      sentence_pairs = [
          ('股票市场今日大涨，投资者乐观。', '持续上涨的市场让投资者感到满意。'),
          ('油价大幅下跌，能源公司面临挑战。', '未来智能城市的建设趋势愈发明显。'),
          ('利率上升，影响房地产市场。', '高利率对房地产有一定冲击。'),
      ]
  
      custom_settings = init_prompts()
      # print(f'custom_settings-->{custom_settings}')
      inference(
          sentence_pairs,
          custom_settings
      )
  ```

  