# Rails_Devise_Mongodb_setup
How to setup rails with devise and mongodb with token_auth and omni_auth

## Initialize the Rails App

```
rails new myapp --skip-active-record
```
Use the GemFile included with this app.

```
bundle install
rails g mongoid:config
rails generate devise:install
```

##Changes to be made in config/initializers/devise.rb

Navigational Formats -> this basically applies to which type of request will be sent to the sign in or sign up paths.
any path not mentioned here will respond with a 401.

```
config.navigational_formats = [:"*/*", :html, :json, :js]
```

###Omniauth


the omniauth details for google are to be put here:
How to get the client-id and the google-serret.

Go to Google Developer console ->

Create A New Project ->

Now Click on Google-Plus-Api, GMail-Api and any other Api that you want, when you go to the Api, click enable ->

Enable as many as you want ->

Now Click on Credentials in the side-bar ->

Now Click on OAuth-Consent-Screen. ->

Now just enter a name for the project (GitHub-Documentation) ->

Now click on "Create Credentials" ->

Now click on "OAuth-whatever" ->

Now in the popup box, fill in the __Authorized Redirect Urls__ with the redirect callback url.

For localhost using Devise this is __http://localhost:3000/users/auth/google_oauth2/callback__

Then click create->

It will then provide you with the client id, and client secret, go to the devise.rb configuration file and under the commented out omniauth section, add the following:


```
config.omniauth :google_oauth2, 'google client id', 'google secret', {
      :scope => "email, profile",
      :prompt => "select_account",
      :image_aspect_ratio => "square",
      :image_size => 50
  }
```

A similar procedure has to be followed for any oauth providers that you wish to support.

### Custom Failure Strategy for failed sign-ins in devise

Paste the following under the relevant section in devise.rb configuration:

```
config.warden do |manager|
    manager.failure_app = CustomFailure
end
```

Then in /lib folder in the rails app, create a file called __custom_failure.rb__
In this file use the following structure:

```
class CustomFailure < Devise::FailureApp
  def redirect_url
  	 puts "the request referrer is: #{request.referrer}"
  	 puts "the flash request type is"
  	 puts flash[:request_type]
	 
     flash[:failure_message] = i18n_message
  	 
     if flash[:request_type] == "oauth"
  		oauth_sign_in_failed_users_path 	
     else
  	 	sign_in_failed_users_path
     end
     
  end

  # You need to override respond to eliminate recall
  def respond
    redirect
  end


end
```

Some points to note about what is happening here:
1. It has to extend Devise::FailureAPp
2. It has to override response, and simply return "redirect"
3. THe redirect method basically redirects by calling the "redirect_url" method, which you override to have custom behaviour
4. Here I use the flash to access information about what kind of request has failed and redirect based on that.

Finally you need to add the following line application.rb , to make it load the lib folder into the asset pipeline.

```
config.autoload_paths << Rails.root.join('lib')
```

===

Now the next steps configure a User Model ,controllers and views for the same.

```
rails g devise User
```

Go and add :omniauthable and :confirmable modules to the user module.

Then continue

```
rails g devise:views user
rails g devise:controllers user
```

Now go and change the routes file so that the controllers point to the views in the correct users subfolders.

```
devise_for :users, :controllers => { :omniauth_callbacks => "users/omniauth_callbacks" , :registrations => "users/registrations", :confirmations => "users/confirmations", :passwords => "users/passwords", :sessions => "users/sessions"}
```

delete the _devise_ subfolder from the views folder because it will have a _users_ subfolder already 

Run rails s , and see that sign_in, sign_up, omniauth and other things are working, before proceeding.
