<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>通用可编辑文档页面</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: "Microsoft Yahei", sans-serif;
        }
        body {
            max-width: 1200px;
            margin: 30px auto;
            padding: 0 20px;
            background: #f5f7fa;
        }
        .tool-bar {
            background: #2563eb;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 20px;
            display: flex;
            gap: 12px;
            flex-wrap: wrap;
        }
        button {
            padding: 8px 16px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 14px;
            background: #fff;
            color: #2563eb;
            font-weight: bold;
        }
        button.danger {
            background: #ef4444;
            color: white;
        }
        button.success {
            background: #10b981;
            color: white;
        }
        .edit-box {
            background: white;
            padding: 25px;
            border-radius: 8px;
            box-shadow: 0 2px 12px #00000010;
            margin-bottom: 20px;
        }
        #titleInput {
            width: 100%;
            border: none;
            font-size: 28px;
            font-weight: bold;
            margin-bottom: 20px;
            outline: none;
            border-bottom: 2px solid #eee;
            padding-bottom: 10px;
        }
        #contentInput {
            width: 100%;
            min-height: 180px;
            border: 1px solid #ddd;
            padding: 12px;
            font-size: 16px;
            border-radius: 6px;
            resize: vertical;
            margin-bottom: 20px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 15px 0;
        }
        table td, table th {
            border: 1px solid #ccc;
            padding: 10px;
            min-width: 100px;
        }
        table th {
            background: #eef5ff;
        }
        .table-control {
            margin: 10px 0;
            display: flex;
            gap: 10px;
        }
        .tip-text {
            color: #666;
            font-size: 14px;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <!-- 顶部工具栏 -->
    <div class="tool-bar">
        <button onclick="addRow()">表格新增一行</button>
        <button onclick="addCol()">表格新增一列</button>
        <button onclick="delRow()">删除最后一行</button>
        <button onclick="delCol()">删除最后一列</button>
        <button class="success" onclick="saveLocal()">自动保存</button>
        <button onclick="exportFile()">导出文档</button>
        <button class="danger" onclick="clearAll()">清空所有内容</button>
    </div>

    <!-- 编辑区域 -->
    <div class="edit-box">
        <!-- 标题 -->
        <input type="text" id="titleInput" placeholder="在这里输入文档标题">
        <!-- 正文文本 -->
        <textarea id="contentInput" placeholder="在这里输入正文内容，直接修改即可自动保存"></textarea>

        <!-- 表格控制按钮 -->
        <div class="table-control">
            <span>可编辑表格（单元格直接点击修改文字）</span>
        </div>

        <!-- 可编辑表格 -->
        <table id="editTable" contenteditable="true">
            <thead>
                <tr>
                    <th>表头1</th>
                    <th>表头2</th>
                    <th>表头3</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>内容1</td>
                    <td>内容2</td>
                    <td>内容3</td>
                </tr>
                <tr>
                    <td>数据A</td>
                    <td>数据B</td>
                    <td>数据C</td>
                </tr>
            </tbody>
        </table>

        <p class="tip-text">提示：修改标题、文字、表格后点【自动保存】，关闭页面再打开内容不会消失；点导出可下载文本文件发给别人。</p>
    </div>

    <script>
        // 本地存储key
        const STORAGE_KEY = "editPageData";

        // 页面加载读取保存的数据
        window.onload = function(){
            let oldData = localStorage.getItem(STORAGE_KEY);
            if(oldData){
                let data = JSON.parse(oldData);
                document.getElementById("titleInput").value = data.title;
                document.getElementById("contentInput").value = data.content;
                document.getElementById("editTable").innerHTML = data.tableHtml;
            }
        }

        // 保存到浏览器本地
        function saveLocal(){
            let saveData = {
                title: document.getElementById("titleInput").value,
                content: document.getElementById("contentInput").value,
                tableHtml: document.getElementById("editTable").innerHTML
            }
            localStorage.setItem(STORAGE_KEY, JSON.stringify(saveData));
            alert("保存成功！当前内容已存在浏览器");
        }

        // 导出文件给他人
        function exportFile(){
            let title = document.getElementById("titleInput").value;
            let content = document.getElementById("contentInput").value;
            let table = document.getElementById("editTable").innerText;
            let text = `文档标题：${title}\n正文：${content}\n表格数据：${table}`;
            let blob = new Blob([text], {type:"text/plain"});
            let a = document.createElement("a");
            a.href = URL.createObjectURL(blob);
            a.download = "可编辑文档.txt";
            a.click();
        }

        // 清空全部内容
        function clearAll(){
            if(confirm("确定清空所有已保存内容？无法恢复！")){
                localStorage.removeItem(STORAGE_KEY);
                document.getElementById("titleInput").value = "";
                document.getElementById("contentInput").value = "";
                location.reload();
            }
        }

        // 表格操作
        function addRow(){
            let table = document.querySelector("#editTable tbody");
            let colNum = table.children[0].cells.length;
            let newTr = document.createElement("tr");
            for(let i=0;i<colNum;i++){
                let td = document.createElement("td");
                td.innerText = "新单元格";
                newTr.appendChild(td);
            }
            table.appendChild(newTr);
        }
        function addCol(){
            let allTr = document.querySelectorAll("#editTable tr");
            allTr.forEach(tr=>{
                let cell = document.createElement(tr.parentElement.tagName=="THEAD" ? "th":"td");
                cell.innerText = "新列";
                tr.appendChild(cell);
            })
        }
        function delRow(){
            let tbody = document.querySelector("#editTable tbody");
            if(tbody.children.length>1) tbody.removeChild(tbody.lastChild);
            else alert("至少保留一行数据");
        }
        function delCol(){
            let firstTr = document.querySelector("#editTable tr");
            if(firstTr.cells.length>1){
                let allTr = document.querySelectorAll("#editTable tr");
                allTr.forEach(tr=>tr.removeChild(tr.lastChild));
            }else alert("至少保留一列");
        }
    </script>
</body>
</html>
