---
layout: post
title: 'Angular, Rails, Capybara, wait for ajax.'
author: "Wenceslao Páez"
description: 'How much time we have to wait(sleep n) for all ajax requests in a test to finish?'  
keywords: "ruby, rails, angular, poltergeist, wait, ajax, capybara"
author_avatar_path: "/assets/images/wcpaez_user.png"
date:   2015-01-18 20:04:14
comments: true
categories: blog
---

When dealing with ajax integrations tests using Capybara and Rails, most of 
the time we don't know how long it will take to our backend to respond to 
that request. In this cases, Capybara is very good about handling ajax, 
waiting for elements to appear on the page,

{% highlight ruby %}
click_link("Send an ajax request")

expect(page).to have_content("My expected ajax response message")
{% endhighlight %} 

Capybara is smart enough to retry finding the content for a brief period of 
time before giving up and throwing an error. This period of time is defined by 

{% highlight ruby %}
Capybara.default_wait_time
{% endhighlight ruby %}

So, to solve your problem you can adjust this period, by 5, by 6, by 7. 
Only when it reaches the default wait time, Capybara stops finding  the content 
and your test fails. Other approach is setting a sleep, guessing how long it 
takes your request.

{% highlight ruby %}
click_link("Send an ajax request")
sleep(1)
expect(page).to have_content("My expected ajax response message")
{% endhighlight %} 

<p></p>
<h4>Wait for ajax!!!</h4>  
In order to stop gueesing, there is a also a good solution in [CODERWALL](https://coderwall.com/p/aklybw/wait-for-ajax-with-capybara-2-0 ) 
when you use JQuery for your ajax request. 

But what happens if like me, you are using AngularJS in your frontend app, 
making the requests with the $http service. In this case you have to find a 
way to figure out if there are pending request with AngularJS. Here is a solution:

{% highlight ruby %}
# spec/support/wait_for_ajax.rb

module WaitForAjax
  def wait_for_ajax(max_wait_time = 30)
    timeout.timeout(max_wait_time) do
      while pending_ajax_requests?
        sleep 0.1
      end
    end
  end

  def pending_ajax_requests?
    page.evaluate_script("angular.element(document.body).injector().get('$http').pendingrequests.length") > 0
  end
end

RSpec.configure do |config|
  config.include WaitForAjax
end
{% endhighlight ruby %}

<p></p>
<h4>Usage</h4>  
Just add this method in the place you want to make sure an ajax request has finished.

{% highlight ruby %}
click_link("Send an ajax request")

wait_for_ajax #wait for ajax response

expect(page).to have_content("My expected ajax response message")
{% endhighlight %} 

<p></p>
<h4>Poltergeist solution</h4>

In case you are using Poltergeist as your Capybara driver, you can inspect 
network traffic, helping know if there are pending response parts:

{% highlight ruby %}
# spec/support/wait_for_ajax.rb

def pending_ajax_requests?
  page.driver.network_traffic.collect(&:response_parts).any?(&:empty?)
end
{% endhighlight ruby %}

You can use this approach regardless of your JS frontend framework.
