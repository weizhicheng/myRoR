	- see app/views/home/_sign_in.html.erb for its usage

* Controllers: displaying views to the user based on their inputs and the state of data models
	- three controllers (follows, home, and twets)
	- application_controller (provides some base methods and properties that our other controllers will inherit)
	- home_controller (the landing page of the app. You can sign in and sign up here)
	- wets_controller (twets_controller requires the user to be authenticated)

* Concerns: the concerns construct gives a way to reuse code shared by multiple controllers

* Helpers: Helper files provide convenience functions that you can use in your Rails views

* Models: 
	- models are responsible for persisting the state of your data
	- Models can also provide methods
	- Follow (representing a “following” relationship between two users)
	- Twet (twets belong to users, and they must have a content field)
	- User (inherits several properties and methods from Devise)

* Views:
	- Devise (the templates required by Devise)
	- Follows ( the templates that our follows_controller will serve)
	- Home (the html that will be rendered when you view the index for the site)
	- Shared (partials that are shared by multiple views)
	- Twets (renders each twet that the user has published, as well as those of the users he or she follows)

* The Config Folder:
	- Environments (contains respective settings for your development, production, and test environments.)
	- database.yml (contains your configuration settings for your development, test, and production environments)
	- routes.rb (routing urls to controllers)

* The Public Folder:
	- contains static html files that the user will see if there are errors for a given url they request.
	limingth@gmail ~/Github/myRoR/twetter$ cat app/views/home/_sign_in.html.erb 
	<div class="row">
	  <div class="col-md-4 col-md-offset-4 panel panel-default">
	    <%= form_for :user, :url => user_session_path,
	                        :builder => InlineErrorsBuilder,
	                        :method => :POST,
	                        :role => :form do |f| %>
	      <div>
	        <%= content_tag :div, :class => f.validation_class(:email, "form-group") do %>
	          <%= f.text_field :email, :placeholder => "Username or email", :class => 'form-control' %>
	          <%= f.errors_for :email %>
	        <% end %>

	        <%= content_tag :div, :class => f.validation_class(:password, "form-group") do %>
	          <%= f.password_field :password, :placeholder => "Password", :class => 'form-control left' %>
	          <%= f.errors_for :password %>
	          <%= f.submit "Sign in", :class => "btn btn-primary pull-right" %>
	        <% end %>

	        <div class="clearfix"></div>

	        <div class="form-group">
	          <div class="pull-left">
	            <label><%= f.check_box :remember_me %> Remember me</label> -
	            <%= link_to "Forgot password?", new_user_password_path %>
	          </div>
	        </div>
	      </div>
	    <% end %>
	  </div>
	</div>
	limingth@gmail ~/Github/myRoR/twetter$ 
# Developing New Features for Twetter
## Develop a Feature Branch for User Profiles
### cd into your Twetter project folder and Create a new branch myProfileBranch
	limingth@gmail ~/Github/myRoR/twetter$ git branch
	* master
	limingth@gmail ~/Github/myRoR/twetter$ git branch -r
	  origin/HEAD -> origin/master
	  origin/complete
	  origin/gravatar
	  origin/master
	  origin/profile
	  origin/retwet
	limingth@gmail ~/Github/myRoR/twetter$ git checkout -b myProfileBranch
	Switched to a new branch 'myProfileBranch'
	limingth@gmail ~/Github/myRoR/twetter$ git branch
	  master
	* myProfileBranch
	limingth@gmail ~/Github/myRoR/twetter$ 
### compare the diff between 2 branches
	limingth@gmail ~/Github/myRoR/twetter$ git diff myProfileBranch origin/profile 
	diff --git a/app/controllers/follows_controller.rb b/app/controllers/follows_controller.rb
	index 33d19a8..5c7e2ae 100644
	--- a/app/controllers/follows_controller.rb
	+++ b/app/controllers/follows_controller.rb
	@@ -15,7 +15,7 @@ class FollowsController < ApplicationController
	   # This action first attempts to find an existing Follow instance between the current user and
	   # the user whose id matches params[:follow][:following_id]. If one is not found, it is created.
	   # If the Follow is created successfully, a success notice is set. Otherwise, an error notice
	-  # is set. Either way we are directed back to the index action.
	+  # is set. The #smart_return method is called to take us back to the appropriate page.
	   #
	   def create
	     following = current_user.follows.where(:following_id => follow_params[:following_id]).first ||
	@@ -25,7 +25,7 @@ class FollowsController < ApplicationController
	     else
	       flash[:error] = "Your attempt to follow was unsuccessful"
	     end
	-    redirect_to :action => :index
	+    smart_return
	   end
	 
	   # DELETE /follows/:id
	@@ -33,8 +33,8 @@ class FollowsController < ApplicationController
	   # Responsible for unfollowing a user. Works by deleting the Follow instance. The use of the
	   # resource method (defined below) ensures that only follows which belong to the authenticated
	   # user can be matched and deleted. If the Follow instance is found and deleted successfully,
	-  # a success notice is set. Otherwise, an error notice is set. Either way we are directed back
	-  # to the index action.
	+  # a success notice is set. Otherwise, an error notice is set. The #smart_return method is
	+  # called to take us back to the appropriate page.
	   #
	   def destroy
	     if resource and resource.destroy
	@@ -42,7 +42,7 @@ class FollowsController < ApplicationController
	     else
	       flash[:error] = "Your attempt to unfollow was not successful"
	     end
	-    redirect_to :action => :index
	+    smart_return
	   end
	 
	   private
	@@ -58,7 +58,22 @@ class FollowsController < ApplicationController
	     params.require(:follow).permit(:following_id)
	   end
	 
	+  # Finds a Follow instance that matches the id passed and assigns it to @resource
	+  # unless @resource is already assigned. This ensures that we only look for the
	+  # Follow instance until we find it the first time.
	+  #
	   def resource
	     @resource ||= current_user.follows.where(:id => params[:id]).first
	   end
	+
	+  # Leverages the params[:return_to] value to direct the user back to a requested
	+  # page. If no value is present, the user is directed back to the index.
	+  #
	+  def smart_return
	+    if params[:return_to].present?
	+      redirect_to params[:return_to]
	+    else
	+      redirect_to :action => :index
	+    end
	+  end
	 end
	diff --git a/app/controllers/twets_controller.rb b/app/controllers/twets_controller.rb
	index 2266efd..21d39be 100644
	--- a/app/controllers/twets_controller.rb
	+++ b/app/controllers/twets_controller.rb
	@@ -41,7 +41,12 @@ class TwetsController < ApplicationController
	 
	   # Sets the @twets instance variable to all twets viewable by the current user
	   def get_twets
	-    @twets = current_user.all_twets
	+    if params[:username]
	+      @user = User.where(:username => params[:username]).first
	+      @twets = Twet.by_user_ids(@user.id) if @user
	+    else
	+      @twets = current_user.all_twets
	+    end
	   end
	 
	   # http://guides.rubyonrails.org/action_controller_overview.html#strong-parameters
	diff --git a/app/helpers/application_helper.rb b/app/helpers/application_helper.rb
	index af78764..9c42855 100644
	--- a/app/helpers/application_helper.rb
	+++ b/app/helpers/application_helper.rb
	@@ -5,16 +5,21 @@ module ApplicationHelper
	   # authenticated user is already following the user passed, an unfollow (DELETE) form
	   # will be generated. Otherwise, a follow (CREATE) form will be generated.
	   #
	-  def follow_link(user)
	+  def follow_link(user, return_path = nil)
	     follow = Follow.where(:user => current_user, :following => user)
	     if follow.exists?
	-      button_to("Unfollow", follow_path(follow.first), :method => :delete,
	-                                                       :class => 'btn btn-danger mar-top-5',
	-                                                       :form => { :class => 'form-inline pull-right' })
	+      form_for(:follow, :url => follow_path(follow.first), :method => 'DELETE', :html => { :class => 'form-inline pull-right' }) do |f|
	+        html = ''
	+        html += hidden_field_tag(:return_to, return_path) if return_path
	+        html += f.submit('Unfollow', :class => "btn btn-danger mar-top-5")
	+        html.html_safe
	+      end
	     else
	       form_for(:follow, :url => follows_path, :method => 'POST', :html => { :class => 'pull-right' }) do |f|
	-        f.hidden_field(:following_id, :value => user.id.to_s) +
	-        f.submit('Follow', :class => "btn btn-primary mar-top-5")
	+        html = f.hidden_field(:following_id, :value => user.id.to_s)
	+        html += hidden_field_tag(:return_to, return_path) if return_path
	+        html += f.submit('Follow', :class => "btn btn-primary mar-top-5")
	+        html.html_safe
	       end
	     end
	   end
	diff --git a/app/views/twets/index.html.erb b/app/views/twets/index.html.erb
	index 7af60e1..288a216 100644
	--- a/app/views/twets/index.html.erb
	+++ b/app/views/twets/index.html.erb
	@@ -7,6 +7,7 @@
	     <div class="pull-left">
	       <h4>Twets</h4>
	     </div>
	+    <%= follow_link(@user, profile_path(:username => @user.username)) if @user and @user.id != current_user.id %>
	     <div class="clearfix"></div>
	     <hr />
	     <ol class="list-unstyled">
	diff --git a/config/application.rb b/config/application.rb
	index 4364368..88abcf1 100644
	--- a/config/application.rb
	+++ b/config/application.rb
	@@ -6,6 +6,8 @@ require 'rails/all'
	 # you've limited to :test, :development, or :production.
	 Bundler.require(:default, Rails.env)
	 
	+require './lib/mention_linker.rb'
	+
	 module Twetter
	   class Application < Rails::Application
	     # Settings in config/environments/* take precedence over those specified here.
	@@ -25,5 +27,8 @@ module Twetter
	     config.action_view.field_error_proc = Proc.new { |html_tag, instance|
	       "#{html_tag}".html_safe
	     }
	+
	+    # Use our MentionLinker class to replace @mentions with a profile link
	+    config.middleware.insert_before ActionDispatch::ParamsParser, MentionLinker
	   end
	 end
	diff --git a/config/routes.rb b/config/routes.rb
	index 98a293b..08f58ba 100644
	--- a/config/routes.rb
	+++ b/config/routes.rb
	@@ -6,6 +6,7 @@ Twetter::Application.routes.draw do
	   authenticated :user do
	     resources :follows, :except => [:new, :edit, :show, :update]
	     resources :twets, :except => [:new, :edit, :show, :update]
	+    get ':username', :to => 'twets#index', :as => :profile
	     root :to => 'follows#index', :as => :user_root
	   end
	 
	diff --git a/lib/mention_linker.rb b/lib/mention_linker.rb
	new file mode 100644
	index 0000000..84d0058
	--- /dev/null
	+++ b/lib/mention_linker.rb
	@@ -0,0 +1,43 @@
	+# ref: http://guides.rubyonrails.org/rails_on_rack.html
	+#
	+# This is Rack Middleware designed to wrap all instances of @username with a link to '/username'.
	+# It is included in the application in config/application.rb#32.
	+#
	+# NOTE: An easier way to accomplish this, would be to wrap all output of @username in a link
	+# on the Rails side of things. If you take / took this approach, make sure to remember linking
	+# @username mentions in Twets as you display them. You can accomplish this by making the #link_mentions
	+# method a view helper and running all Twet#content displays through it.
	+#
	+class MentionLinker
	+
	+  # Make the middleware aware of the application.
	+  #
	+  def initialize(app)
	+    @app = app
	+  end
	+
	+  # Gets called by the stack after the page is generated. Based on where we are in the Rack stack,
	+  # if the response is an Array instance, the first value will be the HTML for the page. In this case,
	+  # the HTML is parsed by the #link_mentions method to perform the substitution described above.
	+  #
	+  def call(env)
	+    status, headers, response = @app.call(env)
	+    if headers["Content-Type"] and
	+       headers["Content-Type"].match(Regexp.new("text/html")) and
	+       response.respond_to?(:first)
	+      [status, headers, [link_mentions(response.first)]]
	+    else
	+      [status, headers, response]
	+    end
	+  end
	+
	+  # Uses a simple regex to find all instances of @username which occur after a space or '>' character
	+  # and replace these with a link to that user. When working with regular expressions in Ruby,
	+  # http://rubular.com will be your best friend.
	+  #
	+  # additional ref: http://ruby-doc.org/core-2.0.0/String.html#method-i-gsub
	+  #
	+  def link_mentions(html)
	+    html.gsub(/(?<prefix>[>| ])@(?<username>(\w+))/, '\k<prefix><a href="/\k<username>">@\k<username></a>') if html
	+  end
	+end
	diff --git a/spec/lib/mention_linker_spec.rb b/spec/lib/mention_linker_spec.rb
	new file mode 100644
	index 0000000..61b09d6
	--- /dev/null
	+++ b/spec/lib/mention_linker_spec.rb
	@@ -0,0 +1,28 @@
	+require 'spec_helper'
	+require Rails.root + 'lib/mention_linker.rb'
	+
	+describe MentionLinker do
	+  let(:linker) { MentionLinker.new(double) }
	+
	+  describe "#link_mentions" do
	+    context "when passed text with an @ symbol" do
	+      context "which is not inside an a element" do
	+        let(:txt) { 'I like @bluefocus' }
	+        let(:expected) { 'I like <a href="/bluefocus">@bluefocus</a>' }
	+
	+        it "should add a link to the user's twets page" do
	+          linker.link_mentions(txt).should == expected
	+        end
	+      end
	+
	+      context "which is a component of an a larger word" do
	+        let(:txt) { 'dan@bluefoc.us' }
	+
	+        it "should not change the text" do
	+          linker.link_mentions(txt).should == txt
	+        end
	+      end
	+    end
	+  end
	+end
	+
	limingth@gmail ~/Github/myRoR/twetter$ 
### write code that converts @username mentions into links to user profile pages

### Step 1 - write routes
	limingth@gmail ~/Github/myRoR/twetter$ vi config/routes.rb 
	  1 Twetter::Application.routes.draw do
	  2   devise_for :users
	  3   # The priority is based upon order of creation: first created -> highest priority.
	  4   # See how all your routes lay out with "rake routes".
	  5 
	  6   authenticated :user do
	  7     resources :follows, :except => [:new, :edit, :show, :update]
	  8     resources :twets, :except => [:new, :edit, :show, :update]		<--- add this line to routes
	  9     get ':username', :to => 'twets#index', :as => :profile
	 10     root :to => 'follows#index', :as => :user_root
	 11   end

### Step 2 - write views
	limingth@gmail ~/Github/myRoR/twetter$ vi app/views/twets/index.html.erb 

	  1 <div class="clearfix top-space small"></div>
	  2 <div class="row">
	  3 
	  4   <%= render :partial => 'shared/left_nav' %>
	  5 
	  6   <div class="panel panel-default col-md-8 text-left">
	  7     <div class="pull-left">
	  8       <h4>Twets</h4>
	  9     </div>
	 10     <%= follow_link(@user, profile_path(:username => @user.username)) if @user and @user.id != current_user.id %>			<--- add this line 
	 11     <div class="clearfix"></div>
	 12     <hr />