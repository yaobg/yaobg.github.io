# LangChain 核心模块：Data Conneciton - Document Loaders


<!--more-->

Document 类
这段代码定义了一个名为Document的类，允许用户与文档的内容进行交互，可以查看文档的段落、摘要，以及使用查找功能来查询文档中的特定字符串。

# 基于BaseModel定义的文档类。

```python
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

BaseLoader 类定义
BaseLoader
类定义了如何从不同的数据源加载文档，并提供了一个可选的方法来分割加载的文档。使用这个类作为基础，开发者可以为特定的数据源创建自定义的加载器，并确保所有这些加载器都提供了加载数据的方法。load_and_split方法还提供了一个额外的功能，可以根据需要将加载的文档分割为更小的块。

# 基础加载器类。

```python
class BaseLoader(ABC):
"""基础加载器类定义。"""

    # 抽象方法，所有子类必须实现此方法。
    @abstractmethod
    def load(self) -> List[Document]:
        """加载数据并将其转换为文档对象。"""

    # 该方法可以加载文档，并将其分割为更小的块。
    def load_and_split(
        self, text_splitter: Optional[TextSplitter] = None
    ) -> List[Document]:
        """加载文档并分割成块。"""
        # 如果没有提供特定的文本分割器，使用默认的字符文本分割器。
        if text_splitter is None:
            _text_splitter: TextSplitter = RecursiveCharacterTextSplitter()
        else:
            _text_splitter = text_splitter
        # 先加载文档。
        docs = self.load()
        # 然后使用_text_splitter来分割每一个文档。
        return _text_splitter.split_documents(docs)
```

# 使用 TextLoader 加载 Txt 文件

```python
from langchain.document_loaders import TextLoader

docs = TextLoader('../tests/state_of_the_union.txt',encoding='utf-8').load()
```

# 使用 ArxivLoader 加载 ArXiv 论文

```python
ArxivLoader 类定义
ArxivLoader 类专门用于从Arxiv平台获取文档。用户提供一个搜索查询，然后加载器与Arxiv API交互，以检索与该查询相关的文档列表。这些文档然后以标准的Document格式返回。

# 针对Arxiv平台的加载器类。
class ArxivLoader(BaseLoader):
    """从`Arxiv`加载基于搜索查询的文档。

    此加载器负责将Arxiv的原始PDF文档转换为纯文本格式，以便于处理。
    """

    # 初始化方法。
    def __init__(
        self,
        query: str,
        load_max_docs: Optional[int] = 100,
        load_all_available_meta: Optional[bool] = False,
    ):
        self.query = query
        """传递给Arxiv API进行搜索的特定查询或关键字。"""
        self.load_max_docs = load_max_docs
        """从搜索中检索文档的上限。"""
        self.load_all_available_meta = load_all_available_meta
        """决定是否加载与文档关联的所有元数据的标志。"""

    # 基于查询获取文档的加载方法。
    def load(self) -> List[Document]:
        arxiv_client = ArxivAPIWrapper(
            load_max_docs=self.load_max_docs,
            load_all_available_meta=self.load_all_available_meta,
        )
        docs = arxiv_client.search(self.query)
        return docs
ArxivLoader有以下参数：

query：用于在ArXiv中查找文档的文本
load_max_docs：默认值为100。使用它来限制下载的文档数量。下载所有100个文档需要时间，因此在实验中请使用较小的数字。
load_all_available_meta：默认值为False。默认情况下只下载最重要的字段：发布日期（文档发布/最后更新日期）、标题、作者、摘要。如果设置为True，则还会下载其他字段。
以 GPT-3 论文（Language Models are Few-Shot Learners） 为例，展示如何使用 ArxivLoader

GPT-3 论文的 Arxiv 链接：https://arxiv.org/abs/2005.14165
```

# 使用 UnstructuredURLLoader 加载网页内容

使用非结构化分区函数(Unstructured)来检测MIME类型并将文件路由到适当的分区器(partitioner)。

支持两种模式运行加载程序："single"和"elements"。如果使用"single"模式，文档将作为单个langchain Document对象返回。如果使用"
elements"模式，非结构化库将把文档拆分成标题和叙述文本等元素。您可以在mode后面传入其他非结构化kwargs以应用不同的非结构化设置。

UnstructuredURLLoader 主要参数：

urls：待加载网页 URL 列表
continue_on_failure：默认True，某个URL加载失败后，是否继续
mode：默认single，
以 ReAct 网页为例（https://react-lm.github.io/） 展示使用

```python
from langchain.document_loaders import UnstructuredURLLoader
urls = [
    "https://react-lm.github.io/",
]
loader = UnstructuredURLLoader(urls=urls)
data = loader.load()
```

---

> 作者:   
> URL: https://yaobg.github.io/posts/notes/ai/langchain/data_connection/1-data-loader/  

