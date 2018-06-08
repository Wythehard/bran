# 基于PHPUnit和Guzzle的Api单元测试工具

## 安装

执行`composer require subtlephp/bran`

## 使用

1. 使用`Bran\Raven`代替`PHPUnit\Framework\TestCase`来创建新的测试类

2. 示例代码如下:

```php
<?php

namespace tests\Api\CpsConsoleApi;

use Bran\Raven;
use PHPUnit\Framework\Assert;

class OrderApiTest extends Raven
{
    protected $clientConfigList = [
        'docker' => [
            'base_uri' => 'http://openresty_1',
            'headers' => [
                'host' => 'xxxx',
            ],
        ]
    ];

    public function __construct($name = null, array $data = [], $dataName = '')
    {
        parent::__construct($name, $data, $dataName);
        $this->clientConfig = $this->clientConfigList[ENV];
    }

    protected $apiConfig = [
        'getOrderDetails' => [
            'pattern' => '/api/order/getOrde',
            'method' => 'get',
        ],
    ];

    /**
     * 针对具体的测试实例，这是框架的写法，符合 phpunit 的模式。
     */
    public function testGetOrderDetails()
    {
        $this->call('getOrder', [
            'query' => [
                'orderNumber' => 123729102900263941,
            ]
        ])
            ->assertStatusCode(200)
            ->assertJsonBodyAttributeEquals(10000, 'errno')
            ->assertJsonBodyHas('data');
    }

    
}
```


或者使用 checkflow 的方式进行

```php
  public function testListValue ()
    {
        // 传参  注意一下，get 方式传参，数组 key 为  `query`     post put delete 等方式，key 为  `form_params` 如果上传的是文件
        // 更多参数参见 guzzle 中文文档 http://guzzle-cn.readthedocs.io/zh_CN/latest/quickstart.html#post
        $request_opt = [
            'query' => [
                'region' => 'xxx',
                'status' => "Running",
            ]
        ];
        // checkflow 对于数据的寻找是使用  data.xxx.xxx 的方式进行寻找，接口放回的数据是  json 格式，会先转化为数组 $response
        // 如果我们 断言 errno 的值为 10000，那么就是说 $response[’errno‘] = 10000 ，如果层级比较深，断言第一个 value 的 name 为 asd
        // 那么需要  $response['data']['value'][0]['name'] = 'asd', 层级过深，不雅观，于是我们改为  data.value.0.name 来寻址。
        // 
        // checkflow 数组中：
        // status 断言响应码的值
        // checkValues  以键值对的方式断言值，key 为寻址地址，value 为断言值
        // checkKeys  是断言，数组是否含有某某  key，如果是根节点，寻址 key 为  `.`
        //  'data' => ['pageNumber', 'pageSize', 'totalCount'] 代表 data 节点下一定有 pageNumber pageSize totalCount 这三个 key。要断言具
        //体的值是多少，请放到  checkValues 中。
        $checkFlow = [
            'status' => 200,
            'checkValues' => [
                'errno' => 10000,
                'data.value.0.name' => 'asd',
            ],
            'checkKeys' => [
                '.' => 'data',
                'data' => ['pageNumber', 'pageSize', 'totalCount']
            ]
        ];

        // 执行请求，并按照 checkflow 断言执行。
        $raven = $this->call('listvalue', $request_opt)
            ->checkFlow($checkFlow);

        // 如果需要取出返回值，可以使用 getData 方法，寻址方式仍然用  data.value.0.name 的格式
        $data = $raven->getData('data');
        if (count($data['value']) < $data['pageSize']) {
            $raven->assertEquals(count($data['value']), $data['totalCount']);
        } else {
            $raven->assertEquals(count($data['value']), $data['pageSize']);
        }

        if ($data['totalCount'] > 0) {
            $region = [];
            foreach ($data['values'] as $value) {
                array_push($region, $value['region']);
            }
            $region = array_unique($region);

            // 调用 phpunit 的方法来执行，某些零碎的断言。
            Assert::assertEquals('1', count($region));
            Assert::assertEquals($request_opt['query']['region'], array_shift($region));
        }
    }
```
