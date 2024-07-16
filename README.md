# Llama2_with_llamaindex
RAG System Using Llama2 With Hugging Face

## RAG System Using Llama2 With Hugging Face
!pip install pypdf
!pip install -q transformers einops accelerate langchain bitsandbytes
## Embedding
!pip install install sentence_transformers
!pip install llama_index
from llama_index import VectorStoreIndex,SimpleDirectoryReader,ServiceContext
from llama_index.llms import HuggingFaceLLM
from llama_index.prompts.prompts import SimpleInputPrompt
documents=SimpleDirectoryReader("/content/data").load_data()
documents
system_prompt="""
You are a Q&A assistant. Your goal is to answer questions as
accurately as possible based on the instructions and context provided.
"""
## Default format supportable by LLama2
query_wrapper_prompt=SimpleInputPrompt("<|USER|>{query_str}<|ASSISTANT|>")
!huggingface-cli login
import torch

llm = HuggingFaceLLM(
    context_window=4096,
    max_new_tokens=256,
    generate_kwargs={"temperature": 0.0, "do_sample": False},
    system_prompt=system_prompt,
    query_wrapper_prompt=query_wrapper_prompt,
    tokenizer_name="meta-llama/Llama-2-7b-chat-hf",
    model_name="meta-llama/Llama-2-7b-chat-hf",
    device_map="auto",
    # uncomment this if using CUDA to reduce memory usage
    model_kwargs={"torch_dtype": torch.float16 , "load_in_8bit":True}
)
from langchain.embeddings.huggingface import HuggingFaceEmbeddings
from llama_index import ServiceContext
from llama_index.embeddings import LangchainEmbedding

embed_model=LangchainEmbedding(
    HuggingFaceEmbeddings(model_name="sentence-transformers/all-mpnet-base-v2"))
service_context=ServiceContext.from_defaults(
    chunk_size=1024,
    llm=llm,
    embed_model=embed_model
)
service_context
index=VectorStoreIndex.from_documents(documents,service_context=service_context)
index
query_engine=index.as_query_engine()
response=query_engine.query("what is attention is all you need?")
print(response)
response=query_engine.query("what is YOLO?")
print(response)
