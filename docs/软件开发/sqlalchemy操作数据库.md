---
tags: ["python", "sqlalchemy"]
---

# 代码

```python
import time

from sqlalchemy import create_engine, text

DB_URI = "mysql+pymysql://{}:{}@{}:{}/{}".format(
    USERNAME, PASSWORD, HOSTNAME, POST, DATABASE
)
while True:
    # SQLAlchemy 2.0.23
    db_engine = create_engine(DB_URI)
    with db_engine.connect() as conn:
        sql_query = "SELECT * FROM some_table WHERE id=:id"
        result = conn.execute(text(sql_query), {"id": 1})
        print(result.all())
        print("完成", result.rowcount, "行被更新。")
        # 如果是更新等语句要提交
        conn.commit()
    time.sleep(5)
```
