---
layout: post
title:      "ActiveRecord Validations"
date:       2020-03-09 22:07:39 +0000
permalink:  activerecord_validations
---


If you're like me you're prone to diving into making code from scratch to solve a problem that already has prebuilt solutions available.  

Recently I build an app with Sinatra and setup some basic validation for my database.  Unfortunately I did not think about using ActiveRecord validations until I had written all of my own validation methods in my ApplicationController class!    

Although this was kind of a waste of time, it did make me appreciate what ActiveRecord can do for me!  Here's a little comparison of my made-from-scratch-validation and my revised ActiveRecord validation.

## MadeFromScratch

```
       def valid_name?(str)
         
            str.downcase.chars.none? { |c| !c.match?(/\A[ a-z]+\z/) }
        end

        def valid_language?(str)
            str.downcase.chars.none? { |c| !c.match?(/\A[ a-z]+\z/) }
            #separate method in case I want to make the validation more specific
        end

        def valid_location?(hash)
            hash.none? do |k, v|
                v.downcase.chars.any? { |c| !c.match?(/\A[a-z]+\z/) }
            end
        end

        def valid_input?
            valid_age?(params["survey"]["age"].to_i) && valid_name?(params["survey"]["name"]) && valid_language?   (params["survey"]["first_language"]) && valid_language?(params["survey"]["second_language"]) && valid_location?(params["location"])
        end
				
```

## ActiveRecord

```
class Survey < ActiveRecord::Base 
    belongs_to :user 
    belongs_to :location 
    validates :name, :gender, :age, :first_language, :second_language, presence: true 
    validates :name, :first_language, :second_language, format: { with: /\A[ a-zA-Z]+\z/, message: "only allows letters and spaces"}
    validates :age, numericality: { only_integer: true, greater_than: 0, less_than: 115 }
    validates_associated :location
end 
```

### a little analysis...

First of all, the ActiveRecord version is a lot nicer to look at!  (15 lines of code compared to 4)

In order to make my homemade version look nice in the controller (`valid_input?`), I had to combine a whole bunch of evaluations into one line...
```
 valid_age?(params["survey"]["age"].to_i) && valid_name?(params["survey"]["name"]) && valid_language?   (params["survey"]["first_language"]) && valid_language?(params["survey"]["second_language"]) && valid_location?(params["location"])
```

That didn't even feel right before I knew about ActiveRecord validations.

Also, once I switched to using ActiveRecord validations I got access to a whole bunch of validation messages as a bonus!  How does that work???

```
@survey = Survey.new(params["survey"])
@survey.build_location(params["location"])
 if @survey.save 
     redirect "/surveys/#{@survey.id}"
 else 
     erb :"surveys/new"
 end 
```

When I called `@survey.save` it also called the validation methods on my Survey object AND my Location object(thanks to this little line of code: `validates_associated :location`).  If any of the validations did not pass `@survey.save` will return false BUT `@survey` will also include a new array of error messages.

Now I can easily display these errors to the user:

```
<% @survey.errors.full_messages.each do |m| %>
        <p><%= m %></p>
    <% end %>
    <% @survey.location.errors.full_messages.each do |m| %>
        <p><%= m %></p>
    <% end %>
```

ActiveRecord made this functionality so much more enjoyable to build!  If you're working on your own validation I highly recommend diving into the [ActiveRecord documentation](http://guides.rubyonrails.org/active_record_validations.html)  on this subject.  It was extremely helpful for me!


