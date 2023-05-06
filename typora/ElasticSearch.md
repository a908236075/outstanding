# ElastichSearch 

## CRUD

1. ```java
   public Map<String, Integer> queryEventType(QeuryVehicleDetailReq req, String index) {
       SearchRequest searchRequest = new SearchRequest(index);
       IndexRequest addRequest = new IndexRequest(index);
       UpdateByQueryRequest updateByQueryRequest = new UpdateByQueryRequest(index);
       DeleteByQueryRequest deleteByQueryRequest = new DeleteByQueryRequest("sub_bank1031");
       SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
       //拼接查询条件
       BoolQueryBuilder queryBuilder =
           filterTime(req.getStartTimeStr(), req.getEndTimeStr());
       queryBuilder.must(QueryBuilders.termQuery(getKeyword(VehicleSearchConstant.VEHICLE_ID), req.getVin()));
       Group eventLevelGroup = new Group("cat2", "CAT2.keyword");
       AggregationBuilder aggregationBuilder = EsAggregationUtil.creatAggregationBuilder(eventLevelGroup);
       sourceBuilder.query(queryBuilder);
       sourceBuilder.aggregation(aggregationBuilder);
       searchRequest.source(sourceBuilder);
        searchRequest.source(sourceBuilder);
       RestHighLevelClient client = null ;
        client.index(addRequest, RequestOptions.DEFAULT);
        client.updateByQuery(updateByQueryRequest, RequestOptions.DEFAULT);
        client.deleteByQuery(deleteByQueryRequest, RequestOptions.DEFAULT);
        response = client.search(searchRequest, RequestOptions.DEFAULT);
               client.delete(deleteByQueryRequest)
    List<? extends MultiBucketsAggregation.Bucket> list = getTermsAggregation(searchResponse, "cat2");
           if (ObjectUtils.isEmpty(list)) {
               return null;
           }
           Map<String, Integer> levelMap = list.stream()
                   .collect(
                           Collectors.groupingBy(
                                   bucket -> {
                                       return bucket.getKeyAsString();
                                   },
                                   Collectors.summingInt(bucket -> Math.toIntExact(bucket.getDocCount()))
                           )
                   );
           return levelMap;
   }
   
   
     public List<? extends MultiBucketsAggregation.Bucket> getTermsAggregation(SearchResponse searchResponse, String key) {
           return Optional.ofNullable(searchResponse)
                   .map(SearchResponse::getAggregations)
                   .map(aggregations -> aggregations.get(key))
                   .map(aggregation -> (MultiBucketsAggregation) aggregation)
                   .map(MultiBucketsAggregation::getBuckets)
                   .orElse(null);
       }
   ```
   
2. 批量插入

   1. ~~~java
       public void insert(List<Map<String, Object>> list,String index) {
              long d1 = System.currentTimeMillis();
              RestHighLevelClient client = null;
              try {
                  client = elasticSearchPool.getClient();
                  // BulkRequest
                  BulkRequest bulkRequest = new BulkRequest();
                  int ss = 0;
                  LocalDate localDate = LocalDate.now();
                  String indexFormat = localDate.format(DateTimeFormatter.BASIC_ISO_DATE);
                  if (list != null && list.size() > 0) {
                      for (int i = 0; i < list.size(); i++) {
                          try {
                              //  bulkRequest.add(indexRequest.source(mmp)); 与单条不同
                              Map<String, Object> mmp = list.get(i);
                              IndexRequest indexRequest = new IndexRequest(index + indexFormat);
                              bulkRequest.add(indexRequest.source(mmp));
      //                        client.bulk(bulkRequest, RequestOptions.DEFAULT);
                              ss++;
                          } catch (NumberFormatException e) {
                              logger.error("NumberFormatException", e);
                          } catch (ElasticsearchGenerationException e) {
                              logger.error("ElasticsearchGenerationException", e);
                          } catch (Exception e) {
                              logger.error("数据处理异常", e);
                          }
                      }
                  }
                  long d2 = System.currentTimeMillis();
                  if (ss > 0) {
                      // RestHighLevelClient 调用 bulkAsync 方法
                      client.bulkAsync(bulkRequest, RequestOptions.DEFAULT, new ActionListener<BulkResponse>() {
                          @Override
                          public void onResponse(BulkResponse bulkResponse) {
                              //成功
                              if (bulkResponse.hasFailures()) {
                                  logger.debug("DB INSERT success!");
                              }
                          }
      
                          @Override
                          public void onFailure(Exception e) {
                              //失败
                              logger.debug("DB INSERT fail:" + e.getMessage());
                          }
                      });
                  }
                  long d3 = System.currentTimeMillis();
      //            System.err.println("DB INSERT status:process data size(" + ss + "),format(" + ((d2 - d1) / 1000.000) + "s),insert(" + ((d3 - d2) / 1000.000) + "s)");
              } catch (Exception e) {
                  logger.error("batch es error ...", e);
              } finally {
                  if (client != null) {
                      elasticSearchPool.returnClient(client);
                  }
              }
          }
      ~~~

   2. 