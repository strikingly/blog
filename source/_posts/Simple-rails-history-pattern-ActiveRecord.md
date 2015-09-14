title: Simple rails history pattern (ActiveRecord)
date: 2015-09-14 10:42:19
author: fdutey
tags: 
- ruby
- rails
- active record
- pattern
categories:
- Backend
- English
---

## Context

Many times in my career, I had to face cases where I had to keep a history of one or many tables. Most of the time, the need of history comes after the main feature itself is developed and ends up in this kind of architecture:

```
| Post         |   | PostHistory   |
|==============|   |===============|
| author_id    |   | post_id       |
| title        |   | title         |
| content      |   | content       |
| published    |   |---------------|
| published_at |
|--------------|   
```

It comes as the most natural and painless way to add the history feature to an existing entity. You can also find gems doing this job for you, in which case, they will serialize your Post in a generic history table.

Both of those approaches sound sexy because they’re simple to implement and they don’t introduce a “risk” for the Post model. Problem is, they’re very hard to maintain.

In the first case, every time we run a migration on the post table, we will have to run it on the PostHistory too. Double the work. If you have special columns (enum, serialized stuff in any manner), you also have to maintain the same code in both classes. Or you have to extract everything in modules to share it between classes, which makes the code less readable. If you use services, it can lead to even more complicated situations.

In the second case, if your Post model changes, restoring the serialized version from history will become hard, if not impossible. If you want to historize only a subset of properties from Post, it’s also inconvenient. Plus, if you have serialized columns in Post table, you will end with serialized data IN serialized data in the history table. Something you don’t want to deal with in case of bugs.

The other main problem is if you need to use history in another context than just reading values and showing them to a human, you will have a schizophrenic code, using sometimes Post, sometimes PostHistory.

This architecture also come with the classic problem “when do we create a new entry?”:

* before saving will imply that the last version is not in history, force you to read from 2 tables to have the complete history
* after saving will imply that the last version is in two different places (content duplicated)


Let’s take a simple example, restore values from history:

```
# services/posts/historize_service.rb

class HistorizeService
  def run(post)
    history = PostHistory.create!(
      title: post.title,
      content: post.content
    )
  end
end

# services/posts/restore_from_history_service.rb

class RestoreFromHistoryService
  attr_reader :historize_service
  
  def initialize(historize_service:)
    @historize_service = historize_service
  end
  
  def run(post, post_history)
    Post.transaction do
      historize_service.run(post)
      post.title = post_history.title
      post.content = post_history.content
      # this list can be very long
      post.save!
    end
  end
end

```

Also consider that in this specific case, published state is ambiguous. And if many versions in the history have different meanings, it’s hard to identify which one has what meaning. We could transfer “publish” informations into the history table but that would mean history would carry contextual informations and we would have to maintain the whole history to ensure only one version is published at a time. 

## Separate “meta” and “content”

A more flexible solution you can consider is to separate in two separate models what is content (ie: what we need to keep track of over time) and what’s immutable over time.

```
| Post                 |   | Post::Content |
|======================|   |=============|
| author_id            |   | post_id     |
| current_content_id   |   | title       |
| published_content_id |   | content     |
| last_published_at    |   | created_at  |
|----------------------|   |-------------|
```

Using this design, whatever version you use, you will always manipulate the same type of object. You don’t have to double the logic and migrations. You keep your data stable over time. 

To give specific meaning to specific versions, we will rely on foreign keys. Post::Content should not know it’s role in outside models. Let’s keep it as small as possible and focus on data.

Your post model will look like this:

```
# models/post.rb

class Post < ActiveRecord::Base
  # Associations
  has_many :contents
  
  has_one :current_content, class_name: 'Post::Content'
  has_one :published_content, class_name: 'Post::Content'
  
  # Methods
  def published?
    published_content_id?
  end
end

```

Create a new entry in history is as simple as that:

```
# services/posts/historize_service.rb

class HistorizeService
  def run(post)
    Post.transaction do
      new_content = post.current_content.dup
      new_content.save!
      post.update_attributes!(current_content: new_content)
    end
  end
end

```

Compare to the previous version, we don’t need to modify this service every time we add (or remove) a property. It’s stable over time, that’s all we want.

The simplest way to deal with versions is to consider our Post::Content objects as immutable

```
# services/posts/content/update_service.rb

class UpdateService
  # params
  # 
  # * post: an instance of Post
  # * properties: a hash containing the following keys
  #   * title: (string) optional
  #   * content: (string) optional
  def run(post, properties)
    new_content = post.current_content.dup
    new_content.attributes = properties
    new_content.save!
    post.update_attributes!(current_content: new_content)
  end
end
```

This service will create a new content entry every time you want to update content properties (title and/or content). If you want to prevent blank entries that would create useless history entry, you can use a form object to validate the data before calling the service.

Let’s keep going with restoring an entry

```
# services/posts/restore_content_from_history_service.rb

class RestoreContentFromHistoryService
  def run(post, content)
    Post.transaction do
      # in order to keep the structure coherent, we need to
      # create a new entry instead of linking the old one.
      new_content = content.dup
      new_content.save!
      post.update_attributes!(current_version: new_content)
    end
  end
end

```

Let’s see how we do the publishing now

```
# services/posts/publish_service.rb

class PublishService
  def run(post)
    post.update_attributes!(
      published_content: post.current_content,
      last_published_at: Time.now
    )
  end
end

```

And voila! This design is flexible enough to add many features that won’t compromise the structure.

## Making it a bit more transparent

```
# models/post.rb

class Post < ActiveRecord::Base
  # Delegations
  delegate :title, :content, to: :current_content, allow_nil: true
end

# allow you to access directly post.title
# and masking the complexity of post.current_content.title
# (see demeter law https://en.wikipedia.org/wiki/Law_of_Demeter)
```

## Plugging features

Adding version numbers, for example

```
# models/posts/content.rb

class Post::Content < ActiveRecord::Base
  # Callbacks
  before_create :tag_version_number
  
  # Methods
  
  private
  
  def tag_version_number
    self.version = generate_version_number
  end
  
  def generate_version_number
    # optimistic solution
    (self.class.where(post_id: post_id)
               .maximum(:version) || 0) + 1
  end
end

```

This could also be extracted to a module, exposed as a method like

```
has_versions_number scope: :post_id
```

You can even disable update at model level to ensure no one is changing the history

```
# lib/active_record/immutable_model.rb

module ActiveRecord
  module ImmutableModel
    class UpdateImmutableModelException < Exception
      def initialize(msg)
        super("Immutable model can't be updated")
      end
    end
    
    extend ActiveSupport::Concern
    
    included do
      before_update :prevent_update
    end
    
    private
    
    def prevent_update
      raise UpdateImmutableModelException.new
    end
  end
end        

# models/post/content.rb

class Post::Content < ActiveRecord::Base
  include ActiveRecord::ImmutableModel
end
```

## Recommendations

To update Post  from forms, I discourage you to use ActiveRecord nested_attributes pattern as long as it’s a very horrible pattern (even if it looks very convenient). Instead, you would prefer to use a form object and hide the complexity of content versioning from your user.

## Conclusion

This solution solves some problem cases but still have cons, like any other one. It has a higher cost at beginning but offers more flexibility over time, especially if you want to add features based on your history. It also allows you to easily “tag” and access some versions.



