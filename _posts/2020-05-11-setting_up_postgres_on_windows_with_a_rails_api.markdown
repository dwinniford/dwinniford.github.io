---
layout: post
title:      "Setting Up Postgres on Windows with a Rails API"
date:       2020-05-11 19:17:39 +0000
permalink:  setting_up_postgres_on_windows_with_a_rails_api
---


I recently tried to deploy an app on heruko and ran into the problem of switching my database to postgres.  It was a bit complicated and I didn't find all the answers I needed in any single place.  I thought it would be worth it to organize the steps for myself and anyone else who is in need of help.  

## Dave's successful postgres setup:

### 1.  Installed postgres 11.7 for windows 10 64-bit from this[ tutorial ](http://www.postgresqltutorial.com/)
* Note that this is not the latest version of postgres.  I chose this version because it was closer to the version used in the tutorial.
* Remember the password and username from installation. 

### 2. 	Created rails api with postgres with this [command](http://medium.com/@ethanryan/creating-a-new-rails-api-with-a-postgresql-database-488ffce649d9)
```
rails new project-name-here --api --database=postgresql
```
* I still ran into errors similar to [this](http://medium.com/@kevinmircovich/pg-connectionbad-could-not-connect-to-server-no-such-file-or-directory-da18a3764a0a) when I tried to migrate.  Below is the solution I found.

### 3.  Changed the config/database.yml file to look like this: 

```
		default: &default
		  host: localhost # manually added
			adapter: postgresql
			encoding: unicode
			username: (username created on installation, default is postgres) # manually added
			password: (password created on installation) # manually added
			pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
```

* Note that the Rails guide[instructions](http://guides.rubyonrails.org/configuring.html#configuring-a-postgresql-database) on configuring were not enough to make it started working on my computer.

### 4. Rails g resource … 
* Created a model to make sure the database was working

### 5. 	Rails db:create
* An obvious but easy to [forget](http://stackoverflow.com/questions/28404482/rails-fatal-database-myapp-development-does-not-exist/28404606) step.

### 6. Rails db:migrate


Yay!  No errors and everything is working now!!! I hope that worked for you too and you can go on to coding your app!! 


