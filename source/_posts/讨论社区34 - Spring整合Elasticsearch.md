---
title: 讨论社区34  -  Spring整合Elasticsearch
date: 2019-08-08 21:32:50
tags: [Elasticsearch,Spring]
category: [讨论社区项目]
---

- 引入依赖

  - spring-boot-starter-data-elasticsearch

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
  </dependency>
  ```

- 配置Elasticsearch

  ```xml
  # ElasticSearch
  spring.data.elasticsearch.cluster-name=talking
  # 9200是http访问的端口，9300是tcp访问的端口
  spring.data.elasticsearch.cluster-nodes=127.0.0.1:9300
  ```

- cluster-name、cluster-nodes（集群的名字，节点）

- Redis和Es底层都用到了Netty,有启动冲突。解决：在TalkingApplication类加入初始化方法进行配置。

```java
@PostConstruct
public void init() {
    // 解决netty启动冲突问题
    // see Netty4Utils.setAvailableProcessors()
    System.setProperty("es.set.netty.runtime.available.processors", "false");
}
```



- Spring Data Elasticsearch(调用API)
  - `ElasticsearchTemplate`（集成了Es的CRUD方法）
  - `ElasticsearchRepository`（接口，底层为ElasticsearchTemplate，用起来更方便）

```java
// 使用 ElasticsearchRepository
@Test
public void testSearchByRepository(){
    SearchQuery searchQuery = new NativeSearchQueryBuilder()
    	.withQuery(QueryBuilders.multiMatchQuery("test","title","content"))
    	.withSort(SortBuilders.fieldSort("type").order(SortOrder.DESC))
    	.withSort(SortBuilders.fieldSort("score").order(SortOrder.DESC))
    	.withSort(SortBuilders.fieldSort("createTime").order(SortOrder.DESC))
    	.withPageable(PageRequest.of(0,10))
    	.withHighlightFields(
    		new HighlightBuilder.Field("title").preTags("<em>").postTags("</em>"),
    		new HighlightBuilder.Field("content").preTags("<em>").postTags("</em>")
    	).build();


    // elasticTemplate.queryForPage(searchQuery, class, SearchResultMapper)
    // 底层获取得到了高亮显示的值, 但是没有返回.
    Page<DiscussPost> page = discussPostRepository.search(searchQuery);

    System.out.println(page.getTotalElements());
    System.out.println(page.getTotalPages());
    System.out.println(page.getNumber());
    System.out.println(page.getSize());
    
    for (DiscussPost post : page) {
    	System.out.println(post);
    }
}
```

```java
@Test
public void testSearchByTemplate() {
	SearchQuery searchQuery = new NativeSearchQueryBuilder()
    	.withQuery(QueryBuilders.multiMatchQuery("互联网寒冬", "title", "content"))
        .withSort(SortBuilders.fieldSort("type").order(SortOrder.DESC))
         .withSort(SortBuilders.fieldSort("score").order(SortOrder.DESC))
         .withSort(SortBuilders.fieldSort("createTime").order(SortOrder.DESC))
         .withPageable(PageRequest.of(0, 10))
         .withHighlightFields(
                 new HighlightBuilder.Field("title").preTags("<em>").postTags("</em>"),
                 new HighlightBuilder.Field("content").preTags("<em>").postTags("</em>")
          ).build();

        Page<DiscussPost> page = elasticTemplate.queryForPage(searchQuery, DiscussPost.class, new SearchResultMapper() {
        @Override
        public <T> AggregatedPage<T> mapResults(SearchResponse response, Class<T> aClass, Pageable pageable) {
            SearchHits hits = response.getHits();
            if (hits.getTotalHits() <= 0) {
                    return null;
                }
		   //封装数据
            List<DiscussPost> list = new ArrayList<>();
            for (SearchHit hit : hits) {
            DiscussPost post = new DiscussPost();

            String id = hit.getSourceAsMap().get("id").toString();
            post.setId(Integer.valueOf(id));

            String userId = hit.getSourceAsMap().get("userId").toString();
            post.setUserId(Integer.valueOf(userId));

            String title = hit.getSourceAsMap().get("title").toString();
            post.setTitle(title);

            String content = hit.getSourceAsMap().get("content").toString();
            post.setContent(content);

            String status = hit.getSourceAsMap().get("status").toString();
            post.setStatus(Integer.valueOf(status));

            String createTime = hit.getSourceAsMap().get("createTime").toString();
            post.setCreateTime(new Date(Long.valueOf(createTime)));

            String commentCount = hit.getSourceAsMap().get("commentCount").toString();
            post.setCommentCount(Integer.valueOf(commentCount));

            // 处理高亮显示的结果
            HighlightField titleField = hit.getHighlightFields().get("title");
            if (titleField != null) {
            	post.setTitle(titleField.getFragments()[0].toString());
            }

            HighlightField contentField = hit.getHighlightFields().get("content");
            if (contentField != null) {
            	post.setContent(contentField.getFragments()[0].toString());
            }

            list.add(post);
         }

            return new AggregatedPageImpl(list, pageable,
                        hits.getTotalHits(), response.getAggregations(), response.getScrollId(), hits.getMaxScore());
            }

            @Override
            public <T> T mapSearchHit(SearchHit searchHit, Class<T> aClass) {
            	return null;
            	}
            });

            System.out.println(page.getTotalElements());
            System.out.println(page.getTotalPages());
            System.out.println(page.getNumber());
            System.out.println(page.getSize());
            for (DiscussPost post : page) {
                System.out.println(post);
            }
    }
```

匹配的结果是与高亮匹配相关的前后文，不会只高亮匹配显示所有text文本。