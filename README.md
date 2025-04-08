from langchain_core.runnables import RunnableLambda

def search_neo4j(inputs):
    question = inputs["input"]
    query_info_by_xxx = [query_info_by_chunk, query_info_by_pagetable, query_info_by_chunk_Blanketsearch, query_info_by_DMchunk_Blanketsearch]
    json_results = [json_results_product, json_results_topic, json_results_ruletopic]
    data_topic_info_xxx = [data_topic_info, data_ruletopic_info]

    query_result_by_chunk, query_result_by_pagetable, query_result_by_DM_metadata = ask_from_neo4j(
        redis_container_storage, 
        llm_stream, 
        kg, 
        embeddings, 
        question, 
        query_info_by_xxx, 
        json_results, 
        data_topic_info_xxx, 
        RESPONSETHREDHOLD
    )
    
    query_feed_to_llm = query_result_by_chunk + query_result_by_pagetable + query_result_by_DM_metadata[0]
    
    return {
        "refrence": str(query_feed_to_llm),
        "input": question,
        "dm_files": query_result_by_DM_metadata[1]
    }

search_chain = RunnableLambda(search_neo4j)
