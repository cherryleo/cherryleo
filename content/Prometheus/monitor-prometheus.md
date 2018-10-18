# Prometheus



## 数据模型

### 时间序列

**时间序列**（time series）是一组按照时间发生先后顺序进行排列的数据点序列。通常一组时间序列的时间间隔为一恒定值（如1秒，5分钟，12小时，7天，1年），因此时间序列可以作为离散时间数据进行分析处理。



**时间序列特征：**

- **非平稳性**（nonstationarity，也译作**不平稳性**，**非稳定性**）：即时间序列的方差无法呈现出一个长期趋势并最终趋于一个常数或是一个线性函数
- 波动幅度**随时间变化**（Time－varying Volatility）：即一个时间序列变量的方差随时间的变化而变化



### 指标名称

```
<metric name>{<label name>=<label value>, ...}
api_http_requests_total{method="POST", handler="/messages"}
```



### 指标类型

### jobs和instances



