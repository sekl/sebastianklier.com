---
title: "Rails 6 Beta - First Look at Action Text"
date: "2019-02-23T10:30:00.284Z"
layout: post
categories: [rails]
comments: true
---

Rails 6 Beta has been out for a short while now, and it comes with a long list of improvements and new features.

Today we'll look at one of the larger additions to Rails, which is the new **Action Text** framework.

<!--more-->

Action Text enables rich text content and editing in Rails straight out of the box. It uses Basecamp's [Trix Editor](https://trix-editor.org/), which is a neat and modern editor that is highly customizable.

Let's install the latest Rails 6 Beta release and try it out!

### Installing Rails 6 Beta

*First a word of warning: Rails 6 is a major version upgrade and there is a good chance it will break your existing application. If you are not ready to upgrade, try this out on a sample app first. Read the [changelog](https://edgeguides.rubyonrails.org/6_0_release_notes.html) for the details to find out if your app will be affected!*

Rails 6 requires **Ruby 2.5 or higher**, so make sure you have that installed. I recommend `rvm` or `rbenv` to separate different Ruby versions on your system, but this step is up to you.

Once you have the correct Ruby version, I'd also make sure to update bundler for good measure. Then let's install the Rails beta. Take care to separate your gemset if you don't want to update your Rails gem just yet. I usually separate rvm and gemsets, and use docker for most development projects, so I can just upgrade without breaking other environments:

`gem install bundler && gem install rails --pre`

The `--pre flag` should get you the latest code and upgrade your Rails gem. A quick `rails -v` should confirm that you are indeed running the latest version, which at time of writing is `Rails 6.0.0.beta1`.

So now we can create a new app:

`rails new rails6_preview`

This might take a while and will eventually leave you with a new Rails app running the latest code. 

> If it does not, or you want to upgrade an existing app, open your Gemfile and change the first few lines and the `rails` gem to look like this:
>
>``` ruby
>source 'https://rubygems.org'
>git_source(:github) { |repo| "https://github.com/#{repo}.git" }
>
>ruby '2.5.3' # or your version
>
>gem 'rails', github: 'rails/rails'
>```
>
> Then run `bundle update rails`. In this case you'll have to go through the upgrade process though (`rails app:update`), so in this example I prefer to start clean.

At this point try to start your server with `rails s` and open `localhost:3000`. If you get any errors, for example complaints about your sqlite gem version, you can fix it by specifying the sqlite gem in your Gemfile: 

`gem 'sqlite3', '~> 1.3.6'`

... and running `bundle update sqlite3`.

**Rails should be happy now:**

![Yay Rails Beta 6](/img/posts/yay-rails-beta-6.png)

**Sweet! Now let's try Action Text.**

First we need to install Action Text and its assets:

`rails action_text:install`

Then we need to create a scaffold with a model to hold our rich text field:

`rails g scaffold Post title:string`

*If you get errors about the directory being already watched, you can ignore that at this point.*

We've added a `title` field of type `string` to the model, but we still need a way to add the rich text body. Open the model and change it to add the new Action Text class method `has_rich_text` and specify a field for it:

``` ruby{numberLines: true}
class Post < ApplicationRecord
  has_rich_text :body
end
```

Run `rails db:migrate` real quick. This will migrate our model as well as Action Text tables that hold all the rich text data.

We also need to update our view to include the new field. In `app/views/posts/_form.html.erb`, add the following:

```html
<div class="field">
  <%= form.label :body %>
  <%= form.rich_text_area :body %>
</div>
```

Then start the server and navigate to `http://localhost:3000/posts/new`. 

You'll notice that the editor has already been included with a default styling and you can simply start using it! If you want to customize the editor, I suggest to start at the [official documentation](https://edgeguides.rubyonrails.org/action_text_overview.html).
