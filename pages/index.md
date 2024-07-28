---
title: Orders by Category
---

```sql orders_by_month
select
    date_trunc('month', order_datetime) as month,
    category,
    count(*) as orders
from orders
group by all
```


<BarChart data={orders_by_month} series=category/>

<OpenAI data={orders_by_month}/>