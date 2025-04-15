WITH $product AS product_list
UNWIND product_list AS prod
CALL db.index.vector.queryNodes($index_name, $top_k, $question) YIELD node, score
WITH node, score, prod
MATCH (node:DM_Chunk)-[:FROM]->(p:Product {product: prod})
RETURN score, 
       node.filename AS filename, 
       node.content AS content,
       node.dmname AS dmname, 
       p.product AS product
