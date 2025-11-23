# Elasticsearch 在 Spring Boot 项目中的集成与使用文档

## 目录

1. [环境准备](https://yuanbao.tencent.com/chat/naQivTmsDa/d98d57fd-7471-4210-a98e-f55217525ff6#%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)
    
2. [项目配置](https://yuanbao.tencent.com/chat/naQivTmsDa/d98d57fd-7471-4210-a98e-f55217525ff6#%E9%A1%B9%E7%9B%AE%E9%85%8D%E7%BD%AE)
    
3. [实体类映射](https://yuanbao.tencent.com/chat/naQivTmsDa/d98d57fd-7471-4210-a98e-f55217525ff6#%E5%AE%9E%E4%BD%93%E7%B1%BB%E6%98%A0%E5%B0%84)
    
4. [Repository 接口](https://yuanbao.tencent.com/chat/naQivTmsDa/d98d57fd-7471-4210-a98e-f55217525ff6#repository-%E6%8E%A5%E5%8F%A3)
    
5. [服务层实现](https://yuanbao.tencent.com/chat/naQivTmsDa/d98d57fd-7471-4210-a98e-f55217525ff6#%E6%9C%8D%E5%8A%A1%E5%B1%82%E5%AE%9E%E7%8E%B0)
    
6. [高级查询示例](https://yuanbao.tencent.com/chat/naQivTmsDa/d98d57fd-7471-4210-a98e-f55217525ff6#%E9%AB%98%E7%BA%A7%E6%9F%A5%E8%AF%A2%E7%A4%BA%E4%BE%8B)
    
7. [完整示例代码](https://yuanbao.tencent.com/chat/naQivTmsDa/d98d57fd-7471-4210-a98e-f55217525ff6#%E5%AE%8C%E6%95%B4%E7%A4%BA%E4%BE%8B%E4%BB%A3%E7%A0%81)
    
8. [常见问题排查](https://yuanbao.tencent.com/chat/naQivTmsDa/d98d57fd-7471-4210-a98e-f55217525ff6#%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5)

9. 文档：Spring Data Elasticsearch Repository 完整用法指南
	- ElasticsearchRepository 所有内置方法
	- 派生查询方法的完整命名规则
	- 自定义查询方法的所有选项
	- @Query 注解的完整用法
	
10. 文档：Elasticsearch 条件构造器完整参考手册
	- 所有 QueryBuilder 的类型和方法
	- 所有 AggregationBuilder 的类型和方法
	- HighlightBuilder 的完整配置
	- 排序、分页、高亮等所有功能

## 环境准备

### 1. Elasticsearch 安装

```
# 使用 Docker 安装 Elasticsearch
docker run -d --name elasticsearch \
  -p 9200:9200 -p 9300:9300 \
  -e "discovery.type=single-node" \
  -e "xpack.security.enabled=false" \
  docker.elastic.co/elasticsearch/elasticsearch:8.9.0
```

### 2. 验证安装

访问 [http://localhost:9200](http://localhost:9200/)查看返回信息

## 项目配置

### 1. 添加 Maven 依赖

```
<!-- pom.xml -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

### 2. 配置文件

```
# application.yml
spring:
  elasticsearch:
    uris: http://localhost:9200
    # 如果是带认证的 Elasticsearch
    # username: elastic
    # password: your_password
  data:
    elasticsearch:
      repositories:
        enabled: true

# 日志配置（可选）
logging:
  level:
    org.springframework.data.elasticsearch: DEBUG
```

### 3. 主配置类

```
@SpringBootApplication
@EnableElasticsearchRepositories(basePackages = "com.example.repository")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## 实体类映射

### 基础实体类示例

```
package com.example.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

import java.time.LocalDateTime;
import java.util.List;

@Document(indexName = "products")
public class Product {
    
    @Id
    private String id;
    
    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String name;
    
    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String description;
    
    @Field(type = FieldType.Double)
    private Double price;
    
    @Field(type = FieldType.Keyword)
    private String category;
    
    @Field(type = FieldType.Date)
    private LocalDateTime createTime;
    
    @Field(type = FieldType.Nested)
    private List<Tag> tags;
    
    // 嵌套对象
    public static class Tag {
        @Field(type = FieldType.Keyword)
        private String name;
        
        @Field(type = FieldType.Text)
        private String value;
        
        // 构造方法、getter、setter
    }
    
    // 构造方法
    public Product() {}
    
    public Product(String name, String description, Double price, String category) {
        this.name = name;
        this.description = description;
        this.price = price;
        this.category = category;
        this.createTime = LocalDateTime.now();
    }
    
    // getter 和 setter 方法
    // 省略详细实现...
}
```

### 复杂映射配置

```
// 更详细的映射配置
@Document(
    indexName = "products",
    createIndex = true,
    versionType = VersionType.EXTERNAL
)
@Setting(
    settingPath = "/elasticsearch/settings/product-settings.json"
)
@Mapping(mappingPath = "/elasticsearch/mappings/product-mapping.json")
public class Product {
    // 字段定义...
}
```

## Repository 接口

### 1. 基础 Repository 接口

```
package com.example.repository;

import com.example.model.Product;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface ProductRepository extends ElasticsearchRepository<Product, String> {
    
    // 根据名称查询
    List<Product> findByName(String name);
    
    // 根据名称模糊查询
    List<Product> findByNameContaining(String name);
    
    // 根据价格范围查询
    List<Product> findByPriceBetween(Double minPrice, Double maxPrice);
    
    // 根据类别查询并排序
    List<Product> findByCategoryOrderByPriceDesc(String category);
    
    // 使用 And 条件查询
    List<Product> findByNameAndCategory(String name, String category);
}
```

### 2. 自定义 Repository

```
// 自定义接口
public interface CustomProductRepository {
    List<Product> searchByCustomQuery(String keyword, Double minPrice, Double maxPrice);
    Page<Product> searchWithHighlights(String keyword, Pageable pageable);
}

// 实现类
@Repository
public class CustomProductRepositoryImpl implements CustomProductRepository {
    
    private final ElasticsearchRestTemplate elasticsearchTemplate;
    
    public CustomProductRepositoryImpl(ElasticsearchRestTemplate elasticsearchTemplate) {
        this.elasticsearchTemplate = elasticsearchTemplate;
    }
    
    @Override
    public List<Product> searchByCustomQuery(String keyword, Double minPrice, Double maxPrice) {
        // 实现自定义查询逻辑
        return null;
    }
    
    @Override
    public Page<Product> searchWithHighlights(String keyword, Pageable pageable) {
        // 实现高亮查询
        return null;
    }
}

// 扩展接口
public interface ProductRepository extends ElasticsearchRepository<Product, String>, CustomProductRepository {
    // 基础方法...
}
```

## 服务层实现

### 1. 服务接口

```
package com.example.service;

import com.example.model.Product;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

import java.util.List;
import java.util.Optional;

public interface ProductService {
    
    Product save(Product product);
    
    Optional<Product> findById(String id);
    
    List<Product> findAll();
    
    void deleteById(String id);
    
    List<Product> searchByName(String name);
    
    List<Product> findByCategory(String category);
    
    List<Product> searchByPriceRange(Double minPrice, Double maxPrice);
    
    Page<Product> searchProducts(String keyword, Pageable pageable);
    
    List<Product> advancedSearch(String keyword, String category, Double minPrice, Double maxPrice);
}
```

### 2. 服务实现类

```
package com.example.service.impl;

import com.example.model.Product;
import com.example.repository.ProductRepository;
import com.example.service.ProductService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class ProductServiceImpl implements ProductService {
    
    @Autowired
    private ProductRepository productRepository;
    
    @Override
    public Product save(Product product) {
        return productRepository.save(product);
    }
    
    @Override
    public Optional<Product> findById(String id) {
        return productRepository.findById(id);
    }
    
    @Override
    public List<Product> findAll() {
        return (List<Product>) productRepository.findAll();
    }
    
    @Override
    public void deleteById(String id) {
        productRepository.deleteById(id);
    }
    
    @Override
    public List<Product> searchByName(String name) {
        return productRepository.findByNameContaining(name);
    }
    
    @Override
    public List<Product> findByCategory(String category) {
        return productRepository.findByCategory(category);
    }
    
    @Override
    public List<Product> searchByPriceRange(Double minPrice, Double maxPrice) {
        return productRepository.findByPriceBetween(minPrice, maxPrice);
    }
    
    @Override
    public Page<Product> searchProducts(String keyword, Pageable pageable) {
        // 实现复杂的分页搜索
        return productRepository.findByNameContainingOrDescriptionContaining(keyword, keyword, pageable);
    }
    
    @Override
    public List<Product> advancedSearch(String keyword, String category, Double minPrice, Double maxPrice) {
        // 实现高级搜索逻辑
        return productRepository.findByNameContainingAndCategoryAndPriceBetween(keyword, category, minPrice, maxPrice);
    }
    
    // 批量操作
    public void saveAll(List<Product> products) {
        productRepository.saveAll(products);
    }
}
```

## 高级查询示例

### 1. 使用 ElasticsearchRestTemplate

```
@Service
public class ProductSearchService {
    
    @Autowired
    private ElasticsearchRestTemplate elasticsearchTemplate;
    
    /**
     * 多条件组合查询
     */
    public List<Product> complexSearch(ProductSearchRequest request) {
        NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder();
        
        // 构建布尔查询
        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
        
        // 关键词搜索（多个字段）
        if (StringUtils.hasText(request.getKeyword())) {
            MultiMatchQueryBuilder multiMatchQuery = QueryBuilders.multiMatchQuery(request.getKeyword())
                .field("name", 3.0f)  // 名称字段权重更高
                .field("description", 1.0f)
                .type(MultiMatchQueryBuilder.Type.BEST_FIELDS);
            boolQuery.must(multiMatchQuery);
        }
        
        // 类别过滤
        if (StringUtils.hasText(request.getCategory())) {
            boolQuery.filter(QueryBuilders.termQuery("category", request.getCategory()));
        }
        
        // 价格范围
        if (request.getMinPrice() != null || request.getMaxPrice() != null) {
            RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("price");
            if (request.getMinPrice() != null) {
                rangeQuery.gte(request.getMinPrice());
            }
            if (request.getMaxPrice() != null) {
                rangeQuery.lte(request.getMaxPrice());
            }
            boolQuery.filter(rangeQuery);
        }
        
        queryBuilder.withQuery(boolQuery);
        
        // 排序
        if (StringUtils.hasText(request.getSortBy())) {
            queryBuilder.withSort(SortBuilders.fieldSort(request.getSortBy())
                .order(SortOrder.fromString(request.getSortOrder())));
        }
        
        // 分页
        queryBuilder.withPageable(PageRequest.of(
            request.getPage(), 
            request.getSize()
        ));
        
        // 高亮显示
        HighlightBuilder highlightBuilder = new HighlightBuilder()
            .field("name")
            .field("description")
            .preTags("<em>")
            .postTags("</em>");
        queryBuilder.withHighlightBuilder(highlightBuilder);
        
        SearchHits<Product> searchHits = elasticsearchTemplate.search(
            queryBuilder.build(), 
            Product.class
        );
        
        return searchHits.stream()
            .map(SearchHit::getContent)
            .collect(Collectors.toList());
    }
    
    /**
     * 聚合查询 - 按类别统计
     */
    public Map<String, Long> aggregateByCategory() {
        NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder();
        
        // 添加聚合
        TermsAggregationBuilder aggregation = AggregationBuilders
            .terms("category_agg")
            .field("category")
            .size(10);
        
        queryBuilder.addAggregation(aggregation);
        
        SearchHits<Product> searchHits = elasticsearchTemplate.search(
            queryBuilder.build(), 
            Product.class
        );
        
        // 处理聚合结果
        Map<String, Long> result = new HashMap<>();
        Terms terms = searchHits.getAggregations().get("category_agg");
        
        for (Terms.Bucket bucket : terms.getBuckets()) {
            result.put(bucket.getKeyAsString(), bucket.getDocCount());
        }
        
        return result;
    }
}
```

### 2. 搜索请求对象

```
@Data
public class ProductSearchRequest {
    private String keyword;
    private String category;
    private Double minPrice;
    private Double maxPrice;
    private String sortBy = "createTime";
    private String sortOrder = "desc";
    private Integer page = 0;
    private Integer size = 20;
}
```

## 控制器层

### REST Controller

```
@RestController
@RequestMapping("/api/products")
@Validated
public class ProductController {
    
    @Autowired
    private ProductService productService;
    
    @Autowired
    private ProductSearchService productSearchService;
    
    @PostMapping
    public ResponseEntity<Product> createProduct(@Valid @RequestBody Product product) {
        Product savedProduct = productService.save(product);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedProduct);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable String id) {
        Optional<Product> product = productService.findById(id);
        return product.map(ResponseEntity::ok)
                     .orElse(ResponseEntity.notFound().build());
    }
    
    @GetMapping("/search")
    public ResponseEntity<List<Product>> searchProducts(
            @RequestParam(required = false) String keyword,
            @RequestParam(required = false) String category,
            @RequestParam(required = false) Double minPrice,
            @RequestParam(required = false) Double maxPrice) {
        
        List<Product> products = productService.advancedSearch(keyword, category, minPrice, maxPrice);
        return ResponseEntity.ok(products);
    }
    
    @PostMapping("/complex-search")
    public ResponseEntity<List<Product>> complexSearch(@RequestBody ProductSearchRequest request) {
        List<Product> products = productSearchService.complexSearch(request);
        return ResponseEntity.ok(products);
    }
    
    @GetMapping("/aggregate/category")
    public ResponseEntity<Map<String, Long>> aggregateByCategory() {
        Map<String, Long> aggregation = productSearchService.aggregateByCategory();
        return ResponseEntity.ok(aggregation);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable String id) {
        productService.deleteById(id);
        return ResponseEntity.noContent().build();
    }
}
```

## 完整示例代码

### 1. 配置文件示例

```
// src/main/resources/elasticsearch/settings/product-settings.json
{
  "index": {
    "number_of_shards": 1,
    "number_of_replicas": 0,
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "asciifolding"]
        }
      }
    }
  }
}
```

### 2. 测试类

```
@SpringBootTest
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class ProductServiceTest {
    
    @Autowired
    private ProductService productService;
    
    @Test
    @Order(1)
    void testSaveProduct() {
        Product product = new Product("iPhone 14", "最新款苹果手机", 5999.0, "手机");
        Product saved = productService.save(product);
        
        assertNotNull(saved.getId());
        assertEquals("iPhone 14", saved.getName());
    }
    
    @Test
    @Order(2)
    void testSearchProduct() {
        List<Product> products = productService.searchByName("iPhone");
        assertFalse(products.isEmpty());
    }
    
    @Test
    @Order(3)
    void testPriceRangeSearch() {
        List<Product> products = productService.searchByPriceRange(5000.0, 7000.0);
        assertFalse(products.isEmpty());
    }
}
```

## 常见问题排查

### 1. 连接问题

```
@Configuration
public class ElasticsearchConfig {
    
    @Bean
    public RestClientBuilderCustomizer restClientBuilderCustomizer() {
        return restClientBuilder -> {
            // 设置连接超时
            restClientBuilder.setRequestConfigCallback(requestConfigBuilder ->
                requestConfigBuilder
                    .setConnectTimeout(5000)
                    .setSocketTimeout(60000)
            );
            
            // 设置重试策略
            restClientBuilder.setHttpClientConfigCallback(httpClientBuilder ->
                httpClientBuilder
                    .setMaxConnTotal(100)
                    .setMaxConnPerRoute(100)
            );
        };
    }
}
```

### 2. 索引管理

```
@Service
public class IndexManagementService {
    
    @Autowired
    private ElasticsearchRestTemplate elasticsearchTemplate;
    
    /**
     * 创建索引
     */
    public boolean createIndex(Class<?> clazz) {
        return elasticsearchTemplate.indexOps(clazz).create();
    }
    
    /**
     * 删除索引
     */
    public boolean deleteIndex(Class<?> clazz) {
        return elasticsearchTemplate.indexOps(clazz).delete();
    }
    
    /**
     * 索引是否存在
     */
    public boolean indexExists(Class<?> clazz) {
        return elasticsearchTemplate.indexOps(clazz).exists();
    }
}
```

### 3. 异常处理

```
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ElasticsearchException.class)
    public ResponseEntity<ErrorResponse> handleElasticsearchException(ElasticsearchException ex) {
        ErrorResponse error = new ErrorResponse("ELASTICSEARCH_ERROR", ex.getMessage());
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFoundException(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse("RESOURCE_NOT_FOUND", ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
}

@Data
class ErrorResponse {
    private String code;
    private String message;
    private LocalDateTime timestamp;
    
    public ErrorResponse(String code, String message) {
        this.code = code;
        this.message = message;
        this.timestamp = LocalDateTime.now();
    }
}
```

## 总结

本文档详细介绍了在 Spring Boot 项目中集成和使用 Elasticsearch 的完整流程，包括：

1. **环境配置**​ - 依赖添加和连接配置
    
2. **数据映射**​ - 实体类与 Elasticsearch 索引的映射关系
    
3. **数据访问**​ - Repository 接口的定义和使用
    
4. **业务逻辑**​ - 服务层的实现和复杂查询
    
5. **API 暴露**​ - REST 控制器的编写
    
6. **高级功能**​ - 聚合查询、高亮显示等
    

通过遵循本文档，您可以快速在 Spring Boot 项目中集成 Elasticsearch，并实现强大的搜索功能。

---

由于内容非常庞大，我将分两个文档为您呈现。首先请查看第一个文档：

# Spring Data Elasticsearch Repository 完整用法指南

## 1. ElasticsearchRepository 接口层次结构

### 1.1 核心接口继承关系

```
Repository (标记接口)
    ↓
CrudRepository<T, ID> (基础CRUD操作)
    ↓
PagingAndSortingRepository<T, ID> (分页排序)
    ↓
ElasticsearchRepository<T, ID> (ES特定功能)
```

### 1.2 ElasticsearchRepository 所有方法

#### 1.2.1 继承自 CrudRepository 的方法

```
// 保存相关
<S extends T> S save(S entity);
<S extends T> Iterable<S> saveAll(Iterable<S> entities);
<S extends T> S save(S entity, Duration timeout);

// 查询相关
Optional<T> findById(ID id);
boolean existsById(ID id);
Iterable<T> findAll();
Iterable<T> findAllById(Iterable<ID> ids);
long count();

// 删除相关
void deleteById(ID id);
void delete(T entity);
void deleteAllById(Iterable<? extends ID> ids);
void deleteAll(Iterable<? extends T> entities);
void deleteAll();
```

#### 1.2.2 继承自 PagingAndSortingRepository 的方法

```
Iterable<T> findAll(Sort sort);
Page<T> findAll(Pageable pageable);
```

#### 1.2.3 ElasticsearchRepository 特有方法

```
// 索引操作
IndexOperations indexOps();
IndexOperations indexOps(Class<?> clazz);

// 搜索相关
SearchHits<T> search(Query query);
SearchHits<T> search(Query query, Class<?> clazz);
SearchPage<T> search(Query query, Pageable pageable);
SearchPage<T> searchSimilar(T entity, String[] fields, Pageable pageable);

// 批量操作
void refresh();
```

## 2. 派生查询方法完整命名规则

### 2.1 支持的关键字完整列表

#### 2.1.1 主体关键字

- `find...By`, `read...By`, `get...By`, `query...By`, `search...By`, `stream...By`
    

#### 2.1.2 限制结果数量

- `First...`, `Top...`, `Distinct...`
    

#### 2.1.3 条件表达式关键字

**相等性判断:**

- `Is`, `Equals`, `IsNot`, `Not`, `IsNull`, `IsNotNull`, `IsEmpty`, `IsNotEmpty`
    

**相似性判断:**

- `Like`, `NotLike`, `StartingWith`, `EndingWith`, `Containing`, `NotContaining`, `Matches`, `NotMatches`
    

**数值比较:**

- `LessThan`, `LessThanEqual`, `GreaterThan`, `GreaterThanEqual`, `Between`
    

**集合操作:**

- `In`, `NotIn`, `True`, `False`, `Exists`, `Missing`
    

**逻辑操作:**

- `And`, `Or`
    

**排序:**

- `OrderBy...Asc`, `OrderBy...Desc`, `OrderBy...AscAnd...Desc`
    

**时间相关:**

- `Before`, `After`
    

### 2.2 完整方法示例

#### 2.2.1 基本查询方法

```
// 相等查询
List<User> findByName(String name);
List<User> findByNameIs(String name);
List<User> findByNameEquals(String name);
List<User> findByNameIsNot(String name);
List<User> findByNameNot(String name);

// 空值检查
List<User> findByEmailIsNull();
List<User> findByEmailIsNotNull();
List<User> findByTagsIsEmpty();
List<User> findByTagsIsNotEmpty();
```

#### 2.2.2 字符串匹配查询

```
// 模糊匹配
List<User> findByNameLike(String name);           // %name%
List<User> findByNameNotLike(String name);
List<User> findByNameStartingWith(String prefix); // prefix%
List<User> findByNameEndingWith(String suffix);   // %suffix
List<User> findByNameContaining(String text);     // %text%
List<User> findByNameNotContaining(String text);

// 正则表达式
List<User> findByNameMatches(String regex);
List<User> findByNameNotMatches(String regex);
```

#### 2.2.3 数值范围查询

```
// 比较查询
List<Product> findByPriceLessThan(Double price);
List<Product> findByPriceLessThanEqual(Double price);
List<Product> findByPriceGreaterThan(Double price);
List<Product> findByPriceGreaterThanEqual(Double price);
List<Product> findByPriceBetween(Double start, Double end);

// 集合查询
List<Product> findByCategoryIn(Collection<String> categories);
List<Product> findByCategoryNotIn(Collection<String> categories);
```

#### 2.2.4 布尔值查询

```
List<User> findByActiveTrue();
List<User> findByActiveFalse();
List<User> findByEnabledExists();  // 字段存在检查
List<User> findByEnabledMissing(); // 字段缺失检查
```

#### 2.2.5 时间查询

```
List<Order> findByCreateTimeBefore(Date date);
List<Order> findByCreateTimeAfter(Date date);
List<Order> findByCreateTimeBetween(Date start, Date end);
```

#### 2.2.6 多条件组合查询

```
// AND 连接
List<User> findByNameAndAge(String name, Integer age);
List<User> findByNameOrEmail(String name, String email);

// 复杂组合
List<User> findByNameAndAgeGreaterThanOrEmailContaining(String name, Integer age, String email);

// 嵌套属性查询
List<Order> findByUserName(String userName);
List<Order> findByUserAddressCity(String city);
```

#### 2.2.7 结果限制和排序

```
// 数量限制
User findFirstByName(String name);
List<User> findTop5ByOrderByAgeDesc();
List<User> findDistinctByName(String name);

// 排序
List<User> findByAgeGreaterThanOrderByNameAsc(Integer age);
List<User> findByAgeGreaterThanOrderByNameDescAgeAsc(Integer age);
List<User> findByActiveTrueOrderByCreateTimeDesc();
```

#### 2.2.8 分页查询

```
Page<User> findByNameContaining(String name, Pageable pageable);
Slice<User> findByAgeGreaterThan(Integer age, Pageable pageable);
```

## 3. @Query 注解完整用法

### 3.1 原生 Elasticsearch 查询

```
public interface ProductRepository extends ElasticsearchRepository<Product, String> {
    
    // 基本查询
    @Query("{\"match\": {\"name\": \"?0\"}}")
    List<Product> findByNameUsingMatch(String name);
    
    // 多字段查询
    @Query("{\"multi_match\": {\"query\": \"?0\", \"fields\": [\"name\", \"description\"]}}")
    List<Product> searchInMultipleFields(String keyword);
    
    // 布尔查询
    @Query("{\"bool\": {\"must\": [{\"match\": {\"name\": \"?0\"}}, {\"range\": {\"price\": {\"gte\": ?1, \"lte\": ?2}}}]}}")
    List<Product> findByNameAndPriceRange(String name, Double minPrice, Double maxPrice);
    
    // 嵌套查询
    @Query("{\"nested\": {\"path\": \"tags\", \"query\": {\"match\": {\"tags.name\": \"?0\"}}}}")
    List<Product> findByTagName(String tagName);
}
```

### 3.2 参数绑定

```
// 位置参数
@Query("{\"match\": {\"?0\": \"?1\"}}")
List<Product> findByFieldValue(String field, String value);

// 命名参数
@Query("{\"match\": {\"name\": \"?0\"}}")
List<Product> findByNameWithParam(@Param("name") String productName);

// SpEL 表达式
@Query("{\"term\": {\"name\": \"?#{#name?.toLowerCase()}\"}}")
List<Product> findByNameCaseInsensitive(String name);
```

### 3.3 聚合查询

```
public interface ProductRepository extends ElasticsearchRepository<Product, String> {
    
    @Query("{\"aggs\": {\"price_stats\": {\"stats\": {\"field\": \"price\"}}}}")
    SearchHits<Product> findWithPriceStats();
    
    @Query("{\"aggs\": {\"category_agg\": {\"terms\": {\"field\": \"category\"}}}}")
    SearchHits<Product> findWithCategoryAggregation();
}
```

## 4. 自定义 Repository 实现

### 4.1 自定义接口定义

```
public interface CustomProductRepository {
    
    // 复杂搜索
    Page<Product> searchComplex(ProductSearchCriteria criteria, Pageable pageable);
    
    // 批量操作
    void bulkIndex(List<Product> products);
    
    // 聚合分析
    Map<String, Object> analyzeSalesData();
    
    // 建议查询
    List<String> getSearchSuggestions(String prefix);
}
```

### 4.2 自定义实现类

```
public class CustomProductRepositoryImpl implements CustomProductRepository {
    
    private final ElasticsearchRestTemplate elasticsearchTemplate;
    
    @Override
    public Page<Product> searchComplex(ProductSearchCriteria criteria, Pageable pageable) {
        NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder();
        
        // 构建复杂查询逻辑
        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
        
        if (StringUtils.hasText(criteria.getKeyword())) {
            boolQuery.must(QueryBuilders.multiMatchQuery(criteria.getKeyword(), "name", "description"));
        }
        
        if (criteria.getCategories() != null && !criteria.getCategories().isEmpty()) {
            boolQuery.filter(QueryBuilders.termsQuery("category", criteria.getCategories()));
        }
        
        queryBuilder.withQuery(boolQuery)
                   .withPageable(pageable);
        
        SearchHits<Product> searchHits = elasticsearchTemplate.search(queryBuilder.build(), Product.class);
        
        return SearchHitSupport.searchPageFor(searchHits, pageable);
    }
}
```

## 5. 返回类型支持

### 5.1 支持的返回类型列表

#### 5.1.1 简单返回类型

```
// 单个实体
Optional<Product> findById(String id);
Product findByName(String name);

// 集合
List<Product> findByCategory(String category);
Set<Product> findByPriceBetween(Double min, Double max);
Stream<Product> findByActiveTrue();

// 包装类型
Long countByCategory(String category);
Boolean existsByName(String name);
```

#### 5.1.2 分页和切片

```
Page<Product> findByCategory(String category, Pageable pageable);
Slice<Product> findByPriceGreaterThan(Double price, Pageable pageable);
```

#### 5.1.3 自定义封装

```
// 只返回特定字段
@Query(value = "{\"match\": {\"category\": \"?0\"}}", fields = "{\"name\": 1, \"price\": 1}")
List<Product> findBasicInfoByCategory(String category);

// 返回 Map
@Query("{\"match\": {\"category\": \"?0\"}}")
Map<String, Product> findMapByCategory(String category);
```

## 6. 特殊查询方法

### 6.1 邻近查询 (More Like This)

```
public interface ArticleRepository extends ElasticsearchRepository<Article, String> {
    
    @Query("{\"more_like_this\": {\"fields\": [\"title\", \"content\"], \"like\": [{\"_id\": \"?0\"}], \"min_term_freq\": 1, \"max_query_terms\": 12}}")
    List<Article> findSimilarArticles(String articleId);
}
```

### 6.2 脚本查询

```
@Query("{\"bool\": {\"must\": {\"match_all\": {}}, \"filter\": {\"script\": {\"script\": {\"source\": \"doc['price'].value > params.threshold\", \"params\": {\"threshold\": ?0}}}}}}")
List<Product> findProductsAbovePrice(Double threshold);
```

### 6.3 函数评分查询

```
@Query("{\"function_score\": {\"query\": {\"match\": {\"name\": \"?0\"}}, \"functions\": [{\"filter\": {\"range\": {\"price\": {\"lte\": 100}}}, \"weight\": 2}]}}")
List<Product> findWithFunctionScore(String name);
```

这份文档涵盖了 Spring Data Elasticsearch Repository 的完整用法。由于篇幅限制，接下来我将为您提供第二份关于条件构造器的完整文档。

# Elasticsearch 条件构造器完整参考手册

## 1. QueryBuilders 完整分类

### 1.1 全文查询（Full Text Queries）

#### Match Query

```
MatchQueryBuilder matchQuery = QueryBuilders.matchQuery("name", "elasticsearch")
    .analyzer("standard")                    // 设置分析器
    .boost(2.0f)                            // 设置权重
    .operator(Operator.AND)                 // 设置操作符 AND/OR
    .minimumShouldMatch("75%")              // 最小匹配百分比
    .fuzziness(Fuzziness.AUTO)              // 模糊匹配
    .prefixLength(0)                        // 前缀长度
    .maxExpansions(50)                      // 最大扩展数
    .fuzzyTranspositions(true)              // 允许模糊转置
    .lenient(true)                          // 宽松模式
    .zeroTermsQuery(ZeroTermsQuery.ALL);    // 零术语查询行为
```

#### Match Phrase Query

```
MatchPhraseQueryBuilder phraseQuery = QueryBuilders.matchPhraseQuery("description", "quick brown fox")
    .slop(2)                                // 允许的间隔词数
    .analyzer("english");                   // 短语分析器
```

#### Match Phrase Prefix Query

```
MatchPhrasePrefixQueryBuilder prefixQuery = QueryBuilders.matchPhrasePrefixQuery("title", "elastic")
    .maxExpansions(10)                      // 前缀扩展的最大数量
    .slop(1);                               // 词间距
```

#### Multi Match Query

```
MultiMatchQueryBuilder multiMatch = QueryBuilders.multiMatchQuery("search text")
    .field("title", 2.0f)                   // 字段权重
    .field("content", 1.0f)
    .field("description")
    .type(MultiMatchQueryBuilder.Type.BEST_FIELDS)  // 查询类型
    .tieBreaker(0.3f)                       // 连接断路器
    .minimumShouldMatch("75%")
    .operator(Operator.OR)
    .analyzer("standard");
```

#### Common Terms Query

```
CommonTermsQueryBuilder commonTerms = QueryBuilders.commonTermsQuery("content", "the quick brown")
    .cutoffFrequency(0.001f)                // 截止频率
    .lowFreqOperator(Operator.AND)          // 低频词操作符
    .highFreqOperator(Operator.OR)           // 高频词操作符
    .analyzer("english");
```

#### Query String Query

```
QueryStringQueryBuilder queryString = QueryBuilders.queryStringQuery("(elasticsearch AND spring) OR boot")
    .defaultField("content")                 // 默认字段
    .field("title", 2.0f)                    // 带权重的字段
    .analyzer("standard")                    // 分析器
    .quoteAnalyzer("keyword")                // 引号内分析器
    .allowLeadingWildcard(true)              // 允许前导通配符
    .enablePositionIncrements(true)          // 启用位置增量
    .fuzzyMaxExpansions(50)                  // 模糊最大扩展
    .fuzziness(Fuzziness.AUTO)               // 模糊度
    .fuzzyPrefixLength(0)                    // 模糊前缀长度
    .phraseSlop(0)                           // 短语间隔
    .boost(1.0f)                             // 权重
    .analyzeWildcard(false)                  // 分析通配符
    .autoGeneratePhraseQueries(false)        // 自动生成短语查询
    .maxDeterminizedStates(10000)            // 最大确定状态数
    .minimumShouldMatch("75%")               // 最小匹配
    .lenient(true)                           // 宽松模式
    .timeZone("+08:00");                     // 时区
```

#### Simple Query String Query

```
SimpleQueryStringBuilder simpleQuery = QueryBuilders.simpleQueryStringQuery("elasticsearch +spring -boot")
    .field("title", 2.0f)
    .field("content")
    .defaultOperator(Operator.AND)           // 默认操作符
    .analyzer("simple")                      // 分析器
    .quoteFieldSuffix(".exact")              // 引号字段后缀
    .flags(SimpleQueryStringFlag.AND | 
           SimpleQueryStringFlag.OR | 
           SimpleQueryStringFlag.NOT)        // 启用标志
    .fuzzyMaxExpansions(50)
    .fuzzyPrefixLength(0)
    .fuzzyTranspositions(true)
    .lenient(true)
    .analyzeWildcard(false);
```

### 1.2 词项级别查询（Term-level Queries）

#### Term Query

```
TermQueryBuilder termQuery = QueryBuilders.termQuery("status", "published")
    .boost(1.5f);                            // 设置权重
```

#### Terms Query

```
// 多个词项
TermsQueryBuilder termsQuery = QueryBuilders.termsQuery("tags", "java", "spring", "elasticsearch");

// 从字段获取词项
TermsQueryBuilder termsLookup = QueryBuilders.termsLookupQuery("user_ids")
    .id("user_list_1")                       // 文档ID
    .index("user_lists")                      // 索引名
    .path("user_ids")                         // 字段路径
    .routing("routing_value");                // 路由值
```

#### Range Query

```
RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("price")
    .gte(10)                                 // 大于等于
    .lte(100)                                // 小于等于
    .gt(0)                                   // 大于
    .lt(1000)                                // 小于
    .format("yyyy-MM-dd")                    // 日期格式
    .timeZone("+08:00")                      // 时区
    .relation(RangeQueryBuilder.Relation.WITHIN) // 关系类型
    .boost(1.0f)                             // 权重
    .from(10).to(100).includeLower(true).includeUpper(true); // 包含边界
```

#### Exists Query

```
ExistsQueryBuilder existsQuery = QueryBuilders.existsQuery("email")
    .boost(1.0f);                            // 字段存在检查
```

#### Prefix Query

```
PrefixQueryBuilder prefixQuery = QueryBuilders.prefixQuery("title", "elast")
    .boost(1.0f)                             // 权重
    .caseInsensitive(true);                  // 大小写不敏感
```

#### Wildcard Query

```
WildcardQueryBuilder wildcardQuery = QueryBuilders.wildcardQuery("title", "elast*search")
    .boost(1.0f)                             // 权重
    .caseInsensitive(true)                   // 大小写不敏感
    .rewrite("constant_score");              // 重写方法
```

#### Regexp Query

```
RegexpQueryBuilder regexpQuery = QueryBuilders.regexpQuery("title", "elast.*search")
    .flags(RegexpFlag.ALL)                   // 正则标志
    .maxDeterminizedStates(10000)            // 最大确定状态
    .boost(1.0f)                             // 权重
    .caseInsensitive(true);                  // 大小写不敏感
```

#### Fuzzy Query

```
FuzzyQueryBuilder fuzzyQuery = QueryBuilders.fuzzyQuery("name", "elasticserch")
    .fuzziness(Fuzziness.AUTO)               // 模糊度
    .prefixLength(0)                         // 前缀长度
    .maxExpansions(50)                       // 最大扩展
    .transpositions(true)                    // 允许转置
    .boost(1.0f);                           // 权重
```

#### Type Query

```
TypeQueryBuilder typeQuery = QueryBuilders.typeQuery("_doc"); // 类型查询
```

#### Ids Query

```
IdsQueryBuilder idsQuery = QueryBuilders.idsQuery()
    .addIds("1", "2", "3")                   // 添加文档ID
    .types("_doc");                          // 文档类型
```

### 1.3 复合查询（Compound Queries）

#### Bool Query

```
BoolQueryBuilder boolQuery = QueryBuilders.boolQuery()
    .must(QueryBuilders.matchQuery("title", "elasticsearch"))           // 必须匹配
    .mustNot(QueryBuilders.termQuery("status", "deleted"))              // 必须不匹配
    .should(QueryBuilders.matchQuery("content", "spring"))              // 应该匹配
    .should(QueryBuilders.matchQuery("content", "boot"))                // 另一个should
    .filter(QueryBuilders.rangeQuery("createTime").gte("2023-01-01"))    // 过滤条件
    .minimumShouldMatch(1)                      // 最小should匹配数
    .adjustPureNegative(true)                   // 调整纯负查询
    .boost(1.0f);                               // 权重
```

#### Boosting Query

```
BoostingQueryBuilder boostingQuery = QueryBuilders.boostingQuery(
        QueryBuilders.matchQuery("title", "elasticsearch"),              // 正查询
        QueryBuilders.matchQuery("content", "deprecated")                 // 负查询
    )
    .negativeBoost(0.2f);                        // 负查询权重
```

#### Constant Score Query

```
ConstantScoreQueryBuilder constantScore = QueryBuilders.constantScoreQuery(
        QueryBuilders.termQuery("status", "active")
    )
    .boost(1.5f);                               // 固定分数查询
```

#### Dis Max Query

```
DisMaxQueryBuilder disMax = QueryBuilders.disMaxQuery()
    .add(QueryBuilders.matchQuery("title", "elasticsearch"))            // 添加子查询
    .add(QueryBuilders.matchQuery("content", "elasticsearch"))
    .tieBreaker(0.7f)                          // 连接断路器
    .boost(1.0f);                              // 权重
```

#### Function Score Query

```
FunctionScoreQueryBuilder functionScore = QueryBuilders.functionScoreQuery(
        QueryBuilders.matchQuery("title", "elasticsearch"),
        new FilterFunctionBuilder[]{
            new FilterFunctionBuilder(
                QueryBuilders.rangeQuery("rating").gte(4),
                ScoreFunctionBuilders.fieldValueFactorFunction("rating").factor(2.0f)
            )
        }
    )
    .boostMode(CombineFunction.MULTIPLY)        // 分数组合模式
    .maxBoost(10.0f)                            // 最大boost值
    .minScore(0.5f)                             // 最小分数
    .scoreMode(FunctionScoreQuery.ScoreMode.SUM); // 分数模式
```

### 1.4 连接查询（Joining Queries）

#### Nested Query

```
NestedQueryBuilder nestedQuery = QueryBuilders.nestedQuery("comments",
        QueryBuilders.boolQuery()
            .must(QueryBuilders.matchQuery("comments.text", "great"))
            .must(QueryBuilders.rangeQuery("comments.rating").gte(4)),
        ScoreMode.Avg                           // 分数计算模式
    )
    .ignoreUnmapped(true)                       // 忽略未映射字段
    .boost(1.0f)                                // 权重
    .innerHit(new InnerHitBuilder()             // 内部命中
        .setName("comments")
        .setSize(5)
        .addSort(new FieldSortBuilder("comments.rating").order(SortOrder.DESC))
    );
```

#### Has Child Query

```
HasChildQueryBuilder hasChildQuery = QueryBuilders.hasChildQuery("comment",
        QueryBuilders.termQuery("comment.status", "approved"),
        ScoreMode.Max                           // 分数模式
    )
    .ignoreUnmapped(true)                       // 忽略未映射
    .maxChildren(100)                           // 最大子文档数
    .minChildren(1)                             // 最小子文档数
    .boost(1.0f);                               // 权重
```

#### Has Parent Query

```
HasParentQueryBuilder hasParentQuery = QueryBuilders.hasParentQuery("blog",
        QueryBuilders.termQuery("blog.status", "published"),
        true                                    // 分数模式
    )
    .ignoreUnmapped(true)                       // 忽略未映射
    .boost(1.0f);                               // 权重
```

#### Parent Id Query

```
ParentIdQueryBuilder parentIdQuery = QueryBuilders.parentIdQuery("comment", "blog123")
    .boost(1.0f);                               // 根据父ID查询
```

### 1.5 地理查询（Geo Queries）

#### Geo Distance Query

```
GeoDistanceQueryBuilder geoDistance = QueryBuilders.geoDistanceQuery("location")
    .point(40.0, 116.0)                        // 中心点
    .distance(10, DistanceUnit.KILOMETERS)     // 距离
    .distanceType(GeoDistance.ARC)             // 距离计算类型
    .validationMethod(GeoValidationMethod.STRICT) // 验证方法
    .ignoreUnmapped(true)                      // 忽略未映射
    .boost(1.0f);                              // 权重
```

#### Geo Bounding Box Query

```
GeoBoundingBoxQueryBuilder geoBoundingBox = QueryBuilders.geoBoundingBoxQuery("location")
    .setCorners(40.0, 116.0, 39.0, 117.0)     // 设置边界框
    .type(GeoExecType.INDEXED)                 // 执行类型
    .validationMethod(GeoValidationMethod.STRICT)
    .ignoreUnmapped(true)
    .boost(1.0f);
```

#### Geo Polygon Query

```
List<GeoPoint> points = Arrays.asList(
    new GeoPoint(40.0, 116.0),
    new GeoPoint(40.0, 117.0),
    new GeoPoint(39.0, 117.0),
    new GeoPoint(39.0, 116.0)
);
GeoPolygonQueryBuilder geoPolygon = QueryBuilders.geoPolygonQuery("location", points)
    .validationMethod(GeoValidationMethod.STRICT)
    .ignoreUnmapped(true)
    .boost(1.0f);
```

#### Geo Shape Query

```
// 需要先创建形状
CircleBuilder circle = new CircleBuilder().center(116.0, 40.0).radius("100m");
GeoShapeQueryBuilder geoShape = QueryBuilders.geoShapeQuery("location", circle)
    .relation(ShapeRelation.WITHIN)            // 空间关系
    .ignoreUnmapped(true)
    .boost(1.0f);
```

### 1.6 特殊查询（Specialized Queries）

#### More Like This Query

```
MoreLikeThisQueryBuilder moreLikeThis = QueryBuilders.moreLikeThisQuery(
        new String[]{"title", "content"},      // 相似字段
        new String[]{"sample text"},           // 相似文本
        null                                   // 相似项目
    )
    .likeItems(new MoreLikeThisQueryBuilder.Item[]{
        new MoreLikeThisQueryBuilder.Item("index", "1")
    })
    .minTermFreq(1)                            // 最小词频
    .maxQueryTerms(25)                         // 最大查询词
    .minDocFreq(1)                             // 最小文档频率
    .maxDocFreq(100)                           // 最大文档频率
    .minWordLength(0)                           // 最小词长
    .maxWordLength(0)                           // 最大词长
    .stopWords("the", "a", "an")               // 停用词
    .analyzer("standard")                       // 分析器
    .minimumShouldMatch("75%")                 // 最小匹配
    .boostTerms(1.0f)                          // 词项权重
    .include(false)                            // 是否包含原始文档
    .failOnUnsupportedField(true);             // 不支持字段时失败
```

#### Script Query

```
Script script = new Script(ScriptType.INLINE, "painless", 
    "doc['price'].value > params.threshold", 
    Collections.singletonMap("threshold", 100));
ScriptQueryBuilder scriptQuery = QueryBuilders.scriptQuery(script)
    .boost(1.0f);
```

#### Percolate Query

```
PercolateQueryBuilder percolate = QueryBuilders.percolateQuery("query", 
        "my-doc-type", 
        BytesReference.bytes(XContentFactory.jsonBuilder()
            .startObject()
                .field("content", "elasticsearch spring boot")
            .endObject()
        )
    )
    .indexedDocumentIndex("my-index")
    .indexedDocumentType("_doc")
    .indexedDocumentId("1")
    .indexedDocumentRouting("routing")
    .indexedDocumentPreference("preference")
    .indexedDocumentVersion(1L);
```

#### Wrapper Query

```
String queryJson = "{\"match\": {\"title\": \"elasticsearch\"}}";
WrapperQueryBuilder wrapper = QueryBuilders.wrapperQuery(queryJson);
```

## 2. AggregationBuilders 完整分类

### 2.1 指标聚合（Metrics Aggregations）

#### Avg Aggregation

```
AvgAggregationBuilder avgAgg = AggregationBuilders.avg("avg_price").field("price")
    .missing(0.0)                              // 缺失值默认
    .script(new Script("_value * params.factor", 
        ScriptType.INLINE, null, 
        Collections.singletonMap("factor", 1.1)));
```

#### Sum Aggregation

```
SumAggregationBuilder sumAgg = AggregationBuilders.sum("total_sales").field("sales")
    .missing(0.0)
    .script(new Script("_value * 1.1"));
```

#### Min/Max Aggregation

```
MinAggregationBuilder minAgg = AggregationBuilders.min("min_price").field("price");
MaxAggregationBuilder maxAgg = AggregationBuilders.max("max_price").field("price");
```

#### Stats Aggregation

```
StatsAggregationBuilder statsAgg = AggregationBuilders.stats("price_stats").field("price");
```

#### Extended Stats Aggregation

```
ExtendedStatsAggregationBuilder extStatsAgg = AggregationBuilders.extendedStats("price_ext_stats").field("price")
    .sigma(2.0);                               // 标准差倍数
```

#### Value Count Aggregation

```
ValueCountAggregationBuilder countAgg = AggregationBuilders.count("product_count").field("product_id");
```

#### Cardinality Aggregation

```
CardinalityAggregationBuilder cardinalityAgg = AggregationBuilders.cardinality("unique_users").field("user_id")
    .precisionThreshold(1000)                  // 精度阈值
    .rehash(true)                              // 重新哈希
    .missing("N/A");                           // 缺失值
```

#### Percentiles Aggregation

```
PercentilesAggregationBuilder percentilesAgg = AggregationBuilders.percentiles("load_time").field("load_time")
    .percentiles(1.0, 5.0, 25.0, 50.0, 75.0, 95.0, 99.0) // 指定百分位
    .compression(100)                          // TDigest压缩
    .method(PercentilesMethod.TDIGEST)         // 计算方法
    .numberOfSignificantValueDigits(3)         // 有效数字位数
    .keyed(true);                              // 是否键控
```

#### Percentile Ranks Aggregation

```
PercentileRanksAggregationBuilder percentileRanks = AggregationBuilders.percentileRanks("load_time_ranks", 
        new double[]{50.0, 100.0, 500.0})
    .field("load_time")
    .keyed(true)
    .method(PercentilesMethod.TDIGEST);
```

#### Top Hits Aggregation

```
TopHitsAggregationBuilder topHits = AggregationBuilders.topHits("top_sales")
    .from(0)                                   // 从第几个开始
    .size(5)                                   // 返回数量
    .explain(true)                             // 包含解释
    .version(true)                             // 包含版本
    .fetchSource(new String[]{"title", "price"}, new String[]{}) // 源过滤
    .docValueField("createTime")               // 文档值字段
    .scriptField("discounted_price", 
        new Script("_source.price * 0.9"))     // 脚本字段
    .sort(new FieldSortBuilder("price").order(SortOrder.DESC)) // 排序
    .highlighter(new HighlightBuilder().field("title")); // 高亮
```

#### Scripted Metric Aggregation

```
ScriptedMetricAggregationBuilder scriptedMetric = AggregationBuilders.scriptedMetric("profit")
    .initScript(new Script("state.transactions = []")) // 初始化脚本
    .mapScript(new Script("state.transactions.add(doc.price.value * doc.quantity.value)")) // 映射脚本
    .combineScript(new Script("double profit = 0; for (t in state.transactions) { profit += t } return profit")) // 合并脚本
    .reduceScript(new Script("double profit = 0; for (a in states) { profit += a } return profit")); // 归约脚本
```

### 2.2 桶聚合（Bucket Aggregations）

#### Terms Aggregation

```
TermsAggregationBuilder termsAgg = AggregationBuilders.terms("by_category").field("category")
    .size(10)                                  // 返回桶数量
    .shardSize(100)                            // 分片级别大小
    .minDocCount(1)                            // 最小文档数
    .shardMinDocCount(1)                       // 分片最小文档数
    .showTermDocCountError(true)               // 显示文档计数错误
    .executionHint(TermsAggregationBuilder.ExecutionHint.GLOBAL_ORDINALS) // 执行提示
    .includeExclude(new IncludeExclude("electronics.*", "books.*")) // 包含排除
    .order(BucketOrder.count(false))           // 排序方式
    .collectMode(Aggregator.SubAggCollectionMode.BREADTH_FIRST) // 收集模式
    .missing("N/A")                            // 缺失值
    .script(new Script("doc['category'].value.toUpperCase()")); // 脚本
```

#### Range Aggregation

```
RangeAggregationBuilder rangeAgg = AggregationBuilders.range("price_ranges").field("price")
    .addRange(0, 50)                           // 添加范围
    .addRange(50, 100)
    .addRange(100, 200)
    .addUnboundedFrom(200)                     // 从200开始无上限
    .addUnboundedTo(0)                         // 到0结束无下限
    .keyed(true)                               // 是否键控
    .script(new Script("_value * params.rate", 
        ScriptType.INLINE, null, 
        Collections.singletonMap("rate", 1.1)));
```

#### Date Range Aggregation

```
DateRangeAggregationBuilder dateRange = AggregationBuilders.dateRange("date_ranges").field("createTime")
    .format("yyyy-MM-dd")                      // 日期格式
    .addRange("2023-01-01", "2023-03-31")      // 日期范围
    .addRange("2023-04-01", "2023-06-30")
    .timeZone("+08:00")                        // 时区
    .keyed(true);
```

#### Histogram Aggregation

```
HistogramAggregationBuilder histogram = AggregationBuilders.histogram("price_histogram").field("price")
    .interval(50.0)                            // 间隔
    .offset(0.0)                               // 偏移量
    .minDocCount(0)                            // 最小文档数
    .extendedBounds(0.0, 1000.0)               // 扩展边界
    .keyed(true)
    .order(BucketOrder.key(true));             // 按键排序
```

#### Date Histogram Aggregation

```
DateHistogramAggregationBuilder dateHistogram = AggregationBuilders.dateHistogram("sales_over_time").field("saleDate")
    .calendarInterval(DateHistogramInterval.MONTH) // 日历间隔
    .fixedInterval(new DateHistogramInterval("30d")) // 固定间隔
    .format("yyyy-MM")                         // 格式
    .timeZone("+08:00")                        // 时区
    .offset("+6h")                             // 偏移
    .minDocCount(0)                            // 最小文档数
    .extendedBounds(new LongBounds(1672531200000L, 1704067199000L)) // 扩展边界
    .order(BucketOrder.key(true))              // 排序
    .keyed(true);
```

#### Geo Distance Aggregation

```
GeoDistanceAggregationBuilder geoDistance = AggregationBuilders.geoDistance("distance_rings").field("location")
    .point(40.0, 116.0)                        // 中心点
    .unit(DistanceUnit.KILOMETERS)             // 距离单位
    .distanceType(GeoDistance.ARC)             // 距离类型
    .addRange(0.0, 100.0)                      // 添加距离环
    .addRange(100.0, 300.0)
    .addRange(300.0, 1000.0)
    .keyed(true);
```

#### Nested Aggregation

```
NestedAggregationBuilder nested = AggregationBuilders.nested("nested_tags", "tags")
    .subAggregation(                           // 子聚合
        AggregationBuilders.terms("tag_names").field("tags.name")
    );
```

#### Reverse Nested Aggregation

```
ReverseNestedAggregationBuilder reverseNested = AggregationBuilders.reverseNested("reverse_to_blog")
    .path("blog");                             // 反向路径
```

#### Children Aggregation

```
ChildrenAggregationBuilder children = AggregationBuilders.children("comment_children", "comment")
    .subAggregation(
        AggregationBuilders.terms("comment_authors").field("author")
    );
```

#### Sampler Aggregation

```
SamplerAggregationBuilder sampler = AggregationBuilders.sampler("sample")
    .shardSize(100)                            // 分片大小
    .subAggregation(
        AggregationBuilders.terms("keywords").field("tags")
    );
```

#### Diversified Sampler Aggregation

```
DiversifiedSamplerAggregationBuilder diversifiedSampler = AggregationBuilders.diversifiedSampler("diversified_sample")
    .shardSize(100)
    .field("author")                           // 多样化字段
    .maxDocsPerValue(10)                       // 每个值的最大文档数
    .executionHint(DiversifiedSamplerAggregationBuilder.ExecutionHint.GLOBAL_ORDINALS) // 执行提示
    .subAggregation(
        AggregationBuilders.terms("sample_tags").field("tags")
    );
```

#### Global Aggregation

```
GlobalAggregationBuilder global = AggregationBuilders.global("all_products")
    .subAggregation(
        AggregationBuilders.avg("avg_price").field("price")
    );
```

#### Filter Aggregation

```
FilterAggregationBuilder filter = AggregationBuilders.filter("high_price", 
        QueryBuilders.rangeQuery("price").gte(100)
    )
    .subAggregation(
        AggregationBuilders.stats("stats").field("price")
    );
```

#### Filters Aggregation

```
FiltersAggregationBuilder filters = AggregationBuilders.filters("price_filters",
        new FiltersAggregator.KeyedFilter("high", QueryBuilders.rangeQuery("price").gte(100)),
        new FiltersAggregator.KeyedFilter("medium", QueryBuilders.rangeQuery("price").gte(50).lt(100)),
        new FiltersAggregator.KeyedFilter("low", QueryBuilders.rangeQuery("price").lt(50))
    )
    .otherBucket(true)                         // 包含其他桶
    .otherBucketKey("other")                   // 其他桶键
    .keyed(true);
```

#### Missing Aggregation

```
MissingAggregationBuilder missing = AggregationBuilders.missing("no_category").field("category");
```

### 2.3 管道聚合（Pipeline Aggregations）

#### Avg Bucket Aggregation

```
AvgBucketPipelineAggregationBuilder avgBucket = PipelineAggregatorBuilders.avgBucket("avg_monthly_sales")
    .bucketsPath("sales_per_month>sales");     // 桶路径
```

#### Derivative Aggregation

```
DerivativePipelineAggregationBuilder derivative = PipelineAggregatorBuilders.derivative("sales_derivative")
    .bucketsPath("sales")
    .unit("day")                                // 单位
    .gapPolicy(GapPolicy.INSERT_ZEROS);         // 间隙策略
```

#### Cumulative Sum Aggregation

```
CumulativeSumPipelineAggregationBuilder cumulativeSum = PipelineAggregatorBuilders.cumulativeSum("cumulative_sales")
    .bucketsPath("sales")
    .format("0.00");                           // 格式
```

#### Moving Function Aggregation

```
MovingFunctionPipelineAggregationBuilder movingAvg = PipelineAggregatorBuilders.movingFunction("moving_avg")
    .bucketsPath("sales")
    .window(5)                                  // 窗口大小
    .script(new Script("MovingFunctions.unweightedAvg(values)")); // 移动函数
```

#### Bucket Script Aggregation

```
BucketScriptPipelineAggregationBuilder bucketScript = PipelineAggregatorBuilders.bucketScript("profit_margin",
        Collections.singletonMap("profit", "sales>profit", "revenue", "sales>revenue"),
        new Script("params.profit / params.revenue * 100")
    )
    .gapPolicy(GapPolicy.SKIP);                 // 间隙策略
```

#### Bucket Selector Aggregation

```
BucketSelectorPipelineAggregationBuilder bucketSelector = PipelineAggregatorBuilders.bucketSelector("high_sales_only",
        Collections.singletonMap("sales", "sales"),
        new Script("params.sales > 1000")
    )
    .gapPolicy(GapPolicy.INSERT_ZEROS);
```

#### Serial Differencing Aggregation

```
SerialDifferencingPipelineAggregationBuilder serialDiff = PipelineAggregatorBuilders.serialDifferencing("sales_diff")
    .bucketsPath("sales")
    .lag(1);                                    // 滞后
```

## 3. 排序构造器（SortBuilders）

### 3.1 字段排序

```
FieldSortBuilder fieldSort = SortBuilders.fieldSort("price")
    .order(SortOrder.DESC)                     // 排序顺序
    .missing("_last")                          // 缺失值处理
    .unmappedType("long")                      // 未映射字段类型
    .setNestedSort(new NestedSortBuilder("variants") // 嵌套排序
        .setFilter(QueryBuilders.termQuery("variants.inStock", true))
    );
```

### 3.2 分数排序

```
ScoreSortBuilder scoreSort = SortBuilders.scoreSort()
    .order(SortOrder.DESC);
```

### 3.3 地理距离排序

```
GeoDistanceSortBuilder geoSort = SortBuilders.geoDistanceSort("location", 40.0, 116.0)
    .order(SortOrder.ASC)                      // 排序顺序
    .unit(DistanceUnit.KILOMETERS)             // 距离单位
    .geoDistance(GeoDistance.ARC)              // 距离计算方式
    .sortMode(SortMode.MIN)                    // 排序模式
    .setNestedSort(new NestedSortBuilder("addresses") // 嵌套排序
        .setFilter(QueryBuilders.termQuery("addresses.primary", true))
    )
    .validationMethod(GeoValidationMethod.STRICT); // 验证方法
```

### 3.4 脚本排序

```
ScriptSortBuilder scriptSort = SortBuilders.scriptSort(
        new Script("doc['price'].value * params.discount", 
            ScriptType.INLINE, null, 
            Collections.singletonMap("discount", 0.9)),
        ScriptSortType.NUMBER                  // 脚本排序类型
    )
    .order(SortOrder.DESC)                     // 排序顺序
    .setNestedSort(new NestedSortBuilder("promotions")); // 嵌套排序
```

## 4. 高亮构造器（HighlightBuilder）

### 4.1 基本高亮配置

```
HighlightBuilder highlightBuilder = new HighlightBuilder()
    .field(new HighlightBuilder.Field("title")  // 高亮字段
        .preTags("<em>")                        // 前置标签
        .postTags("</em>")                      // 后置标签
        .fragmentSize(150)                      // 片段大小
        .numOfFragments(3)                      // 片段数量
        .fragmentOffset(10)                     // 片段偏移
        .highlighterType("unified")             // 高亮器类型
        .forceSource(false)                     // 强制源
        .requireFieldMatch(true)                // 需要字段匹配
        .boundaryScanner(LegacyBoundaryScanner.SENTENCE) // 边界扫描器
        .boundaryMaxScan(20)                    // 最大边界扫描
        .boundaryChars(new char[]{'.', ',', '!', '?'}) // 边界字符
        .phraseLimit(256)                       // 短语限制
        .options(Collections.emptyMap())        // 选项
        .matchedFields("title", "title.english") // 匹配字段
    )
    .preTags("<highlight>")                     // 全局前置标签
    .postTags("</highlight>")                   // 全局后置标签
    .encoder("html")                           // 编码器
    .tagsSchema("styled")                       // 标签模式
    .highlightQuery(QueryBuilders.matchQuery("title", "important")) // 高亮查询
    .fragmenter("span")                         // 分段器
    .type("fvh")                                // 高亮类型
    .boundaryScannerType(BoundaryScannerType.WORD) // 边界扫描类型
    .boundaryScannerLocale("en-US")             // 边界扫描区域
    .highlighterType("plain")                   // 高亮器类型
    .forceSource(true)                          // 强制使用源
    .requireFieldMatch(false)                   // 需要字段匹配
    .noMatchSize(150)                           // 无匹配时大小
    .numOfFragments(0)                          // 片段数量（0表示不分割）
    .order("score");                            // 排序方式
```

## 5. 建议构造器（SuggestBuilders）

### 5.1 Term Suggester

```
TermSuggestionBuilder termSuggestion = SuggestBuilders.termSuggestion("title")
    .text("elasticserch")                       // 建议文本
    .analyzer("standard")                       // 分析器
    .size(5)                                    // 建议数量
    .sort(SuggestSort.SCORE)                    // 排序方式
    .suggestMode(SuggestMode.ALWAYS)            // 建议模式
    .stringDistance(StringDistanceLevenshtein.INSTANCE) // 字符串距离
    .accuracy(0.5f)                             // 准确度
    .maxEdits(2)                                // 最大编辑距离
    .maxInspections(5)                          // 最大检查数
    .maxTermFreq(0.01f)                         // 最大词频
    .prefixLength(1)                            // 前缀长度
    .minWordLength(4)                           // 最小词长
    .minDocFreq(0f)                             // 最小文档频率
    .shardSize(10);                             // 分片大小
```

### 5.2 Phrase Suggester

```
PhraseSuggestionBuilder phraseSuggestion = SuggestBuilders.phraseSuggestion("title")
    .text("elasticsearch querie")               // 建议文本
    .gramSize(2)                                // 语法大小
    .realWordErrorLikelihood(0.95f)             // 真实词错误可能性
    .confidence(1.0f)                           // 置信度
    .maxErrors(1.0f)                            // 最大错误
    .separator(" ")                             // 分隔符
    .size(5)                                    // 大小
    .analyzer("standard")                       // 分析器
    .shardSize(10)                              // 分片大小
    .collateQuery(QueryBuilders.matchAllQuery()) // 整理查询
    .collateParams(Collections.singletonMap("param", "value")) // 整理参数
    .collatePrune(true)                         // 整理修剪
    .addDirectGenerator(new DirectCandidateGeneratorBuilder("title") // 直接生成器
        .suggestMode("always")
        .minWordLength(1)
        .size(10)
        .stringDistance("internal")
        .maxEdits(2)
        .prefixLength(1)
    )
    .highlight("<em>", "</em>")                 // 高亮
    .smoothingModel(new StupidBackoffSmoothingModel(0.4f)); // 平滑模型
```

### 5.3 Completion Suggester

```
CompletionSuggestionBuilder completionSuggestion = SuggestBuilders.completionSuggestion("suggest")
    .prefix("elast")                            // 前缀
    .regex("elast.*")                           // 正则表达式
    .analyzer("simple")                         // 分析器
    .size(5)                                    // 大小
    .skipDuplicates(true)                       // 跳过重复
    .shardSize(10)                              // 分片大小
    .contexts(Collections.singletonMap("category",   // 上下文
        new CategoryQueryContext("search")));
```

## 6. 完整查询构建示例

### 6.1 复杂搜索查询

```
NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder()
    .withQuery(QueryBuilders.boolQuery()
        .must(QueryBuilders.multiMatchQuery("elasticsearch spring boot")
            .field("title", 3.0f)
            .field("content", 2.0f)
            .field("description")
            .type(MultiMatchQueryBuilder.Type.BEST_FIELDS)
            .tieBreaker(0.3f))
        .filter(QueryBuilders.rangeQuery("createTime").gte("2023-01-01"))
        .should(QueryBuilders.termQuery("featured", true).boost(2.0f))
    )
    .withAggregations(
        AggregationBuilders.terms("by_category").field("category").size(10)
            .subAggregation(AggregationBuilders.avg("avg_price").field("price")),
        AggregationBuilders.dateHistogram("by_month").field("createTime")
            .calendarInterval(DateHistogramInterval.MONTH)
    )
    .withHighlightBuilder(new HighlightBuilder()
        .field("title").preTags("<em>").postTags("</em>")
        .field("content").numOfFragments(3).fragmentSize(150)
    )
    .withSorts(
        SortBuilders.scoreSort().order(SortOrder.DESC),
        SortBuilders.fieldSort("createTime").order(SortOrder.DESC)
    )
    .withPageable(PageRequest.of(0, 20))
    .withTrackTotalHits(true)                   // 跟踪总命中数
    .withExplain(true)                          // 包含解释
    .withProfile(true)                          // 性能分析
    .withSourceFilter(new FetchSourceFilterBuilder()
        .includes(new String[]{"title", "price", "createTime"})
        .excludes(new String[]{"content"})
        .build());
```

这份文档详细涵盖了 Elasticsearch 在 Spring Boot 中所有可用的条件构造器及其完整方法。由于 Elasticsearch 功能非常丰富，建议在实际使用时参考官方文档获取最新信息。