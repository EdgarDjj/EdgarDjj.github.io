# Chrome DevTools


# Chrome DevTools

## 网络面板

1. **控件**。控制功能
2. **过滤器**。可以控制在请求表中显示的内容。
3. **概述**。显示了何时检索资源的时间表。
4. **请求表**。列出检索的到的资源列表，默认按时间顺序排序。
5. **摘要**。告诉请求总数，传输数据量和加载时间等。

![panels](./Chrome-DevTools/panes.png)

默认情况，“请求表”显示以下列。可以“添加”和“删除列”。

![panes2](./Chrome-DevTools/panes2.png)

- **Name**. The name of the resource.
- **Status**. The HTTP status code.
- **Type**. The MIME type of the requested resource.
- **Initiator**. The object or process that initiated the request. It can have one of the following values:
  - **Parser**. Chrome's HTML parser initiated the request.
  - **Redirect**. An HTTP redirect initiated the request.
  - **Script**. A script initiated the request.
  - **Other**. Some other process or action initiated the request, such as the user navigating to a page via a link, or by entering a URL in the address bar.
- **Size**. 服务器提供的响应标头的组合大小（通常为几百个字节）加上响应正文。
- **Time**. 从请求开始到响应中最后一个字节接收的总持续时间。
- **Timeline**. “时间轴”列显示所有网络请求的可视瀑布。单击此列的标题将显示其他排序字段的菜单。

> MIME（Mutipurpose Internet Mail Extensions）互联网标准



**Headers**：与资源关联的HTTP标头

**Preview**：JSON，图像和文本资源的预览

**Response**：HTTP响应数据（如果有）

**Timing**：资源的请求生命周期的详细细分
