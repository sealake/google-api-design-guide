# 简介

## API设计指南

当前API设计指南版本： 2017-02-21

## 介绍

这是网络API的一般设计指南。 自2014年以来，它已在Google内部使用，是Google在设计[Cloud API](https://cloud.google.com/apis/docs/overview)和其他[Google API](https://github.com/googleapis/googleapis)时所遵循的标准。 本设计指南在此共享，以通知外部开发人员，使人们更容易合作。

[Google Cloud Endpoints](https://cloud.google.com/endpoints/docs/grpc)开发人员可能会发现本指南在设计gRPC API时特别有用，我们强烈建议此类开发人员使用这些设计原则。 但是，我们不强制要求使用它。 您仍然可以使用Cloud Endpoints和gRPC，而无需遵循指南。

本指南适用于`REST API`和`RPC API`，特别是`gRPC API`。 gRPC API使用`Protocol Buffers`定义其`API表面`和`API服务配置`来配置其API服务，包括HTTP映射，日志和监控。 HTTP映射特性由`Google API`以及使用 `JSON/HTTP` 到 `Protool Buffers/RPC` 转码的 `Cloud Endpoints gRPC API` 使用。

本指南是"活"的文档，随着时间的推移，新的风格和设计模式得到采纳和批准。 按照这种思路，它永远不会是完整的，而API设计的艺术和技巧将永远有足够的用武之地。

## 本文档中使用的约定

要求等级关键字"MUST"，"MUST NOT"，"REQUIRED"，"SHALL"，"SHALL NOT"，"SHOULD"，"SHOULD NOT"，"RECOMMENDED"，"MAY"和"OPTIONAL"在本文档中的使用方式如[RFC 2119](https://www.ietf.org/rfc/rfc2119.txt)中所述 。

在本文档中，使用**粗体字**体突出显示这些关键字。

