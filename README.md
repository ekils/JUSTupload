建CHUNK NODE 
DM_CHUNK_QUERY= """
MERGE(CreateChunks:DM_Chunk {content: $content})
    ON CREATE SET 
        CreateChunks.dmname = $dmname,
        CreateChunks.filename = $filename
RETURN CreateChunks
"""
def create_DM_Node_Chunks(kg, data_frame):
    node_count = 0
    failed_count = 0
    for index, row in data_frame.iterrows():
        try:
            params = {
                'content': row['content'], 
                'dmname': row['filename'],
                'filename': row['product_name']
            }
            _ = kg.query(DM_CHUNK_QUERY, params=params)
            node_count += 1
        except:
            print(f"Failed to create node for row {index}: {e}")
            failed_count += 1
    print(f"Created {node_count} nodes, failed to create {failed_count} nodes")
    return

create_DM_Node_Chunks(kg, data_frame)
