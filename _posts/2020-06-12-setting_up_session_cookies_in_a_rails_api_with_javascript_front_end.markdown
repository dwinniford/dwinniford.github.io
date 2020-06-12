---
layout: post
title:      "Setting up Session Cookies in a Rails API with Javascript front end "
date:       2020-06-12 13:38:51 +0000
permalink:  setting_up_session_cookies_in_a_rails_api_with_javascript_front_end
---


When recently building a Rails API/Javascript app i discovered that session cookies are not built into the Rails API controller.  I found this [guide](http://pragmaticstudio.com/tutorials/rails-session-cookies-for-api-authentication) very helpful in solving the problem but it did not contain all of the details I needed to get this functionality working.

Below are the exact steps that worked for me:

## 1.  Add necessary session and cookie middleware into the app.
  
In `config/application.rb`

```
module AppName
  class Application < Rails::Application
    # Initialize configuration defaults for originally generated Rails version.
    config.load_defaults 6.0
    # Settings in config/environments/* take precedence over those specified here.
    # Application configuration can go into files in config/initializers
    # -- all .rb files in that directory are automatically loaded after loading
    # the framework and any gems in your application.
    # Only loads a smaller set of middleware suitable for API only apps.
    # Middleware like session, flash, cookies can be added back manually.
    # Skip views, helpers and assets when generating a new resource.
    config.api_only = true   
    
    # add the 2 lines below
    config.middleware.use ActionDispatch::Cookies
    config.middleware.use ActionDispatch::Session::CookieStore
  end 
end

```

When looking for a solution to this problem I saw several suggestions that involved adding new files to the initializers folder to add in the above middleware.  This is not necessary.

## 2.  Setup Rack Cors

In your gem file don't forget to uncomment `gem 'rack-cors'` and run `gem install rack-cors`.

In `initializers/cors.rb` add the first line below and adjust the rest accordingly.  `credentials: true` works in conjunction with the corresponding fetch config object setting.

```
Rails.application.config.action_controller.forgery_protection_origin_check = false
 
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins 'localhost:8080'
    
    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head], 
      credentials: true 
  end
end

```
## 3.  Setup the Application Controller

The solution that worked for me involved having the app send an initial GET request on page load to obtain the CSRF-TOKEN.  The token will be needed to make the login request.

```
class ApplicationController < ActionController::API
    before_action :set_csrf_cookie
    include ActionController::Cookies
    include ActionController::RequestForgeryProtection
  
    protect_from_forgery with: :exception 
		
    def cookie 
        "ok"
    end
		
    private 
		
    def set_csrf_cookie
        cookies["CSRF-TOKEN"] = form_authenticity_token
    end
end
```

In `routes.rb` add 

```
root to: "application#cookie"
```

## 4. Set up the front end to make a get request for the cookie when the page loads.

```
fetch(BACKEND_URL, {credentials: 'include'})
```

The browser should now have a `_session_id` and `CSRF-TOKEN`.

To send the CSRF-TOKEN back with future fetch requests add this function to the global scope (or wherever you need it accessible from).

```
function getCSRFToken() {
    return unescape(document.cookie.split('=')[1])
  }

```

## 5. Configure your requests.

The config object for login needs to look like this:

```
const configObject = {
                method: "POST",
                credentials: 'include',
                headers: {
                    'X-CSRF-Token': getCSRFToken(),
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(data)
            }

```

At minimum every non GET request needs to include this in the config object:

```
{credentials: 'include', headers: {'X-CSRF-Token': getCSRFToken()}}
```

Now nothing special is needed to set the user id in the session controller:

```
session[:user_id] = @user.id 
```

Everything works now!!!


