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




```
config.omniauth :google_oauth2, 'google client id', 'google secret', {
      :scope => "email, profile",
      :prompt => "select_account",
      :image_aspect_ratio => "square",
      :image_size => 50
  }
```
