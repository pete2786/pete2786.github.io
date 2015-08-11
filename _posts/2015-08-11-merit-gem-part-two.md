---
layout: post
title:  Earn A Gamification Badge with Merit (Part Two)
tags: [professional]
---

The [Merit](https://github.com/merit-gem/merit) gem can get you a _long_ way towards gamifiying you application. If you are just getting started with Merit, checkout [part one](/2015/07/05/merit-gem.html) of this series. I found three specific areas of the Merit gem/documentation lacking: Notifications (via Observers, using callbacks instead of the DSL and working with Badges. I'm going to dive in a bit deeper to each of these areas.

### Notifications via an Observer ###
The [documentation for notifications](https://github.com/merit-gem/merit#getting-notifications) is lacking. Merit requires an `Observer` class be implemented, which is simply a Ruby class which defines the `update` method. The `update` method a the `changed_data` param, which is a hash of changed data. This observer will be called anytime there is a reputation change and try to describe the change.. The ability to send notifications at a finer grain should be a first class action of this gem. Until that is implemented, check out the following example to try and pluck a specific action from an observer. To the code:

{% highlight ruby %}
# app/observers/new_badge_observer.rb

class NewBadgeObserver
  def update(changed_data)
    return unless changed_data[:merit_object].is_a?(Merit::BadgesSash)

    user = User.where(sash_id: changed_data[:sash_id]).first
    badge = changed_data[:merit_object].badge
    granted_at = changed_data[:granted_at]

    BadgeMailer.earned_badge(user, badge, granted_at).deliver_now
  end
end
{% endhighlight %}

This observer throws away anything which is not `Merit::BadgesDash` change. From there, the code is fairly self explainatory, I grab the user, the badge and the time from the changed_data hash and notify the user via email. If you are wondering why this does not also catch badge removal, you can observe below that the Badge Removal changed_data hash does not include a `merit_object` entry.

The spec of the `changed_data` hash is not well documented, so here goes. The `update` method is called for three reasons: Badge granted, Badge removed or Points applied.

* Badge granted keys: `description, merit object (the badge earned), sash_id`
* Badge removed keys:  `description, sash_id`
* Points applied keys: `description, merit_object (the point), sash_id`.

In addition, every changed_data hash includes: `granted_at, merit_action_id`.

### Using callbacks instead of the Merit DSL ###
The Merit DSL is clean and a great way to keep your badge and point conferring rules in one place. However, there are certain times when a controller action does not line up perfectly with the action you want to record. For example, I wanted the first 100 users of my application to receive a `pioneer` badge. However, I use Google as my identity provider via Omniauth, so I had no user#create action for which to bind the badge. Instead, I added an `after_create` callback to my user model:

{% highlight ruby %}
# app/user.rb

class User < ActiveRecord::Base
  PIONEER_BADGE_ID = 1
  ...

  after_create :process_badges

  def process_badges
    self.add_badge(PIONEER_BADGE_ID) unless self.has_badge?(PIONEER_BADGE_ID) && self.id <= 100
  end

end
{% endhighlight %}

### Working with Badges ###
If you look under the covers, you'll find the Merit::Badge model is an [Ambry model](https://github.com/norman/ambry). This is what allows the flexibility to define static badges in an initializers instead of creating database records. These work fairly well with standard Rails conventions, but I found a few key methods missing. The biggest of which was finding a badge for a user. I added the following mix in to my user model to max dealing with Badges slightly easier:

{% highlight ruby %}
# app/concerns/has_badge.rb

module HasBadge
  extends ActiveSupport::Concern


  def has_badge?(badge_id)
    !find_badge(badge_id).nil?
  end

  def find_badge(badge_id)
    badges.select{|b| b.id == badge_id}.first
  end

end

# app/user.rb

class User < ActiveRecord::Base
  include HasBadge
  ...

end

{% endhighlight %}

I also added a badge decorator using Draper, which I'd also recommend otherwise your view logic could get quite complex.

{% highlight ruby %}
class BadgeDecorator < Draper::Decorator
  delegate_all

  def title
    "#{name.capitalize}#{level_description}"
  end

  def level_description
    level.present? ? " (Level: #{level})" : ""
  end

  def image_path(suffix='')
    filename = level ? "#{name}_level#{level}" : name
    "badges/#{filename}#{suffix}.png"
  end

  def badge_names
    @badge_names ||= badges.map(&:name)
  end

  def earned_on(user)
    Merit::BadgesSash.where(sash_id: user.sash_id, badge_id: id).first.try(:created_at)
  end

end

{% endhighlight %}

That's it, hopefully this helped you get on your way to earning "Advanced magification badge" with the Merit gem.



