---
tags: ["pocketbase"]
---

# 权限放行

表对应的api_rules需要设置为`@request.auth.id != ""`，只有登录认证后的用户才可以操作。

# 函数封装

```python
headers = {
    "Authorization": token
}


def records():
    url = "https://test.com/api/collections/posts/records?page=1&perPage=50"
    return requests.get(url, headers=headers).json()


def record(id):
    url = f"https://test.com/api/collections/posts/records/{id}"
    return requests.get(url, headers=headers).json()


def create_record(data):
    url = "https://test.com/api/collections/posts/records"
    return requests.post(url, headers=headers, json=data).json()


def delete_record(id):
    url = f"https://test.com/api/collections/posts/records/{id}"
    try:
        data = requests.delete(url, headers=headers).json()
    except:
        data = {"id": id}
    return data


def update_record(id, data):
    url = f"https://test.com/api/collections/posts/records/{id}"
    return requests.patch(url, headers=headers, json=data).json()


print(delete_record("c9w208d7mgh0ziu"))
print(create_record({"name": "qfx"}))
print(records())
print(record("c9w208d7mgh0ziu"))
print(update_record("44z3c02j65203k0", {"name": "钱甫新"}))
```

# 类封装

```python
import requests


class PostAPI:
    BASE_URL = "https://test.com/api/collections/posts/records"
    LOGIN_URL = "https://test.com/api/collections/users/auth-with-password"

    def __init__(self, identity, password):
        self.token = self.login(identity, password)
        self.headers = {"Authorization": self.token}

    def login(self, identity, password):
        """登录并获取 Token"""
        response = requests.post(self.LOGIN_URL, json={"identity": identity, "password": password}).json()
        if "token" in response:
            return response["token"]
        raise ValueError(f"登录失败: {response}")

    def get_all(self, page=1, per_page=50):
        """获取所有记录"""
        url = f"{self.BASE_URL}?page={page}&perPage={per_page}"
        return requests.get(url, headers=self.headers).json()

    def get_one(self, record_id):
        """获取单条记录"""
        url = f"{self.BASE_URL}/{record_id}"
        return requests.get(url, headers=self.headers).json()

    def create(self, data):
        """创建新记录"""
        return requests.post(self.BASE_URL, headers=self.headers, json=data).json()

    def update(self, record_id, data):
        """更新记录"""
        url = f"{self.BASE_URL}/{record_id}"
        return requests.patch(url, headers=self.headers, json=data).json()

    def delete(self, record_id):
        """删除记录"""
        url = f"{self.BASE_URL}/{record_id}"
        try:
            data = requests.delete(url, headers=self.headers).json()
        except:
            data = {"id": record_id}
        return data


identity = "test@163.com"
password = "12345678"

api = PostAPI(identity, password)

# 获取所有记录
records = api.get_all()
print(records)
# 获取单条记录
record = api.get_one("515v170sh79pt0h")
print(record)

# 创建记录
new_data = {"name": "作者"}
created_record = api.create(new_data)
print(created_record)

# 更新记录
update_data = {"name": "哈哈哈哈"}
updated_record = api.update("515v170sh79pt0h", update_data)
print(updated_record)

# 删除记录
deleted_record = api.delete("8q5k6618bkimfy5")
print(deleted_record)

```
