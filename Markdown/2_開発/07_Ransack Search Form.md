[![hackmd-github-sync-badge](https://hackmd.io/YIWaeRG_SriNtTCLaNXx-w/badge)](https://hackmd.io/@hayat01sh1da/ransack-search-form)

<img src="https://user-images.githubusercontent.com/37478830/194869304-c70b8392-995f-43eb-81b7-6a4affba3975.png" alt="Ruby on Rails" />

## 1. Environment

* Ubuntu 20.04.5 LTS
* ruby 3.1.2
* Rails 7.0.4

## 2. Requirements

Users can search for events with

* `name`: contained
* `place`: contained
* `content`: contained
* `start_at`: later than or equal to
* `end_at`: earlier than or equal to

## 3. Gemfile

Add `ransack` in [Gemfile](https://github.com/oasis-forever/perfect-ruby-on-rails/blob/master/Gemfile#L51) and run `docker-compose exec app bin/rails bundle`.

```ruby
source 'https://rubygems.org'
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby '2.7.2'

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
gem 'rails', '~> 6.0.3', '>= 6.0.3.2'
...

# SearchForm
gem 'ransack', '~> 2.3.2'
```

## 4. Controllers

Get `params[:q]` value sent from views with `ransack` class method, call the result with `result` instance method and assign events to the instance variable `@events` in [controllers](https://github.com/oasis-forever/perfect-ruby-on-rails/blob/master/app/controllers/events_controller.rb#L7).

```ruby
def index
  @q = Event.ransack(params[:q])
  @events = @q.result.page(params[:page]).per(paginate_per).default
end
```

## 5. Views

Ransack provides the following `{attr}_{options}` helper methods that can be used in [views](https://github.com/oasis-forever/perfect-ruby-on-rails/blob/master/app/views/events/index.html.erb).

|Option  |Meaning               |Symbol |
|:-------|:---------------------|:------|
|*_eq    |equal                 |`==`   |
|*_noteq |not equal             |`!=`   |
|*_lt    |less than             |`<`    |
|*_lteq  |less than or equal    |`<=`   |
|*_gt    |greater than          |`>`    |
|*_gteq  |greater than or equal |`>=`   |
|*_cont  |contains value        |`≒`    |

Apply appropriate options to each attribute and implement the search form with `form_with` helper method.

```html
<div class="panel panel-default panel-search">
  <div class="panel-body">
    <%= search_form_for(@q, class: 'mb-5') do |f| %>
      <div class="row form-group">
        <div class="col-sm-6">
          <%= f.label :name_cont, Event.human_attribute_name(:name), class: 'control-label col-sm-5 nowrap' %>
          <div class="col-sm-11 col-md-9">
            <%= f.search_field :name_cont, class: 'form-control' %>
          </div>
        </div>
        <div class="col-sm-6">
          <%= f.label :place_cont, Event.human_attribute_name(:place), class: 'control-label col-sm-5 nowrap' %>
          <div class="col-sm-11 col-md-9">
            <%= f.search_field :place_cont, class: 'form-control' %>
          </div>
        </div>
      </div>
      <div class="row form-group">
        <div class="col-sm-12">
          <%= f.label :content_cont, Event.human_attribute_name(:content), class: 'control-label col-sm-5 nowrap' %>
          <div class="col-sm-11 col-md-10">
            <%= f.search_field :content_cont, class: 'form-control' %>
          </div>
        </div>
      </div>
      <div class="row form-group">
        <div class="col-sm-6">
          <%= f.label :start_at_gteq, Event.human_attribute_name(:start_at), class: 'control-label col-sm-5 nowrap' %>
          <div class="col-sm-11 col-md-9">
            <%= f.datetime_field :start_at_gteq, class: 'form-control' %>
          </div>
        </div>
        <div class="col-sm-6">
          <%= f.label :end_at_lteq, Event.human_attribute_name(:end_at), class: 'control-label col-sm-5 nowrap' %>
          <div class="col-sm-11 col-md-9">
            <%= f.datetime_field :end_at_lteq, class: 'form-control' %>
          </div>
        </div>
      </div>
      <div class="form-btn-field">
        <%= f.submit class: 'btn btn-primary' %>
      </div>
    <% end %>
  </div>
</div>
```

## 6. Assets

Define styles for the search form in `/app/assets/stylesheets/ransack.scss`.

```css
.panel {
  margin-top: 20px;
  margin-bottom: 20px;
  background-color: #fff;
  border: 1px solid transparent;
  border-radius: 4px;
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.05);
  &-default {
    border-color: #ddd;
  }
  &-body {
    padding: 15px;
  }
}

.form-btn-field {
  margin-top: 20px;
  text-align: center;
}
```

## 7. Deliverables

<img src="https://user-images.githubusercontent.com/37478830/194869332-1f9d693f-0953-43dd-930f-bf47d44cacdb.png" alt="Ransack Search Form" />
