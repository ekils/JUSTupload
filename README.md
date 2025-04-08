# prompt -> model
prompt = ChatPromptTemplate.from_messages(LLM_RAG_PROMPT)
model_chain = prompt | llm_stream

# 把整個 pipeline 接起來
full_chain = search_chain | model_chain






with st.chat_message("assistant"):
    response = st.write_stream(
        full_chain.stream({"input": question})
    )






with st.chat_message("assistant"):
    response = st.write_stream(
        full_chain.stream({"input": question})
    )
