---
layout: post
title:      "SurveyLance - Rails Portfolio Project"
date:       2017-10-03 21:44:25 -0400
permalink:  surveylance_-_rails_portfolio_project
---


## Introduction
SurveyLance is an online survey app that lets you create, published and participate in published surveys. Users can also view stats of the surveys they’ve created. Users can also create questions and answer choices for each question while creating the survey. To keep things simple, the answer choices are displayed as multi-choice check boxes. Unpublished surveys are only viewable to the creators.

The database relation consists of a user model, a submission model and a survey model. Users and surveys have many-to-many relationship with submission acting as a join table. A survey also has many question, a question has many answer choices, and an answer choice has many answers (answers are the answer choices selected by a survey submitter for a particular question). 

```
#models/user.rb
has_many :created_surveys, :class_name=> "Survey", :foreign_key=>"creator_id", dependent: :destroy
has_many :submissions, :foreign_key=>"submitter_id"
has_many :participated_surveys, :through=> :submissions, :source=> :survey
```
```
#models/survey.rb
belongs_to :creator, :class_name=>"User", :foreign_key=>"creator_id"
has_many :submissions, dependent: :destroy
has_many :submitters, :through => :submissions, :source => :submitter #user
has_many :questions, dependent: :destroy
```
```
#models/submission.rb
belongs_to :survey
belongs_to :submitter, :class_name=>"User", :foreign_key=>"submitter_id"
has_many :answers, dependent: :destroy
```
The resources are nested as following:
```
#config/routes.rb
resources :surveys, :shallow=> true do
    get 'take_survey' => "submissions#new" , :as => "take_survey"
    resources :submissions, :only => [:create]
    resources :questions do
      resources :answer_choices
    end
 end
 ```
Adding ‘shallow: true’ on the parent resource will generate shallow nesting, ie, nesting only for ‘[:index, :create, :new]’ and only goes 1 level deep instead of creating deeply nested resources.

## Authentication
User authentication for signup/signin and omniauth login is achieved through ‘*devise*’. GitHub and Facebook omniauth authentications are supported. User authentication is required to take and create surveys. Logged in users land on survey index page which is also the project’s root page. Once logged in, users can participate in published surveys or create their own survey and publish it.

Buttons to take, edit and view stats for the survey are displayed using helper methods based on user role and status of survey.
```
#helpers/surveys_helper.rb
module SurveysHelper
  def button_take_survey(survey)
    if user_signed_in? && survey.published?
      link_to("Take Survey", survey_take_survey_path(survey), :class => "btn btn-primary btn-sm")
    end
  end
end
```
## Create and Publish a Survey
Create Survey form is a simple form with inputs for title and description. ‘*Formtastic*’ gem was used to create forms for the project.
```
#views/surveys/new.html.erb
<%= title "Create New Survey" %>
<div class="container">
  <%= semantic_form_for @survey, :html=>{:class=> "form-group"} do |f| %>
    <%= render 'new_survey_form', f:f %>
  <% end %>
</div>
```

The survey’s show page doubles as an edit page and also show pages for all the questions and answer choices belonging to that survey.
```
#views/surveys/show.html.erb
<% @survey.questions.each do |question| %>
    <% if question.valid? && question.question_text %>
       <li class="list-group-item">
         <%= question.question_text %>
         <%= link_to "Add Answer Options", new_question_answer_choice_path(question), :class => "btn btn-primary btn-sm" %>
         <% question.answer_choices.each do |answer_choice| %>
            <ul>
             <% if answer_choice.answer_text %>
               <li class="list-group-item"><%= answer_choice.answer_text %></li>
             <% end %>
           </ul>
         <% end %>
      </li>
   <% end %>
<% end %>
```
Buttons to add question and add answer choices take you to ‘questions#new’ and ‘answer_choicess#new’ respectively within the nested routes as described earlier. Once all the questions and answer choices are entered, the user clicks Publish button that calls ‘surveys#publish’ method and changes the status of the survey to true.
```
#controllers/surveys_controller.rb
def publish
    authorize @survey, :show?
    @survey.update_attributes(:status => true)
    redirect_to root_path, :notice => "Survey successfully published."
end
```
## Take Survey
When a user takes a published survey, they are routed to ‘submissions#new’ action. This builds a new submission for the survey and displays all the questions and answer choices for the survey. Answer choices are displayed check box collections and will pass their ‘answer_choice_ids’ to ‘params’ as an array. The create action creates a submission and loops through the ‘answer_choice_ids’ array to create multiple answer objects.
```
#controllers/submissions_controller.rb
def create
    @submission = @survey.submissions.build
    @submission.submitter = current_user
    @submission.build_from_answer_choice(params[:submission][:answer_choice_ids])
    if @submission.save
      redirect_to survey_show_stat_path(@survey), :notice => "Survey sucessfully submitted."
    else
      render "surveys/take_survey"
    end
end
```
```
#models/submission.rb
def build_from_answer_choice(answer_choice_ids)
    answer_choice_ids.reject{|ac| ac.empty?}.each do |ac_id|
      answers.build({:answer_choice_id => ac_id})
    end
end
```
## View Survey Stats
The surveys#stat page viewable to the survey creator will show the total number of submissions for the survey and the total number of answers submitted for each answer choices for a question using a scope method.
```
#models/question.rb
def answer_count
    answer_count_hash = {}
    answer_choices.each do |ac|
      answer_count_hash[ac.answer_text] = ac.answers.size
    end
    answer_count_hash
end
```
## Authorization using Pundit
App authorization was generated using ‘Pundit’ gem and creating policy classes for the models. Following authorization policies were employed for this app:
1. Only logged in users can take and create surveys.
2. Users can view published surveys and surveys they have created.
3. Only creators can view the show page and add questions, add answer choices and publish surveys for unpublished surveys.
4. Only creators can view stats for published surveys.

Policy 1 was achieved by ‘devises’ #authenticate_user! method with a before_action.
Policy 2 was achieved by creating a scope class with the SurveyPolicy class and defining the scope as shown below. The ‘.or’ ActiveRecord method was possible because of ‘where-or’ gem. Rails 5 and up will have this method available natively.
Policies 3 & 4 were created by defining individual policy methods with their names matching the corresponding controller actions we are trying to authorize, followed by a question mark (?). For example, a policy method for surveys#show would be survey_policy#show?. The controller action would then invoke the policy by calling ‘authorize instance’. If an action has the same authorization requirement as another one, it call pass the policy name as the second argument to the ‘authorize’ call. For eg: surveys#show and surveys#publish have the same authorization requirement; #publish can invoke #show’s policy by calling ‘authorize @survey, :show?’.
Unauthorized access by default throws and Pundit::NotAuthorizedError. This can be better handled by redirecting user back to the referrer in the request or the root path with a flash warning by rescuing the error as below:
```
#policies/survey_policy.rb
class SurveyPolicy <  ApplicationPolicy
  def show?
    record.creator == user && record.published? == false
  end

  def show_stat?
    record.creator == user && record.published? == true
  end

  class Scope < Scope
    def resolve
        scope.where(creator: user).or(scope.where(status: true))
    end
  end
end
```
```
#controllers/application_controller.rb
rescue_from Pundit::NotAuthorizedError, with: :user_not_authorized
private
def user_not_authorized
    flash[:warning] = "You are not authorized to perform this action."
    redirect_to(request.referrer || root_path)
end
```

All in all, this project exposed me to a host of features and coding best practices that is in-built in Rails as well as external sources. It has definitely made me more confident in the path of being a career web developer.

**Github**: [https://github.com/sarjumulmi/rails-portfolio-survey-app.git](https://github.com/sarjumulmi/rails-portfolio-survey-app.git)




