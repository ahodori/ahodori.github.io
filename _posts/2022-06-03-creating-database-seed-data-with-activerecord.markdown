---
layout: post
title:  "Creating database seed data with ActiveRecord"
date:   2022-06-03 03:39:01 -0400
---

In order to test and debug code running on a database, we need data in that database. Seeds are a way of creating temporary or filler data we can use as a simulation of real, user-created data. But how do we create good seed data? I'll go over a few ideas from debugging that can help inform the seed data, then the basics of how to use an external API to fill in seed data for you.

For these examples, I'll use the database structure of a recent project, where we had 3 tables, Users, Locations, and Visits. Users and Locations are connected by Visits, which each have one corresponding User and Location, acting as a join table. One User and one Location may have multiple associated Visits, but each Visit has only one connected User and Location. A Visit represents a User visiting a particular Location. Here's how the schema looks in ActiveRecord for this setup:

{% highlight ruby %}
  create_table "locations", force: :cascade do |t|
    t.string "country"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end

  create_table "users", force: :cascade do |t|
    t.string "name"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end

  create_table "visits", force: :cascade do |t|
    t.integer "user_id"
    t.integer "location_id"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end
{% endhighlight %}

Now that we have our database layout, we can start creating seed data. To start with, let's make a few locations and users in our seeds.rb file:

{% highlight ruby %}
    bob = User.create(name: "Bob")
    alice = User.create(name: "Alice")
    joe = User.create(name: "Joe")

    us = Location.create(country: "United States")
    ca = Location.create(country: "Canada")
    mx = Location.create(country: "Mexico")
{% endhighlight %}

In order to create our sample visits, let's think about what we would want to test in some function that takes visits. For a function that takes in an array as input, it's important to test what happens when that array has zero, one, or multiple entries: So let's make it so that our three example users explore those three possibilities.

{% highlight ruby %}
    Visit.create(user_id: alice.id, location_id: us.id)

    Visit.create(user_id: joe.id, location_id: us.id)
    Visit.create(user_id: joe.id, location_id: ca.id)
    Visit.create(user_id: joe.id, location_id: mx.id)
{% endhighlight %}

Now, by looking at the visits of each user we can get those zero, one, or multiple entry responses we were looking for. As we build out the functionality of our app's backend, we may want more functions that operate on different data sources; maybe we will have a function that operates on users, filtered by the first initial of their name. In that case, we can create new seed data to make sure we still have options with zero, one, and multiple valid users; if we create a new user named "Billy", we can capture that multiple-case scenario, and we can capture the zero-case scenario by just looking for users whose names begin with C.

## Seeding via API and at random

Handwriting seeds is useful for testing the basic functionality of your app, but for something closer to a real situation, we now want a lot of data. It's slow and difficult for us as developers to create data with the size and scope of real user data by hand; for that, we can use an API to make our work easier.

Let's use the randomuser.me API to generate a whole bunch of users. In order to access this on our backend, we need to install the rest-client gem by adding the line to our project's Gemfile:
`gem "rest-client", "~> 2.1.0"`

Then run `bundle install` to install the dependencies. Now, in the seed file, let's write a request to this api using rest-client that will get us 100 randomly generated users:

{% highlight ruby %}
users = RestClient.get("https://randomuser.me/api/?results=100")
users_array = JSON.parse(users)["results"]
{% endhighlight %}

This API will give us a whole lot of information that's not useful to us right now, so let's drill down and create 100 users using just the user's first name the API provides.

{% highlight ruby %}
users_array.map do |user|
    User.create(name: user["name"]["first"])
end
{% endhighlight %}

Now our database has one hundred random users, and we didn't need to come up with one hundred names ourselves!

I'm sure there's a similar API that can be used to do the same for location, but for now I'd like to skip ahead to creating a lot of visits. Visits only depend on the users and locations we've already defined, so there's nothing to go to an outside API for; instead, we want to create a whole bunch of visits by randomly pairing users and locations. Let's do that:

{% highlight ruby %}
user_ids = User.pluck(:id)
location_ids = Location.pluck(:id)

(0..100).each do
    random_user_id = user_ids.sample
    random_location_id = location_ids.sample

    Visit.create(user_id: random_user_id, location_id: random_location_id)
end
{% endhighlight %}

First, we get two arrays containing all the valid User and Location ids. (This step can be very slow for a large database, but because we only need to seed our database once, this is acceptable.)

Next, 100 times, we sample a random id from each array, and create a Visit with those ids. Now we've got 100 randomly generated visits we can use to test our database.

This is just the simplest case of random generation; there's a whole lot more you can do. For example, if we didn't want duplicate visits, we might introduce a conditional that checks if a visit with a particular user_id and location_id already exists before we create it.

We might also want to avoid leaving things up to chance, and iterate through every user to make sure each user has at least one visit.

Keep in mind how you can use APIs and randomization to generate your seed database, and you'll be able to quickly and easily create data for testing your application.