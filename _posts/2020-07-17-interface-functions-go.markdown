---
layout: post
title: "Polymorphism with Functions in Go"
date: 2020-07-17 11:46:09 -0500
comments: true
author: crro
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

But, what if we had simpler transformers that did not require an entire struct to perform the transformation? Say we need to generate the serial number or add the current year to our car. For that we can have a function implement our interface and add it to our list of transformers!

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

Finally, we can then apply all of our transformations in a single function:

{% highlight go %}
func applyTransformations(cars []Car, transformations []Transform) {   
  for i := range cars {
    for _, t := range transformations {
      cars[i] = t.Transform(cars[i])
    }
  }
}
{% endhighlight %}

I think this post is long enough already but the last thing I want to mention is that you can use a similar pattern for testing your code. If you are working with a single method interface, using this pattern can save you quite a bit of time! 

If you liked this please share it and if you would like to hear more from me consider subscribing to my newsletter below. I generally write about container technology, and about different programming patterns in Go and/or Python.

Abrazos!

David

<!-- Begin Mailchimp Signup Form -->
<link href="//cdn-images.mailchimp.com/embedcode/horizontal-slim-10_7.css" rel="stylesheet" type="text/css">
<style type="text/css">
	#mc_embed_signup{background:#fff; clear:left; font:14px Helvetica,Arial,sans-serif; width:100%;}
	/* Add your own Mailchimp form style overrides in your site stylesheet or in this style block.
	   We recommend moving this block and the preceding CSS link to the HEAD of your HTML file. */
</style>
<div id="mc_embed_signup">
<form action="https://github.us10.list-manage.com/subscribe/post?u=85f552eee256467fd023266f3&amp;id=af3bc86a4e" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate" target="_blank" novalidate>
    <div id="mc_embed_signup_scroll">
	<label for="mce-EMAIL">Subscribe</label>
	<input type="email" value="" name="EMAIL" class="email" id="mce-EMAIL" placeholder="email address" required>
    <!-- real people should not fill this in and expect good things - do not remove this or risk form bot signups-->
    <div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text" name="b_85f552eee256467fd023266f3_af3bc86a4e" tabindex="-1" value=""></div>
    <div class="clear"><input type="submit" value="Subscribe" name="subscribe" id="mc-embedded-subscribe" class="button"></div>
    </div>
</form>
</div>

<!--End mc_embed_signup-->
<div id="disqus_thread"></div>
<script>

/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/

var disqus_config = function () {
this.page.url = https://crro.github.io/;  // Replace PAGE_URL with your page's canonical URL variable
};

(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://crros-blog.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
                            
