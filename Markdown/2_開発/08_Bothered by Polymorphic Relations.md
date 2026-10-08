[![hackmd-github-sync-badge](https://hackmd.io/YIWaeRG_SriNtTCLaNXx-w/badge)](https://hackmd.io/@hayat01sh1da/bothered-by-polymorphic-relations)

<img src="https://hackmd.io/_uploads/r1p5_FHOA.png" alt="Ruby on Rails" />

## 1. Environment

* Ubuntu 20.04.5 LTS
* ruby 3.1.2
* Rails 7.0.4

## 2. Requirements

Add username to the search form item in an auto-sent mail history page.

## 3. Tables

\* Partially quoted

### 3-1. mail_events

|id |email              |
|:--|:------------------|
|1  |user@example.com   |
|2  |office@example.com |

### 3-2. mail_event_sent_to

|id |mail_event_id  |type     |holder_id |office_id |
|:--|:--------------|:--------|:---------|:---------|
|1  |1              |User     |1         |1         |
|2  |2              |Office   |1         |1         |

### 3-3. offices

|id |name     |email                |
|:--|:--------|:--------------------|
|1  |Office01 |office01@example.com |

### 3-4. users

|id |office_id |name   |
|:--|:---------|:------|
|1  |1         |User01 |

## 4. Relations

\* Class name is non-existent.

### 4-1. MailEvent

```ruby
class MailEvent < ApplicationRecord
  has_many :mail_event_sent_to, foreign_key: :mail_event_id, inverse_of: :mail_event, dependent: :destroy
end
```

### 4-2. MailEventSentTo

```ruby
class MailEventSentTo < ApplicationRecord
  belongs_to :mail_event
  belongs_to :sent_to, polymorphic: true
  belongs_to :office
end
```

### 4-3. Office

```ruby
class Office < ApplicationRecord
  has_many :users
end
```

### 4-4. User

```ruby
class User < ApplicationRecord
  has_many :entrance_and_exits
end
```

## 5. Troubles

1. Normally, I should define a scope in a class `MailEventSentTo` as the related one.  However, `sent_to` attribute is related to `polymorphic: true`, so it does not exist as the `SentTo` class.
2. `type` and `holder_id` defines a receiver(e.g. `User` + `1` is a user in `users` table where id is equal to `1`. `Office` + `1` is a user in `offices` table where id is equal to `1`).  So the keyword input in the search form must run SQL which searches both `users` table and `offices` table.

## 6. Source Code

\* Class name is non-existent.

### 6-1. Views

Add `sent_to` to form_with

```html
<%= search_form_field(f, :sent_to, 'name') %>
```

### 6-2. Controllers

Add `sent_to` to the white list of Strong Parameters.

```ruby
class MailEventsController < ApplicationController
  def index
    @offices = Office.all
    @search_form = MailEvent::SearchForm.new(search_params)
    @mail_events = @search_form.search...
  end

  private

  def search_params
    params.fetch(:mail_event_search_form, {}).permit(..., :sent_to)
  end
end
```

***

Add `sent_to` as String data in the class for search.  
`search` method calls a scope `MailEvent#sent_to`.

```ruby
class MailEvent::SearchForm < SearchForm::Base
  ...
  attribute :sent_to, String

  def search
    MailEvent
    ...
    .sent_to(@name)
  end
end
```

### 6-3. Models

Call `MailEventSentTo#sent_to` via `MailEvent#sent_to` when search is done with a name.
\* I would originally like to make an inner join with `joins(:mail_event_sent_to)` in `MailEvent::SearchForm#search` methods and directly implement a search logic.  But some `MailEvent` does not have `sent_to`.  That is why the logic looks complicated and a little strange.

```ruby
class MailEvent < ApplicationRecord
  ...
  scope :sent_to, ->(name) {
    return if name.blank?
    joins(:mail_event_sent_to).merge(MailEventSentTo.sent_to(name)).distinct
  }
end
...
```

***

Call `MailEventSentTo#sent_to`.

```ruby
class MailEventSentTo < ApplicationRecord
  ...
  scope :sent_to, ->(name) {
    return if name.blank?
    office_ids = Office.like_office_name(name).pluck(:id)
    user_ids = User.like_user_name(name).pluck(:id)
    search_office_ids = where(type: 'Office').where(holder_id: office_ids)
    search_user_ids = where(type: 'User').where(holder_id: user_ids)
    sql = search_office_ids.or(search_user_ids).select(:id).to_sql
    where("#{table_name}.id IN (#{sql})")
  }
  ...
end
```

What `MailEventSentTo#sent_to` does are:

Select the records from `offices` table which include the keyword input in the search form and assign projection of `id` column to `office_ids` variable.

```ruby
office_ids = Office.like_office_name(name).pluck(:id)
```

Select the records from `users` table which include the keyword input in the search form and assign projection of `id` column to `user_ids` variable.

```ruby
user_ids = User.like_user_name(name).pluck(:id)
```

Select all records where `type` is `Office` in `mail_event_sent_to` table and where `holder_id` is office_ids.  
Then assign them to the `search_office_ids` variable.

```ruby
search_office_ids = where(type: 'Office').where(holder_id: office_ids)
```

Select all records where `type` is `User` in `mail_event_sent_to` table and where `holder_id` is user_ids.  
Then assign them to `search_user_ids` variable.

```ruby
search_user_ids = where(mail_address_holder_type: 'User').where(mail_address_holder_id: user_ids)
```

Take the logical disjunction of `search_office_ids` and `search_user_ids`, extract foreign_key, then convert it to SQL.
Then assign them to the `sql` variable.

```ruby
sql = search_office_ids.or(search_user_ids).select(:id).to_sql
```

Select records which are equal to the sql from `mail_events`.

```ruby
where("#{table_name}.id IN (#{sql})")
```

***

`Office#like_office_name` and `User#like_user_name` are:

1. Call [arel_table](https://apidock.com/rails/v6.0.0/ActiveRecord/Core/ClassMethods/arel_table) with a receiver, `Office` or `User`, and load `name` column as an instance of `Arel::Table`.
2. Sanitise the keyword input in the search form and convert it into SQL, which is assigned to the LIKE phrase.
3. Issue SQL which selects all records where the keyword is included.

## 7. Conclusion

Do you remember the requirements of this task?  
It was just to add a `name` section to a search form in an index page.  
I thought that I could make it done soon because I viewed it as a simple task, but it was actually not.

The complicated table and SendGrid makes this task a profound one.  
I asked for a review so many times after a lot of tries and errors, and finally made it.  
I thought of writing raw SQL, but the code managed to ride on Rails Way.

This task made me understand:

* What is [Polymorphic Associations](https://guides.rubyonrails.org/association_basics.html#polymorphic-associations)？
* How to project the same named columns in plural tables with only a transaction
* How deprecated [arel_table](https://apidock.com/rails/v6.0.0/ActiveRecord/Core/ClassMethods/arel_table) and [sanitize_sql_like](https://api.rubyonrails.org/classes/ActiveRecord/Sanitization/ClassMethods.html#method-i-sanitize_sql_like) works
* Crazy table design can make production codes dirtier to supplement faults.
