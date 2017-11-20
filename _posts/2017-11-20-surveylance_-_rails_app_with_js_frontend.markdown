---
layout: post
title:      "Surveylance - Rails App with JS FrontEnd"
date:       2017-11-20 16:29:01 +0000
permalink:  surveylance_-_rails_app_with_js_frontend
---


For this project, I added some features to the SurveyLance app employing JQuery and AJAX. The first improvement was to add the capability of adding and removing question and answer fields while creating or editing a survey from a single page. This is accomplished by adding a helper method ‘link_to_add_fields’. First, partials are created for questions and answers fields respectively. The method would then basically generate a ‘fields_for’ scope for the given form builder object and association. The 'fields_for' scope would render a partial for the desired field and return a link with a ‘data-field’ attribute set to the generated ‘fields_for’ scope. Upon clicking the link, the ‘data-field’ is extracted and appended to the DOM. Since fields were being dynamically added or removed, event delegation is used to handle events by attaching the listener to a parent node.

```
#application_helper.rb
def link_to_add_fields(name, f, association, **args)
    new_object = f.object.send(association).klass.new
    id = new_object.object_id
    fields = f.semantic_fields_for(association, new_object, :child_index=> id) do |builder|
      render(association.to_s.singularize, f:builder)
    end
    link_to(name, '#', :class => "add_fields " + (args[:class] || ''), :data => {:id => id, :fields => fields.gsub('\n', '')})
end
```
```
#surveys.js
$(document).on('turbolinks:load', function(){
  // event listener for Delete links for questions and answer options
  $('form').on('click','.remove-records', function(evt){
    $(this).prev('li').children('input[type=hidden]').val('1')
    $(this).closest('fieldset').hide()
    evt.preventDefault()
  })
  // event listener for add fields for questions and answer options
  $('form').on('click', '.add_fields', function(evt){ 
    time = new Date().getTime()
    regexp = new RegExp($(this).data('id'), 'g')
    $(this).parent().prev('.fields').append($(this).data('fields').replace(regexp, time))
    evt.preventDefault()
  })
})
```

The second feature added displays a user’s created and participated survey lists through a link via AJAX on the user’s profile page. ActiveModel serializers for ‘User’ and ‘UserSurvey’ are created and ‘surveys#index’ action is edited to respond with the user object as a ‘json’ object. In ‘users.js’, a JavaScript class ‘Survey’ is created to replicate the ‘json’ object returned by the action. 
```
#user_serializer.rb
class UserSerializer < ActiveModel::Serializer
  attributes :id, :email
  has_many :created_surveys, serializer: UserSurveySerializer
  has_many :participated_surveys, serializer: UserSurveySerializer
end
```
```
#user_survey_serializer.rb
require 'action_view'
require 'action_view/helpers'
include ActionView::Helpers::DateHelper

class UserSurveySerializer < ActiveModel::Serializer
  attributes :id, :title, :description, :created
  has_many :submissions

  def created
    "about #{time_ago_in_words(object.created_at)} ago"
  end 
end
```
```
#surveys_controller.rb
def index
    if params[:user_id]
      @user = User.find(params[:user_id])
      render :json=> @user
    else
      @surveys = policy_scope(Survey)
      @survey = Survey.new
    end
  end
```
```
#users.js
function Survey(attributes){
  this.id = attributes.id
  this.description = attributes.description
  this.title = attributes.title
  this.creator_id = attributes.creator_id
  this.created = attributes.created
}
```
Handlebars template and partial scripts are introduced at the bottom of the ‘users#show’ page to display the survey lists in tabular format. Upon clicking the link, a new ‘Survey’ object is instantiated and passed on to the compiled Handlebars template. The returning variable is then appended to the DOM.
```
#users.js
$('#show-survey').on('click', function(evt){
    $.get(this.href)
    .done(json => {
      var created_surveys = []
      json.created_surveys.forEach(att => {
        created_surveys.push(new Survey(att))
      })
      var created_surveys_list = Survey.template({title: "Created Surveys", survey: created_surveys})
      $('#user-created-surveys').empty().html(created_surveys_list)
	})
	evt.preventDefault()
  })
```
The last feature displays a user’s created survey details page and adds ‘Next’ and ‘Previous’ pagination links to browse through the surveys without refreshing the page. Virtual attributes ‘next’ and ‘previous’ are created for ‘Survey’ model to return the next and previous ‘Survey’ object belonging to the user respectively. 

```
#survey.rb
scope :previous, lambda {|id| where("id < ?",id).order("id DESC") }
def previous
    Survey.where('creator_id=?',self.creator_id).previous(self.id).first
end
```

Serializers for ‘Survey’, ‘Question’ and ‘AnswerChoice’ are created and ‘surveys#show’ action is modified to respond with both ‘html’ and ‘json’. In the ‘SurveySerializer’, the virtual attributes ‘next’ and ‘previous’ are modified to return only the ‘ids’. Handlebars templates are employed again to generate the template and insert into the DOM.

```
#survey_serializer.rb
class SurveySerializer < ActiveModel::Serializer
  attributes :id, :title, :description, :creator_id, :status, :nextSurvey, :previousSurvey
  has_many :questions

  def previousSurvey
    object.previous.id if object.previous
  end
end
```

The URL is also updated to show the correct survey id using Web API, history.pushState().

```
#survey_show.js
history.pushState(null, null, this.href)
```

One thing that confounded me and go berserk on google was ‘Turbolinks’. I had the ‘Turbolinks’ turned on which, by the way, would make your AJAX requests faster and more efficient. However, it also does not update the links in the DOM upon partial refresh, even with the event delegation in place. So, the link would work as expected the first time, but the second time it’s not loaded in the DOM. The event goes unnoticed and you get a full page reload. The solution was to simply replace the typical `$(document).ready(function (){})` with `$(document).on(‘turbolinks:load’, function (){})` which will force the link to update with each partial refresh.
> References:
> 
> [Railscast - Nested Model Form ](http://railscasts.com/episodes/196-nested-model-form-revised?autoplay=true)

**Github**: [https://github.com/sarjumulmi/Rails-JS-portfolio.git](https://github.com/sarjumulmi/Rails-JS-portfolio.git)


