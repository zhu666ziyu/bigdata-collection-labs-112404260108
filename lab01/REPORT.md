# Lab 1 实验报告

## 学生信息

- **姓名：朱子羽**
- **学号：112404260108**
- **班级：数据1241**
- **GitHub 用户名：zhu666ziyu**
- **仓库地址：https://github.com/zhu666ziyu/bigdata-collection-labs-112404260108**

> 文件位置：`lab01/REPORT.md`  
> 完整代码保存在 `lab1_html_api.ipynb`。  
> 本报告只填写关键发现、关键代码、结果和指定截图，**不要把整份 Notebook 复制进来**。

---

# 任务 1：判断数据加载方式

## 1.1 Device：判断动态加载并找到 API

### 判断结果

- 页面：`/devices`
- 数据加载方式：` 动态加载`
- 判断依据：在网页源码里搜不到设备名称，在浏览器Network的Fetch/XHR里能拿到设备数据接口

请用 1～2 句话说明：

1. 在“查看网页源代码”中是否能搜索到页面上的设备名称；不能
2. 在 Network 的 Fetch/XHR 中是否找到返回设备数据的请求。能

### API 信息

- Request URL：https://115.159.82/api/devices
- Request Method：GET
- Query 参数：page page_size

### 【截图 1】Device API 发现证据

文件名：

```text
images/01_device_api.png
```

**截图内容必须是浏览器 DevTools，不是 Jupyter 代码。**

截图中应能看清：

1. 浏览器的 **Network** 面板；
2. 已选中真正返回设备列表的 Fetch/XHR 请求；
3. 右侧 **Headers** 中的 Request URL；
4. Query String Parameters（或 URL 中的分页参数）。

> 不要截整个桌面；尽量裁剪到 Network 请求和右侧 Headers 区域。

插入：


![Device API 发现证据](images/01_device_api.png)
```

---

## 1.2 Company：判断静态加载

- 页面：`/companies`
- 数据加载方式：`静态加载 `
- 判断依据：

请说明：使用“右键 → 查看网页源代码”后，是否能搜索到页面中真实显示的企业名称。

### 【截图 2】Company 静态加载证据

文件名：

```text
images/02_company_source.png
```

**截图内容必须是“查看网页源代码”页面，不是 Elements，也不是 Jupyter。**

截图中应能看清：

1. 浏览器正在显示 `/companies` 的网页源代码；
2. 使用 `Ctrl + F` 搜索一个页面中真实存在的企业名称；
3. 源代码中能看到该企业名称或对应企业记录。

插入：


![Device API 发现证据](images/02_company_sourse.png
)


---

# 任务 2：Device API 分页采集

## 2.1 API 分页结构

填写你实际从 JSON 中观察到的字段：

| 含义 | JSON 字段 |
|---|---|
| 当前页 | page|
| 每页条数 | page_size|
| 设备总数 |total |
| 设备列表 | items|

---

## 2.2 关键代码

只填写分页采集的核心部分，不要粘贴整个 Notebook。

```python
# 从第一页响应中读取分页信息
total = data["______total__________"]
page_size = data["__page_size______________"]

total_pages = _(total+page_size-1)//page_size_____________________________________

devices = []

for page in range(1, total_pages + 1):
    r = requests.get(
        api_url,
        params={
            "___page_____": page,
            "___page_size_____": page_size
        },
        timeout=10
    )

    page_data = r.json()
    page_items = page_data["____items____________"]
    ________________________________________________
```

---

## 2.3 采集结果

- 每页条数：50
- 设备总数：5000
- 程序计算得到的总页数：100
- 最终采集设备数：5000
- 唯一 `device_id` 数：5000

回答：

> 为什么总页数应由 API 返回的 `total` 和 `page_size` 计算，而不是直接在代码里写死？

答：网站后端设备数据会动态增减，固定写死页数会出现少采或者空请求报错。

---

# 任务 3：Company HTML 解析

## 3.1 企业记录 Selector

- 企业记录 selector：article.company-card
- 第 1 页企业记录数：10

说明为什么选择这个 selector：页面上每一条企业信息都放在<article class="company-card">标签内，class 为company-card专门用来标识企业卡片，用这个选择器可以一次性选中当前页面全部企业条目，定位稳定简洁。

- 主要使用了 `id / class / data-* / 父子层级` 中的哪一种？
class
- 是否参考过浏览器 Copy selector 或 AI？
参考过浏览器开发者工具，查看页面元素后编写
- 最终为什么采用现在这个写法？

答：因为设备总数会动态变化，不能写死固定页数。

---

## 3.2 关键代码

只填写“记录定位 + 一条记录的关键字段”。

```python
cards = soup.select("__article__.company-item____________________________")

card = cards[0]

company_id = card["data-company-id"]______________________________________
name = card.select_one("h2 a").get_text(strip=True)____________________________________________
status = card.select_one(".badge").get_text(strip=True)__________________________________________
```

行业、区域、设备数量使用 `dt/dd` 结构后，请写出你最终如何得到：

```python
industry = dds[0].get_text(strip=True)________________________________________
zone = dds[1].get_text(strip=True)____________________________________________
device_count = dds[3].get_text(strip=True)____________________________________
```

---

## 3.3 字段来源说明

任选 **2 个字段**说明：

1. 字段：company_id
   - 来自：`HTML 属性`
   - 使用的方法：card["data-company-id"]，直接读取卡片标签上的data自定义属性

2. 字段：name
   - 来自：`文本 `
   - 使用的方法：card.select_one("h2 a").get_text(strip=True)，选中a标签后提取内部文本并去除首尾空白

---

# 任务 4：Company “下一页”分页

## 4.1 下一页元素

- “下一页” selector：a:contains("下一页")
- 第 1 页 `href` 示例：/companies?page=2

### 【截图 3】Company “下一页”元素

文件名：

```text
images/03_company_next.png
```

**截图内容必须是浏览器 DevTools 的 Elements 面板。**

截图中应能看清：

1. 页面底部“下一页”对应的 `<a>` 元素被选中；
2. 该元素的 `class`、`href` 等关键属性；
3. 能判断你为什么使用当前 selector。

> 不需要截 Python 代码；这一张图只证明你找到了正确的 DOM 元素。

插入：


![Company 下一页元素](images/03_company_next.png)
```

---

## 4.2 关键代码

```python
next_link = soup.select_one("______________a______________")

next_href = ___next_link["href"]_____________________________________

next_url = urljoin(
    current_url,
    ________next_href________________________________________
)
```

再用 2～3 句话说明：

> 你的程序如何完成“当前页 → 找下一页 href → 请求下一页 → 再解析”的循环？

答：程序先解析当前页面，查找页面里文字为 “下一页” 的 a 标签，提取 href 链接。使用 urljoin 拼接得到完整下一页地址，发起请求获取新页面 HTML，再次解析页面内容，循环执行直到采集满 3 页。

---

## 4.3 三页采集结果

| 页码 | 企业数 |
|---|---:|
| 1 | 10|
| 2 | 10|
| 3 | 10|
| **合计** | 30|

- 唯一 `company_id` 数：60

---

# 任务 5：结果验证

## 5.1 最终结果

| 检查项 | 结果 |
|---|---:|
| Company 总数 |60 |
| 唯一 `company_id` |60|
| Device 总数 | 5000|
| 唯一 `device_id` | 5000|
| Company–Device 不一致企业数 | 0|

### 【截图 4】最终验证结果

文件名：

```text
images/04_validation.png
```

**截图内容必须是 JupyterLab 的运行结果，不是代码截图，也不是浏览器截图。**

截图中应能同时看清以下输出：

```text
Company 总数
唯一 company_id 数
Device 总数
唯一 device_id 数
Company–Device 不一致企业数
```

建议把这几个 `print()` 放在同一个 Notebook 单元格中运行后截图。

插入：


![最终验证结果](images/04_validation.png)
```

---

## 5.2 一致性检查关键代码

只填写最关键的比较逻辑：

```python
from collections import Counter

device_counts = Counter(
    x["company_id"] for x in devices
)

mismatch_count = 0

for company in companies:
    cid = __________company["company_id"]________________________________
    expected = ____int(re.search(r"\d+", company["device"]).group())_________________________________
    actual = ____device_counts[cid]___________________________________

    if __________expected != actual:___________________________________:
        mismatch_count += 1
```

回答：

> 为什么 `status_code == 200` 仍然不能说明“数据已经采集正确”？

答：status_code=200 仅代表**网络请求成功、服务器正常返回了响应**，不代表返回的数据内容是完整、正确、符合预期的。可能返回空 JSON、错误提示文本、旧缓存数据，数据字段缺失或者数量不对。

---

# 任务 6：保存采集结果

确认已经生成：

```text
data/companies.json
data/devices.json
```

填写：

- `companies.json`：`已生成 `
- `devices.json`：`已生成 `

不需要截图 JSON 全文，也不需要把 JSON 内容复制到报告。

---

# 实验结论

用 **80～150 字**总结：

1. 静态加载与动态加载页面的处理方式有什么不同；
2. HTML “下一页”分页和 API `page` 分页有什么不同；
3. 你使用了哪些方法验证采集结果。

答：静态页面数据直接写在 HTML 源码中，用 BeautifulSoup 解析；动态页面数据通过后端 API 异步返回，需要请求接口获取 JSON。HTML 分页依靠页面里的 “下一页” 超链接跳转，API 分页通过请求参数 page 切换页码。本次实验采集完成后，统计总条数与唯一 ID 数量，对比企业卡片标注设备数和实际采集设备数量，校验数据是否匹配，以此验证采集结果的完整性与准确性。
