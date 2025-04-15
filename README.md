node_label:Chunk
product:['台灣人壽吉美世美元利率變動型終身壽險']
question:我想詢問,吉美世美元30歲男性躉繳的的費率
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

node_label:DM_Chunk
product:['台灣人壽吉美世美元利率變動型終身壽險']


有正常query 



question:詢問智能問題該如何手機收訊
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

node_label:DM_Chunk
product:['智能問題']


搜尋尋到




可是neo4j 裡:


DM_Chunk
<elementId>	4:a78ea9c0-8f90-42cd-bd61-874488b395ed:1169
<id>	1169
DMcontentEmbedding	[0.05923118442296982,0.042523812502622604,0.008237045258283615,0.023905759677290916,0.0014607841148972511,-0.028926856815814972,-0.03645850345492363,0.… Show all]
content	Q1. 如何申請手機收信
A1. 
a.開啟瀏覽器進入 [員工網] ，在公司佈告下主題查詢輸入帳號權限按下
search ，會找到員工帳號權限申請手冊 
b.點取員工帳號權限申請手冊 即可下載此文件 
c.打開 [員工帳號權限申請操作手冊 ] ，找到目錄 [ 3. Outlook相關權限類] 
下的… Show all
dmname	智能問題20250224_帳管edited.pdf
filename	智能問題


