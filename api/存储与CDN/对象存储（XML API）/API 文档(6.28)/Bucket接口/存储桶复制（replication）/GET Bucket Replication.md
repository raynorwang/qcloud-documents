## 功能描述

GET Bucket replication 接口用于查询存储桶中用户存储桶复制配置信息。用户发起该请求时需获得请求签名，表明该请求已获得许可。

## 请求

#### 请求示例

```plaintext
GET /?replication HTTP/1.1
Host: <BucketName-APPID>.cos.<Region>.myqcloud.com
Date: GMT Date
Authorization: Auth String
```

>? 
> - Host: &lt;BucketName-APPID>.cos.&lt;Region>.myqcloud.com，其中 &lt;BucketName-APPID> 为带 APPID 后缀的存储桶名字，例如 examplebucket-1250000000，可参阅 [存储桶概览 > 基本信息](https://cloud.tencent.com/document/product/436/48921#.E5.9F.BA.E6.9C.AC.E4.BF.A1.E6.81.AF) 和 [存储桶概述 > 存储桶命名规范](https://cloud.tencent.com/document/product/436/13312#.E5.AD.98.E5.82.A8.E6.A1.B6.E5.91.BD.E5.90.8D.E8.A7.84.E8.8C.83) 文档；&lt;Region> 为 COS 的可用地域，可参阅 [地域和访问域名](http://cloud.tencent.com/document/product/436/6224) 文档。
> - Authorization: Auth String（详情请参见 [请求签名](https://cloud.tencent.com/document/product/436/7778) 文档）。
> 

#### 请求头

此接口仅使用公共请求头部，详情请参见 [公共请求头部](https://cloud.tencent.com/document/product/436/7728) 文档。


#### 请求体

该请求的请求体为空。

## 响应

#### 响应头

此接口仅返回公共响应头部，详情请参见 [公共响应头部](https://cloud.tencent.com/document/product/436/7729) 文档。

#### 响应体

该响应体返回为 **application/xml** 数据，包含完整节点数据的内容展示如下：

```plaintext
<ReplicationConfiguration>
    <Role>qcs::cam::uin/<OwnerUin>:uin/<SubUin></Role>
    <Rule>
        <Status></Status>
        <Priority></Priority>
        <ID></ID>
        <Filter>
            <And>
                <Prefix></Prefix>
                <Tag>
                    <Key></Key>
                    <Value></Value>
                </Tag>
                <Tag>
                    <Key></Key>
                    <Value></Value>
                </Tag>
            </And>
        </Filter>
        <Destination>
            <Bucket>qcs::cos:<Region>::<BucketName-APPID></Bucket>
            <StorageClass></StorageClass>
        </Destination>
        <DeleteMarkerReplication>
            <Status></Status>
        </DeleteMarkerReplication>
    </Rule>
</ReplicationConfiguration>
```

具体的节点描述如下：

| 节点名称（关键字）       | 父节点                                    | 描述                                                         | 类型      | 
| ------------------------ | ----------------------------------------- | ------------------------------------------------------------ | --------- | 
| ReplicationConfiguration | 无                                        | 说明所有复制配置信息                                      | Container | 
| Role                     | ReplicationConfiguration                  | 发起者身份标示：<br>`qcs::cam::uin/&lt;OwnerUin>:uin/&lt;SubUin>` | String    |  
| Rule                     | ReplicationConfiguration                  | 具体配置信息，最多支持1000个，所有策略只能指向一个目标存储桶 | Container | 
| ID                       | ReplicationConfiguration.Rule             | 用来标注具体 Rule 的名称                                     | String    | 
| Status                   | ReplicationConfiguration.Rule             | 标识 Rule 是否生效，枚举值：Enabled，Disabled                | String    |
|Priority                   | ReplicationConfiguration.Rule             | 规则执行优先级，用于处理目标存储桶相同、复制规则命中同一个对象的情况。存储桶复制规则必须全部携带 Prority 或 全部不携带 Priority。支持设置 1-1000 范围内的正整数，不同规则的 Priority 值不可重复。<li>所有规则都携带 Priority 时，当目标存储桶相同时，不同规则的筛选对象 prefix 可以存在重叠。不同规则命中同一个对象时，会优先触发 Priority 值最小的规则。<li>所有规则都不携带 Priority 时，不同规则的筛选对象 prefix 不允许重叠|Integer    |
Filter                   | ReplicationConfiguration.Rule             | 筛选待复制对象。存储桶功能将复制符合 Filter 中设置的对象前缀、对象标签的对象     |Container    |
|And                   | ReplicationConfiguration.Rule             | 筛选待复制对象时，如果同时需要对象前缀与对象标签条件，需要用 And 包装     |Container    |
| Prefix                   | ReplicationConfiguration.Rule.Filter             | 前缀匹配策略，不可重叠，重叠返回错误，前缀匹配根目录为空     | String    |
| Tag                  | ReplicationConfiguration.Rule.Filter.And             | 筛选待复制对象时，可以用对象标签（支持多个）作为过滤条件，最多支持填入10个标签。添加标签作为筛选条件后，同步删除标记选项必须置为 false	  | String    |
| Destination              | ReplicationConfiguration.Rule             | 目标存储桶信息                                               | Container | 
| Bucket                   | ReplicationConfiguration.Rule.Destination | 资源标识符：<br>`qcs::cos:[region]::[BucketName-APPID]`      | String    | 
| StorageClass             | ReplicationConfiguration.Rule.Destination | 存储类型，枚举值：STANDARD，INTELLIGENT_TIERING，STANDARD_IA，ARCHIVE，DEEP_ARCHIVE，默认值：原存储类型 | String    | 
| DeleteMarkerReplication             | ReplicationConfiguration.Rule | 是否同步删除标记 |Container    |
|Status             | ReplicationConfiguration.Rule. DeleteMarkerReplication | 是否同步删除标记，支持`Disabled`或`Enabled`。默认值为 `Enabled`，即同步删除标记 |String    |

#### 错误码

此接口遵循统一的错误响应和错误码，详情请参见 [错误码](https://cloud.tencent.com/document/product/436/7730) 文档。




## 实际案例

#### 请求

下述请求示例展示了从存储桶`originbucket-1250000000`中查询配置信息。

```plaintext
GET /?replication HTTP/1.1
Date: Fri, 14 Apr 2019 07:17:19 GMT
Authorization: q-sign-algorithm=sha1&q-ak=AKIDZfbOAo7cllgPvF9cXFrJD0a1ICvR98JM&q-sign-time=1503895278;1503895638&q-key-time=1503895278;1503895638&q-header-list=host&q-url-param-list=replication&q-signature=f77900be432072b16afd8222b4b349aabd837cb9
Host: originbucket-1250000000.cos.ap-guangzhou.myqcloud.com
Content-Length: 0
```

#### 响应

上述请求后，COS 返回以下响应，表明当前该存储桶内的存储桶复制配置处于启用状态。该规则配置信息中，复制的内容为存储桶`originbucket-1250000000`内包含`testPrefix`前缀的所有对象。对象副本的存储类型默认跟随源存储桶内对象的存储类型。

```plaintext
Content-Type: application/xml
Content-Length: 309
Connection: keep-alive
Date: Fri, 14 Apr 2019 07:17:19 GMT
Server: tencent-cos
x-cos-replication-rule-creation-time: Fri, 14 Apr 2019 07:06:19 GMT
x-cos-request-id: NWQwMzQ5ZmZfMjBiNDU4NjRfNjAwOV84MzA2****
<ReplicationConfiguration>
    <Role>qcs::cam::uin/100000000001:uin/100000000001</Role>
    <Rule>
        <Status>Enabled</Status>
        <ID>RuleId_01</ID>
        <Prefix>testPrefix</Prefix>
        <Destination>
            <Bucket>qcs::cos:ap-guangzhou::destinationbucket-1250000000</Bucket>
        </Destination>
    </Rule>
</ReplicationConfiguration>
```
