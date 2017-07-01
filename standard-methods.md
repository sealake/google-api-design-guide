
# 标准方法

本章定义了标准方法的概念，即`List`，`Get`，`Create`， `Update`和`Delete`。 标准方法降低复杂性并提高一致性。 Google API存储库中有超过70％的API方法是标准方法，这使得它们更容易学习和使用。

下表介绍了如何将标准方法映射到HTTP方法：

标准方法 | HTTP映射 | HTTP请求体 | HTTP响应体
--- | --- | --- | ---
List | GET <collection URL> | Empty | Resource* list
Get | GET <resource URL> | Empty | Resource*
Create | POST <collection URL> | Resource | Resource*
Update | PUT or PATCH <resource URL> | Resource | Resource*
Delete | DELETE <resource URL> | Empty | Empty**

`*` 如果方法支持`字段隐藏(field masks)`，指定要返回的字段的子集，从`List`，`Get`，`Create`和`Update`方法返回的资源`可能(MAY)`包含部分数据， 在某些情况下，API平台原生支持所有方法的字段隐藏。

`**` 对于`不立即删除`资源（例如更新标志或创建长时间运行的删除操作）的Delete方法，返回的响应应包含长时间运行的操作或修改的资源。

对于在单个API调用的时间范围内未完成的请求，标准方法也`可以(MAY)`返回长时间运行的操作 。

以下部分详细介绍了每种标准方法。示例显示 `.proto` 文件中定义的方法，其中包含`HTTP映射`的特殊注释。 你可以在Google API资源库中找到使用标准方法的许多示例。

## List

`List`方法将集合名称和零个或多个参数作为输入，并返回与输入匹配的资源列表。

List通常用于搜索资源。List适合来自单个集合的数据，该集合的大小有限，而不是高速缓存。对于更广泛的情况，`应该(SHOULD)`使用自定义`Search`方法 。

批处理获取（例如，用多个资源ID并返回每个ID的对象）的方法应该被实现为自定义`BatchGet`方法而不是`List`。但如果有一个已经存在的`List`方法提供相同的功能，则`可以(MAY)`重新使用List方法。如果使用自定义`BatchGet`方法，则`应该(SHOULD)`将其映射到`HTTP GET`。

适用的常见模式：分页，结果排序 。
适用的命名约定：过滤器字段，结果字段

**HTTP映射：**

+ List方法**必须(MUST)**使用`HTTP GET`动词。
+ 列表请求的集合资源名称**应该(SHOULD)**映射到URL路径。如果集合名称映射到URL路径，则URL模板的最后一个段(集合ID)**必须(MUST)**是字面值。
+ 所有剩余的请求消息字段**应该(SHALL)**映射到URL查询参数(Request Params)。
+ 没有请求体（Request Body). API的配置中 **不能(MUST NOT)** 声明一个body子句。
+ 响应数据(Response Body)**应该(SHOULD)**包含资源列表以及可选元数据。

以下代码示例定义了列出图书集合的标准List方法：

```cpp
// Lists all the books on a given shelf.
rpc ListBooks(ListBooksRequest) returns (ListBooksResponse) {
  // List method maps to HTTP GET.
  option (google.api.http) = {
    // The `parent` captures the parent resource name, such as "shelves/shelf1".
    get: "/v1/{parent=shelves/*}/books"
  };
}

message ListBooksRequest {
  // The parent resource name, for example, "shelves/shelf1".
  string parent = 1;

  // The maximum number of items to return.
  int32 page_size = 2;

  // The next_page_token value returned from a previous List request, if any.
  string page_token = 3;
}

message ListBooksResponse {
  // The field name should match the noun "books" in the method name.  There
  // will be a maximum number of items returned based on the page_size field
  // in the request.
  repeated Book books = 1;

  // Token to retrieve the next page of results, or empty if there are no
  // more results in the list.
  string next_page_token = 2;
}
```

## Get

Get方法接收资源名称，零个或多个参数，并返回指定的资源。

**HTTP映射：**

+ Get方法**必须(MUST)**使用`HTTP GET`动词。
+ 资源名称**应该(SHOULD)**映射到URL路径。
+ 所有剩余的请求消息字段**应该(SHOULD)**映射到URL查询参数(Request Parameters)。
+ 没有请求体（Request Body). API的配置中 **不能(MUST NOT)** 声明一个body子句。
+ 返回的资源**应该(SHOULD)**映射到整个响应体(Response Body)。

以下代码示例定义了一个Get指定书籍的标准Get方法：

```cpp
// Gets the specified book.
rpc GetBook(GetBookRequest) returns (Book) {
  // Get maps to HTTP GET. Resource name is mapped to the URL. No body.
  option (google.api.http) = {
    // Note the URL template variable which captures the multi-segment resource
    // name of the requested book, such as "shelves/shelf1/books/book2"
    get: "/v1/{name=shelves/*/books/*}"
  };
}

message GetBookRequest {
  // The field will contain name of the resource requested, for example:
  // "shelves/shelf1/books/book2"
  string name = 1;
}
```

## Create

`Create`方法接收一个集合名称，资源和零个或多个参数。它在指定的资源集合中创建一个新资源，并返回新创建的资源。

如果API支持创建资源，则 **应该(SHOULD)** 为可以创建的每种资源类型建立一个`Create`方法。

HTTP映射：

+ `Create`方法**必须(MUST)**使用`HTTP POST`动词。
+ 请求消息**应该(SHOULD)**有一个字段`${parent}` ，它指定要创建资源的父资源名称。
+ 所有其他的请求消息字段应映射到URL查询参数。
+ 请求**可能(MAY)**包含名为 `<resource>_id` 的字段，以允许调用方选择客户端分配的ID。 该字段**必须(MUST)**映射到URL查询参数。
+ 资源的消息字段**应该(SHOULD)**映射到请求体(Request Body)。 
+ 返回的资源**应当(SHALL)**映射到整个响应体。

如果Create方法支持客户端分配的资源名称，并且资源已经存在，则该请求**应该(SHOULD)**失败，并出现错误代码`google.rpc.Code.ALREADY_EXISTS`或者使用不同的服务器分配的资源名称，文档应该清楚所创建的资源名称可能不同于传入的名称。

以下代码示例定义了一个标准的Create方法，该方法在父架内创建一本书：

```cpp
rpc CreateBook(CreateBookRequest) returns (Book) {
  // Create maps to HTTP POST. URL path as the collection name.
  // HTTP request body contains the resource.
  option (google.api.http) = {
    // The `parent` captures the parent resource name, such as "shelves/1".
    post: "/v1/{parent=shelves/*}/books"
    body: "book"
  };
}

message CreateBookRequest {
  // The parent resource name where the book to be created.
  string parent = 1;

  // The book id to use for this book.
  string book_id = 3;

  // The book resource to create.
  // The field name should match the Noun in the method name.
  Book book = 2;
}

rpc CreateShelf(CreateShelfRequest) returns (Shelf) {
  option (google.api.http) = {
    post: "/v1/shelves"
    body: "shelf"
  };
}

message CreateShelfRequest {
  Shelf shelf = 1;
}
```


## Update

Update方法接收包含资源和零个或多个参数的请求消息。它更新指定的资源及其属性，并返回更新的资源。

除了包含的资源名称或父级的属性，可变资源属性**应该(SHOULD)**由Update方法进行改变。重命名或移动资源的任何功能**不得(MUST NOT)**在Update方法中发生，而**应该(SHOULD)**由自定义方法处理。

**HTTP映射：**

+ 标准的Update方法**应该(SHOULD)**支持部分资源更新，对于部分更新的场景，使用`HTTP PATCH`动词并携带更新的字段句柄。(TODO: 此处建议参照原文，因为google的文档同时描述了rpc和restful场景，不能简单拿来套用在设计RESTful风格的接口中)
+ **应该(SHOULD)**通过自定义方法提供需要更高级修补语义(如附加到重复字段)的Update方法 。
+ 如果Update方法仅支持完全资源更新，则**必须(MUST)**使用`HTTP PUT`。但是，非常不鼓励完全更新，因为在添加新的资源字段时它具有向后的兼容性问题。
+ 资源名称**必须(MUST)**映射到URL路径。该字段**可能(MAY)**在资源消息本身中。
+ 资源数据**必须(MUST)**映射到请求体。
+ 所有其他的请求消息字段**必须(MUST)**映射到URL查询参数。
+ 响应消息**必须(MUST)**是更新的资源本身。

如果API接受客户端分配的资源名称，则服务器**可以(MAY)**允许客户端指定不存在的资源名称并创建新的资源。否则，Update方法应该返回"资源不存在"的失败响应。 如果它是唯一的错误条件，则应使用错误代码`NOT_FOUND`。

如果API提供了用于资源创建的Update方法，还**应该(SHOULD)**提供Create方法。理由是，如果Update方法是唯一的方法，它不清楚如何创建资源。

以下代码示例定义了Update指定书籍的标准Update方法：

```cpp
rpc UpdateBook(UpdateBookRequest) returns (Book) {
  // Update maps to HTTP PATCH. Resource name is mapped to a URL path.
  // Resource is contained in the HTTP request body.
  option (google.api.http) = {
    // Note the URL template variable which captures the resource name of the
    // book to update.
    patch: "/v1/{book.name=shelves/*/books/*}"
    body: "book"
  };
}

message UpdateBookRequest {
  // The book resource which replaces the resource on the server.
  Book book = 1;

  // The update mask applies to the resource. For the `FieldMask` definition,
  // see https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#fieldmask
  FieldMask update_mask = 2;
}
```


## 删除

Delete方法使用资源名称以及零个或多个参数作为输入，`删除`或`调度以删除`指定的资源。 Delete方法**应该(SHOULD)**返回`google.protobuf.Empty`(这里是rpc的协议)。

API**不应该(SHOULD NOT)**依赖于Delete方法返回的任何信息，因为它不能重复调用。

**HTTP映射：**

* Delete方法**必须(MUST)**使用`HTTP DELETE`动词。
* 资源名称**应该(SHOULD)**映射到URL路径。
* 所有其他的请求消息字段**应该(SHOULD)**映射到URL查询参数。
* 没有请求体。API配置**不能(MUST NOT)**声明一个body子句。
* 如果Delete方法立即删除资源，它**应该(SHOULD)**返回一个空的响应。
* 如果Delete方法启动长时间运行的操作，则**应该(SHOULD)**返回长时间运行的操作。
* 如果Delete方法仅将资源标记为已删除，则**应该(SHOULD)**返回更新的资源。

Delete方法的调用效果应该是幂等的，但不需要产生相同的响应。 任何数量的Delete请求**应当(SHOULD)**导致资源（最终）被删除，但只有第一个请求返回成功代码。 后续请求应该导致一个`google.rpc.Code.NOT_FOUND`。
 
以下代码示例定义了删除指定书籍的标准Delete方法：

```cpp
rpc DeleteBook(DeleteBookRequest) returns (google.protobuf.Empty) {
  // Delete maps to HTTP DELETE. Resource name maps to the URL path.
  // There is no request body.
  option (google.api.http) = {
    // Note the URL template variable capturing the multi-segment name of the
    // book resource to be deleted, such as "shelves/shelf1/books/book2"
    delete: "/v1/{name=shelves/*/books/*}"
  };
}

message DeleteBookRequest {
  // The resource name of the book to be deleted, for example:
  // "shelves/shelf1/books/book2"
  string name = 1;
}
```
