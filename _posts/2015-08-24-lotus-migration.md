---
layout: post
title: Lotus DB Migrations Tips
tags: [lotus, ruby]
date: 2015-08-24
background: '/img/writing.jpg'
---

Lotus, just like Rails, also supports database migration through [Sequel Migrations](http://sequel.jeremyevans.net/rdoc/files/doc/migration_rdoc.html).
It´s pretty simple but some things are not so clear.

**Create a migration**

{% highlight bash %}
lotus g migration create_my_table
{% endhighlight %}

**Enable UUID (PostgreSQL only)**

{% highlight ruby %}
Lotus::Model.migration do
  up do
    execute 'CREATE EXTENSION "uuid-ossp"'
  end

  down do
    execute 'DROP EXTENSION "uuid-ossp"'
  end
end
{% endhighlight %}

**Use UUID as Primary Key**

{% highlight ruby %}
Lotus::Model.migration do
  up do
    create_table :my_table do
      column :id, :uuid, null: false, default: Sequel.function(:uuid_generate_v4), primary_key: true
    end
  end

  down do
    drop_table :my_table
  end
end
{% endhighlight %}

**Use UUID as Foreign Key**

{% highlight ruby %}
Lotus::Model.migration do
  up do
    create_table :my_table do
      column :id, :uuid, null: false, default: Sequel.function(:uuid_generate_v4), primary_key: true
      foreign_key :author_id, :authors, type: 'uuid', null: false
    end
  end
end
{% endhighlight %}

**Index a column**

{% highlight ruby %}
Lotus::Model.migration do
  up do
    create_table :my_table do
      column :code, :integer, null: false
      index :code
    end
  end
end
{% endhighlight %}

**Table documentation**

{% highlight ruby %}
Lotus::Model.migration do
  up do
    create_table :my_table do
      column :code, :integer, null: false
    end

    execute %Q(COMMENT ON TABLE my_table IS 'You should do it')
    execute %Q(COMMENT ON COLUMN my_table.code IS 'Easy and useful')
  end
end
{% endhighlight %}

**Migrate up**

{% highlight bash %}
lotus db migrate
{% endhighlight %}

**Migrate down**

There´s no `db migrate down` command, although you can specify a version to migrate. Just specify `0` to migrate to initial version (before any migration).
{% highlight bash %}
lotus db migrate 0
{% endhighlight %}
