# 建立 VECTOR
DM_BUILD_VECTOR_INDEX_CONTENT = """
 CREATE VECTOR INDEX `DM_emb_index` IF NOT EXISTS
 FOR (e:DM_Chunk) ON (e.DMcontentEmbedding)
  OPTIONS { indexConfig: {
    `vector.dimensions`: 1536,
    `vector.similarity_function`: 'cosine'    
 }}
"""
SHOWINDEX = """
SHOW INDEX
""" 
def create_VecotrIndex_content(kg, embeddings):
    neo4j_vector_store = Neo4jVector.from_existing_graph(embedding=embeddings, 
                                    url=NEO4J_URI, 
                                    username=NEO4J_USERNAME, 
                                    password=NEO4J_PASSWORD, 
                                    database=NEO4J_DATABASE,
                                    index_name="DM_emb_index",
                                    node_label='DM_Chunk', 
                                    embedding_node_property='DMcontentEmbedding', 
                                    text_node_properties=['content'])
    _ = kg.refresh_schema()
    print(kg.schema)
    _ = kg.query(DM_BUILD_VECTOR_INDEX_CONTENT) 
    _ = kg.query(SHOWINDEX)
    return 

create_VecotrIndex_content(kg, embeddings)













Node properties:
Product {product: STRING}
DM_Chunk {content: STRING, dmname: STRING, filename: STRING, DMcontentEmbedding: LIST}
Relationship properties:

The relationships:
(:DM_Chunk)-[:FROM]->(:Product)
/storage/pyenv/versions/3.12.5/envs/ACP_LLM_312/lib/python3.12/site-packages/neo4j/_sync/driver.py:547: DeprecationWarning: Relying on Driver's destructor to close the session is deprecated. Please make sure to close the session. Use it as a context (`with` statement) or make sure to call `.close()` explicitly. Future versions of the driver will not close drivers automatically.
  _deprecation_warn(

