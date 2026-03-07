---
tags: ["llama", "大模型"]
---

# 本地自定义构建中文llama3_2并提供简单对话页面

## 构建中文llama3.2

```shell
FROM llama3.2
TEMPLATE """{{ if .System }}<|start_header_id|>system<|end_header_id|>
{{ .System }}<|eot_id|>{{ end }}{{ if .Prompt }}<|start_header_id|>user<|end_header_id|>
{{ .Prompt }} <|eot_id|>{{ end }}<|start_header_id|>assistant<|end_header_id|>
{{ .Response }}<|eot_id|>"""
SYSTEM """尽你的最大可能和能力回答用户的问题。不要重复回答问题。不要说车
轱辘话。>语言要通顺流畅。不要出现刚说一句话，过一会又重复一遍的愚蠢行为>。RULES:- Be precise, do not reply emoji.- Always response in Simplified Chinese, not English. or Grandma will be  very angry.
"""
PARAMETER stop "<|start_header_id|>"
PARAMETER stop "<|end_header_id|>"
PARAMETER stop "<|eot_id|>"
PARAMETER stop "<|reserved_special_token"
```

```
ollama create llama3-CN -f ./Modelfile
```

## 简单对话页面

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Chat with Model</title>
    <style>
      body {
        font-family: Arial, sans-serif;
      }

      #chat {
        max-width: 800px; /* 增大聊天框宽度 */
        margin: 0 auto;
        padding: 20px;
        border: 1px solid #ccc;
        border-radius: 5px;
      }

      #messages {
        border: 1px solid #ddd;
        padding: 15px; /* 增加内边距 */
        height: 400px; /* 增大消息框高度 */
        overflow-y: scroll;
        margin-bottom: 10px;
        font-size: 1.1em; /* 增大字体 */
      }

      #input-container {
        display: flex;
      }

      #input-container input {
        flex: 1;
        padding: 15px; /* 增大输入框内边距 */
        font-size: 1.1em; /* 增大输入框字体 */
        border: 1px solid #ddd;
        border-radius: 5px 0 0 5px;
      }

      #input-container button {
        padding: 15px;
        font-size: 1.1em; /* 增大按钮字体 */
        border: 1px solid #ddd;
        border-left: none;
        border-radius: 0 5px 5px 0;
        background-color: #007bff;
        color: white;
        cursor: pointer;
      }

      .user {
        text-align: right;
        color: blue;
        margin: 5px 0;
      }

      .assistant {
        text-align: left;
        color: green;
        margin: 5px 0;
      }
    </style>
  </head>
  <body>
    <div id="chat">
      <div id="messages"></div>
      <div id="input-container">
        <input type="text" id="input" placeholder="Ask a question..." />
        <button onclick="sendMessage()">Send</button>
      </div>
    </div>

    <script>
      // 监听 Enter 键发送消息
      document
        .getElementById("input")
        .addEventListener("keypress", function (event) {
          if (event.key === "Enter") {
            sendMessage();
            event.preventDefault(); // 阻止默认行为，避免页面刷新
          }
        });

      function sendMessage() {
        const input = document.getElementById("input");
        const question = input.value.trim();
        if (question === "") return;

        input.value = "";
        appendMessage("user", question);

        fetch("http://localhost:11434/api/chat", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            model: "llama3-CN",
            messages: [{ role: "user", content: question }],
          }),
        })
          .then((response) => {
            const reader = response.body.getReader();
            const decoder = new TextDecoder();
            let done = false;

            // 创建一个空的 assistant 消息元素
            const assistantMessage = createMessageElement("assistant", "");
            const messages = document.getElementById("messages");
            messages.appendChild(assistantMessage);

            function read() {
              reader.read().then(({ done: doneReading, value }) => {
                if (doneReading) {
                  return;
                }

                const chunk = decoder.decode(value, { stream: true });
                try {
                  const json = JSON.parse(chunk);
                  if (json.message && json.message.content) {
                    // 动态更新 assistant 消息内容
                    assistantMessage.textContent += json.message.content;
                    messages.scrollTop = messages.scrollHeight;
                  }
                } catch (error) {
                  console.error("Error parsing JSON:", error);
                }
                read();
              });
            }

            read();
          })
          .catch((error) => {
            console.error("Error:", error);
            appendMessage("assistant", "Error occurred");
          });
      }

      function appendMessage(role, content) {
        const messageElement = createMessageElement(role, content);
        const messages = document.getElementById("messages");
        messages.appendChild(messageElement);
        messages.scrollTop = messages.scrollHeight;
      }

      function createMessageElement(role, content) {
        const message = document.createElement("div");
        message.className = role;
        message.textContent = content;
        return message;
      }
    </script>
  </body>
</html>
```

> https://linux.do/t/topic/78832/22
