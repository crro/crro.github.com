---
layout: post
title: "How to make a function implement an interface in Go"
date: 2020-07-17 11:46:09 -0500
comments: true
categories: golang interface functions
---

Having a function implement an interface can be really useful when testing or programming Go code. Being able to do this allows you to have a data structure such as a map or an array hold both functions and structs that implement the interface. Let's consider the following example, imagine that you are writing a data pipeline where you have to populate the various fields inside a car object. In order to populate these fields you have to fetch the information from various sources. You might have to query your internal database, a third party database, and other third party websites. In order to do this you decide to have a Transformer interface that takes in a Car and returns a Car with new information. The struct and the code look something like this:

{% highlight go %}
type Car struct {
	Model string
	Year int
	PremiumFeatures []string
	NumberOfSeats int
	Insurance string
	Owners map[string]string
  SerialNumber string
}

// Transformer is the interface that we are going to use to 
// apply various transformations to the car
type Transformer interface {
  // Transform takes in a car and returns a car with updated information.
	Transform(Car) (Car, error)
}
{% endhighlight %}

This setup makes sense because we can then build different clients that interact with the different data sources to populate the car. You decide to make a database client and an http client that implment the interface:

{% highlight go %}
type DBClient struct {
  /* Implementation details ... */
}

func (c *DBClient) Transform(c Car) (Car, error) {
  /* Implementation details ... */
}

type HttpClient struct {
  /* Implementation details ... */
}

func (c *HttpClient) Transform(c Car) (Car, error) {
  /* Implementation details ... */
}
{% endhighlight %}

At this point you are able to have a few instances and of your clients talking to different source and have them all grouped in a single array:

{% highlight go %}

func collectTransformations() []Transformer {
  transformers := make([]Transformer, 0, 0)
  dbC := new(DBClient)
  /* Implementation details ... */
  transformers = append(transformers, dbC)
  httpC := new(HttpClient)
  /* Implementation details ... */
  transformers = append(transformers, httpC)
  // Transformers now holds both DBClient and HttpClient and is ready to apply the transformations.
}


{% endhighlight %}

But, what if we had simpler transformers that did not required an entire struct to perform the transformation? For that we can have a function implement our interface and add it to our list!

{% highlight go %}

type TransformFunc func(Car) (Car, error)

func (tf TransformFunc) Transform(c Car) (Car, error) {
  return tf(c)
}

func AddYear(Car) (Car, error) {
  // Uses the current year and adds it to the car
}

func AddSerialNumber(Car) (Car, error) {
  // Generates serial numbers and adds it to the car
}


func collectTransformations() []Transformer {
  /* Code from previous section. */
  transformers = append(transformers, TransformFunc(AddYear))
  transformers = append(transformers, TransformFunc(AddSerialNumber))
  // Transformers now holds both DBClient and HttpClient AND our functions defined above.
}

{% endhighlight %}

{% highlight go %}
func applyTransformations(cars []Car, transformations []Transform) { 
  
  for i := range cars {
    for _, t := range transformations {
      cars[i] = t.Transform(cars[i])
    }
  }
}
  
{% endhighlight %}
