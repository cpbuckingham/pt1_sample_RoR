ample_RoR

pt.1
rails new sample_RoR -d mysql
cd sample_RoR
rails g model user name:string email:string password_digest:string
rails g controller users new
rails g controller sessions new
rails g controller home show
bundle exec rake db:drop
bundle exec rake db:create
bundle exec rake db:migrate
gem 'bcrypt-ruby'
bundle install
mysql.server start
mysql -u root -p;
SHOW DATABASES;
USE sample_RoR_development;
select * from users;

pt.2
root to: 'home#show'
get '/signup' => 'users#new'
post '/users' => 'users#create'
get '/login' => 'sessions#new'
post '/login' => 'sessions#create'
get '/logout' => 'sessions#destroy'
<%= form_for :user, url: '/users' do |f| %>
<%= f.text_field :name, placeholder:"name" %>
<%= f.text_field :email, placeholder:"email" %>
<%= f.password_field :password, placeholder:"password" %>
<%= f.password_field :password_confirmation, placeholder:"retype password" %>
<%= f.submit "Submit" %>
<% end %>
def create
  user = User.new(user_params)
  if user.save
    session[:user_id] = user.id
    redirect_to '/'
  else
    redirect_to '/signup'
  end
end
private
def user_params
params.require(:user).permit(:name, :email, :password, :password_confirmation)
end

has_secure_password


pt.3
def create
  user = User.find_by_email(params[:email])
  # if the user exists AND the password entered is correct
  if user && user.authenticate(params[:password])
    # save the user id inside the browser cookie. This is how we keep the user logged in when they navigate around our website.
    session[:user_id] = user.id
    redirect_to '/'
  else
    redirect_to '/login'
  end
end

def destroy
  session[:user_id] = nil
  redirect_to '/login'
end
<%= form_tag '/login' do %>
email: <%= text_field_tag :email %>
password:<%= password_field_tag :password %>
<%= submit_tag "Submit" %>
<% end %>


pt.4
class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception

  def current_user
    @current_user ||= User.find(session[:user_id]) if session[:user_id]
  end
  helper_method :current_user

  def authorize
    redirect_to '/login' unless current_user
  end

end
<% if current_user %>
  Signed in as <%= current_user.name %> | <%= link_to "Logout", '/logout' %>
<% else %>
  <%= link_to 'Login', '/login' %> | <%= link_to 'Signup', '/signup' %>
<% end %>

pt.5

mailers/user_mailer.rb
class UserMailer < ActionMailer::Base
  default from: 'sample_RoR_app@camjam.com'

  def welcome_email(user)
    @user = user
    mail(to: user.email, subject: "Welcome to my sample RoR app")
  end
end

views/user_mailer/welcome_email.html.erb
<!DOCTYPE html>
<html>
<head>
  <meta content='text/html; charset=UTF-8' http-equiv='Content-Type'/>
</head>

<body>

<h1>Welcome to my Sample Ruby App, <%= @user.email %>!</h1>

<p>Thanks for signing up!</p>

</body>
</html>

UserMailer.welcome_email(user).deliver



Homework -
use Pony app to deliver emails

