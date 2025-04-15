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
        RESPONSETHREDHOLD
    )


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
        RESPONSETHREDHOLD
    )


cypher_query: 
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

index_name: DM_emb_index
node_label: DM_Chunk
question:我想詢問,吉美世美元30歲男性躉繳的的費率
---------------------------------------------------------------------------
ClientError                               Traceback (most recent call last)
Cell In[164], line 69
     66 json_results_ruletopic = json.dumps(results, ensure_ascii=False, indent=4)
     67 json_results = [json_results_product, json_results_topic, json_results_ruletopic]
---> 69 query_result_by_DM_metadata = ask_from_neo4j(
     70         redis_container_storage, 
     71         llm_stream, 
     72         kg, 
     73         embeddings, 
     74         question, 
     75         query_info_by_xxx, 
     76         RESPONSETHREDHOLD
     77     )

Cell In[163], line 10, in ask_from_neo4j(redis_container_storage, llm_stream, kg, embeddings, user_question, query_info_by_xxx, ResponseThredshold)
      7 query_info_by_DMchunk_Blanketsearch = query_info_by_xxx[0]
      9 #TODO: 20250325新增
---> 10 query_result_by_DMchunk = get_query_result(kg, embeddings, \
     11                                     query_info_by_DMchunk_Blanketsearch, \
     12                                     user_question,\
     13                                     threshold = 0.75)
     14 print("query_result_by_DMchunk: {}".format(query_result_by_DMchunk))
     18 # 將列表中的字典轉換為可哈希的形式

Cell In[163], line 42, in get_query_result(kg, embeddings, query_info, user_question, threshold)
     41 def get_query_result(kg, embeddings, query_info, user_question, threshold):
---> 42     query_result = neo4j_vector_search(kg, embeddings, query_info, user_question)
     43     query_result =[i for i in query_result if i['score'] >= threshold]
     44     return query_result

Cell In[163], line 62, in neo4j_vector_search(kg, embeddings, query_info, question)
     56 print("node_label: {}".format(node_label))
     60 print("question:{}".format(question))
---> 62 similar = kg.query(cypher_query, 
     63                  params={
     64                   'question': question_emb, 
     65                   'index_name':index_name, 
     66                   'node_label': node_label, 
     67                   'top_k': 1000})
     68 return similar

File /storage/pyenv/versions/3.12.5/envs/ACP_LLM_312/lib/python3.12/site-packages/langchain_community/graphs/neo4j_graph.py:430, in Neo4jGraph.query(self, query, params, retry_on_session_expired)
    428 with self._driver.session(database=self._database) as session:
    429     try:
--> 430         data = session.run(Query(text=query, timeout=self.timeout), params)
    431         json_data = [r.data() for r in data]
    432         if self.sanitize:

File /storage/pyenv/versions/3.12.5/envs/ACP_LLM_312/lib/python3.12/site-packages/neo4j/_sync/work/session.py:327, in Session.run(self, query, parameters, **kwargs)
    325 bookmarks = self._get_bookmarks()
    326 parameters = dict(parameters or {}, **kwargs)
--> 327 self._auto_result._run(
    328     query,
    329     parameters,
    330     self._config.database,
    331     self._config.impersonated_user,
    332     self._config.default_access_mode,
    333     bookmarks,
    334     self._config.notifications_min_severity,
    335     self._config.notifications_disabled_classifications,
    336 )
    338 return self._auto_result

File /storage/pyenv/versions/3.12.5/envs/ACP_LLM_312/lib/python3.12/site-packages/neo4j/_sync/work/result.py:231, in Result._run(self, query, parameters, db, imp_user, access_mode, bookmarks, notifications_min_severity, notifications_disabled_classifications)
    229 self._pull()
    230 self._connection.send_all()
--> 231 self._attach()

File /storage/pyenv/versions/3.12.5/envs/ACP_LLM_312/lib/python3.12/site-packages/neo4j/_sync/work/result.py:425, in Result._attach(self)
    423 if self._exhausted is False:
    424     while self._attached is False:
--> 425         self._connection.fetch_message()

File /storage/pyenv/versions/3.12.5/envs/ACP_LLM_312/lib/python3.12/site-packages/neo4j/_sync/io/_common.py:184, in ConnectionErrorHandler.__getattr__.<locals>.outer.<locals>.inner(*args, **kwargs)
    182 def inner(*args, **kwargs):
    183     try:
--> 184         func(*args, **kwargs)
    185     except (Neo4jError, ServiceUnavailable, SessionExpired) as exc:
    186         assert not asyncio.iscoroutinefunction(self.__on_error)

File /storage/pyenv/versions/3.12.5/envs/ACP_LLM_312/lib/python3.12/site-packages/neo4j/_sync/io/_bolt.py:994, in Bolt.fetch_message(self)
    990 # Receive exactly one message
    991 tag, fields = self.inbox.pop(
    992     hydration_hooks=self.responses[0].hydration_hooks
    993 )
--> 994 res = self._process_message(tag, fields)
    995 self.idle_since = monotonic()
    996 return res

File /storage/pyenv/versions/3.12.5/envs/ACP_LLM_312/lib/python3.12/site-packages/neo4j/_sync/io/_bolt5.py:496, in Bolt5x0._process_message(self, tag, fields)
    494 self._server_state_manager.state = self.bolt_states.FAILED
    495 try:
--> 496     response.on_failure(summary_metadata or {})
    497 except (ServiceUnavailable, DatabaseUnavailable):
    498     if self.pool:

File /storage/pyenv/versions/3.12.5/envs/ACP_LLM_312/lib/python3.12/site-packages/neo4j/_sync/io/_common.py:254, in Response.on_failure(self, metadata)
    252 handler = self.handlers.get("on_summary")
    253 Util.callback(handler)
--> 254 raise self._hydrate_error(metadata)

ClientError: {code: Neo.ClientError.Statement.ParameterMissing} {message: Expected parameter(s): product}
1
