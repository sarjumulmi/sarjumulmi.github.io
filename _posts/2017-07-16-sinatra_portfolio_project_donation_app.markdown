---
layout: post
title:  "Sinatra Portfolio Project – Donation App"
date:   2017-07-16 16:26:42 +0000
---


After deliberating for over half a day, I finally came up with a scheme for my Sinatra Portfolio project. The basic requirements for the project were to use *ActiveRecord* CRUD methods, have user authentication, employ a model with a *has_many* relationship, and do input validation. Much to my wife’s chagrin (she wanted a closet organizer), my idea was to come up with a Donation app that will match the donors to the various charities.

I ended up with a pretty straightforward database structure with a Donor, a Charity and a Donation models. A Donor has a name, email, password, and many Donations. A Charity has a name, a cause and many Donations. A Donation will act as a join table that belongs to both Donor and Charity, and also has item and item_price attributes. Thus a Donor will have many Chairities through Donation and vice-versa. I wanted to model the Items as a separate entity but it soon got very complicated, so I decided to stick with this simple model for the project. I have helper methods to check whether the user is logged in or not, if logged in, give the current user. I also created two Donor methods to get the total amount donated by a Donor and the total amount donated to each Charity by a Donor using the *ActiveRecord::Calculations* sum method.
```
def total_donation
    self.donations.sum(:item_price)
  end

  def donated_to_charity(charity)
    self.donations.where(:charity_id=>charity.id).sum(:item_price)
  end
```

The homepage shows a list of donors and their respective charities and will redirect to the login/signup page if the user is not logged in. After logging in, a donor can view a list of charities, edit his/her profile, or make a donation. The navigation is achieved with links and buttons within a form with method set to ‘get’. When a donor clicks the “Donate” button, it “get”s the */donations/new* route which will route the donor to a */donations/new* view if logged in. The view consists of a form showing the donor’s name, a dropdown list of charities, input boxes for the item to donate and the price of the item donated. The donor can select the charity he wants to donate to, put in the donation item and its price, and hit the *Donate* button. This will “post” to */donations* action in the *DonationsController*. The action creates a new donation and sets its donor and charity from the params hash. It then saves the donation and redirect the donor to his profile page.

I used a lot of *tux* and *binding.pry* to see and tinker around with the database and the params hash. Overall, I felt good to be able to build a simple working web app from scratch.

