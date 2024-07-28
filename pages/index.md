```sql orders_by_month
select
    date_trunc('month', order_datetime) as month,
    category,
    count(*) as orders
from orders
group by all
```

<DataTable data={orders_by_month} compact rows=5/>

<OpenAI data={orders_by_month}/>