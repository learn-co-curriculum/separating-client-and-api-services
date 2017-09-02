# Separating Client And API Services

## Objectives

* Learn the benefits of creating separate applications using an API layer and Client App Layer
* Walk through building a simple API/Client layer app
* Discuss how CORS works and how to use Rack CORS Gem

## Introduction: 

In the last lesson we built out the beginning of our Iron Starter API. We've moved that app to this lesson, so we should be able to run:

```bash 
bundle install
rails db:migrate 
rails server 
``` 

To run the application on port 3000. 

In this lesson we are going to create a small client application that connects to our API. 

### Client APP 

First let's create a simple index.html file in the root of our directory:

```bash 
touch index.html
```

and add some basic html:

```html 
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Simple Client App</title>
</head>

<body>

  <h1>Fetch All The Things</h1>
  <h4>Create Campaign</h4>
  <form id="campaign-form">
    <div>
        <div>
            <label for="title">Title: </label>
        </div>
        <input type="text" placeholder="title" name="title" />
    </div>
    <div>
        <div>
            <label for="description">Description: </label>
        </div>
        <textarea placeholder="description" name="description"></textarea>
    </div>
    <div>
        <div>
            <label for="goal">Goal: </label>
        </div>
        <input type="number" placeholder="goal" name="goal" />
    </div>
    <div>
        <div>
            <label for="pledged">Pledged: </label>
        </div>
        <input type="number" placeholder="pledged" name="pledged" />
    </div>
    <div>
        <input type="submit" value="Add Campaign" />
    </div>
  </form>
  <h3>Campaigns:</h3>
  <ol id="campaigns-list">
  </ol>
</body>
</html>
```

We can check out this code by running a mock ruby server on port 8000.

```bash 
ruby -rwebrick -e'WEBrick::HTTPServer.new(:Port => 8000, :DocumentRoot => Dir.pwd).start'
```

If we open the browser and go to localhost:8000 we should see something like this

![client app image 1](https://s3.amazonaws.com/learn-verified/basic-client-app-image-1-react-and-rails.png)

Kind of bland at the moment, but lets add some code that allows ut to make a __fetch__ request to our rails api to get a list of campaigns and render them to the DOM. We should also add POST __fetch__ request to create a new campaign.

let's add the following code inside of the `<script></script>` tags in the `index.html` file

```javascript 
<script>
    (function() {

        document.querySelector('#campaign-form').addEventListener('submit', function(event) { 
            event.preventDefault();
            const form = this;
            const campaignParams = {
                title: getValue('input', 'title'),
                description: getValue('textarea', 'description'),
                goal: getValue('input', 'goal'),
                pledged: getValue('input', 'pledged')
            };
            return fetch('http://localhost:3000/api/campaigns', { 
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    campaign: campaignParams
                })
            })
            .then(response => response.json())
            .then(campaign => renderCampaign(campaign));
        });

        const getValue = (inputType, name) => {
            return document.querySelector(`${inputType}[name="${name}"]`).value;
        }

        const renderCampaign = campaign => {
            const orderedCampaignsListElement = document.querySelector('#campaigns-list');
            const li = document.createElement('li');
            li.innerHTML = `
                <h3>${campaign.title}</h3>
                <h4>Goal: $${campaign.goal}</h4>
                <h4>Pledged: $${campaign.pledged}</h4>
                <p>${campaign.description}</p>
            `
            orderedCampaignsListElement.appendChild(li);
        }

        fetch('http://localhost:3000/api/campaigns')
            .then(response => response.json())
            .then(campaigns => {
                campaigns.forEach(campaign => renderCampaign(campaign));
            });
    }());
</script>
```

If we refresh the browser we should see the list of campaigns rendered neatly inside of our __<ol>__ tag (depending on the browser in use the campaigns might not render). let's open up our dev tools console and try to make a campaign with the form. Once we click submit we should be able to create the campaign.

Oh wait!! nothing was added to the list. If we check out the dev console we should see an error like this. 

![basic client app image 2 console errors](https://s3.amazonaws.com/learn-verified/basic-client-app-image-2-react-and-rails.png)

So what does Cross-Origin Request Blocked: The Same Origin Policy disallows reading the remote resource at http://localhost:3000/api/campaigns. (Reason: CORS header ‘Access-Control-Allow-Origin’ missing) mean?

### CORS (Cross-Origin Resource Sharing) 101

So we've uncovered a new error and now is a good time to grab and seat and buckle up as we talk about Cross-Origin Resource Sharing(CORS). CORS is a mechanism in place that gives web servers cross-domain access controls, so that it can secure who is able to make data transfers. We probably don't want just anyone to access our application, CORS allows us to lock down which Domains can access our app. More information can be found at [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS).

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

Let's restart our __rails server__ (since we changed an initializer we need to restart for this to run correctly) and run the app in the browser one more time and try it out. This is what the app should look like now.

![Successful API Request](https://s3.amazonaws.com/learn-verified/basic-client-app-image-3-react-and-rails.png)

### Summary

In this lesson we learned how to setup a basic Ruby server and run a small client side app that makes a fetch request to our Iron Starter API. We also ran into an issue with CORS, and used the out of the box system that Rails 5 API Template provides for us.

#### Resources 

[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)
[Rack CORS Middleware GEM](https://github.com/cyu/rack-cors)
[What is CORS?](https://www.maxcdn.com/one/visual-glossary/cors/)

<p class='util--hide'>View <a href='https://learn.co/lessons/separating-client-and-api-services'>Separating Client and API Services</a> on Learn.co and start learning to code for free.</p>
