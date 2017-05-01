## heroku-addon

[https://addons.heroku.com/transloadit](https://addons.heroku.com/transloadit)

A **Heroku addon** for [Transloadit](https://transloadit.com)'s file uploading and encoding service

## Intro

[Transloadit](https://transloadit.com) is a service that helps you handle file uploads, resize, crop and watermark your images, make GIFs, transcode your videos, extract thumbnails, generate audio waveforms, and so much more. In short, [Transloadit](https://transloadit.com) is the Swiss Army Knife for your files.

[this](https://addons.heroku.com/transloadit) is an [add-on](https://addons.heroku.com) to make it easy to talk to the [Transloadit](https://transloadit.com) REST API.

![Transloadit Uploads](https://s3.amazonaws.com/heroku.devcenter/heroku_assets/images/235-original.jpg "Upload any file with Transloadit")

Dealing with hundreds of different media formats and running a scalable architecture
that can encode even the biggest files swiftly is no joke. Transloadit has spent
the last 4 years perfecting this and abstracting all this complexity into one
beatifully flexible and easy to use API.

## Install

Transloadit can be attached to a Heroku application via the CLI:

```bash
$ heroku addons:add transloadit
-----> Adding transloadit to sharp-mountain-4005... done, v18 (free)
```

<div class="callout">
A list of all plans available can be found <a href="https://addons.heroku.com/transloadit">here</a>.
</div>

Once Transloadit has been added the `TRANSLOADIT_AUTH_KEY` and `TRANSLOADIT_SECRET_KEY` settings will be available in the app configuration and will contain the credentials needed to authenticate to the [Transloadit API](https://transloadit.com/docs/api-docs). This can be confirmed using the `heroku config:get` command.

```bash
$ heroku config:get TRANSLOADIT_AUTH_KEY
4bba21cf6d744fd1aeef0f0b72ec3212
```

## Usage

With your `TRANSLOADIT_AUTH_KEY` and `TRANSLOADIT_SECRET_KEY` in place you can now integrate it with the SDK corresponding to 
your project. Here's a list of our SDKs:

- [Go](https://github.com/transloadit/go-sdk)
- [Java](https://github.com/transloadit/java-sdk)
- [Node.js](https://github.com/transloadit/node-sdk)
- [PHP](https://github.com/transloadit/php-sdk)
- [Rails](https://github.com/transloadit/rails-sdk)
- [Ruby](https://github.com/transloadit/ruby-sdk)

### Using with Ruby <sub>*(Uses our Ruby SDK)*</sub>

Verify that the `TRANSLOADIT_AUTH_KEY` and `TRANSLOADIT_SECRET_KEY` variables are set.

Ruby applications need to add the following entry into their `Gemfile` specifying the Transloadit client library.

```ruby
gem 'transloadit'
```

Then update application dependencies with bundler.

```bash
$ bundle install
```

Finally re-deploy your application.

```bash
$ git add .
$ git commit -a -m "add transloadit instrumentation"
$ git push heroku master
```

#### First encoding job

After installing the `transloadit` gem and deploying your app you can start talking to
the [Transloadit API](https://transloadit.com/docs/api-docs):


```ruby
require 'transloadit'

puts "Resizing lolcat.jpg on #{ENV['TRANSLOADIT_URL']}"

transloadit = Transloadit.new(
  :service => ENV['TRANSLOADIT_URL'],
  :key     => ENV['TRANSLOADIT_AUTH_KEY'],
  :secret  => ENV['TRANSLOADIT_SECRET_KEY']
)

resize = transloadit.step 'resize', '/image/resize',
  :width  => 320,
  :height => 240

assembly = transloadit.assembly(
  :steps => [ resize ]
)

response = assembly.submit! open('lolcat.jpg')

# loop until processing is finished
until response.finished?
  sleep 1; response.reload! # you'll want to implement a timeout in your production app
end

if response.error?
 # handle error
else
 # handle other cases
end
```

### Using with Ruby on Rails <sub>*(Uses or Rails SDK)*</sub>

Here we'll show how to use transloadit in a freshly
setup rails project and Heroku app.

If you haven't already done so, go ahead and install Rails.

```bash
$ gem install rdoc rails
```

With rails installed, let's create a new app called 'transloku'.

```bash
$ rails new transloku
$ cd transloku
```

In order to use transloadit in this app, we need to add the gem to our Gemfile
and bundle things up.

Remove `sqlite3` from your Gemfile

```bash
$ echo "ruby '2.0.0'" >> Gemfile
$ echo "gem 'transloadit-rails'" >> Gemfile
$ echo "gem 'pg'" >> Gemfile
$ bundle install
```

With that in place, it's time to generate our Transloadit configuration, as
well as a basic UploadsController and a dummy Upload model.

```bash
$ rails g transloadit:install
$ rails g controller uploads new create
$ rails g model upload
$ rake  db:migrate
```

The controller generator we just executed has probably put two GET routes into
your `config/routes.rb`. We don't want those, so lets go ahead an overwrite
them with this.

```ruby
Transloku::Application.routes.draw do
  resources :uploads
end
```

Next we need to configure our `config/transloadit.yml` file. For this tutorial,
just put in your credentials, and define an image resize step as indicated
below:

```yaml
auth:
  key     : <%= ENV['TRANSLOADIT_AUTH_KEY'] %>
  secret  : <%= ENV['TRANSLOADIT_SECRET_KEY'] %>

templates:
  image_resize:
    steps:
      resize:
        robot : '/image/resize'
        format: 'jpg'
        width : 320
        height: 200
```

Note that we encourage you to enable authentication in your Transloadit Account
and put your secret into the ```config/transloadit.yml``` to have your requests
signed.

Make your `config/database.yml` look like this:

```yaml
development:
  adapter: postgresql
  encoding: unicode
  database: transloku_development
  pool: 5
  password:

test:
  adapter: postgresql
  encoding: unicode
  database: transloku_test
  pool: 5
  password:

production:
  adapter: postgresql
  encoding: unicode
  database: transloku_production
  pool: 5
  password:
```

Alright, time to create our upload form. In order to do that, please open
`app/views/uploads/new.html.erb`, and put the following code in:

```erb
<%= javascript_include_tag '//ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js' %>

<h1>Upload an image</h1>
<%= form_for Upload.new, :html => { :id => 'upload' } do |form| %>
  <%= transloadit :image_resize %>
  <%= form.label      :file, 'File to upload' %>
  <%= form.file_field :file %>
  <%= form.submit %>
<% end %>

<%= transloadit_jquerify :upload, :wait => true %>
```

With this in place, we can modify the `app/views/uploads/create.html.erb` view
to render the uploaded and resized image:

```erb
<h1>Resized upload image</h1>
<%= image_tag params[:transloadit][:results][:resize].first[:url] %>
```

In order to use the transloadit params in your controller and views you
have to include the ParamsDecoder into your controller. Let's do that for our
UploadsController.

Open up `app/controllers/uploads_controller.rb` and adapt it like that:

```ruby
class UploadsController < ApplicationController
  include Transloadit::Rails::ParamsDecoder

  def new
  end

  def create
  end
end
```

That's it. If you've followed the steps closely, you should now be able to
try your first upload. Don't forget do start your rails server first:

```bash
$ rails server
```

Then go to http://localhost:3000/uploads/new, and upload an image. If you did
everything right, you should see the uploaded and resized file as soon as the
upload finishes.

All looking sharp? Let's publish this to Heroku

```bash
$ git init
$ git add .
$ git commit -m "init"
$ heroku login
$ heroku create
$ heroku addons:add transloadit
$ heroku config:get TRANSLOADIT_AUTH_KEY
$ git push heroku master
$ heroku run rake db:migrate
$ heroku open && heroku logs --tail
```

Point your browser to `/uploads/new`

### Using with any Language

Instead of talking server-to-server, your website visitors can directly upload
to Transloadit's specialized upload servers, so in theory there's no need for
serverside languages.

The easiest way to accomplish this would be to to include our
[jQuery SDK](https://transloadit.com/docs#jquery-plugin) in your HTML.

It includes a Twitter Bootstrap compatible progress bar, and it saves you
development time having to handle the file uploads yourself, and then pushing it
to our API.

```html
<script type="text/javascript" src="//assets.transloadit.com/js/jquery.transloadit2-v2-latest.js"></script>
<script type="text/javascript">
   // We call .transloadit() after the DOM is initialized:
   $(function() {
     $('#MyForm').transloadit();
   });
</script>
```

## Migrating between plans

As long as the plan you are migrating to includes enough allocated measurements for your usage, you can migrate between plans at any time without any interruption to your encoding.

Use the `heroku addons:upgrade` command to migrate to a new plan.

```bash
$ heroku addons:upgrade transloadit:enterprise
-----> Upgrading transloadit:enterprise to sharp-mountain-4005... done, v18 ($299/mo)
       Your plan has been updated to: transloadit:enterprise
```

## Removing the add-on

Transloadit can be removed via the CLI.

<div class="warning">This will destroy all associated data and cannot be undone!</div>

```bash
$ heroku addons:remove transloadit
-----> Removing transloadit from sharp-mountain-4005... done, v20 (free)
```

## Support

All Transloadit support and runtime issues should be submitted via one of the [Heroku Support channels](support-channels). Any non-support related issues or product feedback for Transloadit is welcome via [email](mailto:support@transloadit.com).
