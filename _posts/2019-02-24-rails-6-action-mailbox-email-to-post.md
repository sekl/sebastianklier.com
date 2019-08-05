---
title: "Building a Post-via-Email Feature with Rails 6 Action Mailbox"
date: "2019-02-24T12:00:00.284Z"
layout: post
categories: [rails]
comments: true
---

Last time I looked at the new [Action Text](/rails/2019/02/23/rails-6-action-text.html) framework coming in Rails 6. Now let's see how we can use Action Mailbox - also new in Rails 6 - to create a simple "Post-via-Email" blogging feature.

<!--more-->

Action Mailbox already has [great documentation](https://edgeguides.rubyonrails.org/action_mailbox_basics.html), so getting started is easy.

For now, let's skip integration with an email service such as Sendgrid or Amazon SES. We can test incoming emails using just the framework features in local development.

### Installing Action Mailbox

Setup is a simple two-step process - assuming you already installed the latest Rails 6 beta or newer:

```
rails action_mailbox:install
rails db:migrate
```

This will install Action Mailbox assets, routes and set up your database.

If you **start the Rails server** now, you can already play around with incoming emails by pointing your browser to `http://localhost:3000/rails/conductor/action_mailbox/inbound_emails`.

Then create a new inbound email. It will be added to the list but have the status `bounced`. This is because we don't yet have any controller that can handle it.

**Let's fix that.**

In this example I pick up where I left off [last time](/rails/2019/02/23/rails-6-action-text.html), so I already have a `Post` scaffold ready to extend.

In `app/mailboxes/application_mailbox.rb`, add a new route:

``` ruby
class ApplicationMailbox < ActionMailbox::Base
  routing /^blog_via_email@/i     => :blog_via_email
end
```
    
Routing is described in more detail in the Action Mailbox [source](https://github.com/rails/rails/blob/master/actionmailbox/lib/action_mailbox/base.rb):
``` ruby
# Any of the recipients of the mail (whether to, cc, bcc) are matched against the regexp.
routing /^replies@/i => :replies

# Any of the recipients of the mail (whether to, cc, bcc) needs to be an exact match for the string.
routing "help@example.com" => :help

# Any callable (proc, lambda, etc) object is passed the inbound_email record and is a match if true.
routing ->(inbound_email) { inbound_email.mail.to.size > 2 } => :multiple_recipients

# Any object responding to #match? is called with the inbound_email record as an argument. Match if true.
routing CustomAddress.new => :custom

# Any inbound_email that has not been already matched will be sent to the BackstopMailbox.
routing :all => :backstop 
```

In our case, this means that any email sent to `blog_via_email@ourdomain.com` will end up in the mailbox we call `blog_via_email`, which we will create now. 

**Run `rails generate mailbox blog_via_email` on the command line.**

The result is a new, empty mailbox at `app/mailboxes/blog_via_email_mailbox.rb`

The `process` method is already stubbed out and will be used to handle the incoming email. We also have several callbacks available here: `before_processing`, `after_processing` and `around_processing`, which do what you would expect. 

As an example use case, you could use `before_processing` to make sure that the sender of the email has a corresponding user in the database and is actually authorized to spam your system with blog posts.

To make it simple and demonstrate only the most basic flow, let's just naively create a new post from our email subject and content.

**You should probably implement your own logic, validations and error handling before trying this on production!**

``` ruby
class BlogViaEmailMailbox < ApplicationMailbox
  def process
    Post.create!(title: mail.subject, body: mail.body.decoded)
  end
end
```

Now if you go to `http://localhost:3000/rails/conductor/action_mailbox/inbound_emails/new` again and send a new email to `blog_via_email@yourdomain.com`, it should go through, change status to `delivered` and create a new blog post at `http://localhost:3000/posts/`!

For a real use case, you could go back to add the missing validation and callbacks, then hook it up with your Email service of choice as described in the [docs](https://edgeguides.rubyonrails.org/action_mailbox_basics.html).

By the way, the code for this example is available on [GitHub](https://github.com/sekl/rails6_preview).
