gem 'devise'
gem 'redis'
gem 'braintree'
gem 'gon'
rails generate devise Admin
class Admin < ActiveRecord::Base
  devise :database_authenticatable, :trackable, :timeoutable
end
Rails.application.routes.draw do
  namespace :admin do
    resources :categories
    resources :products
  end
end
rails generate model Category name:string
rails generate model Product name:string description:text price:decimal category:references
rails generate model Cart
class ApplicationController < ActionController::Base
  before_action :set_cart

  private

  def set_cart
    @cart = Cart.find(session[:cart_id])
  rescue ActiveRecord::RecordNotFound
    @cart = Cart.create
    session[:cart_id] = @cart.id
  end
end
# products/show.html.erb
<%= button_to "Add to Cart", carts_path(product_id: @product.id) %>
class CartsController < ApplicationController
  def create
    @cart = Cart.find(session[:cart_id])
    @product = Product.find(params[:product_id])
    @cart.add_product(@product)
    redirect_to root_path, notice: "Product added to cart"
  end

  def destroy
    @cart = Cart.find(session[:cart_id])
    @product = Product.find(params[:product_id])
    @cart.remove_product(@product)
    redirect_to root_path, notice: "Product removed from cart"
  end
end
class PaymentsController < ApplicationController
  def create
    # Braintree payment logic here
  end
end
# application.html.erb
<p>Visits: <%= Rails.application.secrets.visits_counter %></p>
class ApplicationController < ActionController::Base
  before_action :increment_visits_counter

  private

  def increment_visits_counter
    Rails.application.secrets.visits_counter ||= 0
    Rails.application.secrets.visits_counter += 1
  end
end
# payments/confirmation.html.erb
<h1>Thank you for your purchase!</h1>
<p>Please leave us some feedback:</p>
<%= form_for(:feedback, url: { controller: 'feedbacks', action: 'create' }) do |f| %>
  <%= f.text_area :body %>
  <%= f.submit %>
<% end %>
class Feedback < ApplicationRecord
  def classify
    # Open AI classification logic here
  end
end

