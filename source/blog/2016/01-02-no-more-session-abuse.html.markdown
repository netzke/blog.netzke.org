---
title: No more session abuse
date: 2016-01-02 11:54 UTC
tags:
excerpt: "With latest Netzke Core we can do better than keeping a component's state in the session."
---
In a [typical example](https://www.airpair.com/javascript/posts/getting-started-with-netzke) of having 2 grids that
handle models bound with a one-to-many relationship, we used to need an endpoint that would receive the id of the
currently selected parent record and store it in the session, so that the next endpoint call that is processed by the
second grid would already have that grid reconfigured for us, scoping it out to the record selected. It usually looked
something like this:

```ruby
endpoint :server_set_author do |params, this|
  # store author_id in the session:
  component_session[:current_author_id] = params[:author_id]
end

component :books do |c|
  # filter books data by author id stored in the session:
  c.scope = {author_id: component_session[:current_author_id]}
end
```

Then on the client we would do something like this:


```javascript
this.netzkeGetComponent('authors').on('rowclick', function(grid, record) {
  // call the "server_update_author" endpoint of the server side
  this.serverSetAuthor({author_id: record.getId()});

  // inform the client-side of the Books grid to reload itself
  this.netzkeGetComponent('books').getStore().load();
}, this);
```


Given we could have dozens of configurations like this, this was very heavy on the session, which resulted in session size
exceeded very quickly, and the remedy for that would be to use the database to store the session. Not very performant a
solution, and besides it didn't feel clean. Now there's a better way.

Currently Netzke implements a way to reconfigure any instantiated component directly from the client side by using
special `serverConfig` property. Whatever is stored in that object is passed along with every endpoint call and is
accessible on the server side via `client_config` method. So, on the client we can now do this:

```javascript
this.netzkeGetComponent('authors').on('rowclick', function(grid, record) {
  // configure our top component with the new current_author_id
  this.serverConfig['current_author_id'] = record.getId();

  // inform the client-side of the Books grid to reload itself
  this.netzkeGetComponent('books').getStore().load();
}, this);
```

No more call to the `server_set_author` endpoint, so, we can get rid of it. And the books grid can be configured like
this now:

```ruby
component :books do |c|
  # filter books data by author id passed from the client
  c.scope = {author_id: client_config[:current_author_id]}
end
```

I updated the ["Bosses and Clerks" example](http://demo.netzke.org/#bosses_and_clerks) in the official demo to use this technique, too.
