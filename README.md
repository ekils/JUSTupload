# prompt -> model
prompt = ChatPromptTemplate.from_messages(LLM_RAG_PROMPT)
model_chain = prompt | llm_stream

# 把整個 pipeline 接起來
full_chain = search_chain | model_chain






with st.chat_message("assistant"):
    response = st.write_stream(
        full_chain.stream({"input": question})
    )





prompt = ChatPromptTemplate.from_messages(LLM_RAG_PROMPT)
model_chain = prompt | llm_stream
search_chain = RunnableLambda(search_neo4j)
full_chain = search_chain | model_chain

# 當使用者輸入問題
if question := st.chat_input("What is up?"):
    st.session_state.history.append(HumanMessage(question))
    st.session_state.messages.append({"role": "user", "content": question})

    with st.chat_message("user"):
        st.markdown(question)

    with st.chat_message("assistant"):
        response = st.write_stream(
            full_chain.stream({"input": question})
        )

    st.session_state.messages.append({"role": "assistant", "content": response})






1. 先把 yield_func 包成 LCEL 的 RunnableLambda
from langchain_core.runnables import RunnableLambda

def append_sources(inputs):
    """inputs 會包含 stream response 跟 dm_files"""
    stream_response = inputs["stream_response"]
    dm_files = inputs["dm_files"]
    
    for words in stream_response:
        time.sleep(0.01)
        yield words.content

    if dm_files:
        yield "\n\n以上參考資料來源為："
        for dm in dm_files:
            yield f"\n{dm}"

append_sources_chain = RunnableLambda(append_sources)





2. 然後你的 Full Chain 變成這樣
# 基礎
prompt = ChatPromptTemplate.from_messages(LLM_RAG_PROMPT)
model_chain = prompt | llm_stream
search_chain = RunnableLambda(search_neo4j)

# 包一層，讓 model_chain 的結果和 dm_files 都送到 append_sources_chain
full_chain = search_chain | (
    lambda inputs: {
        "stream_response": model_chain.stream(inputs),
        "dm_files": inputs["dm_files"]
    }
) | append_sources_chain





3. 在 Streamlit 呼叫時：

if question := st.chat_input("What is up?"):
    st.session_state.history.append(HumanMessage(question))
    st.session_state.messages.append({"role": "user", "content": question})

    with st.chat_message("user"):
        st.markdown(question)

    with st.chat_message("assistant"):
        response = st.write_stream(
            full_chain.stream({"input": question})
        )

    st.session_state.messages.append({"role": "assistant", "content": response})

