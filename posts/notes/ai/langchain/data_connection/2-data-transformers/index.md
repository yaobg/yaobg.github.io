# LangChain 核心模块：Data Conneciton - Document Transformers


<!--more-->

# Document 类

这段代码定义了一个名为Document的类，允许用户与文档的内容进行交互，可以查看文档的段落、摘要，以及使用查找功能来查询文档中的特定字符串。

```python
# 基于BaseModel定义的文档类。
class Document(BaseModel):
"""接口，用于与文档进行交互。"""

    # 文档的主要内容。
    page_content: str
    # 用于查找的字符串。
    lookup_str: str = ""
    # 查找的索引，初次默认为0。
    lookup_index = 0
    # 用于存储任何与文档相关的元数据。
    metadata: dict = Field(default_factory=dict)

    @property
    def paragraphs(self) -> List[str]:
        """页面的段落列表。"""
        # 使用"\n\n"将内容分割为多个段落。
        return self.page_content.split("\n\n")

    @property
    def summary(self) -> str:
        """页面的摘要（即第一段）。"""
        # 返回第一个段落作为摘要。
        return self.paragraphs[0]

    # 这个方法模仿命令行中的查找功能。
    def lookup(self, string: str) -> str:
        """在页面中查找一个词，模仿cmd-F功能。"""
        # 如果输入的字符串与当前的查找字符串不同，则重置查找字符串和索引。
        if string.lower() != self.lookup_str:
            self.lookup_str = string.lower()
            self.lookup_index = 0
        else:
            # 如果输入的字符串与当前的查找字符串相同，则查找索引加1。
            self.lookup_index += 1
        # 找出所有包含查找字符串的段落。
        lookups = [p for p in self.paragraphs if self.lookup_str in p.lower()]
        # 根据查找结果返回相应的信息。
        if len(lookups) == 0:
            return "No Results"
        elif self.lookup_index >= len(lookups):
            return "No More Results"
        else:
            result_prefix = f"(Result {self.lookup_index + 1}/{len(lookups)})"
            return f"{result_prefix} {lookups[self.lookup_index]}"
```

# Text Splitters 文本分割器

当你想处理长篇文本时，有必要将文本分成块。听起来很简单，但这里存在着潜在的复杂性。理想情况下，你希望将语义相关的文本片段放在一起。"
语义相关"的含义可能取决于文本类型。这个笔记本展示了几种实现方式。

从高层次上看，文本分割器的工作原理如下：

将文本分成小而有意义的块（通常是句子）。
开始将这些小块组合成较大的块，直到达到某个大小（通过某个函数进行测量）。
一旦达到该大小，使该块成为自己独立的一部分，并开始创建一个具有一定重叠（以保持上下文关系）的新文本块。
这意味着您可以沿两个不同轴向定制您的文本分割器：

如何拆分文字
如何测量块大小
使用 RecursiveCharacterTextSplitter 文本分割器
该文本分割器接受一个字符列表作为参数，根据第一个字符进行切块，但如果任何切块太大，则会继续移动到下一个字符，并以此类推。默认情况下，它尝试进行切割的字符包括 ["\n\n", "\n", " ", ""]

除了控制可以进行切割的字符外，您还可以控制其他一些内容：

- length_function：用于计算切块长度的方法。默认只计算字符数，但通常在这里传递一个令牌计数器。
- chunk_size：您的切块的最大大小（由长度函数测量）。
- chunk_overlap：切块之间的最大重叠部分。保持一定程度的重叠可以使得各个切块之间保持连贯性（例如滑动窗口）。
- add_start_index：是否在元数据中包含每个切块在原始文档中的起始位置。

## 分割文本

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
# 加载待分割长文本
with open('../tests/state_of_the_union.txt', encoding='utf-8') as f:
    state_of_the_union = f.read()

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size = 100,
    chunk_overlap  = 20,
    length_function = len,
    add_start_index = True,
)
docs = text_splitter.create_documents([state_of_the_union])
print(docs[0])
print(docs[1])
# 多个文档
metadatas = [{"document": 1}, {"document": 2}]
documents = text_splitter.create_documents([state_of_the_union, state_of_the_union], metadatas=metadatas)
print(documents[0])
```

## 分割代码

```python
from langchain.text_splitter import Language
# 支持编程语言的完整列表
[e.value for e in Language]
html_splitter = RecursiveCharacterTextSplitter.from_language(
    language=Language.HTML, chunk_size=60, chunk_overlap=0
)
html_docs = html_splitter.create_documents([html_text])
print(len(html_docs))
```

---

> 作者:   
> URL: https://yaobg.github.io/posts/notes/ai/langchain/data_connection/2-data-transformers/  

