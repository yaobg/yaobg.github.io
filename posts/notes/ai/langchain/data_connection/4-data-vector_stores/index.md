# LangChain 核心模块：Data Conneciton - Vector Stores


<!--more-->

本篇文章采用Chroma Vector Store作为示例，其他Vector Store的安装方式可以参考官方文档。

# 语义搜索
```shell
from langchain.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain.text_splitter import CharacterTextSplitter
from langchain.vectorstores import Chroma

# 加载长文本
raw_documents = TextLoader('../tests/state_of_the_union.txt',encoding='utf-8').load()
# 实例化文本分割器
text_splitter = CharacterTextSplitter(chunk_size=200, chunk_overlap=0)
# 分割文本
documents = text_splitter.split_documents(raw_documents)
# 采用OpenAI Embeddings
embeddings_model = OpenAIEmbeddings()
```

# 使用文本进行语义相似度搜索
```shell
query = "What did the president say about Ketanji Brown Jackson"
docs = db.similarity_search(query)
print(docs[0].page_content)
```
# 使用向量进行语义相似度搜索
```shell
embedding_vector = embeddings_model.embed_query(query)
docs = db.similarity_search_by_vector(embedding_vector)
print(docs[0].page_content)
```

---

> 作者:   
> URL: https://yaobg.github.io/posts/notes/ai/langchain/data_connection/4-data-vector_stores/  

