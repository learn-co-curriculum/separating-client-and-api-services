# Seperating Client And API Services

## Objectives

* Learn the benefits of creating seperate applications using an API layer and Client App Layer
* Walk through building a simple API/Client layer app
* Discuss how CORS works and how to use Rack CORS Gem

## Introduction: 

// Walkthrough

- Explain the benefits of creating seperate applications using an API Layer and Cient App Layer
- Walk them through some very very very simple version of separate API/Client Layer. Maybe just have students run their rails server with rails s. And then server an index.html file from ruby -rwebrick -e'WEBrick::HTTPServer.new(:Port => 8000, :DocumentRoot => Dir.pwd).start' .Then put a basic fetch request in there. Make sure they see the error
-  Explain what CORS is. (Show them the error! in a random Inspect Element instance)
- Walkthrough of how to Rack-Cors gem (https://github.com/cyu/rack-cors) Built into Rails 5 API mode
- Just show them * and localhost:3000

### Summary

// Go Over What We Learned In This Readme

<p class='util--hide'>View <a href='https://learn.co/lessons/seperating-client-and-api-services'>Sepearting Client and API Services</a> on Learn.co and start learning to code for free.</p>