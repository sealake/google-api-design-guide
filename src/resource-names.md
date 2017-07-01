# 资源名称

在面向资源的API中，资源被命名为实体，资源名称是它们的标识符。 每个资源必须有自己唯一的资源名称。 资源名称由资源本身的ID，任意父资源的ID及其API服务名称组成。 下面我们一起看看资源ID以及如何构建资源名称。

gRPC API应该使用无模式(`schema-less`)的URI作为资源名称。它们通常遵循REST URL约定，并且表现得非常像网络文件路径。它们可以轻松映射到REST URL: 相关详细信息，请参阅[标准方法](https://cloud.google.com/apis/design/standard_methods)部分。

集合是一种特殊类型的资源，其中包含相同类型的子资源列表。 例如，目录是文件资源的集合。集合的资源ID称为集合ID。

资源名称使用集合ID和资源ID进行分层组织，以正斜杠`"/"`分隔。 如果一个资源包含一个子资源，子资源的名称是通过指定父资源名称后跟子资源的ID来形成的，再次用正斜杠分隔。

示例1：存储服务具有`buckets`的集合，其中每个桶具有一组`objects`：

API服务名称 | 集合ID | 资源ID | 集合ID | 资源ID
--- | --- | --- | --- | ---
//storage.googleapis.com |  /buckets | /bucket-id | /objects | /object-id

示例2：电子邮件服务包含`users`的集合。 每个用户都有一个`settings`子资源，并且`settings`子资源有许多其他子资源，其中包括`customFrom`：

API服务名称 | 集合ID | 资源ID | 资源ID | 资源ID
--- | --- | --- | --- | ---
//mail.googleapis.com | /users | /name@example.com | /settings | / customFrom

只要在资源层次结构中它们是唯一的，API生产者可以为资源和集合ID选择任何可接受的值。你可以在下面找到更多有关选择恰当的资源和集合ID的准则。

假设没有一个段包含任何可能引起歧义的正斜杠，通过拆分资源名称，例如`name.split("/")[n]`，可以获取各个集合ID和资源ID。

## 完整资源名称

一个无模式(不带http，https，ftp等前缀)的URI 由`DNS兼容`的API服务名称和`资源路径`组成。 资源路径也称为相对资源名称。例如：

```
 "//library.googleapis.com/shelves/shelf1/books/book2"
```

API服务名称用于客户端定位API服务端点; 它`可以(MAY)`是内部服务伪造的DNS名称。 如果API服务名称在上下文中显而易见，则通常使用相对的资源名称。

## 相对资源名称

一个无模式path——没有前导"/"的URI路径。它标识API服务中的资源。例如：

```
"shelves/shelf1/books/book2"
```

## 资源ID

通过其`父资源`中`资源`的非空URI段（[segment-nz-nc](https://tools.ietf.org/html/rfc3986#appendix-A)）识别，参见上述示例。

资源中的资源ID后面可能尾随有多个URI段。 例如：

集合ID | 资源ID
--- | ---
files | /source/py/parser.py

API服务`应该(SHOULD)`在可行时使用URL友好的资源ID。 资源ID `必须(MUST)`清楚记录，无论是由客户端，服务器还是由其他参与者分配。 例如，文件名通常由客户端分配，而电子邮件消息ID通常由服务端分配。

## 集合ID

由其`父资源`中的`集合资源`的非空URI段([segment-nz-nc](https://tools.ietf.org/html/rfc3986#appendix-A))标识，请参见上述示例。

因为集合ID通常出现在生成好的客户端库中，所以它们必须符合以下要求：

+ `必须(MUST)`是有效的C/C++标识符。
+ `必须(MUST)`是复数形式, 符合 `lowerCamel` 命名规则(首字母小写的驼峰式)。
+ `必须(MUST)`使用清晰简明的英文术语。
+ `应该(SHOULD)` 避免过于常见的名称。例如，`RowValue`优于`Value`。以下术语应该无条件避免使用：
    + Element
    + Entry
    + Instance
    + Item
    + Object
    + Resource
    + Type
    + Value

## 资源名称与URL

虽然完整的资源名称与普通URL相似，但它们的涵义并不相同。 单一资源可以由不同版本的API和不同的API协议公开。 完整的资源名称不指定此类信息，因此必须将其映射到特定协议和API版本以供实际使用。

要通过REST API使用完整的资源名称， `必须(MUST)`通过在服务名称之前添加`HTTPS schema`(译者按：google早已全站https，因此其内部协议规定https，这里也可以使用http); 在资源路径之前添加`API主版本`; 以及 对资源路径进行URL转义(URL-escape); 将其转换为REST URL。 例如：

```
// This is a calendar event resource name.
"//calendar.googleapis.com/users/john smith/events/123"

// This is the corresponding HTTP URL.
"https://calendar.googleapis.com/v3/users/john%20smith/events/123"
```
      
## 资源名称作为字符串

除非是向后兼容性问题，否则Google API `必须(MUST)`使用字符串来表示资源名称。 资源名称`应该(SHOULD)`像普通文件路径一样处理，并且不支持％-encoding。 

对于资源定义，第一个字段`应该(SHOULD)`是资源名称的字符串字段，它`应该(SHOULD)`称为`name`。

> 注意：其他与名称相关的字段应符合条件以避免混淆，如`display_name`， `first_name`，`last_name`，`full_name`。

例如：

```
service LibraryService {
    rpc GetBook(GetBookRequest) returns (Book) { 
        option (google.api.http) = { 
            get: "/v1/{name=shelves/*/books/*}"
        };
    }; 

    rpc CreateBook(CreateBookRequest) returns (Book) { 
        option (google.api.http) = { 
            post: "/v1/{parent=shelves/*}/books" 
            body: "book" 
        }; 
    }; 
}

message Book {
    // Resource name of the book. It must have the format of "shelves/*/books/*".
    // For example: "shelves/shelf1/books/book2".
    string name = 1;

    // ... other properties
}

message GetBookRequest {
    // Resource name of a book. For example: "shelves/shelf1/books/book2".
    string name = 1;
}

message CreateBookRequest {
    // Resource name of the parent resource where to create the book.
    // For example: "shelves/shelf1".
    string parent = 1;
    // The Book resource to be created. Client must not set the `Book.name` field.
    Book book = 2;
}

``` 

> 注意：为了资源名称的一致性，前导正斜杠`不能(MUST NOT)`被任何URL模板变量捕获。 例如，`必须(MUST)`使用URL "/v1/{name=shelves/*/books/*}"来代替"/v1{name=/shelves/*/books/*}" 。

