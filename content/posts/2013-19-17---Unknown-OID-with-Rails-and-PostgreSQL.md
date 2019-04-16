---
title: "Unknown OID with Rails and PostgreSQL"
date: "2013-09-17T22:25:06-05:00"
template: "post"
draft: false
slug: "/posts/unknown-oid-with-rails-and-postgresql/"
category: "Web Development"
tags:
  - "rails"
  - "postgresql"
  - "ruby"
  - "citext"
description: "Fixing unknown OID postgresql errors with rails"
---


I was running into an issue where my Rails application log was full of warnings.

The warning was the following:

```
unknown OID: name(209925)
```

It was only happening for one certain attribute. This attribute was using a postgresql-contrib provided datatype ['citext'](http://www.postgresql.org/docs/9.3/static/citext.html).

After some digging it appears that Rails PostgreSQL adapter uses a cache that stores the oid of the default datatypes. The relevant code can be found [here on github](https://github.com/rails/rails/blob/4-0-stable/activerecord/lib/active_record/connection_adapters/postgresql/oid.rb#L294). Any column using a datatype that isn't found in the pg\_type table will [throw a warning](https://github.com/rails/rails/blob/4-0-stable/activerecord/lib/active_record/connection_adapters/postgresql/database_statements.rb#L147).

To fix this warning you need register the datatype like so:

```ruby
ActiveRecord::ConnectionAdapters::PostgreSQLAdapter.tap do |klass|
  klass::OID.register_type('citext', klass::OID::Identity.new)
end
```

This is relevent for any custom datatype just replace 'citext' with the one you are using.
After adding this to an initializer the warnings disapeared.

Hope that was helpful, cheers!
