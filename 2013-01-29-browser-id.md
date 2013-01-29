I really like Mozilla's Persona. From a user standpoint, I trust a service that a) is from
Mozilla, and, importantly, b) does _nothing_ but authenticate. Even if I know that Google, or
Github, or Twitter's OAuth service isn't giving another web application any data beyond
authentication, I nevertheless avoid using them and feel annoyed if I am forced to.

And it's dead simple to add to Rails. No big authentication system to add (Devise), no
complicated user model.

## Dead-simple Browser ID authentication for Rails

Here's our migration to create the `Users` table:

    create_table :users do |t|
      t.string :email, null: false
      t.timestamps
    end

We just need an email field.

Add to the Gemfile:

    gem 'omniauth-browserid'

We'll need an omniauth initializer. Create `config/initializers/omniauth.rb`:

    Rails.application.config.middleware.use OmniAuth::Builder do
      provider :browser_id
    end

Add this route:

    post '/auth/browser_id/callback', to: 'sessions#create'

And create a `SessionsController`:

    class SessionsController < ApplicationController
      skip_before_filter :verify_authenticity_token
      
      def create
        auth = request.env["omniauth.auth"]
        email = auth["uid"]
        user = User.authorize_with_uid email
        session[:uid] = user.id
        redirect_to root_path, notice: "Signed in as #{email}"
      end
    end

And the `User` class:

    class User < ActiveRecord::Base
      def self.authorize_with_uid uid
        where(email: uid).fetch(0) {create_with_uid uid}
      end

      def self.create_with_uid uid
        create! email: uid
      end
    end

That's pretty much it! A `SessionsHelper` might look like this:

```
module SessionsHelper
  def current_user
    @current_user ||= User.find_by id: session[:uid]
  end

  def signed_in?
    current_user.present?
  end

  def authenticate_user!
    if !current_user
      redirect_to root_url, alert: 'Must be signed in.'
    end
  end

  def signed_in_email
    current_user.email
  end
end
```

(Put `helper :sessions` in `ApplicationController` to access those methods
in views.)
