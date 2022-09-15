# HttpClient    Tutorial

## Before

~~~xml
       <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>RELEASE</version>
            <scope>compile</scope>
        </dependency>
        <!--httpClient apache-->
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.9</version>
        </dependency>
        <!--http上传文件-->
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpmime</artifactId>
            <version>4.5.9</version>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.8.2</version>
            <scope>test</scope>
        </dependency>
~~~



## Posting with Apache HttpClient

1. ### Post With Json

   ~~~java
   @Test
   public void whenPostJsonUsingHttpClient_thenCorrect() 
     throws ClientProtocolException, IOException {
       CloseableHttpClient client = HttpClients.createDefault();
       HttpPost httpPost = new HttpPost("http://www.example.com");
   
       String json = "{"id":1,"name":"John"}";
       StringEntity entity = new StringEntity(json);
       httpPost.setEntity(entity);
       httpPost.setHeader("Accept", "application/json");
       httpPost.setHeader("Content-type", "application/json");
   
       CloseableHttpResponse response = client.execute(httpPost);
       assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
       client.close();
   }
   ~~~

2. ### POST Multipart  Request

   ~~~java
   @Test
   public void whenSendMultipartRequestUsingHttpClient_thenCorrect() 
     throws ClientProtocolException, IOException {
       CloseableHttpClient client = HttpClients.createDefault();
       HttpPost httpPost = new HttpPost("http://www.example.com");
   
       MultipartEntityBuilder builder = MultipartEntityBuilder.create();
       builder.addTextBody("username", "John");
       builder.addTextBody("password", "pass");
       builder.addBinaryBody(
         "file", new File("test.txt"), ContentType.APPLICATION_OCTET_STREAM, "file.ext");
   	// 注意这里 	
       HttpEntity multipart = builder.build();
       httpPost.setEntity(multipart);
   
       CloseableHttpResponse response = client.execute(httpPost);
       assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
       client.close();
   }
   ~~~

   ## Adding Parameters to Apache  HttpClient  Requests

   1. ~~~java
      // 单个参数添加
      public CloseableHttpResponse sendHttpRequest() {
          HttpGet httpGet = new HttpGet("https://example.com");
          URI uri = new URIBuilder(httpGet.getURI())
            .addParameter("param1", "value1")
            .addParameter("param2", "value2")
            .build();
         ((HttpRequestBase) httpGet).setURI(uri);
          CloseableHttpResponse response = client.execute(httpGet);
          client.close();
      }
      ~~~

   2. ~~~java
      // 很多参数添加
      public CloseableHttpResponse sendHttpRequest() {
          List nameValuePairs = new ArrayList();
          nameValuePairs.add(new BasicNameValuePair("param1", "value1"));
          nameValuePairs.add(new BasicNameValuePair("param2", "value2"));
          HttpGet httpGet = new HttpGet("https://example.com");
          URI uri = new URIBuilder(httpGet.getURI())
            .addParameters(nameValuePairs)
            .build();
         ((HttpRequestBase) httpGet).setURI(uri);
          CloseableHttpResponse response = client.execute(httpGet);
          client.close();
      }
      ~~~

   3. ~~~java
      // 使用UrlEncodedFormEntity 添加参数
      public CloseableHttpResponse sendHttpRequest() {
          List nameValuePairs = new ArrayList();
          nameValuePairs.add(new BasicNameValuePair("param1", "value1"));
          nameValuePairs.add(new BasicNameValuePair("param2", "value2"));
          HttpPost httpPost = new HttpPost("https://example.com");
          httpPost.setEntity(new UrlEncodedFormEntity(nameValuePairs, StandardCharsets.UTF_8));
          CloseableHttpResponse response = client.execute(httpPost);
          client.close();
      }
      ~~~

## Apache HttpClient Basic Authentication

1. ~~~java
    @Test
       void basicAutTest() throws IOException {
           String urlOverHttps
                   = "https://www.baidu.com";
           BasicCredentialsProvider provider = new BasicCredentialsProvider();
           UsernamePasswordCredentials credentials = new UsernamePasswordCredentials("usr1", "password1");
           provider.setCredentials(AuthScope.ANY, credentials);
           CloseableHttpClient client = HttpClientBuilder.create().setDefaultCredentialsProvider(provider).build();
           CloseableHttpResponse response = client.execute(new HttpGet(urlOverHttps));
           Assertions.assertEquals(response.getStatusLine().getStatusCode(), HttpStatus.SC_OK);
       }
   ~~~

2. 

​	