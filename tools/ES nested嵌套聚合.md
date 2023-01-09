

# ES nested嵌套聚合

```
// 优化1.select * from goods_index where ancestryCategoryId=2 and  hotelPrices.sellPrice//>=200 and hotelPrices.sellPrice<=1000 and hotelPrices.stockQuantity>0 //这块不对，因为聚合计算的不是 过滤之后的嵌套文档，即不是inner_hits,不合适
GET goods_test2/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "ancestryCategoryId": {
              "value": "2"
            }
          }
        },
        {
          "nested": {
            "path": "hotelPrices",
            "ignore_unmapped": true,
            "score_mode": "none",
            "boost": 1,
            "inner_hits": {
              "ignore_unmapped": true,
              "from": 0,
              "size": 3,
              "version": false,
              "seq_no_primary_term": false,
              "explain": false,
              "track_scores": false
            },
            "query": {
              "bool": {
                "must": [
                  {
                    "range": {
                      "hotelPrices.sellPrice": {
                        "gte": 200,
                        "lte": 1000
                      }
                    }
                  },
                  {
                    "range": {
                      "hotelPrices.stockQuantity": {
                        "gt":0
                      }
                    }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  },
  "aggregations": {
    "salesNested": {
      "nested": {
        "path": "hotelPrices"
      },
      "aggregations": {  
        "group_by": {
          "terms": {
            "field": "hotelPrices.goodsId",
            "size": 10,
            "order": {
              "_key": "asc"
            }
          }
        }
      }
    }
  }
}
```





```
// 优化2.select * from goods_index where ancestryCategoryId=2 and  hotelPrices.sellPrice//>=200 and hotelPrices.sellPrice<=1000 and hotelPrices.stockQuantity>0 group by goodId having(count>天数) 
//这块不对，因为聚合计算的不是 过滤之后的嵌套文档，即不是inner_hits,所以在聚合时也对nested里面的内容做了层层的过滤，但是聚合过滤后每个商品符合条件的天数还需要再过滤，目前没做having(count>天数)  
GET goods_test2/_search
{
  "from": 0,
  "size": 1,
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "ancestryCategoryId": {
              "value": "2"
            }
          }
        },
        {
          "nested": {
            "path": "hotelPrices",
            "ignore_unmapped": true,
            "score_mode": "none",
            "boost": 1,
            "inner_hits": {
              "ignore_unmapped": true,
              "from": 0,
              "size": 90,
              "version": false,
              "seq_no_primary_term": false,
              "explain": false,
              "track_scores": false
            },
            "query": {
              "bool": {
                "must": [
                  {
                    "range": {
                      "hotelPrices.sellPrice": {
                        "gte": 200,
                        "lte": 1000
                      }
                    }
                  },
                  {
                    "range": {
                      "hotelPrices.stockQuantity": {
                        "gt": 0
                      }
                    }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "filtered_nested": {
      "nested": {
        "path": "hotelPrices"
      },
      "aggs": {
        "where": {
          "range": {
            "field": "hotelPrices.sellPrice",
            "ranges": [
              {
                "from": 200,
                "to": 1000
              }
            ]
          },
          "aggs": {
            "and_where": {
              "range": {
                "field": "hotelPrices.stockQuantity",
                "ranges": [
                  {
                    "from": 0.1
                  }
                ]
              },
              "aggs": {
                "group_by": {
                  "terms": {
                    "field": "hotelPrices.goodsId",
                    "size": 10,
                    "order": {
                      "_key": "asc"
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```



```
// 优化3.select * from goods_index where ancestryCategoryId=2 and  hotelPrices.sellPrice//>=200 and hotelPrices.sellPrice<=1000 and hotelPrices.stockQuantity>0 and hotelPrices.specValue>? and hotelPrices.specValue<? group by goodId having(count>天数)  //这块不对，因为聚合计算的不是 过滤之后的嵌套文档，即不是inner_hits,所以在聚合时也对nested里面的内容做了层层过滤
GET goods_test2/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "ancestryCategoryId": {
              "value": "2"
            }
          }
        },
        {
          "nested": {
            "path": "hotelPrices",
            "ignore_unmapped": true,
            "score_mode": "none",
            "boost": 1,
            "inner_hits": {
              "ignore_unmapped": true,
              "from": 0,
              "size": 90,
              "version": false,
              "seq_no_primary_term": false,
              "explain": false,
              "track_scores": false
            },
            "query": {
              "bool": {
                "must": [
                  {
                    "range": {
                      "hotelPrices.sellPrice": {
                        "gte": 200,
                        "lte": 1000
                      }
                    }
                  },
                  {
                    "range": {
                      "hotelPrices.stockQuantity": {
                        "gt": 0
                      }
                    }
                  },
                  {
                    "range": {
                      "hotelPrices.specValue": {
                        "gte": "2022-12-01",
                        "lte": "2022-12-20"
                      }
                    }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "filtered_nested": {
      "nested": {
        "path": "hotelPrices"
      },
      "aggs": {
        "where": {
           "filter": {
             "range": {
                      "hotelPrices.sellPrice": {
                        "gte": 200,
                        "lte": 1000
                      }
                    }
           }
        }
      }
    }
  }
}
```



```
// 优化4.select * from goods_index where ancestryCategoryId=2 and hotelPrices.specValue>? and hotelPrices.specValue<? group by goodId having(avg(hotelPrices.sellPrice)>? and avg(hotelPrices.sellPrice)<? )  //这块不对，因为聚合计算的不是 过滤之后的嵌套文档，即不是inner_hits,所以在聚合时也对nested里面的内容做了层层过滤
GET goods_test2/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "ancestryCategoryId": {
              "value": "2"
            }
          }
        },
        {
          "nested": {
            "path": "hotelPrices",
            "ignore_unmapped": true,
            "score_mode": "none",
            "boost": 1,
            "inner_hits": {
              "ignore_unmapped": true,
              "from": 0,
              "size": 90,
              "version": false,
              "seq_no_primary_term": false,
              "explain": false,
              "track_scores": false
            },
            "query": {
              "bool": {
                "must": [
                  {
                    "range": {
                      "hotelPrices.sellPrice": {
                        "gte": 200,
                        "lte": 1000
                      }
                    }
                  },
                  {
                    "range": {
                      "hotelPrices.stockQuantity": {
                        "gt": 0
                      }
                    }
                  },
                  {
                    "range": {
                      "hotelPrices.specValue": {
                        "gte": "2022-12-01",
                        "lte": "2022-12-20"
                      }
                    }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "filtered_nested": {
      "nested": {
        "path": "hotelPrices"
      },
      "aggs": {
        "where": {
          "range": {
            "field": "hotelPrices.specValue",
            "ranges": [
              {
                "from": "2022-12-01",
                "to": "2022-12-30"
              }
            ]
          },
          "aggs": {
            "group_by": {
              "terms": {
                "field": "hotelPrices.goodsId",
                "size": 90,
                "order": {
                  "_key": "asc"
                }
              },
              "aggs": {
                "avg_price": {
                  "avg": {
                    "field": "hotelPrices.sellPrice"
                  }
                }
                ,
                 "aggs": {
                    "bucket_selector": {
                    "buckets_path": {
                      "avgprice": "avg_price"
                    },
                    //基于筛选后的内容算平均值，且平均值大于600小于900
                    "script": "params.avgprice>600 && params.avgprice<900"
                  }
                   
                 }
              }
            }
          }
        }
      }
    }
  }
}
```





```
// 优化5.select * from goods_index where ancestryCategoryId=2 and hotelPrices.specValue>? and hotelPrices.specValue<? group by goodId having(avg(hotelPrices.sellPrice)>? and avg(hotelPrices.sellPrice)<? )  //这块不对，因为聚合计算的不是 过滤之后的嵌套文档，即不是inner_hits,所以在聚合时也对nested里面的内容做了单次过滤
GET goods_test2/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "ancestryCategoryId": {
              "value": "2"
            }
          }
        },
        {
          "nested": {
            "path": "hotelPrices",
            "ignore_unmapped": true,
            "score_mode": "none",
            "boost": 1,
            "inner_hits": {
              "ignore_unmapped": true,
              "from": 0,
              "size": 90,
              "version": false,
              "seq_no_primary_term": false,
              "explain": false,
              "track_scores": false
            },
            "query": {
              "bool": {
                "must": [
                  {
                    "range": {
                      "hotelPrices.sellPrice": {
                        "gte": 200,
                        "lte": 1000
                      }
                    }
                  },
                  {
                    "range": {
                      "hotelPrices.stockQuantity": {
                        "gt": 0
                      }
                    }
                  },
                  {
                    "range": {
                      "hotelPrices.specValue": {
                        "gte": "2022-12-01",
                        "lte": "2022-12-30"
                      }
                    }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "filtered_nested": {
      "nested": {
        "path": "hotelPrices"
      },
      "aggs": {
        "where": {
          "filter": {
            "bool": {
              "filter": [
                {
                  "range": {
                    "hotelPrices.specValue": {
                      "gte": "2022-12-08",
                      "lte": "2022-12-30"
                    }
                  }
                },
                {
                  "range": {
                    "hotelPrices.stockQuantity": {
                      "gt": 0
                    }
                  }
                }
              ]
            }
          },
          "aggs": {
            "group_by": {
              "terms": {
                "field": "hotelPrices.goodsId",
                "size": 90,
                "order": {
                  "_key": "asc"
                }
              },
              "aggs": {
                "avg_price": {
                  "avg": {
                    "field": "hotelPrices.sellPrice"
                  }
                },
                "aggs": {
                  //"having": {
                    //基于筛选后的内容算平均值，且平均值大于600小于900
                    "bucket_selector": {
                      "buckets_path": {
                        "avgprice": "avg_price",
                        "counts": "_count"
                      },
                      "script": "params.avgprice>600 && params.avgprice<900 &&params.counts==22"
                    }
                 // }
                }
              }
            }
          }
        }
      }
    }
  }
}
```









GET _search
{"query":{"match_all":{}}}


GET _search
{
  "query": {
    "match_all": {}
  }
}

GET expo_enterprise_test/_search
{}

GET expo_enterprise_test/_mapping
{}


GET expo_enterprise_test/_search
{
  "query": {
    "term": {
      "_id": {
        "value": "98132208"
      }
    }
  }
}


GET stores_test/_search
{
  "query": {
    "terms": {
      "channelId": [
        "126"
      ]
    }
  }
}

GET goods_test2/_mapping

GET goods_test/_mapping
GET stores_test/_mapping

GET goods_test/_search
{
 "query": {
   "term": {
     "id": {
       "value": "43746"
     }
   }
 }
}

// 优化1.select * from goods_index where ancestryCategoryId=2 and  hotelPrices.sellPrice//>=200 and hotelPrices.sellPrice<=1000 and hotelPrices.stockQuantity>0 //这块不对，因为聚合计算的不是 过滤之后的嵌套文档，即不是inner_hits,不合适
GET goods_test2/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "ancestryCategoryId": {
              "value": "2"
            }
          }
        },
        {
          "nested": {
            "path": "hotelPrices",
            "ignore_unmapped": true,
            "score_mode": "none",
            "boost": 1,
            "inner_hits": {
              "ignore_unmapped": true,
              "from": 0,
              "size": 3,
              "version": false,
              "seq_no_primary_term": false,
              "explain": false,
              "track_scores": false
            },
            "query": {
              "bool": {
                "must": [
                  {
                    "range": {
                      "hotelPrices.sellPrice": {
                        "gte": 200,
                        "lte": 1000
                      }
                    }
                  },
                  {
                    "range": {
                      "hotelPrices.stockQuantity": {
                        "gt":0
                      }
                    }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  },
  "aggregations": {
    "salesNested": {
      "nested": {
        "path": "hotelPrices"
      },
      "aggregations": {  
        "group_by": {
          "terms": {
            "field": "hotelPrices.goodsId",
            "size": 10,
            "order": {
              "_key": "asc"
            }
          }
        }
      }
    }
  }
}


// 优化2.select * from goods_index where ancestryCategoryId=2 and  hotelPrices.sellPrice//>=200 and hotelPrices.sellPrice<=1000 and hotelPrices.stockQuantity>0 group by goodId having(count>天数) 
//这块不对，因为聚合计算的不是 过滤之后的嵌套文档，即不是inner_hits,所以在聚合时也对nested里面的内容做了层层过滤，但是聚合过滤后每个商品符合条件的天数还需要再过滤，目前没做having(count>天数)  
GET goods_test2/_search
{
  "from": 0,
  "size": 1,
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "ancestryCategoryId": {
              "value": "2"
            }
          }
        },
        {
          "nested": {
            "path": "hotelPrices",
            "ignore_unmapped": true,
            "score_mode": "none",
            "boost": 1,
            "inner_hits": {
              "ignore_unmapped": true,
              "from": 0,
              "size": 90,
              "version": false,
              "seq_no_primary_term": false,
              "explain": false,
              "track_scores": false
            },
            "query": {
              "bool": {
                "must": [
                  {
                    "range": {
                      "hotelPrices.sellPrice": {
                        "gte": 200,
                        "lte": 1000
                      }
                    }
                  },
                  {
                    "range": {
                      "hotelPrices.stockQuantity": {
                        "gt": 0
                      }
                    }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "filtered_nested": {
      "nested": {
        "path": "hotelPrices"
      },
      "aggs": {
        "where": {
          "range": {
            "field": "hotelPrices.sellPrice",
            "ranges": [
              {
                "from": 200,
                "to": 1000
              }
            ]
          },
          "aggs": {
            "and_where": {
              "range": {
                "field": "hotelPrices.stockQuantity",
                "ranges": [
                  {
                    "from": 0.1
                  }
                ]
              },
              "aggs": {
                "group_by": {
                  "terms": {
                    "field": "hotelPrices.goodsId",
                    "size": 10,
                    "order": {
                      "_key": "asc"
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}


// 优化3.select * from goods_index where ancestryCategoryId=2 and  hotelPrices.sellPrice//>=200 and hotelPrices.sellPrice<=1000 and hotelPrices.stockQuantity>0 and hotelPrices.specValue>? and hotelPrices.specValue<? group by goodId having(count>天数)  //这块不对，因为聚合计算的不是 过滤之后的嵌套文档，即不是inner_hits,所以在聚合时也对nested里面的内容做了层层过滤
GET goods_test2/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "ancestryCategoryId": {
              "value": "2"
            }
          }
        },
        {
          "nested": {
            "path": "hotelPrices",
            "ignore_unmapped": true,
            "score_mode": "none",
            "boost": 1,
            "inner_hits": {
              "ignore_unmapped": true,
              "from": 0,
              "size": 90,
              "version": false,
              "seq_no_primary_term": false,
              "explain": false,
              "track_scores": false
            },
            "query": {
              "bool": {
                "must": [
                  {
                    "range": {
                      "hotelPrices.sellPrice": {
                        "gte": 200,
                        "lte": 1000
                      }
                    }
                  },
                  {
                    "range": {
                      "hotelPrices.stockQuantity": {
                        "gt": 0
                      }
                    }
                  },
                  {
                    "range": {
                      "hotelPrices.specValue": {
                        "gte": "2022-12-01",
                        "lte": "2022-12-20"
                      }
                    }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "filtered_nested": {
      "nested": {
        "path": "hotelPrices"
      },
      "aggs": {
        "where": {
           "filter": {
             "range": {
                      "hotelPrices.sellPrice": {
                        "gte": 200,
                        "lte": 1000
                      }
                    }
           }
        }
      }
    }
  }
}





// 优化4.select * from goods_index where ancestryCategoryId=2 and hotelPrices.specValue>? and hotelPrices.specValue<? group by goodId having(avg(hotelPrices.sellPrice)>? and avg(hotelPrices.sellPrice)<? )  //这块不对，因为聚合计算的不是 过滤之后的嵌套文档，即不是inner_hits,所以在聚合时也对nested里面的内容做了层层过滤
GET goods_test2/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "ancestryCategoryId": {
              "value": "2"
            }
          }
        },
        {
          "nested": {
            "path": "hotelPrices",
            "ignore_unmapped": true,
            "score_mode": "none",
            "boost": 1,
            "inner_hits": {
              "ignore_unmapped": true,
              "from": 0,
              "size": 90,
              "version": false,
              "seq_no_primary_term": false,
              "explain": false,
              "track_scores": false
            },
            "query": {
              "bool": {
                "must": [
                  {
                    "range": {
                      "hotelPrices.sellPrice": {
                        "gte": 200,
                        "lte": 1000
                      }
                    }
                  },
                  {
                    "range": {
                      "hotelPrices.stockQuantity": {
                        "gt": 0
                      }
                    }
                  },
                  {
                    "range": {
                      "hotelPrices.specValue": {
                        "gte": "2022-12-01",
                        "lte": "2022-12-20"
                      }
                    }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "filtered_nested": {
      "nested": {
        "path": "hotelPrices"
      },
      "aggs": {
        "where": {
          "range": {
            "field": "hotelPrices.specValue",
            "ranges": [
              {
                "from": "2022-12-01",
                "to": "2022-12-30"
              }
            ]
          },
          "aggs": {
            "group_by": {
              "terms": {
                "field": "hotelPrices.goodsId",
                "size": 90,
                "order": {
                  "_key": "asc"
                }
              },
              "aggs": {
                "avg_price": {
                  "avg": {
                    "field": "hotelPrices.sellPrice"
                  }
                }
                ,
                 "aggs": {
                    "bucket_selector": {
                    "buckets_path": {
                      "avgprice": "avg_price"
                    },
                    //基于筛选后的内容算平均值，且平均值大于600小于900
                    "script": "params.avgprice>600 && params.avgprice<900"
                  }
                   
                 }
              }
            }
          }
        }
      }
    }
  }
}


// 优化5.select * from goods_index where ancestryCategoryId=2 and hotelPrices.specValue>? and hotelPrices.specValue<? group by goodId having(avg(hotelPrices.sellPrice)>? and avg(hotelPrices.sellPrice)<? )  //这块不对，因为聚合计算的不是 过滤之后的嵌套文档，即不是inner_hits,所以在聚合时也对nested里面的内容做了单次过滤
GET goods_test2/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "ancestryCategoryId": {
              "value": "2"
            }
          }
        },
        {
          "nested": {
            "path": "hotelPrices",
            "ignore_unmapped": true,
            "score_mode": "none",
            "boost": 1,
            "inner_hits": {
              "ignore_unmapped": true,
              "from": 0,
              "size": 90,
              "version": false,
              "seq_no_primary_term": false,
              "explain": false,
              "track_scores": false
            },
            "query": {
              "bool": {
                "must": [
                  {
                    "range": {
                      "hotelPrices.sellPrice": {
                        "gte": 200,
                        "lte": 1000
                      }
                    }
                  },
                  {
                    "range": {
                      "hotelPrices.stockQuantity": {
                        "gt": 0
                      }
                    }
                  },
                  {
                    "range": {
                      "hotelPrices.specValue": {
                        "gte": "2022-12-01",
                        "lte": "2022-12-30"
                      }
                    }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "filtered_nested": {
      "nested": {
        "path": "hotelPrices"
      },
      "aggs": {
        "where": {
          "filter": {
            "bool": {
              "filter": [
                {
                  "range": {
                    "hotelPrices.specValue": {
                      "gte": "2022-12-08",
                      "lte": "2022-12-30"
                    }
                  }
                },
                {
                  "range": {
                    "hotelPrices.stockQuantity": {
                      "gt": 0
                    }
                  }
                }
              ]
            }
          },
          "aggs": {
            "group_by": {
              "terms": {
                "field": "hotelPrices.goodsId",
                "size": 90,
                "order": {
                  "_key": "asc"
                }
              },
              "aggs": {
                "avg_price": {
                  "avg": {
                    "field": "hotelPrices.sellPrice"
                  }
                },
                "aggs": {
                  //"having": {
                    //基于筛选后的内容算平均值，且平均值大于600小于900
                    "bucket_selector": {
                      "buckets_path": {
                        "avgprice": "avg_price",
                        "counts": "_count"
                      },
                      "script": "params.avgprice>600 && params.avgprice<900 &&params.counts==23"
                    }
                 // }
                }
              }
            }
          }
        }
      }
    }
  }
}

 

GET goods_test2/_search
{
 "query": {
   "term": {
     "ancestryCategoryId": {
       "value": "2"
     }
   }
 }
}

GET goods_test2/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "ancestryCategoryId": {
              "value": "2"
            }
          }
        },
        {
          "nested": {
            "path": "hotelPrices",
            "ignore_unmapped": true,
            "score_mode": "none",
            "boost": 1,
            "inner_hits": {
              "ignore_unmapped": true,
              "from": 0,
              "size": 3,
              "version": false,
              "seq_no_primary_term": false,
              "explain": false,
              "track_scores": false
            },
            "query": {
              "range": {
                "hotelPrices.sellPrice": {
                  "gte": 200,
                  "lte": 1000
                }
              }
            }
          }
        }
      ]
    }
  },
  "aggregations": {
    "agg": {
      "terms": {
        "field": "_id"
      },
      "salesNested": {
        "nested": {
          "path": "hotelPrices"
        },
        "aggregations": {
          "priceByRange": {
            "range": {
              "field": "hotelPrices.sellPrice",
              "ranges": [
                {
                  "from": 200,
                  "to": 1000
                }
              ]
            }
          }
        }
      }
    }
  }
}

GET goods_test2/_search
{
  "from": 0,
  "size": 10,
  "timeout": "5s",
  "query": {
    "function_score": {
      "query": {
        "bool": {
          "must": [
            {
              "terms": {
                "ancestryCategoryId": [
                  "2"
                ],
                "boost": 1
              }
            },
            {
              "term": {
                "projectCode": {
                  "value": 3300,
                  "boost": 1
                }
              }
            },
            {
              "range": {
                "totalCommentStar": {
                  "from": 0,
                  "to": null,
                  "include_lower": true,
                  "include_upper": true,
                  "boost": 1
                }
              }
            },
            {
              "nested": {
                "query": {
                  "bool": {
                    "must": [
                      {
                        "range": {
                          "hotelPrices.sellPrice": {
                            "from": 400,
                            "to": null,
                            "include_lower": true,
                            "include_upper": true,
                            "boost": 1
                          }
                        }
                      },
                      {
                        "range": {
                          "hotelPrices.stockQuantity": {
                            "from": 0,
                            "to": null,
                            "include_lower": false,
                            "include_upper": true,
                            "boost": 1
                          }
                        }
                      }
                    ],
                    "adjust_pure_negative": true,
                    "boost": 1
                  }
                },
                "path": "hotelPrices",
                "ignore_unmapped": true,
                "score_mode": "none",
                "boost": 1,
                "inner_hits": {
                  "ignore_unmapped": true,
                  "from": 0,
                  "size": 3,
                  "version": false,
                  "seq_no_primary_term": false,
                  "explain": false,
                  "track_scores": false
                }
              }
            },
            {
              "match_all": {
                "boost": 1
              }
            }
          ],
          "adjust_pure_negative": true,
          "boost": 1
        }
      },
      "functions": [
        {
          "filter": {
            "bool": {
              "must": [
                {
                  "terms": {
                    "ancestryCategoryId": [
                      "2"
                    ],
                    "boost": 1
                  }
                },
                {
                  "term": {
                    "projectCode": {
                      "value": 3300,
                      "boost": 1
                    }
                  }
                },
                {
                  "range": {
                    "totalCommentStar": {
                      "from": 0,
                      "to": null,
                      "include_lower": true,
                      "include_upper": true,
                      "boost": 1
                    }
                  }
                },
                {
                  "nested": {
                    "query": {
                      "bool": {
                        "must": [
                          {
                            "range": {
                              "hotelPrices.sellPrice": {
                                "from": 400,
                                "to": null,
                                "include_lower": true,
                                "include_upper": true,
                                "boost": 1
                              }
                            }
                          },
                          {
                            "range": {
                              "hotelPrices.stockQuantity": {
                                "from": 0,
                                "to": null,
                                "include_lower": false,
                                "include_upper": true,
                                "boost": 1
                              }
                            }
                          }
                        ],
                        "adjust_pure_negative": true,
                        "boost": 1
                      }
                    },
                    "path": "hotelPrices",
                    "ignore_unmapped": true,
                    "score_mode": "none",
                    "boost": 1,
                    "inner_hits": {
                      "ignore_unmapped": true,
                      "from": 0,
                      "size": 3,
                      "version": false,
                      "seq_no_primary_term": false,
                      "explain": false,
                      "track_scores": false
                    }
                  }
                },
                {
                  "match_all": {
                    "boost": 1
                  }
                }
              ],
              "adjust_pure_negative": true,
              "boost": 1
            }
          },
          "script_score": {
            "script": {
              "source": "Math.log((doc['goodCommentRate'].value == 0.0 ? 0.96 : doc['goodCommentRate'].value * 10 +1) ) * 30 + Math.log10((doc['salesSum'].value == 0 ? 1 : doc['salesSum'].value * 100 +1)) + Math.log10((doc['createTime'].value == 0 ? 1 : doc['createTime'].value+1)) * 300 + Math.log((doc['visitCount'].value == 0 ? 1 : doc['visitCount'].value+1) ) + _score",
              "lang": "painless"
            }
          }
        }
      ],
      "score_mode": "sum",
      "boost_mode": "sum",
      "max_boost": 3.4028235e+38,
      "boost": 1
    }
  },
  "_source": {
    "includes": [
      "supplierId",
      "storeId",
      "goodsTags",
      "price",
      "promPrice",
      "promType",
      "promStock",
      "label",
      "preheatStarttime",
      "preheatImage",
      "showStarttime",
      "showEndtime",
      "seckillStarttime",
      "seckillEndtime",
      "salesSum",
      "imageUrls",
      "atmosphereImage",
      "name",
      "trait",
      "id",
      "ancestryCategoryId",
      "projectCode",
      "isDistribute",
      "onOff",
      "shareNum",
      "commissionValue",
      "validStartTime",
      "validEndTime",
      "siteName",
      "frontendIds",
      "province",
      "city",
      "district",
      "provinceName",
      "cityName",
      "districtName",
      "imageMainUrl",
      "goodsAttributes",
      "goodsAttribute",
      "hotelPrices",
      "storeName",
      "totalCommentStar"
    ],
    "excludes": [
      "backCatId",
      "salesSumTrue",
      "goodCommentRate",
      "sellerCity",
      "sellerDistinct",
      "sellerProvince"
    ]
  },
  "aggregations": {
    "filtered_nested": {
      "nested": {
        "path": "hotelPrices"
      },
      "aggregations": {
        "where": {
          "filter": {
            "bool": {
              "filter": [
                {
                  "range": {
                    "hotelPrices.stockQuantity": {
                      "from": 0,
                      "to": null,
                      "include_lower": false,
                      "include_upper": true,
                      "boost": 1
                    }
                  }
                }
              ],
              "adjust_pure_negative": true,
              "boost": 1
            }
          },
          "aggregations": {
            "group_by": {
              "terms": {
                "field": "hotelPrices.goodsId",
                "size": 10,
                "min_doc_count": 1,
                "shard_min_doc_count": 0,
                "show_term_doc_count_error": false,
                "order": [
                  {
                    "_key": "asc"
                  },
                  {
                    "_key": "asc"
                  }
                ]
              },
              "aggregations": {
                "avg_price": {
                  "avg": {
                    "field": "hotelPrices.sellPrice"
                  }
                },
                "having": {
                  "bucket_selector": {
                    "buckets_path": {
                      "counts": "_count",
                      "avgprice": "avg_price"
                    },
                    "script": {
                      "source": "params.avgprice>=200 && params.avgprice<400 &&params.counts==90",
                      "lang": "painless"
                    },
                    "gap_policy": "skip"
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}

GET homepage_recommend_test/_search
{
  "query": {
    "term": {
      "isRecommend": {
        "value": "1"
      }
    }
  }
}

GET homepage_recommend_test/_search
{
  "query": {
    "term": {
      "recommendType": {
        "value": "9"
      }
    }
  }
}

GET expo_enterprise_test/_mapping

GET expo_enterprise_test/_search
{
  "from": 0,
  "size": 10,
  "timeout": "2s",
  "query": {
    "bool": {
      "must": [
        {
          "nested": {
            "query": {
              "match_phrase": {
                "expoEnterpriseGoods.goodsName": {
                  "query": "自制莲子",
                  "boost": 1
                }
              }
            },
            "path": "expoEnterpriseGoods",
            "ignore_unmapped": true,
            "score_mode": "none",
            "boost": 1,
            "inner_hits": {
              "ignore_unmapped": true,
              "from": 0,
              "size": 3,
              "version": false,
              "seq_no_primary_term": false,
              "explain": false,
              "track_scores": false
            }
          }
        }
      ],
      "adjust_pure_negative": true,
      "boost": 1
    }
  },
  "_source": {
    "includes": [
      "id",
      "suppliersId",
      "suppliersName",
      "storeName",
      "storeId",
      "headPic",
      "mainGoods",
      "expoPosition",
      "expoEnterpriseGoods",
      "csiteId",
      "psiteId",
      "siteId"
    ],
    "excludes": [
      ""
    ]
  }
}

//嵌套
GET expo_enterprise_test/_search
{
  "query": {
    "nested": {
      "path": "expoEnterpriseGoods",
      "query": {
        "match_phrase": {
          "expoEnterpriseGoods.goodsName": "自制莲子"
        }
      },
      "inner_hits": {
        "ignore_unmapped": true
      }
    }
  }
}

GET expo_enterprise_test/_search
{
  "query": {
    "terms": {
      "_id": [
        "1654",
        "573"
      ]
    }
  }
}

//更新
POST expo_enterprise_test/_update_by_query
{
  "query": {
    "terms": {
      "_id": [
        "573",
        "1654"
      ]
    }
  }
}

GET expo_enterprise_test/_search
{
  "query": {
    "nested": {
      "path": "expoEnterpriseGoods",
      "query": {
        "match_phrase": {
          "expoEnterpriseGoods.goodsName": "自制莲子"
        }
      },
      "inner_hits": {
        "ignore_unmapped": true
      }
    }
  }
}




GET expo_enterprise_test/_search
{
  //"explain": true, 
  "from":0,"size":10,"timeout":"2s","query":{"bool":{"must":[{"match":{"expoEnterpriseGoods.goodsName":{"query":"秋田小町","operator":"OR","prefix_length":0,"max_expansions":50,"fuzzy_transpositions":true,"lenient":false,"zero_terms_query":"NONE","auto_generate_synonyms_phrase_query":true,"boost":1.0}}}],"adjust_pure_negative":true,"boost":1.0}},"_source":{"includes":["id","suppliersId","suppliersName","mainGoods","expoPosition","expoEnterpriseGoods"],"excludes":[""]}}

GET stores_test/_search
{"from":0,"size":1000,"timeout":"2s","query":{"function_score":{"query":{"bool":{"must":[{"term":{"projectCode":{"value":3300,"boost":1.0}}},{"terms":{"ancestryCategory":["2","5","6"],"boost":1.0}}],"adjust_pure_negative":true,"boost":1.0}},"functions":[{"filter":{"match_all":{"boost":1.0}},"weight":3.0,"field_value_factor":{"field":"onSaleGoodsCount","factor":10.0,"missing":1.0,"modifier":"none"}},{"filter":{"match_all":{"boost":1.0}},"weight":3.0,"field_value_factor":{"field":"stars","factor":10.0,"missing":1.0,"modifier":"none"}}],"score_mode":"sum","boost_mode":"sum","max_boost":3.4028235E38,"boost":1.0}},"_source":{"includes":["storeId","supplierId","storeName","signPic","storeStar","location","stars","features","salesSumTrue","provinceName","cityName","districtName","onSaleGoodsCount","channelId"],"excludes":["sellerCity","sellerDistinct","sellerProvince","provinceCode","cityCode","districtCode"]},"sort":[{"_score":{"order":"desc"}},{"createTime":{"order":"asc"}}]}


//筛选

GET goods_test/_search
{
  "from": 0,
  "size": 10,
  "timeout": "2s",
  "query": {
    "function_score": {
      "query": {
        "bool": {
          "must": [
            {
              "term": {
                "projectCode": {
                  "value": 3300,
                  "boost": 1
                }
              }
            },
            {
              "bool": {
                "adjust_pure_negative": true,
                "boost": 1
              }
            },
            {
              "term": {
                "projectCode": {
                  "value": 3300,
                  "boost": 1
                }
              }
            },
            {
              "terms": {
                "ancestryCategoryId": [
                  "7"
                ],
                "boost": 1
              }
            },
            {
              "term": {
                "goodsAttribute.beeStatus.keyword": {
                  "value": "蜂王健康",
                  "boost": 1
                }
              }
            },

            {
              "match_all": {
                "boost": 1
              }
            }
          ],
          "adjust_pure_negative": true,
          "boost": 1
        }
      },
      "functions": [
        {
          "filter": {
            "bool": {
              "must": [
                {
                  "term": {
                    "projectCode": {
                      "value": 3300,
                      "boost": 1
                    }
                  }
                },
                {
                  "bool": {
                    "adjust_pure_negative": true,
                    "boost": 1
                  }
                },
                {
                  "term": {
                    "projectCode": {
                      "value": 3300,
                      "boost": 1
                    }
                  }
                },
                {
                  "terms": {
                    "ancestryCategoryId": [
                      "7"
                    ],
                    "boost": 1
                  }
                },
                {
                  "range": {
                    "goodsAttribute.beeboxTemperature": {
                      "from": "-50",
                      "to": "10.0",
                      "include_lower": true,
                      "include_upper": true,
                      "boost": 1
                    }
                  }
                },
                {
                  "range": {
                    "goodsAttribute.beeCount": {
                      "from": "1000.0",
                      "to": "3000.0",
                      "include_lower": true,
                      "include_upper": true,
                      "boost": 1
                    }
                  }
                },
                {
                  "term": {
                    "goodsAttribute.beeStatus.keyword": {
                      "value": "蜂王健康",
                      "boost": 1
                    }
                  }
                },
                {
                  "term": {
                    "goodsAttribute.beeProduction.keyword": {
                      "value": "3.5",
                      "boost": 1
                    }
                  }
                },
                {
                  "match_all": {
                    "boost": 1
                  }
                }
              ],
              "adjust_pure_negative": true,
              "boost": 1
            }
          },
          "script_score": {
            "script": {
              "source": "Math.log((doc['goodCommentRate'].value == 0.0 ? 0.96 : doc['goodCommentRate'].value * 10 +1) ) * 30 + Math.log10((doc['salesSum'].value == 0 ? 1 : doc['salesSum'].value * 100 +1)) + Math.log10((doc['createTime'].value == 0 ? 1 : doc['createTime'].value+1)) * 300 + Math.log((doc['visitCount'].value == 0 ? 1 : doc['visitCount'].value+1) ) + _score",
              "lang": "painless"
            }
          }
        }
      ],
      "score_mode": "sum",
      "boost_mode": "sum",
      "max_boost": 3.4028235e+38,
      "boost": 1
    }
  },
  "_source": {
    "includes": [
      "supplierId",
      "storeId",
      "goodsTags",
      "price",
      "promPrice",
      "promType",
      "promStock",
      "label",
      "preheatStarttime",
      "preheatImage",
      "showStarttime",
      "showEndtime",
      "seckillStarttime",
      "seckillEndtime",
      "salesSum",
      "imageUrls",
      "atmosphereImage",
      "name",
      "trait",
      "id",
      "ancestryCategoryId",
      "projectCode",
      "isDistribute",
      "onOff",
      "shareNum",
      "commissionValue",
      "validStartTime",
      "validEndTime",
      "longitude",
      "latitude",
      "siteName",
      "frontendIds",
      "province",
      "city",
      "district",
      "provinceName",
      "cityName",
      "districtName",
      "imageMainUrl",
      "goodsAttributes",
      "goodsAttribute"
    ],
    "excludes": [
      "backCatId",
      "salesSumTrue",
      "goodCommentRate",
      "sellerCity",
      "sellerDistinct",
      "sellerProvince"
    ]
  },
  "aggregations": {
    "beeSource": {
      "terms": {
        "field": "goodsAttribute.beeSource.keyword",
        "size": 3,
        "min_doc_count": 1,
        "shard_min_doc_count": 0,
        "show_term_doc_count_error": false,
        "order": [
          {
            "_count": "desc"
          },
          {
            "_key": "asc"
          }
        ]
      }
    },
    "beeStatus": {
      "terms": {
        "field": "goodsAttribute.beeStatus.keyword",
        "size": 3,
        "min_doc_count": 1,
        "shard_min_doc_count": 0,
        "show_term_doc_count_error": false,
        "order": [
          {
            "_count": "desc"
          },
          {
            "_key": "asc"
          }
        ]
      }
    },
    "beeboxTemperature": {
      "range": {
        "field": "goodsAttribute.beeboxTemperature",
        "ranges": [
          {
            "from": -50,
            "to": 10
          },
          {
            "from": 10,
            "to": 25
          },
          {
            "from": 25,
            "to": 40
          },
          {
            "from": 40,
            "to": 100
          }
        ],
        "keyed": false
      }
    },
    "beeCount": {
      "range": {
        "field": "goodsAttribute.beeCount",
        "ranges": [
          {
            "from": 1000,
            "to": 3000
          },
          {
            "from": 3000,
            "to": 6000
          },
          {
            "from": 6000,
            "to": 10000
          }
        ],
        "keyed": false
      }
    },
    "beeProduction": {
      "terms": {
        "field": "goodsAttribute.beeProduction.keyword",
        "size": 3,
        "min_doc_count": 1,
        "shard_min_doc_count": 0,
        "show_term_doc_count_error": false,
        "order": [
          {
            "_count": "desc"
          },
          {
            "_key": "asc"
          }
        ]
      }
    },
    "packaging": {
      "terms": {
        "field": "goodsAttribute.packaging.keyword",
        "size": 3,
        "min_doc_count": 1,
        "shard_min_doc_count": 0,
        "show_term_doc_count_error": false,
        "order": [
          {
            "_count": "desc"
          },
          {
            "_key": "asc"
          }
        ]
      }
    },
    "netWeight": {
      "terms": {
        "field": "goodsAttribute.netWeight.keyword",
        "size": 3,
        "min_doc_count": 1,
        "shard_min_doc_count": 0,
        "show_term_doc_count_error": false,
        "order": [
          {
            "_count": "desc"
          },
          {
            "_key": "asc"
          }
        ]
      }
    }
  }
}






// 筛选

GET goods_test/_search
{
  "from": 0,
  "size": 10,
  "timeout": "2s",
  "query": {
    "function_score": {
      "query": {
        "bool": {
          "must": [
            {
              "term": {
                "projectCode": {
                  "value": 3300,
                  "boost": 1
                }
              }
            },
            {
              "bool": {
                "adjust_pure_negative": true,
                "boost": 1
              }
            },
            {
              "term": {
                "projectCode": {
                  "value": 3300,
                  "boost": 1
                }
              }
            },
            {
              "terms": {
                "ancestryCategoryId": [
                  "7"
                ],
                "boost": 1
              }
            },


            {
              "term": {
                "goodsAttribute.beeStatus.keyword": {
                  "value": "蜂王健康"
                }
              }
            },
       
            {
              "range": {
                "goodsAttribute.beeCount": {
                  "gte": 1000,
                  "lte": 3000
                }
              }
            },
            {
              "range": {
                "goodsAttribute.beeboxTemperature": {
                  "gte": 0,
                  "lte": 10
                }
              }
            },
             {
              "term": {
                "goodsAttribute.beeProduction.keyword": {
                  "value": "3.5"
                }
              }
            },
         
            {
              "match_all": {
                "boost": 1
              }
            }
          ],
          "adjust_pure_negative": true,
          "boost": 1
        }
      },
      "functions": [
        {
          "filter": {
            "bool": {
              "must": [
                {
                  "term": {
                    "projectCode": {
                      "value": 3300,
                      "boost": 1
                    }
                  }
                },
                {
                  "bool": {
                    "adjust_pure_negative": true,
                    "boost": 1
                  }
                },
                {
                  "term": {
                    "projectCode": {
                      "value": 3300,
                      "boost": 1
                    }
                  }
                },
                {
                  "terms": {
                    "ancestryCategoryId": [
                      "7"
                    ],
                    "boost": 1
                  }
                },
            
                {
                  "match_all": {
                    "boost": 1
                  }
                }
              ],
              "adjust_pure_negative": true,
              "boost": 1
            }
          },
          "script_score": {
            "script": {
              "source": "Math.log((doc['goodCommentRate'].value == 0.0 ? 0.96 : doc['goodCommentRate'].value * 10 +1) ) * 30 + Math.log10((doc['salesSum'].value == 0 ? 1 : doc['salesSum'].value * 100 +1)) + Math.log10((doc['createTime'].value == 0 ? 1 : doc['createTime'].value+1)) * 300 + Math.log((doc['visitCount'].value == 0 ? 1 : doc['visitCount'].value+1) ) + _score",
              "lang": "painless"
            }
          }
        }
      ],
      "score_mode": "sum",
      "boost_mode": "sum",
      "max_boost": 3.4028235e+38,
      "boost": 1
    }
  },
  "_source": {
    "includes": [
      "supplierId",
      "storeId",
      "goodsTags",
      "price",
      "promPrice",
      "promType",
      "promStock",
      "label",
      "preheatStarttime",
      "preheatImage",
      "showStarttime",
      "showEndtime",
      "seckillStarttime",
      "seckillEndtime",
      "salesSum",
      "imageUrls",
      "atmosphereImage",
      "name",
      "trait",
      "id",
      "ancestryCategoryId",
      "projectCode",
      "isDistribute",
      "onOff",
      "shareNum",
      "commissionValue",
      "validStartTime",
      "validEndTime",
      "longitude",
      "latitude",
      "siteName",
      "frontendIds",
      "province",
      "city",
      "district",
      "provinceName",
      "cityName",
      "districtName",
      "imageMainUrl",
      "goodsAttributes",
      "goodsAttribute"
    ],
    "excludes": [
      "backCatId",
      "salesSumTrue",
      "goodCommentRate",
      "sellerCity",
      "sellerDistinct",
      "sellerProvince"
    ]
  },
  "aggregations": {
    "beeSource": {
      "terms": {
        "field": "goodsAttribute.beeSource.keyword",
        "size": 3,
        "min_doc_count": 1,
        "shard_min_doc_count": 0,
        "show_term_doc_count_error": false,
        "order": [
          {
            "_count": "desc"
          },
          {
            "_key": "asc"
          }
        ]
      }
    },
    "beeStatus": {
      "terms": {
        "field": "goodsAttribute.beeStatus.keyword",
        "size": 3,
        "min_doc_count": 1,
        "shard_min_doc_count": 0,
        "show_term_doc_count_error": false,
        "order": [
          {
            "_count": "desc"
          },
          {
            "_key": "asc"
          }
        ]
      }
    },
    "beeboxTemperature": {
      "range": {
        "field": "goodsAttribute.beeboxTemperature",
        "ranges": [
          {
            "to": 10
          },
          {
            "from": 10,
            "to": 25
          },
          {
            "from": 25,
            "to": 40
          },
          {
            "from": 40
          }
        ],
        "keyed": false
      }
    },
    "beeCount": {
      "range": {
        "field": "goodsAttribute.beeCount",
        "ranges": [
          {
            "from": 1000,
            "to": 3000
          },
          {
            "from": 3000,
            "to": 6000
          },
          {
            "from": 6000,
            "to": 10000
          }
        ],
        "keyed": false
      }
    },
    "beeProduction": {
      "terms": {
        "field": "goodsAttribute.beeProduction.keyword",
        "size": 3,
        "min_doc_count": 1,
        "shard_min_doc_count": 0,
        "show_term_doc_count_error": false,
        "order": [
          {
            "_count": "desc"
          },
          {
            "_key": "asc"
          }
        ]
      }
    },
    "packaging": {
      "terms": {
        "field": "goodsAttribute.packaging.keyword",
        "size": 3,
        "min_doc_count": 1,
        "shard_min_doc_count": 0,
        "show_term_doc_count_error": false,
        "order": [
          {
            "_count": "desc"
          },
          {
            "_key": "asc"
          }
        ]
      }
    },
    "netWeight": {
      "terms": {
        "field": "goodsAttribute.netWeight.keyword",
        "size": 3,
        "min_doc_count": 1,
        "shard_min_doc_count": 0,
        "show_term_doc_count_error": false,
        "order": [
          {
            "_count": "desc"
          },
          {
            "_key": "asc"
          }
        ]
      }
    }
  }
}





GET goods_test/_search
{}

GET goods_test/_search
{
  "from": 0, 
  "size": 2000
  , "query": {
    "term": {
      "storeId.keyword": {
        "value": "62"
      }
    }
  }
}