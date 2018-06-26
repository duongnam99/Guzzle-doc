 # Guzzle 6
## Khởi đầu nhanh  
Trang này cung cấp phần khởi đầu nhanh với Guzzle và một vài ví dụ giới thiệt. Nếu bạn chưa cài Guzzle, hãy đến trang [cài đặt](http://docs.guzzlephp.org/en/stable/overview.html#installation)

## Tạo một request  
Bạn có thể gửi các request với Guzzle sử dụng đối tượng GuzzleHttp\ClientInterface  

## Tạo  một Client 
``` Sử dụng GuzzleHttp\Client;
$client = new Client([
    // Base URI is used with relative requests
    'base_uri' => 'http://httpbin.org',
    // You can set any number of default request options.
    'timeout'  => 2.0,
]);
```
Các Clients là bất biến trong Guzzle 6, điều đó có nghĩa bạn không thể thay đổi chế độ mặc định của clients sau khi tạo nó.  
Client constructor chấp nhận mảng liên kết của ác tùy chọn:
`base_uri`  
(string|UriInterface) Base URI của client được sát nhập vào thành các URI tương đối. Có thể là một chuỗi hoặc một trường hợp của UriInterface. Khi một URI tương đối được cung cấp cho client, client sẽ kết hợp base URI với URI tương đối và sử dụng các quy tắc được môt tả trong [RFC 3986, section 2](https://tools.ietf.org/html/rfc3986#section-5.2)  

```
// Create a client with a base URI
$client = new GuzzleHttp\Client(['base_uri' => 'https://foo.com/api/']);
// Send a request to https://foo.com/api/test
$response = $client->request('GET', 'test');
// Send a request to https://foo.com/root
$response = $client->request('GET', '/root');
```
Không cảm thấy rằng minh đang đọc RFC 3986? dưới đây là vài ví dụ nhanh về cách mà một `base_uri` được xử lí nhanh bởi URI khác.  

| base_uri                  | URI              | Result                |
| ------------------------- | ---------------- | --------------------- |
| http://foo.com            | /bar             | http://foo.com/bar    |
| http://foo.com/foo        | /bar             | http://foo.com/bar    |
| http://foo.com/foo        | bar              | http://foo.com/bar    |
| http://foo.com/foo/       | bar              | http://foo.com/foo/bar|
| http://foo.com            | http://baz.com   | http://baz.com        |
| http://foo.com/?bar       | bar              | http://foo.com/bar    |

`handler` : (callable) 1 hàm chuyển các HTTP request trên đường dẫn. Hàm được gọi với 1 Psr7HttpMessageRequestInterface và mảng các tùy chọn chuyển, và phải trả về 1 GuzzleHttpPromisePromiseInterface mà làm đúng theo Psr7HttpMessageResponseInterface khi thành công. handler là 1 khởi tạo mà tùy chọn không thể được ghi đè trong các tùy chọn per/request

... : (mixed) Tất cả các tùy chọn khác được truyền vào hàm khởi tạo được sử dụng như các tùy chọn request mặc định với mỗi request được tạo bởi client.

## Gửi request
Các Magic methods trên client khiến việc của các request đồng bộ trở nên dễ dàng
```
$response = $client->get('http://httpbin.org/get');
$response = $client->delete('http://httpbin.org/delete');
$response = $client->head('http://httpbin.org/get');
$response = $client->options('http://httpbin.org/get');
$response = $client->patch('http://httpbin.org/patch');
$response = $client->post('http://httpbin.org/post');
$response = $client->put('http://httpbin.org/put');
```  
Bạn có thể tạo một request và sau đó gửi request bằng client khi bạn sẵn sàng:
```
**sử dụng** GuzzleHttp\Psr7\Request;

$request = new Request('PUT', 'http://httpbin.org/put');
$response = $client->send($request, ['timeout' => 2]);
```  
Các đối tượng client cung cấp một giải pháp linh động trong việc làm thế nào để request được vận chuyển với các tùy chon mặc định, các middleware mặc định được xử dụng  bởi mỗi request, và  1 URI gốc cho phép bạn gửi các request với các URI tương đối.  
Bạn có thể tìm hiểu thêm về client middleware trong trang [Handlers and Middleware](http://docs.guzzlephp.org/en/stable/handlers-and-middleware.html) của tài liệu.

## Các request không đồng bộ
Bạn có thẻ sử dụng các request bất đồng bộ bằng cách sử dụng các magic method được cung cấp bởi client:
```
$promise = $client->getAsync('http://httpbin.org/get');
$promise = $client->deleteAsync('http://httpbin.org/delete');
$promise = $client->headAsync('http://httpbin.org/get');
$promise = $client->optionsAsync('http://httpbin.org/get');
$promise = $client->patchAsync('http://httpbin.org/patch');
$promise = $client->postAsync('http://httpbin.org/post');
$promise = $client->putAsync('http://httpbin.org/put');
```
Bạn cũng có thể sử dụng phương thức sendAsync() và requestAsync() của client:
```
use GuzzleHttp\Psr7\Request;

// Create a PSR-7 request object to send
$headers = ['X-Foo' => 'Bar'];
$body = 'Hello!';
$request = new Request('HEAD', 'http://httpbin.org/head', $headers, $body);

// Or, if you don't need to pass in a request instance:
$promise = $client->requestAsync('GET', 'http://httpbin.org/get');
```
Promise trả về của những hàm này implements Promises/A+ spec, được cung cấp bởi [Guzzle promises library](https://github.com/guzzle/promises). Điều đó có nghĩa bạn có thể gọi hàm then() nối sau promise. Những lời gọi sau cũng thỏa mãn 1 PsrHttpMessageResponseInterface thành công hoặc bị từ chối với 1 ngoại lệ.
```
use Psr\Http\Message\ResponseInterface;
use GuzzleHttp\Exception\RequestException;

$promise = $client->requestAsync('GET', 'http://httpbin.org/get');
$promise->then(
    function (ResponseInterface $res) {
        echo $res->getStatusCode() . "\n";
    },
    function (RequestException $e) {
        echo $e->getMessage() . "\n";
        echo $e->getRequest()->getMethod();
    }
);
```
## Các request đồng thời  
Bạn có thể gửi nhiều request cùng một lúc bằng các sử dụng các promise và các request bất đồng bộ
```
use GuzzleHttp\Client;
use GuzzleHttp\Promise;

$client = new Client(['base_uri' => 'http://httpbin.org/']);

// Initiate each request but do not block
$promises = [
    'image' => $client->getAsync('/image'),
    'png'   => $client->getAsync('/image/png'),
    'jpeg'  => $client->getAsync('/image/jpeg'),
    'webp'  => $client->getAsync('/image/webp')
];

// Wait on all of the requests to complete. Throws a ConnectException
// if any of the requests fail
$results = Promise\unwrap($promises);

// Wait for the requests to complete, even if some of them fail
$results = Promise\settle($promises)->wait();

// You can access each result using the key provided to the unwrap
// function.
echo $results['image']['value']->getHeader('Content-Length')[0]
echo $results['png']['value']->getHeader('Content-Length')[0]
```
Bạn có thể sử dụng GuzzleHttp\Pool object khi bạn có một số request không định lượng đưuọc mà bạn cần gửi:
```
use GuzzleHttp\Pool;
use GuzzleHttp\Client;
use GuzzleHttp\Psr7\Request;

$client = new Client();

$requests = function ($total) {
    $uri = 'http://127.0.0.1:8126/guzzle-server/perf';
    for ($i = 0; $i < $total; $i++) {
        yield new Request('GET', $uri);
    }
};

$pool = new Pool($client, $requests(100), [
    'concurrency' => 5,
    'fulfilled' => function ($response, $index) {
        // this is delivered each successful response
    },
    'rejected' => function ($reason, $index) {
        // this is delivered each failed request
    },
]);

// Initiate the transfers and create a promise
$promise = $pool->promise();

// Force the pool of requests to complete.
$promise->wait();
```
Hoặc sử dụng một closure mà sẽ trả về promise một khi pool gọi closure
```
$client = new Client();

$requests = function ($total) use ($client) {
    $uri = 'http://127.0.0.1:8126/guzzle-server/perf';
    for ($i = 0; $i < $total; $i++) {
        yield function() use ($client, $uri) {
            return $client->getAsync($uri);
        };
    }
};

$pool = new Pool($client, $requests(100));
```

## Sử dụng các Response
Ở phần ví dụ trước, chúng ta đã lấy một biến $response  hoặc chúng ta cung cấp một reponse từ promise. Đối tượng reponse thực thi một PSR-7 response, Psr\Http\Message\ResponseInterface, và chứa đựng rất nhiều thông tin hữu ích.  
Chúng ta có thể lấy được status code và reason phrase của response:
```
$code = $response->getStatusCode(); // 200
$reason = $response->getReasonPhrase(); // OK
```
Bạn có thể lấy được header từ response:
```
// Check if a header exists.
if ($response->hasHeader('Content-Length')) {
    echo "It exists";
}

// Get a header from the response.
echo $response->getHeader('Content-Length');

// Get all of the response headers.
foreach ($response->getHeaders() as $name => $values) {
    echo $name . ': ' . implode(', ', $values) . "\r\n";
}
```
Phần body của response cũng có thể lấy được bằng cách sử dụng  phương thức getBody. Phần body có thể dược dùng như một chuỗi, ép thành chuỗi, hoặc được dùng như một luồng đối tượng
```
$body = $response->getBody();
// Implicitly cast the body to a string and echo it
echo $body;
// Explicitly cast the body to a string
$stringBody = (string) $body;
// Read 10 bytes from the body
$tenBytes = $body->read(10);
// Read the remaining contents of the body as a string
$remainingBytes = $body->getContents();
```

## Query String Parameters (Truy vấn tham số chuỗi)
Bạn có thể tạo query string parameters với một request bằng một vài cách:
 Truy vấn tham số chuỗi trong URI của request
 ```
$response = $client->request('GET', 'http://httpbin.org?foo=bar');
```
Bạn có thể xác định query string parameters bằng cách sử dụng query request dưới dạng chuỗi.
```
$client->request('GET', 'http://httpbin.org', [
    'query' => ['foo' => 'bar']
]);
```
Tạo một tùy chọn dạng mảng sẽ sử dụng hàm http_build_query của PHP để định dạng chuỗi query
Và cuối cùng, bạn có thể tạo lựa chọn query request như là một chuỗi
```
$client->request('GET', 'http://httpbin.org', ['query' => 'foo=bar']);
```

## Upload dữ liệu
Guzzle cung cấp một số phương thức để upload dữ liệu.
Bạn có thể gửi các request chứa luồng dữ liệu bằng các truyền vào chuỗi, resource trả về từ fopen, hay một instance của Psr\Http\Message\StreamInterface cho tùy chọn body request
```
// Provide the body as a string.
$r = $client->request('POST', 'http://httpbin.org/post', [
    'body' => 'raw data'
]);

// Provide an fopen resource.
$body = fopen('/path/to/file', 'r');
$r = $client->request('POST', 'http://httpbin.org/post', ['body' => $body]);

// Use the stream_for() function to create a PSR-7 stream.
$body = \GuzzleHttp\Psr7\stream_for('hello!');
$r = $client->request('POST', 'http://httpbin.org/post', ['body' => $body]);
```
Cách dễ dàng để upload dữ liêu json và cài đặt header thích hợp là sử dụng tùy chọn `json` request
```
$r = $client->request('PUT', 'http://httpbin.org/put', [
    'json' => ['foo' => 'bar']
]);
```

## Các POST/Form Request
Để xác định raw data của request sử dụng tùy chọn body request, Guzzle cung cấp các abstraction hữu dụng khi gửi dữ liệu POST 

### Gửi từ các trường
Gửi các request POST application/x-www-form-urlencoded yêu cầu bạn phải đặc tả trường POST như 1 mảng dưới các tùy chọn request form_params
```
$response = $client->request('POST', 'http://httpbin.org/post', [
    'form_params' => [
        'field_name' => 'abc',
        'other_field' => '123',
        'nested_field' => [
            'nested' => 'hello'
        ]
    ]
]);
```
### Gửi từ file 
Bạn có thể gửi các file qua một form (multipart/form-data POST requests), sử dụng tùy chọn multipart request. multipart chấp nhận một mảng của các mảng liên kết, nơimà tưng mảng liên kết chứa các keys sau:
- name: (required, string) khóa nôis đến trường tên trong form
- contents: (required, mixed) cung cấp một chuỗi để gửi nội dung của file như là một chuỗi, cung cấp fopen resource để chuyển nội dung từ luồng php, hoặc cung cấp Psr\Http\Message\StreamInterface để chuyển nội dung từ một luồng PSR-7
    ```
    $response = $client->request('POST', 'http://httpbin.org/post', [
        'multipart' => [
            [
                'name'     => 'field_name',
                'contents' => 'abc'
            ],
            [
                'name'     => 'file_name',
                'contents' => fopen('/path/to/file', 'r')
            ],
            [
                'name'     => 'other_file',
                'contents' => 'hello',
                'filename' => 'filename.txt',
                'headers'  => [
                    'X-Foo' => 'this is an extra header to include'
                ]
            ]
        ]
    ]);
    ```
## Cookies
Guzzle có thể duy trì cookie session cho bạn nếu được yêu cầu sử dụng tùy chọn cookies request. Khi gửi môt request, tùy chọn cookie phải được gắn với instance của GuzzleHttp\Cookie\CookieJarInterface.
```
// Use a specific cookie jar
$jar = new \GuzzleHttp\Cookie\CookieJar;
$r = $client->request('GET', 'http://httpbin.org/cookies', [
    'cookies' => $jar
]);
```
Bạn có thể cài đặt cookie về true trong client contructor nếu bạn muốn dùng shared cookie jar cho tất cả các request
```
// Use a shared client cookie jar
$client = new \GuzzleHttp\Client(['cookies' => true]);
$r = $client->request('GET', 'http://httpbin.org/cookies');
```

## Chuyển hướng: 
Guzzle sẽ tự động theo dõi chuyển hướng trừ bạn không muốn nó làm vậy. Bạn có thể tùy chỉnh hành động điều hướng bằng các sử dụng tùy chọn allow_redirects request
- Đặt thành `true` để bật điều hướng bình thường với tối đa 5 lần điều hướng. Đây là cài đặt mặc định.
- Cài về `false` để tắt điều hướng
- Truyền một mảng liên kết bao gòm `max` key để xác định số điều hướng tối đa và  có thể thêm khóa 'strict' để chỉ rõ rằng liệu điều hướng RFC compliant có được sử dụng hay không (có nghĩa là điều hướng các POST request với các POST so với việc hầu hết các trình duyệt sẽ làm đó là điều hướng các POST request với các GET request).
```
$response = $client->request('GET', 'http://github.com');
echo $response->getStatusCode();
// 200
```
Ví dụ dưới dây cho thấy ridirect có thể bị tắt
```
$response = $client->request('GET', 'http://github.com', [
    'allow_redirects' => false
]);
echo $response->getStatusCode();
// 301
```

## Ngoại lệ
Guzzle sẽ ném ngoại lệ cho những lỗi xảy ra tron g  quá tr ình  truyền tải:
- Trong sự kiện lỗi mạng:  ()connection timeout, DNS errors, vv...).GuzzleHttp\Exception\RequestException được ném ra. Ngoại lệ  này kết thừa từ  GuzzleHttp\Exception\TransferException. Bắt được ngoại lệ này sẽ bắt được bất kì ngọa lệ  nào mà có thể xuất hiện trong qúa trình tryuyền t ải request.
    ```
    use GuzzleHttp\Psr7;
    use GuzzleHttp\Exception\RequestException;
    
    try {
        $client->request('GET', 'https://github.com/_abc_123_404');
    } catch (RequestException $e) {
        echo Psr7\str($e->getRequest());
        if ($e->hasResponse()) {
            echo Psr7\str($e->getResponse());
        }
    }
    ```
- Ngoại lệ GuzzleHttp\Exception\ConnectException được ném ra khi có sự kiện  llỗi mạng. Ngọa lệ này kết tthùa từ  GuzzleHttp\Exception\RequestException.
- GuzzleHttpExceptionClientException được ném ra mức lỗi 400 nếu tùy chọn request http_errors được gán thành true. Ngoại lệ này kế thừa từ GuzzleHttpExceptionBadResponseException và GuzzleHttpExceptionBadResponseException kế thừa từ GuzzleHttpExceptionRequestException.
    ```
      use GuzzleHttpExceptionClientException;
    try { $client->request('GET', 'https://github.com/_abc_123_404'); } catch (ClientException $e) { echo Psr7str($e->getRequest()); echo Psr7str($e->getResponse()); }
    ```
- GuzzleHttpExceptionServerException được ném ra cho lỗi mức 500 nếu tùy chọn request http_errors được gán thành true. Ngoại lệ này kế thừa từ GuzzleHttpExceptionBadResponseException.

- GuzzleHttpExceptionTooManyRedirectsException sẽ được ném ra khi có quá nhiều lần điều hướng xảy ra. Ngoại lệ này kế thừa từ GuzzleHttpExceptionRequestException.

Tất cả những ngoại lệ trên đều kế thừa từ GuzzleHttpExceptionTransferException.

## Biến môi trường

Guzzle cung cấp một vài biến môi trường đmà có thể sđược dùngể tuỳ chỉnh hành động của thư viện.  

`GUZZLE_CURL_SELECT_TIMEOUT` : 
điều chỉnh khoảng thời gian (vài giây) mà một curl_multi_* handler sử dụng khi chọn curl handles bằng cách sử dụng curl_multi_select(). Một vài hệ thống có vấn đề với các triển khai của PHP của curl_multi_select() khi mà việc gọi hàm này luôn dẫn tới việc phải chờ trong khoảng thời gian timeout tối đa.  

`HTTP_PROXY :`
Định nghĩa proxy được sử dụng khi gửi các request sử dụng giao thức "http".

Ghi chú: vì biến HTTP_PROXY có thể chứa đầu vào tùy ý từ người dùng àrtn một vài môi trường (CGI), nên nó chỉ được sử dụng trên CLI SAPI. Xem thêm thông tin phía dưới.

`HTTPS_PROXY :`  
Định nghĩa proxy nào được sử dụng khi gửi các request mà sử dụng giao thức "https".







