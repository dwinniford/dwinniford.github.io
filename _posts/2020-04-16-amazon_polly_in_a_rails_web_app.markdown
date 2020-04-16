---
layout: post
title:      "Amazon Polly in a Rails Web App"
date:       2020-04-16 19:53:49 +0000
permalink:  amazon_polly_in_a_rails_web_app
---


I built a language learning related web app and wanted to incorparate an AWS text-to-speech functionality to allow learners to hear how to pronounce specific words.  

Unfortunately, I was not able to find a lot of documentation specific to Rails. (I spent a lot of time looking at an [example](http://https://docs.aws.amazon.com/polly/latest/dg/examples-python.html) in python.)  After struggling with the code for a while I arrived at a somewhat imperfect solution.  Hopefully it will be improved soon!

Here's what I learned:

Configure your app with aws according the documentation [here](http://https://docs.aws.amazon.com/sdk-for-ruby/v3/developer-guide/setup-config.html).  (This part was well documented for ruby and very simple.)

In my web app I want users to be able to click a button next to a chinese word and hear it played back. 

Here's my html:

```
<h1 lang="zh"><%= @restaurant.chinese_name %> </h1>
            <form id="input" method="GET" action="/read">
                <%= hidden_field_tag(:text, @restaurant.chinese_name) %>
                <input type="submit" value="Listen" id="submit" />
            </form>

<audio id="player"></audio>
```

To do this I created a form with a hidden input set to the value of the text being displayed to the user.  

Here's the javascript I used to submit the form:

```
<script>
        document.addEventListener("DOMContentLoaded", function () {
            var input = document.getElementById('input'),
                text = document.getElementById('text'),
                player = document.getElementById('player'),
                submit = document.getElementById('submit');
                
            input.addEventListener('submit', function (event) {
            
                player.src = '/read?text=' + encodeURIComponent(text.value);
                player.play();
            
                event.preventDefault();
            });
        });
    </script>
```

This request is routed to the PollyController with a corresponding method:

```
def read 
        polly = Aws::Polly::Client.new

        resp = polly.synthesize_speech({
            output_format: "mp3",
            text: params[:text],
            voice_id: "Zhiyu",
        })
      
        IO.copy_stream(resp.audio_stream, "#{params[:text]}.mp3") 
        send_file "#{params[:text]}.mp3
    end
```

The above code saves a copy of the response to an mp3 file in the app directory and then sends it to the audio element in the browser.  This is far from perfect.  I would like to create an mp3 file without saving it and send it directly to the browser.

### Taking a look at Amazon Polly:

`Aws::Polly::Client.new` creates and API client for Amazon Polly([see docs](http://https://docs.aws.amazon.com/sdk-for-ruby/v2/api/Aws/Polly/Client.html)).  Note that this requires a region and credentials to be configured.

I used a `.env` file to configure this:

```
AWS_ACCESS_KEY_ID =
AWS_SECRET_ACCESS_KEY=
AWS_REGION= "us-east-2"
```

 `polly.synthesize_speech(options)` synthesizes text input to a stream of bytes.  This object in turn responds to `.audio_stream` returning an IO object.  This is where I ran into significant problems.  I could not find a way to send this IO object to the browser.  So in the end I settled with saving it as an mp3 first.

### More Functionality

My web app only required very minimal functionality from Amazon Polly.  However Amazon Polly will allow you to choose from multiple voices and audio formats.  You can easily change them through the options hash in`synthesize_speech`.


### Summing it up

Amazon Polly was very simple to configure and start using.  However, integrating it into Rails for my specific use is presenting some challenges!



