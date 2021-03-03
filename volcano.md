## Volcano查询处理模型

volcano是最经典的数据库查询处理模型，许多数据库如mysql等都是基于volcano模型。

### 执行计划
![volcano模型图](static/volcano.png)

图中左边的查询语句被翻译成了右边的执行计划。下面根据些简单说明volcano模型设计与实现

### Operator算子抽象
```java
Operator{
   void open();
   Tuple next();
   void close();
}
```
volcano的所有算子都是包含open/next/close三个方法。
- open 用于资源初始化操作
- next 获取下一行
- close 释放资源

### TableScan

```java
TableScan implement Operator{

  TableData data;
  public void open(){
     data = loadData();
  }
  
  public Tuple next(){
     if(data.hasNext()){
        return data.next();
     }
     return END;
  }
  
  public void close(){
    data.close();
  }
}
```

### Filter
```java
Filter implement Operator{
   Operator input;
   Predicate predicate;
   
   public void open(){
      input.open();
   }
   
   public Tuple next(){
      Tuple tuple = input.next();
      while(tuple != END && !predicate.test(tuple)){
         tuple = input.next();
      }
      return tuple;
   }
   public void close(){
      input.close;
   }
}
```

### Projection
```java
Project implement Operator{
   Operator input;
   Function<Tuple,Tuple> mapFunction;
   public void open(){
      input.open();
   }
   
   public Tuple next(){
      Tuple tuple = input.next();
      if(tuple!=END){
        return mapFunction.apply(tuple);
      }
      return END;
   }
   public void close(){
      input.close();
   }
}
```

### Aggregation
```java
Aggregation implement Operator{
   Operator input;
   Function<Tuple,Object> keyFunction;
   List<AggCall> aggCalls;
   HashTable<Object, List<AggState>> hashTable;
   Iterator<Tuple> reduceIterator;
   public void open(){
      input.open();
      
      Tuple tuple = input.next();
      while(tuple!=END){
         Object key = keyFunction.apply(tuple);
         List<AggState> state = hashTable.get(key);
         if(state==null){
            state = new ArrayList<>();
            for(AggCall agg:aggCalls){
               state.add(agg.init());
            }
            hashTable.put(key, state);
         }
         for(int i=0;i<aggCalls.size();i++){
            aggCalls.get(i).combine(state.get(i), tuple);
         }
      }
      reduceIteratur = hashTable.values().map(state->state.reduce()).iterator();
   }
   
   public Tuple next(){
      if(!reduceIterator.hasNext()){
         return END;
      }
      return reduceIterator.next();
   }
   public void close(){
      input.close();
   }
}
```

### Driver
```java
Driver {
   public static void main(String[] args){
      Operator op = new TableScan(new TableData('lineitem'));
      op = new Filter(new Predicate("l_shipdate <= date '1998-12-01' - interval '90' day"));
      op = new Project(new Function("[l_returnflat, l_linesatus, l_extendedprice, l_extendedprice*(1-l_discount)...]"));
      op = new Aggregation(new Function("[l_returnflat, l_linesatus]"),new AggCallList("[sum(l_extendedprice), sum(l_extendedprice*(1-l_discount))...]"));
      op.open();
      while(op.next()!=END){
         op.next();
      }
      op.close();
   }
}
```
