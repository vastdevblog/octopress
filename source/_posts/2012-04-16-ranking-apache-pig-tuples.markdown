---
layout: post
title: "Ranking Apache Pig Tuples with a Python UDF"
date: 2012-04-16 15:46
comments: true
categories: [hadoop, pig, python]
author: Josh Levy, Ph.D.
---

The [ORDER ... BY](http://pig.apache.org/docs/r0.9.2/basic.html#order-by) statement in Pig Latin sorts the ```tuples``` in a ```bag```.
Sometimes it is useful to know the rank of each ```tuple``` -- its position in the ```bag``` for a given sort order. This post demonstrates a [user defined function (UDF)](http://pig.apache.org/docs/r0.9.2/udf.html) written in Python that injects rank into each ```tuple``` in a ```bag```.  The code below has been tested on Pig 0.9.2.

<!-- more -->
Suppose that we have the following structure in Pig: ``` data: {group: int, record: {(make: chararray, model: chararray, mpg: int)}} ```

This Pig script sorts the records in each group in descending order by mpg.  Its output is in the same format as its input.

{% codeblock Sorting in Pig %}
data = foreach data {
    sorted = order record by mpg desc;
    generate group, sorted;
};
{% endcodeblock %}


This next Pig script sorts the records in each group by mpg and stores the rank with each record. Its output is formatted as: ```  data: {group: int, record: {(rank:int, make: chararray,model: chararray, mpg: int)}} ```  

{% codeblock Ranking in Pig with a Python UDF %}
register enumerate_bag.py using jython as eb;

data = foreach data {
    sorted = order record by mpg desc;
    generate group, eb.enumerate_bag(sorted) as record: {(rank: int, make: chararray, model: chararray, mpg: int)};
};
{% endcodeblock %}

Line 1 registers the functions in the Python module ```enumerate_bag``` as the Pig namespace ```eb```.  Line 5 calls the ```eb.enumerate_bag``` UDF to add rank to each ```tuple``` in ```sorted```.   Here is the Python source code.


{% codeblock Python UDF to Enumerate Tuples in a Bag -- enumerate_bag.py %}
def enumerate_bag(input):
    output = []
    for rank, item in enumerate(input):
        output.append(tuple([rank] + list(item)))
    return output
{% endcodeblock %}

Pig passes a ```bag``` to a Python UDF as a ```list``` of ```tuple```.  Line 3 uses the Python builtin ```enumerate``` to get the rank and value for each ```tuple``` in ```input```.  Each Python ```tuple``` is immutable, so line 4 of the function using ```list``` concatenation to create a new ```tuple``` containing the rank followed by the elements of the original ```tuple```.  The function returns a ```list``` of these new ```tuple```s, and Pig maps that result back to a ```bag```.  

Although Pig provides a way for a Python UDF to [specify its return type](http://pig.apache.org/docs/r0.9.2/udf.html#decorators) we have chosen not to do so.  The ```outputSchema``` decorator is too restrictive.  We want our UDF to be able to inject rank into any tuple.  The Pig documentation says the ```outputFunctionSchema``` decorator can be used to write a UDF that can accept multiple input types; however, constructing the corresponding ```schemaFunction``` is non-trivial.  Because the return type is not specified, Pig treats the result as ```bytearray``` and we specify its structure on line 5 of the Pig script.


In this blog post we discussed the relationship between sorting and ranking.   Pig has native support for sorting via the ```ORDER ... BY``` command.  We provided a Python UDF that adds ranking to a sorted bag. 



