
prompt = ChatPromptTemplate.from_messages( LLM_RAG_PROMPT)
chain = prompt | llm_stream


if "history" not in st.session_state:
    st.session_state.history = [SystemMessage(content="")]

AIMessage_temp = []
def yield_func(query_feed_to_llm, dm_files, qution):
    for words in chain.stream({'refrence': query_feed_to_llm , 'input':qution}):
        time.sleep(0.01)
        AIMessage_temp.append(words.content)
        yield words.content
    if dm_files !=[]:
        yield """\n
        以上參考資料來源為:"""
        for dm in dm_files:
            yield f"""\n
            {dm}"""

st.title('Chat Model: {}'.format(AZURE_OPENAI_DEPLOYMENT_NAME))
# Initialize chat history
if "messages" not in st.session_state:
    st.session_state.messages = []

# Display chat messages from history on app rerun
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

# Accept user input
if question := st.chat_input("What is up?"):
    st.session_state.history.append(HumanMessage(question))
    # Add user message to chat history
    st.session_state.messages.append({"role": "user", "content": question})
    # Display user message in chat message container
    with st.chat_message("user"):
        st.markdown(question)

        query_info_by_xxx = [query_info_by_chunk, query_info_by_pagetable, query_info_by_chunk_Blanketsearch, query_info_by_DMchunk_Blanketsearch]
        json_results = [json_results_product, json_results_topic, json_results_ruletopic]
        data_topic_info_xxx = [data_topic_info, data_ruletopic_info]

        query_result_by_chunk, query_result_by_pagetable, query_result_by_DM_metadata = ask_from_neo4j(  redis_container_storage, \
                                                                            llm_stream, \
                                                                            kg, \
                                                                            embeddings,\
                                                                            question,\
                                                                            query_info_by_xxx, \
                                                                            json_results,\
                                                                            data_topic_info_xxx,\
                                                                            RESPONSETHREDHOLD  
                                                                            )

    # Display assistant response in chat message container
    with st.chat_message("assistant"):
        query_feed_to_llm =[] 
        query_feed_to_llm += query_result_by_chunk
        query_feed_to_llm += query_result_by_pagetable
        query_feed_to_llm += query_result_by_DM_metadata[0]

        print("\n ========== Query_result_by_chunk ========")
        print(query_result_by_chunk)
        print("\n")
        print("\n ========== Query_result_by_pagetable ========")
        print(query_result_by_pagetable)
        print("\n")
        print("\n ========== Query_result_by_DM_chunk ========")
        print(query_result_by_DM_metadata[0])
        print("\n")

        response = st.write_stream(yield_func(str(query_feed_to_llm),query_result_by_DM_metadata[1], st.session_state.history))
        AIMessage_temp = ''.join([item for item in AIMessage_temp if item])
        st.session_state.history.append(AIMessage(AIMessage_temp))

        redis_product_list = redis_container_storage.lrange('product', 0, -1) 
        if len(redis_product_list) > 1 :
            if redis_product_list[-1]!= redis_product_list[-2]:
                st.session_state.history = []
                last_n_elements = [element.decode('utf-8') for element in redis_container_storage.lrange('product', -1, -1)][0]
                _ = redis_container_storage.flushdb()
                _ = redis_container_storage.rpush('product', last_n_elements)

    # Add assistant response to chat history
    st.session_state.messages.append({"role": "assistant", "content": response})
