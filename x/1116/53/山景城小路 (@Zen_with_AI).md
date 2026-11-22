---
author: "山景城小路"
handle: "@Zen_with_AI"
source: "https://x.com/Zen_with_AI/status/2121837934328627358"
date: 2026-11-15T05:29:20.000Z
type: thread
likes: 4
replies: 2
bookmarks: 3
views: 1162
---
# 山景城小路 (@Zen_with_AI)

Meta 数据岗 SQL 单题 “Point of Entry”  
  
Point of Entry：250M 行的 event_data 表，统计每位访客的购买次数，但只算那些第一条记录就是 page_view 的访客；没买过的人记 0；结果按购买次数降序。  
  
关键在两步：先用 ROW_NUMBER() 按 user_id 分组、按 event_id 排序，取每人的第一条事件，筛出第一条是 page_view 的人；再把购买次数聚合表左连接上去，没匹配到的用 COALESCE 补 0。PySpark 等价实现是加分项。  
  
SQL答案：  
  
WITH first_event AS (  
-- 第一步：找出第一条事件就是 page_view 的访客  
SELECT user_id  
FROM (  
SELECT user_id, event_type,  
ROW_NUMBER() OVER (  
PARTITION BY user_id  
ORDER BY event_id  
) AS rn  
FROM event_data  
) t  
WHERE rn = 1  
AND event_type = 'page_view'  
),  
purchase_counts AS (  
-- 第二步：每人买了几次  
SELECT user_id, COUNT(*) AS purchase_count  
FROM event_data  
WHERE event_type = 'purchase'  
GROUP BY user_id  
)  
SELECT f.user_id,  
COALESCE(p.purchase_count, 0) AS purchase_count  
FROM first_event f  
LEFT JOIN purchase_counts p  
ON f.user_id = p.user_id  
ORDER BY purchase_count DESC;

---

有人问 “但只算那些第一条记录就是 page_view 的访客 这句话什么意思” 解释一下：  
  
意思是先给每个访客定一个“入口”，只统计从浏览页面进来的人。  
  
event_data 表里一个访客会有很多条记录，按时间排：page_view（浏览页面）、purchase（下单）等等。“第一条记录”是指这个人最早的那一条事件，不是全表第一行。  
  
举个例子：  
  
访客 A：page_view → purchase → purchase，第一条是 page_view，算，买了 2 次  
访客 B：purchase → page_view，第一条是 purchase，不算，哪怕他买了  
访客 C：page_view → page_view，第一条是 page_view，算，买了 0 次，记 0