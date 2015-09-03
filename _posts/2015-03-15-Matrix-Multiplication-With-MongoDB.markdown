---
layout: post
title:  "Matrix Multiplication with MongoDB's Aggregation Framework"
date:   2015-03-15 10:06:00
categories: MongoDB
comments: True
---
A while ago, I have been challenged by a banking prospect about how much could be done on the server side with MongoDB. One of his problem was to store large matrices (thousands of lines and columns) and manipulate them. Doing such thing within MongoDB raised two challenges. First, you need to represent such a large matrix while complying with the 16MB BSON document size limit. Second, the map/reduce framework isn't the fastest thing on earth, so you really want to give the aggregation framework a shot first, but as you'll see below, the syntax is a bit more complicated than JavaScript (to say the least).

So here we go. After trying several models, the following one finally fit my needs.


{% highlight json %}
{ "_id" : ObjectId("5405dcc0717fa3897dbcec2e"), "name" : "a", "row" : "0", "values" : [ { "col" : "0", "value" : 1 }, { "col" : "1", "value" : 2 }, { "col" : "2", "value" : 3 } ] }
{ "_id" : ObjectId("5405dcc0717fa3897dbcec2f"), "name" : "a", "row" : "1", "values" : [ { "col" : "0", "value" : 2 }, { "col" : "1", "value" : 3 }, { "col" : "2", "value" : 4 } ] }
{ "_id" : ObjectId("5405dcc0717fa3897dbcec30"), "name" : "a", "row" : "2", "values" : [ { "col" : "0", "value" : 3 }, { "col" : "1", "value" : 4 }, { "col" : "2", "value" : 5 } ] }
{ "_id" : ObjectId("5405dca0717fa3897dbcec2b"), "name" : "b", "row" : "0", "values" : [ { "col" : "0", "value" : 10 }, { "col" : "1", "value" : 11 }, { "col" : "2", "value" : 12 } ] }
{ "_id" : ObjectId("5405dca0717fa3897dbcec2c"), "name" : "b", "row" : "1", "values" : [ { "col" : "0", "value" : 11 }, { "col" : "1", "value" : 12 }, { "col" : "2", "value" : 13 } ] }
{ "_id" : ObjectId("5405dca0717fa3897dbcec2d"), "name" : "b", "row" : "2", "values" : [ { "col" : "0", "value" : 12 }, { "col" : "1", "value" : 13 }, { "col" : "2", "value" : 14 } ] }
{% endhighlight %}

Each document is the line of a matrix. Matrix "a" and "b" both have three lines, indexed by the "row" property. Then the "values" property is an array of sub-documents, containing the column number and the actual value. With this model, I should be able to remain way below the 16MB limits, and I don't really need to have a more granular model has I won't use values outside of the context of a row.

Now moving on to the actual matrix multiplication. Each line is a stage of the pipeline. I will document this deeper later, but for the time being I encourage you to remove all the steps, execute the aggregation and then adding the next one. It should then be quite easy to understand the transformations.

{% highlight js %}
db.matrix2.aggregate([
  {$match : { $or : [{name: 'a'}, {name : 'b'}]}},
  {$unwind : '$values'},  
  {$project : {  cell : { $cond : { if: { $eq: [ "$name", "a" ] },  then: { x : '$row', y : '$values.col', valx :'$values.value'},  else: {x: '$values.col', y: '$row', valy :'$values.value' }  }}  }},  
  {$group : {_id : {x : '$cell.x', y : '$cell.y'} , valx : {$sum : '$cell.valx'}, valy : {$sum: '$cell.valy'}}},
  {$project : {val : {$multiply : ['$valx', '$valy']}}},
  {$sort : {'_id.x' : 1, '_id.y' : 1}},  
  {$group : {_id : '$_id.x', row : {$first : '$_id.x'}, values : {$push : {col : '$_id.y', value : '$val'}}}}
])
{% endhighlight %}

And the result is a new matrix complying with the same schema we used for the input matrices.

{% highlight json %}
{ "_id" : "2", "row" : "2", "values" : [ { "col" : "0", "value" : 36 }, { "col" : "1", "value" : 52 }, { "col" : "2", "value" : 70 } ] }
{ "_id" : "1", "row" : "1", "values" : [ { "col" : "0", "value" : 22 }, { "col" : "1", "value" : 36 }, { "col" : "2", "value" : 52 } ] }
{ "_id" : "0", "row" : "0", "values" : [ { "col" : "0", "value" : 10 }, { "col" : "1", "value" : 22 }, { "col" : "2", "value" : 36 } ] }
{% endhighlight %}


