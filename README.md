CALL db.index.vector.queryNodes($index_name, $top_k, $question) YIELD node, score
OPTIONAL MATCH (node)-[:FROM]->(p:Product)
WITH node, score, p.product AS product
WHERE product IN $product
RETURN score, 
       node.filename AS filename, 
       node.content AS content,
       node.dmname AS dmname, 
       product
