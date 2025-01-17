# Day05 笔记

------

## 1 什么是LangChain

<div align=center><img src="./img/01.png" style="zoom:50%" ><img/></div>

LangChain由 Harrison Chase 创建于2022年10月，它是围绕LLMs（大语言模型）建立的一个框架，LangChain自身并不开发LLMs，它的核心理念是为各种LLMs实现通用的接口，把LLMs相关的组件“链接”在一起，简化LLMs应用的开发难度，方便开发者快速地开发复杂的LLMs应用。

LangChain目前有两个语言的实现：python、nodejs。

参考官网介绍：https://python.langchain.com/docs/integrations/text_embedding/huggingfacehub

------

## 2 LangChain主要组件

一个LangChain的应用是需要多个组件共同实现的，LangChain主要支持6种组件：

- Models：模型，各种类型的模型和模型集成，比如GPT-4
- Prompts：提示，包括提示管理、提示优化和提示序列化
- Memory：记忆，用来保存和模型交互时的上下文状态
- Indexes：索引，用来结构化文档，以便和模型交互
- Chains：链，一系列对各种组件的调用
- Agents：代理，决定模型采取哪些行动，执行并且观察流程，直到完成为止

------

### 2.1 Models

LangChain目前支持三种类型的模型：LLMs、Chat Models(聊天模型)、Embeddings Models(嵌入模型）.

- LLMs: 大语言模型接收文本字符作为输入，返回的也是文本字符.


- 聊天模型: 基于LLMs, 不同的是它接收聊天消息(一种特定格式的数据)作为输入，返回的也是聊天消息.


- 文本嵌入模型: 文本嵌入模型接收文本作为输入, 返回的是浮点数列表.

LangChain支持的三类模型，它们的使用场景不同，输入和输出不同，开发者需要根据项目需要选择相应。

------

安装必备的工具包

```properties
pip install langchain
pip install ollama
pip install langchain_community
```

#### 2.1.1 LLMs (大语言模型)

一般基于LLMs模型接受的是一段文本，输出也是文本

- 代码示例

```python
from langchain_community.llms import Ollama
# model = Ollama(model="qwen2.5:1.5b")
model = Ollama(model="qwen2.5:7b")
result = model.invoke("请给我讲个鬼故事")
print(result)
```

#### 2.1.2 Chat Models (聊天模型)

聊天消息包含下面几种类型，使用时需要按照约定传入合适的值：

- AIMessage: 就是 AI 输出的消息，可以是针对问题的回答.
- HumanMessage: 人类消息就是用户信息，由人给出的信息发送给LLMs的提示信息，比如“实现一个快速排序方法”.
- SystemMessage: 可以用于指定模型具体所处的环境和背景，如角色扮演等。你可以在这里给出具体的指示，比如“作为一个代码专家”，或者“返回json格式”.
- 代码示例：

```python
from langchain_core.messages import HumanMessage, SystemMessage, AIMessage
from langchain_community.chat_models import ChatOllama
model = ChatOllama(model="qwen2.5:7b")
messages = [
        SystemMessage(content="现在你是一个著名的诗人"),
        HumanMessage(content="给我写一首唐诗")
]
res = model.invoke(messages)
print(res)
print(res.content)

# 加入AIMessage
messages = [
        SystemMessage(content="现在你是一个著名的诗人"),
        HumanMessage(content="给我写一首唐诗"),
        AIMessage(content='清风明月夜，独坐思故人。\n江水东流去，孤舟何处人？\n花落知多少，柳暗闭重门。\n唯有心中意，常随鹤归云。'),
        HumanMessage(content="给我写一首宋词"),
]
res = model.invoke(messages)
print(res)
print(res.content)
```

#### 2.1.3 Embeddings Models(嵌入模型)

Embeddings Models特点：将字符串作为输入，返回一个浮动数的列表。在NLP中，Embedding的作用就是将数据进行文本向量化。

- 代码示例：


```python
from langchain_community.embeddings import OllamaEmbeddings

model = OllamaEmbeddings(model="mxbai-embed-large", temperature=0)
res1 = model.embed_query('这是第一个测试文档')
print(res1)
print(len(res1))

res2 = model.embed_documents(['这是第一个测试文档', '这是第二个测试文档'])
print(res2)
```

### 2.2 Prompts

Prompt是指当用户输入信息给模型时加入的提示，这个提示的形式可以是zero-shot或者few-shot等方式，目的是让模型理解更为复杂的业务场景以便更好的解决问题。

提示模板：如果你有了一个起作用的提示，你可能想把它作为一个模板用于解决其他问题，LangChain就提供了PromptTemplates组件，它可以帮助你更方便的构建提示。

zero-shot提示方式：

```python
from langchain import PromptTemplate
from langchain_community.llms import Ollama
model = Ollama(model="qwen2.5:7b")
# 定义模板
template = "我的邻居姓{lastname}，他生了个儿子，给他儿子起个名字"

prompt = PromptTemplate(
    input_variables=["lastname"],
    template=template,
)

prompt_text = prompt.format(lastname="王")
print(prompt_text)
# result: 我的邻居姓王，他生了个儿子，给他儿子起个名字


result = model(prompt_text)
print(result)
```

few-shot提示方式：

```python
from langchain import PromptTemplate, FewShotPromptTemplate
from langchain_community.llms import Ollama
model = Ollama(model="qwen2:1.5b")

examples = [
    {"word": "开心", "antonym": "难过"},
    {"word": "高", "antonym": "矮"},
]

example_template = """
单词: {word}
反义词: {antonym}\\n
"""

example_prompt = PromptTemplate(
    input_variables=["word", "antonym"],
    template=example_template,
)

few_shot_prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_prompt,
    prefix="给出每个单词的反义词",
    suffix="单词: {input}\\n反义词:",
    input_variables=["input"],
    example_separator="\\n",
)

prompt_text = few_shot_prompt.format(input="粗")
print(prompt_text)
print('*'*80)
# 给出每个单词的反义词
# 单词: 开心
# 反义词: 难过

# 单词: 高
# 反义词: 矮

# 单词: 粗
# 反义词:

# 调用模型
print(model(prompt_text))

# 细
```

------

### 2.3 Chains(链)

在LangChain中，Chains描述了将LLM与其他组件结合起来完成一个应用程序的过程.

针对上一小节的提示模版例子，zero-shot里面，我们可以用链来连接提示模版组件和模型，进而可以实现代码的更改：

```python
from langchain import PromptTemplate
from langchain_community.llms import Ollama
from langchain.chains import LLMChain

# 定义模板
template = "我的邻居姓{lastname}，他生了个儿子，给他儿子起个名字"

prompt = PromptTemplate(
    input_variables=["lastname"],
    template=template,
)
llm = Ollama(model="qwen2.5:7b")

chain = LLMChain(llm=llm, prompt=prompt)
# 执行链
print(chain.run("王"))
```

如果你想将第一个模型输出的结果，直接作为第二个模型的输入，还可以使用LangChain的SimpleSequentialChain, 代码如下：

```python
from langchain import PromptTemplate
from langchain_community.llms import Ollama
from langchain.chains import LLMChain, SimpleSequentialChain

# 创建第一条链
template = "我的邻居姓{lastname}，他生了个儿子，给他儿子起个名字"

first_prompt = PromptTemplate(
    input_variables=["lastname"],
    template=template,
)
llm = Ollama(model="qwen2.5:7b")


first_chain = LLMChain(llm=llm, prompt=first_prompt)

# 创建第二条链
second_prompt = PromptTemplate(
    input_variables=["child_name"],
    template="邻居的儿子名字叫{child_name}，给他起一个小名",
)

second_chain = LLMChain(llm=llm, prompt=second_prompt)


# 链接两条链
overall_chain = SimpleSequentialChain(chains=[first_chain, second_chain], verbose=True)

print(overall_chain)
print('*'*80)
# 执行链，只需要传入第一个参数
catchphrase = overall_chain.run("王")
print(catchphrase)
```

### 2.4 Agents (代理)

Agents 也就是代理，它的核心思想是利用一个语言模型来选择一系列要执行的动作。


在 LangChain 中 Agents 的作用就是根据用户的需求，来访问一些第三方工具(比如：搜索引擎或者数据库)，进而来解决相关需求问题。

------

代码示例：

```python
from langchain.agents import initialize_agent
from langchain.agents import AgentType
from langchain_community.llms import Ollama
from langchain_core.prompts import PromptTemplate
from langchain_community.agent_toolkits.load_tools import load_tools

#  实例化大模型
llm = Ollama(model="qwen2.5:7b")

#  设置工具
# "serpapi"实时联网搜素工具、"math": 数学计算的工具
# tools = load_tools(["serpapi", "llm-math"], llm=llm)
tools = load_tools(["llm-math"], llm=llm)

# 实例化代理Agent:返回 AgentExecutor 类型的实例
agent = initialize_agent(tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=False)

# print('agent', agent)
# 准备提示词
prompt_template = """解以下方程：3x + 4(x + 2) - 84 = y; 其中x为3，请问y是多少？"""
prompt = PromptTemplate.from_template(prompt_template)
# print('prompt-->', prompt)

# 代理Agent工作
result = agent.run(prompt)
print(result)

```

>注意，如果运行这个示例你要使用serpapi， 需要申请`serpapi` token，并且设置到环境变量`SERPAPI_API_KEY` ，然后安装依赖包`google-search-results` 

查询所有工具的名称

```python
from langchain.agents import get_all_tool_names
results = get_all_tool_names()
print(results)
# ['python_repl', 'requests', 'requests_get', 'requests_post', 'requests_patch', 'requests_put', 'requests_delete', 'terminal', 'sleep', 'wolfram-alpha', 'google-search', 'google-search-results-json', 'searx-search-results-json', 'bing-search', 'metaphor-search', 'ddg-search', 'google-serper', 'google-scholar', 'google-serper-results-json', 'searchapi', 'searchapi-results-json', 'serpapi', 'dalle-image-generator', 'twilio', 'searx-search', 'wikipedia', 'arxiv', 'golden-query', 'pubmed', 'human', 'awslambda', 'sceneXplain', 'graphql', 'openweathermap-api', 'dataforseo-api-search', 'dataforseo-api-search-json', 'eleven_labs_text2speech', 'google_cloud_texttospeech', 'news-api', 'tmdb-api', 'podcast-api', 'memorize', 'llm-math', 'open-meteo-api']
```

### 2.5 Memory

大模型本身不具备上下文的概念，它并不保存上次交互的内容，ChatGPT之所以能够和人正常沟通对话，因为它进行了一层封装，将历史记录回传给了模型。

因此 LangChain 也提供了Memory组件, Memory分为两种类型：短期记忆和长期记忆。短期记忆一般指单一会话时传递数据，长期记忆则是处理多个会话时获取和更新信息。

目前的Memory组件只需要考虑ChatMessageHistory。举例分析：

```python
from langchain.memory import ChatMessageHistory

history = ChatMessageHistory()
history.add_user_message("在吗？")
history.add_ai_message("有什么事?")

print(history.messages)

```

和 模型结合，直接使用`ConversationChain`：

```python
from langchain import ConversationChain
from langchain_community.llms import Ollama

#  实例化大模型
llm = Ollama(model="qwen2.5:7b")
conversation = ConversationChain(llm=llm)
resut1 = conversation.predict(input="小明有1只猫")
print(resut1)
print('*'*80)
resut2 = conversation.predict(input="小刚有2只狗")
print(resut2)
print('*'*80)
resut3 = conversation.predict(input="小明和小刚一共有几只宠物?")
print(resut3)
print('*'*80)
```

如果要像chatGPT一样，长期保存历史消息，，可以使用`messages_to_dict` 方法

```python
from langchain.memory import ChatMessageHistory
from langchain.schema import messages_from_dict, messages_to_dict

history = ChatMessageHistory()
history.add_user_message("hi!")
history.add_ai_message("whats up?")

dicts = messages_to_dict(history.messages)

print(dicts)

'''
[{'type': 'human', 'data': {'content': 'hi!', 'additional_kwargs': {}, 'type': 'human', 'example': False}}, {'type': 'ai', 'data': {'content': 'whats up?', 'additional_kwargs': {}, 'type': 'ai', 'example': False}}]
'''


# 读取历史消息
new_messages = messages_from_dict(dicts)

print(new_messages)
#[HumanMessage(content='hi!'), AIMessage(content='whats up?')]
```

### 2.6 Indexes (索引)

Indexes组件的目的是让LangChain具备处理文档处理的能力，包括：文档加载、检索等。注意，这里的文档不局限于txt、pdf等文本类内容，还涵盖email、区块链、视频等内容。

Indexes组件主要包含类型：

- 文档加载器
- 文本分割器
- VectorStores
- 检索器

------

#### 2.6.1 文档加载器

文档加载器主要基于`Unstructured` 包，`Unstructured` 是一个python包，可以把各种类型的文件转换成文本。

文档加载器使用起来很简单，只需要引入相应的loader工具：

```python
from langchain.document_loaders import TextLoader
loader = TextLoader('衣服属性.txt', encoding='utf8')
docs = loader.load()
print(docs)
print(len(docs))
a = docs[0].page_content[:4]
print(a)

print('*'*80)
from langchain.document_loaders import UnstructuredFileLoader
loader = UnstructuredFileLoader('衣服属性.txt', encoding='utf8')
docs = loader.load()
print(docs)
print(len(docs))
a = docs[0].page_content[:4]
print(a)
```

#### 2.6.2 文档分割器

由于模型对输入的字符长度有限制，我们在碰到很长的文本时，需要把文本分割成多个小的文本片段。

LangChain中最基本的文本分割器是`CharacterTextSplitter` ，它按照指定的分隔符（默认“\n\n”）进行分割，并且考虑文本片段的最大长度。我们看个例子：

```python
from langchain.text_splitter import CharacterTextSplitter


text_splitter = CharacterTextSplitter(
    separator = " ", # 空格分割，但是空格也属于字符
    chunk_size = 5,
    chunk_overlap  = 1,
)


# 一句分割
a = text_splitter.split_text("a b c d e f")
print(a)

# 多句话分割（文档分割）
texts = text_splitter.create_documents(["a b c d e f", "e f g h"], )
print(texts)
```

#### 2.6.3 VectorStores

VectorStores是一种特殊类型的数据库，它的作用是存储由嵌入创建的向量，提供相似查询等功能。我们使用其中一个`Chroma` 组件`pip install chromadb`作为例子：

```python
from langchain_community.embeddings import OllamaEmbeddings
from langchain.text_splitter import CharacterTextSplitter
from langchain_community.vectorstores import Chroma

# pku.txt内容：<https://www.pku.edu.cn/about.html>
with open('./pku.txt') as f:
    state_of_the_union = f.read()
text_splitter = CharacterTextSplitter(chunk_size=100, chunk_overlap=0)
texts = text_splitter.split_text(state_of_the_union)
print(texts)
embeddings = OllamaEmbeddings(model="mxbai-embed-large")

docsearch = Chroma.from_texts(texts, embeddings)

query = "1937年北京大学发生了什么？"
docs = docsearch.similarity_search(query)
print(docs)
```

LangChain支持的VectorStore如下：

|  VectorStore  |                             描述                             |
| :-----------: | :----------------------------------------------------------: |
|    Chroma     |                     一个开源嵌入式数据库                     |
| ElasticSearch |                        ElasticSearch                         |
|    Milvus     | 用于存储、索引和管理由深度神经网络和其他机器学习（ML）模型产生的大量嵌入向量的数据库 |
|     Redis     |                      基于redis的检索器                       |
|     FAISS     |                  Facebook AI相似性搜索服务                   |
|   Pinecone    |                 一个具有广泛功能的向量数据库                 |

#### 2.6.4 检索器

检索器是一种便于模型查询的存储数据的方式，LangChain约定检索器组件至少有一个方法`get_relevant_texts`，这个方法接收查询字符串，返回一组文档。

```python
from langchain.document_loaders import TextLoader
from langchain.text_splitter import CharacterTextSplitter
from langchain_community.vectorstores import FAISS
from langchain_community.embeddings import OllamaEmbeddings

loader = TextLoader('./pku.txt')
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=100, chunk_overlap=0)
texts = text_splitter.split_documents(documents)

embeddings = OllamaEmbeddings(model="mxbai-embed-large")

db = FAISS.from_documents(texts, embeddings)
retriever = db.as_retriever(search_kwargs={'k': 1})
docs = retriever.get_relevant_documents("北京大学什么时候成立的")
print(docs)

#打印结果：
'''
[Document(metadata={'source': './pku.txt'}, page_content='北京大学创办于1898年，是戊戌变法的产物，也是中华民族救亡图存、兴学图强的结果，初名京师大学堂，是中国近现代第一所国立综合性大学，辛亥革命后，于1912年改为现名。')]
'''
```

## 3 LangChain使用场景

- 个人助手
- 基于文档的问答系统
- 聊天机器人
- Tabular数据查询
- API交互
- 信息提取
- 文档总结

------

