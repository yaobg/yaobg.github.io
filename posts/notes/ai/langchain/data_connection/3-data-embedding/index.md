# LangChain 核心模块：Data Conneciton - Text Embedding Models


<!--more-->

LangChain中基础的Embeddings类公开了两种方法：一种用于对文档进行嵌入，另一种用于对查询进行嵌入。前者输入多个文本，而后者输入单个文本。之所以将它们作为两个独立的方法，是因为某些嵌入提供者针对要搜索的文件和查询（搜索查询本身）具有不同的嵌入方法。

# Document 类

```shell
这段代码定义了一个名为Document的类,允许用户与文档的内容进行交互，可以查看文档的段落、摘要，以及使用查找功能来查询文档中的特定字符串。

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

# openAI切入模型

```shell
from langchain_openai import OpenAIEmbeddings
embeddings_model = OpenAIEmbeddings()
embeddings = embeddings_model.embed_documents(
    [
        "Hi there!",
        "Oh, hello!",
        "What's your name?",
        "My friends call me World",
        "Hello World!"
    ]
)
```

# embed_query 方法嵌入问题

```shell
# QA场景：嵌入一段文本，以便与其他嵌入进行比较。
embedded_query = embeddings_model.embed_query("What was the name mentioned in the conversation?")
```

---

> 作者:   
> URL: https://yaobg.github.io/posts/notes/ai/langchain/data_connection/3-data-embedding/  

