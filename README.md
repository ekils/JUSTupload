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
