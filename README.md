def ask_from_neo4j( redis_container_storage, llm_stream, kg, embeddings, user_question, query_info_by_xxx, json_results,data_topic_info_xxx, ResponseThredshold ):

   product = []
   topics1 = []
   topics2 = []

   query_info_by_DMchunk_Blanketsearch = query_info_by_xxx[0]

   json_results_product = json_results[0]
   json_results_topic = json_results[1]
   json_results_ruletopic =  json_results[2]

   data_topic_info = data_topic_info_xxx[0]
   data_topic_info_by_rules = data_topic_info_xxx[1]

   # 依照 條款 topic node搜尋: 
   next_prodcut1, next_topics1 = ask_question(llm_stream, user_question, json_results_product, json_results_topic, data_topic_info)
   next_prodcut1 = rm_duplicate(next_prodcut1)
   next_topics1 = rm_duplicate(next_topics1)
   print("▶ 條款 topic node搜尋 next_topics1: {}".format(next_topics1))

   # 依照 投保規則 topic node 搜尋:  
   next_prodcut2, next_topics2 = ask_question(llm_stream, user_question, json_results_product, json_results_ruletopic, data_topic_info_by_rules)
   next_prodcut2 = rm_duplicate(next_prodcut2)
   next_topics2 = rm_duplicate(next_topics2)
   print("▶ 投保規則 topicrule node 搜尋 next_topics2: {}".format(next_topics2))

   # 合併:
   next_prodcut= next_prodcut1 + next_prodcut2
   next_prodcut = rm_duplicate(next_prodcut)


   if len(next_prodcut)> 0:
       for i, _ in enumerate(next_prodcut):
           redis_container_storage.rpush('product', next_prodcut[i])     
       last_n_elements = redis_container_storage.lrange('product',len(next_prodcut)-2*(len(next_prodcut)) ,-1)
       product = [element.decode('utf-8') for element in last_n_elements]

       # product = next_prodcut
       topics1 = next_topics1
       topics2 = next_topics2
       print("▶ next_prodcut 有值 : {} ".format(next_prodcut))

   else:
       next_prodcut = redis_container_storage.lindex('product', -1).decode('utf-8')
       print("▶ next_prodcut 無值 : {} ".format(product))

   print("▶ PRODUCT : {}  topics1 : {} ;  topics2 : {} ".format(next_prodcut, topics1, topics2))

   #TODO: 20250325新增
   query_result_by_DMchunk = get_query_result(kg, embeddings, \
                                       query_info_by_DMchunk_Blanketsearch, \
                                       user_question,\
                                       next_prodcut, \
                                       next_topics1,\
                                       threshold = 0.75)
   print("query_result_by_DMchunk: {}".format(query_result_by_DMchunk))



   # 將列表中的字典轉換為可哈希的形式
   def make_hashable(d):
       return frozenset((k, tuple(v) if isinstance(v, list) else v) for k, v in d.items())

   topics2 = rm_duplicate(topics2)


   #TODO: 20250325新增
   # 去除重複的字典
   query_result_by_DMchunk = [dict(t) for t in {make_hashable(d) for d in query_result_by_DMchunk}]
   print("▶ query_result_by_DMchunk 數量: {}".format(len(query_result_by_DMchunk)))

   concate_DM_chunks_content = [ i['content']for i in query_result_by_DMchunk]
   concate_DM_chunks_content = "\n".join(concate_DM_chunks_content)

   DM_name =  [ i['dmname']for i in query_result_by_DMchunk]
   DM_name = list(set(DM_name))
   # DM_name = "\n".join(DM_name)

   DM_metadata= [concate_DM_chunks_content, DM_name]

   
   return DM_metadata



def ask_question(llm_stream, user_question, json_results_product, json_results_xxx, data_topic_info_xxx):

    ask_prompt0 = ChatPromptTemplate.from_messages(PROMPT_STEP1)
    
    ask_prompt1 = ChatPromptTemplate.from_messages(PROMPT_STEP2)
    
    ask_prompt2 = ChatPromptTemplate.from_messages(PROMPT_STEP3)
    
    while True:
        try:
            # 確認有無商品
            as_description0= ask_prompt0.format_messages(input=user_question, products =json_results_product )
            llm_recommand_product = llm_stream.invoke(as_description0).content
            llm_recommand_product = ast.literal_eval(llm_recommand_product)
            llm_recommand_product = [i for i in llm_recommand_product if i !=""]
            print_product = ("保險商品為: {}\n".format(llm_recommand_product))
            
            # 從問題確認主題:
            as_description1 = ask_prompt1.format_messages(input=user_question, topics =json_results_xxx )
            llm_recommand_topics = llm_stream.invoke(as_description1).content
            llm_recommand_topics = ast.literal_eval(llm_recommand_topics)
            
            if llm_recommand_topics ==["無"]:
                 print_method1 = ("Method1 ~ 提問問題無法找到對應主題\n")
            else:
                print_method1 = ("Method1 ~ 找到相關聯的主題為: {}\n".format(llm_recommand_topics))
            
            
            #從標籤搜尋確認主題
            as_description2 = ask_prompt2.format_messages(input=user_question )
            llm_keywords = llm_stream.invoke(as_description2).content #取得標籤
            if llm_keywords =="":
                llm_keywords = [user_question]
            else:
                llm_keywords = ast.literal_eval(llm_keywords) #轉 list
            
            # 已索引的文件
            grouped_terms  = data_topic_info_xxx.groupby('Category_int')['Term'].apply(lambda x: ','.join(map(str, x))).tolist()
            documents = grouped_terms 
            # 標籤查詢
            user_query = ",".join(llm_keywords)
            # 取得分數跟index
            sorted_indices, sorted_score = tf_idf_simularity(documents, user_query)
            
            llm_recommand_topics2 = []
            score_list= []
            for index in sorted_indices[:3]:
                llm_recommand_topics2.append("Topic"+ str(index+1))
            # print(top_list)
            for index in sorted_score[:3]:
                score_list.append(index)
            # print(score_list)
            print_method2 = ("Method2 ~ 標籤為:{} 。透過主題標籤找到相關聯的主題為: {}\n".format(llm_keywords, llm_recommand_topics2))
            
            
            llm_recommand_topics = rm_duplicate( llm_recommand_topics2 + llm_recommand_topics)
            llm_recommand_topics=[i for i in llm_recommand_topics if i!='無']
            # most_important_topics = list(set(llm_recommand_topics2) & set(llm_recommand_topics))
            print_llmtopics = ("LLM 搜尋結果主題:{}".format(llm_recommand_topics))
    
            break

        except Exception as e:
             llm_recommand_product= []
             llm_recommand_topics = []
             print("EXCEPTION .....")
             break
    # print(print_product)
    # print(print_method1)
    # print(print_method2)
    # print(print_llmtopics)

    return  llm_recommand_product, llm_recommand_topics
   



from n9_b_imports import *

llm_stream = AzureChatOpenAI(
    azure_endpoint=AZURE_OPENAI_ENDPOINT,
    openai_api_version=AZURE_OPENAI_API_VERSION,
    model_name=AZURE_OPENAI_DEPLOYMENT_NAME,
    openai_api_key=AZURE_OPENAI_API_KEY,
    temperature=0.3,
    streaming=True,
)

embeddings = AzureOpenAIEmbeddings(
    model =AZURE_EMB_MODLE ,
    azure_deployment=AZURE_EMB_DEPLOYMENT,
    azure_endpoint = AZURE_EMB_ENDPOINT,
    openai_api_version=AZURE_EMB_API_VERSION,
    api_key=AZURE_EMB_MODLE_API_KEY
)




# neo4j search:
from n9_b_configs import *
results = kg.query(QUERY_GET_FILENAME)
json_results_product = json.dumps(results, ensure_ascii=False, indent=4)
results = kg.query(QUERY_GET_TOPICS)
json_results_topic = json.dumps(results, ensure_ascii=False, indent=4)
results = kg.query(QUERY_GET_RULETOPICS)

# 讀取條款分類: 
with open('./pickles/data_topic_info.pkl', 'rb') as file:
    data_topic_info = pickle.load(file)

# # 讀取規則分類: 
with open('./pickles/data_ruletopic_info.pkl', 'rb') as file:
    data_ruletopic_info = pickle.load(file)





redis_container_storage = redis.Redis(host='localhost', port=6379, db=0)
kg = Neo4jGraph(url=NEO4J_URI, username=NEO4J_USERNAME, password=NEO4J_PASSWORD, database=NEO4J_DATABASE)


VECTOR_SEARCH_BY_DM_CHUNK_BLANKETSEARCH ="""
        WITH $product AS product
        UNWIND product AS prod
        CALL db.index.vector.queryNodes($index_name, $top_k, $question) YIELD node, score
        WITH node, score, prod
        MATCH (node:DM_Chunk)-[:FROM]->(p:Product {product: prod})
        RETURN score, 
               node.filename AS filename, 
               node.content AS content,
               node.dmname AS dmname, 
               p.product AS product
"""
query_info_by_DMchunk_Blanketsearch = [VECTOR_SEARCH_BY_DM_CHUNK_BLANKETSEARCH , 'DM_emb_index', 'DM_Chunk']
question ="我想詢問,吉美世美元30歲男性躉繳的的費率"

query_info_by_xxx =  [query_info_by_DMchunk_Blanketsearch]
data_topic_info_xxx = [data_topic_info, data_ruletopic_info]
RESPONSETHREDHOLD =0.65


json_results_ruletopic = json.dumps(results, ensure_ascii=False, indent=4)
json_results = [json_results_product, json_results_topic, json_results_ruletopic]

query_result_by_DM_metadata = ask_from_neo4j(
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



    
