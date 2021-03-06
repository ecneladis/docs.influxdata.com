---
title: DerivativeNode
note: Auto generated by tickdoc

menu:
  kapacitor_02:
    name: Derivative
    identifier: derivative_node
    weight: 50
    parent: tick
---

Compute the derivative of a stream or batch. 
The derivative is computed on a single field 
and behaves similarly to the InfluxQL derivative 
function. Deriviative is not a MapReduce function 
and as a result is not part of the normal influxql functions. 

Example: 


```javascript
     stream
         .from().measurement('net_rx_packets')
         .derivative('value')
            .unit(1s) // default
            .nonNegative()
         ...
```

Computes the derivative via: 
(current - previous ) / ( time_difference / unit) 

For batch edges the derivative is computed for each 
point in the batch and because of boundary conditions 
the number of points is reduced by one. 


Properties
----------

Property methods modify state on the calling node. They do not add another node to the pipeline, and always return a reference to the calling node.

### As

The new name of the derivative field. 
Default is the name of the field used 
when calculating the derivative. 


```javascript
node.as(value string)
```


### NonNegative

If called the derivative will skip negative results. 


```javascript
node.nonNegative()
```


### Unit

The time unit of the resulting derivative value. 
Default: 1s 


```javascript
node.unit(value time.Duration)
```


Chaining Methods
----------------

Chaining methods create a new node in the pipeline as a child of the calling node. They do not modify the calling node.

### Alert

Create an alert node, which can trigger alerts. 


```javascript
node.alert()
```

Returns: [AlertNode](/kapacitor/v0.2/tick/alert_node/)


### Derivative

Create a new node that computes the derivative of adjacent points. 


```javascript
node.derivative(field string)
```

Returns: [DerivativeNode](/kapacitor/v0.2/tick/derivative_node/)


### Eval

Create an eval node that will evaluate the given transformation function to each data point. 
A list of expressions may be provided and will be evaluated in the order they are given 
and results of previous expressions are made available to later expressions. 


```javascript
node.eval(expressions ...tick.Node)
```

Returns: [EvalNode](/kapacitor/v0.2/tick/eval_node/)


### GroupBy

Group the data by a set of tags. 

Can pass literal * to group by all dimensions. 
Example: 


```javascript
    .groupBy(*)
```



```javascript
node.groupBy(tag ...interface{})
```

Returns: [GroupByNode](/kapacitor/v0.2/tick/group_by_node/)


### HttpOut

Create an http output node that caches the most recent data it has received. 
The cached data is available at the given endpoint. 
The endpoint is the relative path from the API endpoint of the running task. 
For example if the task endpoint is at &#34;/api/v1/task/&lt;task_name&gt;&#34; and endpoint is 
&#34;top10&#34;, then the data can be requested from &#34;/api/v1/task/&lt;task_name&gt;/top10&#34;. 


```javascript
node.httpOut(endpoint string)
```

Returns: [HTTPOutNode](/kapacitor/v0.2/tick/http_out_node/)


### InfluxDBOut

Create an influxdb output node that will store the incoming data into InfluxDB. 


```javascript
node.influxDBOut()
```

Returns: [InfluxDBOutNode](/kapacitor/v0.2/tick/influx_d_b_out_node/)


### Join

Join this node with other nodes. The data is joined on timestamp. 


```javascript
node.join(others ...Node)
```

Returns: [JoinNode](/kapacitor/v0.2/tick/join_node/)


### MapReduce

Perform a map-reduce operation on the data. 
The built-in functions under `influxql` provide the 
selection,aggregation, and transformation functions 
from the InfluxQL language. 

MapReduce may be applied to either a batch or a stream edge. 
In the case of a batch each batch is passed to the mapper idependently. 
In the case of a stream all incoming data points that have 
the exact same time are combined into a batch and sent to the mapper. 


```javascript
node.mapReduce(mr MapReduceInfo)
```

Returns: [ReduceNode](/kapacitor/v0.2/tick/reduce_node/)


### Sample

Create a new node that samples the incoming points or batches. 

One point will be emitted every count or duration specified. 


```javascript
node.sample(rate interface{})
```

Returns: [SampleNode](/kapacitor/v0.2/tick/sample_node/)


### Union

Perform the union of this node and all other given nodes. 


```javascript
node.union(node ...Node)
```

Returns: [UnionNode](/kapacitor/v0.2/tick/union_node/)


### Where

Create a new node that filters the data stream by a given expression. 


```javascript
node.where(expression tick.Node)
```

Returns: [WhereNode](/kapacitor/v0.2/tick/where_node/)


### Window

Create a new node that windows the stream by time. 

NOTE: Window can only be applied to stream edges. 


```javascript
node.window()
```

Returns: [WindowNode](/kapacitor/v0.2/tick/window_node/)

