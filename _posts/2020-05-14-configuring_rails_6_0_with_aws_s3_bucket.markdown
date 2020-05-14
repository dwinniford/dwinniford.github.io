---
layout: post
title:      "Configuring Rails 6.0 with AWS S3 Bucket"
date:       2020-05-14 20:02:43 +0000
permalink:  configuring_rails_6_0_with_aws_s3_bucket
---


Configuring Rails with an aws-sdk didn't work the same for me as it did a few months ago.  So here is an up-to-date guide on getting setup.

## 1.  Install ActiveStorage
ActiveStorage connects your database records to the S3 bucket.  Here is a good [guide](http://pragmaticstudio.com/tutorials/using-active-storage-in-rails) on getting this part set up.  This is also a good read to understand what ActiveStorage does.

Here's the summary:
```
rails active_storage:install
```
```
rails db:migrate
```
Declare attachment associations in your model like so:
```
has_one_attached :file_name
```


ActiveStorage provides some useful methods for handling the attachement in the controller.

For example, to check if an object has an attached file:
```
@object.file_name.attached?
```
Or to save an attachement to an object:
```
@object.audio_file.attach(params[:file_name])
```
## 2.  Install the aws-sdk gem
add `gem 'aws-sdk-s3', require: false` to your gem file and run `bundle install`
## 3.  Add your credentials (to the right place!!!)
Check out your` config/storage.yml` file.

Rails provides this helpful comment that could have saved me a lot of time if I actually read it/understood what it meant the first time around:

```
# Use rails credentials:edit to set the AWS secrets (as aws:access_key_id|secret_access_key)
# amazon:
#   service: S3
#   access_key_id: <%= Rails.application.credentials.dig(:aws, :access_key_id) %>
#   secret_access_key: <%= Rails.application.credentials.dig(:aws, :secret_access_key) %>
#   region: us-east-1
#   bucket: your_own_bucket
```

Uncomment this code and add your region and bucket name.

```
amazon:
  service: S3
  access_key_id: <%= Rails.application.credentials.dig(:aws, :access_key_id) %>
  secret_access_key: <%= Rails.application.credentials.dig(:aws, :secret_access_key) %>
  bucket: "bucket_name"
  region: 'us-east-2'
```

My first attempt at setting credentials that wasted me a lot of time and involved setting ENV varaibles in a `.env` file and accessing them like this:

```
amazon:
  service: S3
  access_key_id: ENV['AWS_ACCESS_KEY_ID']
  secret_access_key: ENV['AWS_SECRET_ACCESS_KEY']
  bucket: "bucket_name"
  region: 'us-east-2'
```

That definitely did not work... 

```
Aws::S3::Errors::InvalidAccessKeyId (The AWS Access Key Id you provided does not exist in our records.):
```

Moving on, I found out Rails now provides a way to store credentials in an encrypted file.  Read the[ rails edge guide](http://edgeguides.rubyonrails.org/security.html#custom-credentials) and check out this little [explanation](http://medium.com/@kirill_shevch/encrypted-secrets-credentials-in-rails-6-rails-5-1-5-2-f470accd62fc).

Your app should already have this file: `config/credentials.yml.enc`

To edit its contents you can run these commands in the terminal:

```
rails secret
```
None of the blogs I read included the above step but it was in the [edge guide.](http://edgeguides.rubyonrails.org/security.html#custom-credentials)
```
rails credentials:edit
```
(That's straight from the Rails comments in storage.yml!)

This [article](http://www.viget.com/articles/storing-secret-credentials-in-rails-5-2-and-up/) gives a more in depth explanation of how Rails handles credentials and suggests a more refined command to edit the credentials:
```
EDITOR="code --wait" bin/rails credentials:edit
```

"code" tells the terminal to open the file with my preferred text editor, vscode.  "--wait" tells it not to save changes immediately.

When I finally got to this step and opened this file I discovered that my AWS credentials were already there!! But the app still wasn't working.  My guess is Rails imported the credentials from my `.env` file(as suggested in the [edge guide](http://edgeguides.rubyonrails.org/security.html#custom-credentials)) but didn't get the formatting right.

In the end I found that my problem was a simple yaml format issue and the file needed to look exactly like this [example](http://medium.com/@kirill_shevch/encrypted-secrets-credentials-in-rails-6-rails-5-1-5-2-f470accd62fc):

```
aws:
  access_key_id: 123
  secret_access_key: 345
	
secret_key_base: xxx
```

Before there was some wierd indentation that somehow prevented the credentials from showing up.  Something like this...

```
            secret_key_base: xxx

aws:
  access_key_id: 123
  secret_access_key: 345
```

Finally working!

