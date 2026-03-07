---
tags: ["quill_js", "记事本"]
---

# 使用quill_js实现本地记事本

## 代码

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>铅笔</title>
    <style>
      html,
      body {
        height: 100%;
        margin: 0;
        display: flex;
        flex-direction: column;
      }

      #editor-container {
        flex: 1;
      }
    </style>
  </head>
  <body>
    <!-- 工具栏容器 -->
    <div id="toolbar-container">
      <span class="ql-formats">
        <select class="ql-header">
          <option value="1"></option>
          <option value="2"></option>
          <option selected></option>
        </select>
        <select class="ql-font"></select>
      </span>
      <span class="ql-formats">
        <button class="ql-bold"></button>
        <button class="ql-italic"></button>
        <button class="ql-underline"></button>
        <button class="ql-strike"></button>
      </span>
      <span class="ql-formats">
        <select class="ql-color"></select>
        <select class="ql-background"></select>
      </span>
      <span class="ql-formats">
        <button class="ql-script" value="sub"></button>
        <button class="ql-script" value="super"></button>
      </span>
      <span class="ql-formats">
        <button class="ql-blockquote"></button>
        <button class="ql-code-block"></button>
      </span>
      <span class="ql-formats">
        <button class="ql-list" value="ordered"></button>
        <button class="ql-list" value="bullet"></button>
        <button class="ql-indent" value="-1"></button>
        <button class="ql-indent" value="+1"></button>
      </span>
      <span class="ql-formats">
        <button class="ql-direction" value="rtl"></button>
        <select class="ql-align"></select>
      </span>
      <span class="ql-formats">
        <button class="ql-link"></button>
        <button class="ql-image"></button>
        <button class="ql-video"></button>
        <button class="ql-formula"></button>
      </span>
      <span class="ql-formats">
        <button class="ql-clean"></button>
      </span>
      <span class="ql-formats">
        <button class="ql-save">save</button>
      </span>
    </div>
    <!-- 编辑器容器 -->
    <div id="editor-container"></div>
    <!-- 引入Quill的库 -->
    <link
      href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/styles/atom-one-dark.min.css"
      rel="stylesheet"
    />
    <script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/highlight.min.js"></script>
    <link
      href="https://cdn.jsdelivr.net/npm/quill@2.0.2/dist/quill.snow.css"
      rel="stylesheet"
    />
    <script src="https://cdn.jsdelivr.net/npm/quill@2.0.2/dist/quill.js"></script>
    <script>
      // 初始化Quill编辑器
      var quill = new Quill("#editor-container", {
        theme: "snow",
        modules: {
          syntax: true,
          toolbar: "#toolbar-container",
        },
        placeholder: "输入点什么...",
      });

      // 加载保存的内容（如果有）
      window.onload = function () {
        var savedContent = localStorage.getItem("quillContent");
        if (savedContent) {
          quill.setContents(JSON.parse(savedContent));
        }
      };

      // 保存内容到localStorage的函数
      function saveContent() {
        var content = quill.getContents();
        localStorage.setItem("quillContent", JSON.stringify(content));
        console.log("Content saved!");
      }

      // 添加保存按钮的点击事件
      document.querySelector(".ql-save").addEventListener("click", function () {
        saveContent();
        alert("Content saved!");
      });

      // 每十五秒自动保存一次
      setInterval(saveContent, 15000);
    </script>
  </body>
</html>
```

![](/images/img_5.png)
