---
layout: post
title:  "Using Bootstrap with React"
date:   2022-05-10 16:39:01 -0400
categories: jekyll update
---

Bootstrap is a toolkit combining Javascript, CSS, and other tools to create a quick and easy way to get a modern-looking website. Using vanilla Javascript and CSS, you can already access the full scope of Bootstrap's functionality; however, for React there is also a react-bootstrap library available which recasts Bootstrap's functionality in the form of React components. This blogpost will go through the differences in how vanilla and React Bootstrap are used, and some of the advantages React Bootstrap can offer in a React app.

## Vanilla bootstrap example

Vanilla bootstrap is made available via a CDN, a web resource your page can link to. It requires two steps in your HTML file to make the full functionality of bootstrap available; first, the CSS has to be loaded via a <link> tag in the <head>:

{% highlight html %}
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3" crossorigin="anonymous">
{% endhighlight %}

Then, the Javascript has to be inserted via a <script> at the very end of the page:

{% highlight html %}
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-ka7Sk0Gln4gmtz2MlQnikT1wXgYsOg+OMhuP+IlRH9sENBO0LRn5q+8nbTov4+1p" crossorigin="anonymous"></script>
{% endhighlight %}

With Bootstrap loaded, html elements can be given certain class names to take advantage of Bootstrap's styling. Here's an example of what some HTML using Bootstrap might look like:
{% highlight html %}
<div class="container">
  <div class="row">
    <div class="col">
      <div class="card">
        <img src="..." class="card-img-top"/>
        <div class="card-body">
          <p class="card-text">Text</p>
        </div>
      </div>
    </div>
  </div>
</div>
{% endhighlight %}

That's a whole lot of <div> tags, and though we know we're using Bootstrap, the vague class names make it hard to know whether this styling is coming from Bootstrap or our own CSS. All this was possible with either completely vanilla Javascript, or within a React app; however, if we are using React, we have another option in react-bootstrap.

## React Bootstrap example

To begin using react-bootstrap, it needs to be installed via npm with `npm install react-bootstrap`. Then, we can add two imports at the top of the component we would like to use Bootstrap within:

{% highlight javascript %}
import "bootstrap/dist/css/bootstrap.min.css";
import { Container, Row, Col, Card } from "react-bootstrap";
{% endhighlight %}

The first import imports the Bootstrap CSS, like we did from the CDN in the vanilla Javascript example. The second imports react-bootstrap's React components that pre-package the functionality we accessed before via class names. Here's the earlier HTML converted to React JSX, and using react-bootstrap components instead of classes:

{% highlight html %}
<Container>
  <Row>
    <Col>
      <Card>
        <Card.Img src="..."/>
        <Card.Body>
          <Card.Text>Text</Card.Text>
        </Card.Body>
      </Card>
    </Col>
  </Row>
</Container>
{% endhighlight %}

This has no functional difference from the vanilla Javascript example, but in my opinion it is much easier to read. Because every HTML node here is a React component that we have explicitly imported from react-bootstrap, there's no ambiguity about where things are coming from; just look at the import statements if you're not sure.

Another way react-bootstrap can make your code more clear than the vanilla Javascript alternative is through props. In vanilla Bootstrap, variants and optional parameters to a component are indicated through further class names:

`<div class="card bg-dark text-white">`

This can be unclear for the same reasons the main component is unclear; not only are you not sure whether "bg-dark" is something coming from bootstrap or your own CSS, you also don't know how it relates to the primary class "card". (And some Bootstrap classes do actually change depending on where they're applied!)

Here's what that would look like with react-bootstrap:

`<Card className="bg-dark text-white">`

Now at least it is clear that the most important thing about this component is that it is a Card, and the classes applied will depend on that basic framework.
