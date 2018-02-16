Setting up a Shopify App with Rails 5.1, Webpack, React, and Polaris
======================================================

Assumptions:

* `Rails 5.1.4` 
* `Ruby 2.4.3`
* `Postgres` [Setup on Mac](#todo)

We’ll be creating an embedded app that can use Shopify’s new [Polaris components](https://polaris.shopify.com/).

### 1. Generating a new applicaton

Let’s set up a new rails app (make sure you’re using Rails 5.1)

`rails new <APP NAME> --webpack=react -d postgresql -T`

The last two arguments are optional but I like to set up my apps with Postgres (open source, yay!) and skip the tests so that I can replace them rspec.

Add Shopify’s gem to the gem file, run bundle install, and create the database.

```
#Gemfile
gem 'shopify_app'
```

`bundle install`

`rails db:create`

### 2. Tunneling & Installing ngrok

To use some of Shopify’s features such as Application Proxy and Webhooks you’ll need to set up a tunnel to expose your local server to an actual url. I do this for all apps though because it helps avoid loading errors due to mixed content issues. Luckily ngrok makes this pretty easy. Head over to [their site](https://ngrok.com/) and download and install using your OS’s instructions. Then run the following command. (I’m setting this to tunnel from `localhost` port 5000 instead of Rail’s default. This is because foreman.)

\*\* the -subdomain argument is a premium feature but makes Shopify development MUCH easier. Highly recommend the upgrade

`ngrok http -subdomain=nathan-dev`

We’ll now be able use the “https://nathan-dev.ngrok.io” to access our app.

### 3. Shopify Partner Accounts & App Creation

Next let’s head over to [https://www.shopify.com/partners](https://www.shopify.com/partners) and login or set up an account. This will allow us to create and sell apps as well as set up development stores.

Click on the create app button in the app page of your dashboard. Use ngrok url that we generated earlier.

Now head to the app info tab and change the Whitelisted redirection URL to “https://nathan-dev.ngrok.io/auth/shopify/callback”.

Then go to the extensions tab and make sure “Embed in Shopify admin” is enabled.

Still in the dashboard, let’s go back to the overview tab and generate our app’s api credentials. 

Next, we need to configure our app to use our API access key and secret key.

With Rails 5.1 we have a great way of managing our secrets. 

`bin/rails secrets:setup`

The command will output the actions it performs and the password for your encrypted secrets file. You will want to save this password your password manager to share it with your team.

The command creates 2 files. `config/secrets.yml.key` is used to encrypt and decrypt your secrets. It is automatically added your gitignore. `config/secrets.yml.enc` is the encrypted version of you secrets.yml file. It can be added to your repository. If you don't have the key file or password, you can't decrypt it.

The generated `secrets.yml.enc` will be empty, save for a comment. To add secrets run,

`rails secrets:edit`

This will open your default editor and you can add in your secrets. If you already have secrets you can move them in from here.

We still need to tell Rails to load the secrets from the encrypted file. Add the following line to your `config/aplication.rb`

`config.read_encrypted_secrets = true`

If you want to display the secrets in the console run,

`rails runner "puts Rails::Secrets.read"`

Now add the Shopify API key and Secret key into your rails secrets file. 

```
# $rails secrets:edit
developement:
  shopify_app_api_key: <xxxxxxxx>
  shopify_app_secret_key: <xxxxxxxx>
```

> You will want to setup seperate apps in the Shopify dashboard for staging and production.

You can now access your Shopify keys like this, 
`Rails.application.secrets.shopify_app_api_key` and `Rails.application.secrets.shopify_app_secret_key`.


For more information on deploying with encrypted secrets see, [Encrypted Rails Secrets on Rails 5.1](https://www.engineyard.com/blog/encrypted-rails-secrets-on-rails-5.1).

### 4. Generating a Base App Using the Shopify Gem Installer

Now we can get our app set up quickly by running

`rails generate shopify_app`

`rails db:migrate`

> Checkout the [Shopify App](https://github.com/Shopify/shopify_app#home-controller-generator) documentation for more generators.

It will be necessary to run `rails generate shopify_app:controllers` for most apps. This will allow the modification of the Shopify controllers.

We now need to make sure that our app is using the access key and secret key that we set up earlier. In the generated file, `config/initializers/shopify_app.rb`, add in a reference to your Rails secrets. 

```
ShopifyApp.configure do |config|
   config.application_name = "Shopify Demo App"
   config.api_key = Rails.application.secrets.shopify_app_api_key
   config.secret = Rails.application.secrets.shopify_app_secret_key
   config.scope = "read_orders, read_products"
   config.embedded_app = true
   config.after_authenticate_job = false
   config.session_repository = Shop
end
```

### 5. Installing Polaris and Setting up a Basic React Component

Now that the Rails section of our app is set up let’s get React working. First we’ll install Polaris by running

If you are missing the webpacker elements for React, run,

`bundle exec rails webpacker:install`

> Yarn is a JS package manager

`yarn add @shopify/polaris`

When we set up the app with webpacker it generated some basic React components. Let’s modify one to use some Polaris components

{% raw %}
```
# app/javascript/packs/hello_react.jsx

import React from 'react'  
import ReactDOM from 'react-dom'  
import PropTypes from 'prop-types'  
import {Page, Card, Button, Thumbnail} from '@shopify/polaris';

const Hello = props => (  
   <Page title="Products">  
    {props.products.map((product, index) => (  
    <Card key={index}  
      title={product.title}  
      primaryFooterAction={{  
        content: 'View',  
        url: 'https://${shop_session.url}/admin/products/${product.id}',  
      }}  
      sectioned  
    >  
        <Thumbnail  
       source={((product.images == null) ? product.images[0].src : 'https://images-na.ssl-images-amazon.com/images/I/91JKyhY%2BtFL._SL1500_.jpg')}  
          alt={product.title}  
          size="large"  
        />  
        
    </Card>     
    ))}  
  </Page>  
)

// Render component with data  
document.addEventListener('DOMContentLoaded', () => {  
  const node = document.getElementById('hello-react')  
  const data = JSON.parse(node.getAttribute('data'))

ReactDOM.render(<Hello {...data} />, node)  
})
```
{% endraw %}

Now lets add this component to our app’s layout file by adding the following in the head tags

`<%= javascript_pack_tag 'hello_react' %>`

\* Make sure you put this in the embedded_app.html.erb file that the Shopify gem created for us and NOT `application.html.erb`. Other wise you’ll end up like me spending 15 minutes why I couldn’t get anything to show up

Then add the the component to the `views/home/index.html.erb` file

{% raw %}
```
<% content_for :javascript do %>  
  <script type="text/javascript">  
    ShopifyApp.ready(function(){  
      ShopifyApp.Bar.initialize({  
        title: "Home",  
        icon: "<%= asset_path('favicon.ico') %>"  
      });  
    });  
  </script>  
<% end %>

<%= content_tag :div,  
  id: "hello-react",  
  data: {  
    name: 'Products',  
    products: @products,  
    shop_session: @shop_session  
}.to_json do %>  
<% end %>
```
{% endraw %}

Make sure to remove this line or add a favicon in your assets folder,

icon: `"<%= asset_path('favicon.ico') %>"`

### 6. Configuring our local server with Foreman

We’re almost ready to check out our app but first let’s make our lives easier by using Foreman to start or local server. If you don’t you’ll need to run the rails and webpack servers in separate terminal tabs. This gets old pretty quickly. Luckily there’s an easy work around (you can see the original instructions [here](https://github.com/rails/webpacker#webpack-dev-server).

In terminal run,

`gem install foreman`  
`touch Procfile`

In the Procfile add the following lines,

```
web: bundle exec rails s  
webpacker: ./bin/webpack-dev-server
```

This is also a good time to head over to config/webpacker.yml and make sure that the dev_server is set to use https. (This will help us avoid loading and content errors)

```
dev_server:  
    host: 0.0.0.0  
    port: 8080  
    https: true
```

now run,

`foreman start`

> If the server won't run and the issue is not being output. It is likely a compile issue with your React code and running webpacker alone to view errors. 

> `./bin/webpack-dev-server`


### 7. Check Out Your New App on a Development Store

Make sure you have a development store set up and head to your ngrok url.

You’ll be prompted to install your app by entering your shop’s domain like this.

<STORE NAME>.myshopify.com

Once it’s installed you should be able to see a list of products and our React button.

Not much of an app yet but hopefully this set up will let you go out and create something great! I plan on adding a second part to this walk through to show give some more in depth React examples.
