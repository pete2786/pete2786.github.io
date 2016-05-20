---
layout: post
title:  Responders - Rails Junior Senior
tags: [professional rails ruby]
---

After a few months, or a few projects of Rails development, you might start noticing a boring, repetive pattern with your controllers. Something like: 

```
class SomeController < ApplicationController
  def something
    @resource = Resource.find(params[:id])
    @resource.do_something

    respond_to do |format|
      format.html do
        flash[:notice] = "Resource did something!"
        redirect_to resource_path(@resource)
      end
    end
  end

  def something_else
    @resource = Resource.find(params[:id])
    @resource.do_something_else

    respond_to do |format|
      format.html do
        flash[:notice] = "Resource did something else!"
        redirect_to resource_path(@resource)
      end
    end
  end
end
```

The first course action might be some simple refactoring. Another common pattern is controller abstraction, define a base controller which handles the base RESTful actions and override when needed. This approach tends to be complex and difficult follow. Rails offers another convention - Responders (http://edgeapi.rubyonrails.org/classes/ActionController/Responder.html).

## Defining a Responder
To create your own responder, create a new class which inherits from ActionController::ActionController::Responder. For example:

```
class MyFlashResponder < ActionController::Responder
  def to_html
    flash[:notice] = "#{resource.class.to_s} did #{action_verb}!"
    controller.redirect_to(resource)
  end
end
```

## Using a Responder
You could then leverage the MyFlashResponder the controller above and DRY up the code significantly. The respond_with method kicks off the responder.

```
require 'my_flash_responder'

class SomeController < ApplicationController
  self.responder = MyFlashResponder
  respond_to :html

  def something
    @resource = Resource.find(params[:id])
    @resource.do_something
    respond_with(@resource)
  end

  def something_else
    @resource = Resource.find(params[:id])
    @resource.do_something_else
    respond_with(@resource)
  end
end
```

I have found Responders particularly useful when developing a JSON API. It empowers you to define a consistent response pattern, handle errors, response codes, etc. Plataformatec has a "responders" gem (https://github.com/plataformatec/responders) with a number of well formed patterns to get you started. Examine your Rails apps and see if responders could make your life easier.
