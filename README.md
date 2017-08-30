# Seperating Client And API Services

## Objectives

* Learn the benefits of creating seperate applications using an API layer and Client App Layer
* Walk through building a simple API/Client layer app
* Discuss how CORS works and how to use Rack CORS Gem

## Introduction: 

In the last lesson we built out the beginning of our Iron Starter API. We've moved that app to this lesson, so you should be able to run:

```bash 
bundle install
rails db:migrate 
rails server 
``` 

To run the application on port 3000. 

In this lesson we are going to create a small client application that connects to our API. 

### Client APP 

Lets create a simple index.html file in the root of our directory 

```bash 
touch index.html
```

and add some basic html

```html 
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Simple Client App</title>
</head>

<body>

  <h1>Fetch All The Things</h1>

  <h3>Campaigns:</h3>
  <ol id="campaigns-list">
  </ol>
  <script>
  
  </script>
</body>
</html>
```

We can check out this code by running a mock ruby server on port 8000

```bash 
ruby -rwebrick -e'WEBrick::HTTPServer.new(:Port => 8000, :DocumentRoot => Dir.pwd).start'
```

Please use Safari or Firefox browser, as chrome might have security disabled for GET requests (We will talk about this later in the lesson). 

If you open your browser and go to localhost:8000 you should see something like this

![basic client app image 1](https://s3.amazonaws.com/learn-verified/basic-client-app-image-1-react-and-rails.png)

Kind of bland at the moment, but lets make a __fetch__ request to our rails api to get a list of campaigns and render them to the DOM. 

let's add the following code inside of the `<script></script>` tags in the `index.html` file

```javascript 
<script>
    (function() {
        fetch('http://localhost:3000/api/campaigns')
            .then(response => response.json())
            .then(campaigns => {
                const orderedCampaignsListElement = document.querySelector('#campaigns-list');
                campaigns.forEach(campaign => {
                    const li = document.createElement('li');
                    li.innerHTML = `
                        <h3>${campaign.title}</h3>
                        <h4>Goal: $${campaign.goal}</h4>
                        <h4>Pledged: $${campaign.pledged}</h4>
                        <p>${campaign.description}</p>
                    `
                    orderedCampaignsListElement.appendChild(li);
                });
            });
    }());
</script>
```

If you refresh the browser you should see the list of campaigns rendered neatly inside of our __<ol>__ tag.

Oh wait!! nothing (if you see it rendering the campaigns you might have security disabled). If you open up the dev console you should see an error like this. 

![basic client app image 2 console errors](https://s3.amazonaws.com/learn-verified/basic-client-app-image-2-react-and-rails.png)

So what does Cross-Origin Request Blocked: The Same Origin Policy disallows reading the remote resource at http://localhost:3000/api/campaigns. (Reason: CORS header ‘Access-Control-Allow-Origin’ missing) mean?

### CORS (Cross-Origin Resource Sharing) 101

So we've uncovered a new error and now is a good time to grab and seat and buckle up as we talk about Cross-Origin Resource Sharing(CORS). CORS is a mechanism in place that gives web servers cross-domain access controls, so that it can secure who is able to make data transfers. We probably don't want just anyone to access our application, CORS allows us to lock down which Domains can access our app. You can find more information about it at [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS).

### Handling CORS 

So now that we know what CORS is how do we resolve this? Well conceptually we need to allow our Iron Starter API app to allow the localhost:8000 port to make GET request, and probably POST, PUT, DELETE, etc as well. Well Rails 5 API Template to the rescue. This middleware functionality is included out of the box using a gem called [Rack CORS Middleware](https://github.com/cyu/rack-cors). 

Rack CORS Middleware provides support so that our Rails app can allow CORS requests. To use it we need to uncomment it in our Gemfile:

```ruby 
# Gemfile 

# Use Rack CORS for handling Cross-Origin Resource Sharing (CORS), making cross-origin AJAX possible
gem 'rack-cors'
```

and run `bundle install`. This add the gem to our app, but we also need to setup an initializers to run our desired configuration. This is also already in our application. Go to `/config/initializers/cors.rb` an uncomment the following code and add our `localhost:8000` port as an allowed origin.

```ruby 
# /config/initializers/cors.rb
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins 'localhost:8000'  # <- IMPORTANT

    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end
```

This code allows our `localhost:8000` port to make requests using all of HTTP verbs with options and headers. 

Let's restart our __rails server__ (since we changed an initializer we need to restart for this to run correctly) and run the app in the browser one more time and try it out. You should see something like this:

![Successful API Request](https://s3.amazonaws.com/learn-verified/basic-client-app-image-3-react-and-rails.png)

### Summary

In this lesson we learned how to setup a basic Ruby server and run a small client side app that makes a fetch request to our Iron Starter API. We also ran into an issue with CORS, and used the out of the box system that Rails 5 API Template provides for us.

#### Resources 

[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)
[Rack CORS Middleware GEM](https://github.com/cyu/rack-cors)
[What is CORS?](https://www.maxcdn.com/one/visual-glossary/cors/)

<p class='util--hide'>View <a href='https://learn.co/lessons/seperating-client-and-api-services'>Sepearting Client and API Services</a> on Learn.co and start learning to code for free.</p>