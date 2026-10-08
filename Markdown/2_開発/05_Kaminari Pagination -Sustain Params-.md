[![hackmd-github-sync-badge](https://hackmd.io/YIWaeRG_SriNtTCLaNXx-w/badge)](https://hackmd.io/@hayat01sh1da/kaminari-pagination-with-params-sustained)

<img src="https://hackmd.io/_uploads/r1p5_FHOA.png" alt="Ruby on Rails" />

## 1. Environment

* Ubuntu 20.04.5 LTS
* ruby 3.1.2
* Rails 7.0.4

## 2. Requirements

* Sustain pagination data for such extensions as Ransack search form introduction
* Show the number of items of all
* Default number of items is 20
* Switch the number of items among 20, 50, 100, 200 and 500

## 3. Gemfile

Add `rails-i18n` and `kaminari` in [Gemfile](https://github.com/oasis-forever/perfect-ruby-on-rails/blob/master/Gemfile#L44) and run `docker-compose exec app bin/rails bundle`.

```ruby
source 'https://rubygems.org'
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby '2.7.2'

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
gem 'rails', '~> 6.0.3', '>= 6.0.3.2'
...

# Locale
gem 'rails-i18n', '~> 6.0.0'

# Pagination
gem 'kaminari', '~> 1.2.1'
```

## 4. Config

### 4-1. Settings

Define `number_per_page` in [/config/settings.yml](http://github.com/oasis-forever/perfect-ruby-on-rails/blob/master/config/settings.yml) to switch the number of items to show.

```yaml
common: &common
  number_per_page: [20, 50, 100, 200, 500]

development:
  <<: *common

test:
  <<: *common

staging:
  <<: *common

production:
  <<: *common
```

### 4-2. Initializers

First, define `AppConfig` constant in [/config/initializers/app_constants.rb](https://github.com/oasis-forever/perfect-ruby-on-rails/blob/master/config/initializers/app_constants.rb) to call `number_per_page`.

```ruby
AppConfig = YAML.load_file("#{Rails.root}/config/settings.yml")[Rails.env].symbolize_keys
```

Next, define the default number of items to show as 20 in [/config/initializers/kaminari_config.rb](https://github.com/oasis-forever/perfect-ruby-on-rails/blob/master/config/initializers/kaminari_config.rb).

```ruby
Kaminari.configure do |config|
  config.default_per_page = 20
end
```

### 4-3. Locales

Define translation of English words of kaminari in [/config/locales/pagination.ja.yml](http://github.com/oasis-forever/perfect-ruby-on-rails/blob/master/config/locales/pagination.ja.yml).

```yaml
ja:
  views:
    pagination:
      first: "最初へ"
      last: "最後へ"
      previous: "前へ"
      next: "次へ"
      truncate: "&hellip;"
  helpers:
    page_entries_info:
      one_page:
        zero: "該当データがありません。"
        one: "1-1/全1件"
        display_entries: '1-%{count}/全%{count}件'
      more_pages:
        display_entries: '%{first}-%{last}/全%{total}件'
```

## 5. Controllers

### 5-1. ApplicationController

Define some private methods in [/app/controllers/application_controller.rb](https://github.com/oasis-forever/perfect-ruby-on-rails/blob/master/app/controllers/application_controller.rb).

```ruby
class ApplicationController < ActionController::Base

...

  private

  # Return the number of items to show if it is assigned
  def paginate_per
    session[:paginate_per] = params[:per] if params[:per].present?
    session[:paginate_per]
  end

  # Sustain query parameters controller name and action name in the key `last_pagination_data` of session
  def keep_last_pagination_data
    session[:last_pagination_data] = {
      params: request.query_parameters,
      controller: controller_name,
      action: action_name
    }
  end

  # Abandon `session[:last_pagination_data]` and return `session[:last_pagination_data]["params"]`
  # if `session[:last_pagination_data]` is NOT `nil`,
  #    `session[:last_pagination_data]["controller"]` is `controller_name` and
  #    `session[:last_pagination_data]["action"]` is `action.to_s`
  def load_pagination_params(action)
    data = session[:last_pagination_data].presence
    if data["controller"] == controller_name && data["action"] == action.to_s
      session[:last_pagination_data] = nil
      ret = data["params"]
    end
  end

 # Redirect to the index page with `action: action_name` and `params: session[:last_pagination_data]["params"]` key & values
  def redirect_with_kept_pagination_params(action:, **args)
    redirect_to({ action: action, params: load_pagination_params(action) }, args)
  end
end
```

### 5-2. Apply Pagination to Controllers

Apply `paginate_per` private methods to the argument of `per` method of kaminari in the [controllers](http://github.com/oasis-forever/perfect-ruby-on-rails/blob/03c5b208eefd13e818136a3434c8157a7df46bec/app/controllers/welcome_controller.rb#L5) you would like to apply pagination to.

```ruby
def index
  @events = Event.page(params[:page]).per(paginate_per).where('start_at >= ?', Time.now).order(:start_at)
end
```

## 6. Views

### 6-1. Pagination Header

Implement a shared pagination header in `/app/views/shared/_pagination_header.html.erb`

```html
<% if defined?(objects) %>
<div class="pagination-field">
  <div class="pagination-wrapper">
    <div class="entry-content">
      <%= page_entries_info(objects, entry_name: objects&.model_name.to_s) %>
      <p class="display-items">
        表示件数：
        <% AppConfig[:number_per_page].each do |per| %>
          <%= link_to_unless per == objects.limit_value, per, params.permit!.merge(per: per) %>
        <% end %>
         &nbsp;
      </p>
    </div>
    <div class="pagination-content float-right">
      <%= paginate(objects) %>
    </div>
  </div>
</div>
<% end %>
```

### 6-2. Pagination Views

Run `docker-compose exec app bin/rails g kaminari:views bootstrap4`, and required pagination views will be created.

* [/app/views/kaminari/_first_page.html.erb](https://github.com/oasis-forever/perfect-ruby-on-rails/blob/master/app/views/kaminari/_first_page.html.erb)

```html
<li class="page-item">
  <%= link_to_unless current_page.first?, raw(t 'views.pagination.first'), url, remote: remote, class: 'page-link' %>
</li>
```

* [/app/views/kaminari/_gap.html.erb](http://github.com/oasis-forever/perfect-ruby-on-rails/blob/master/app/views/kaminari/_gap.html.erb)

```html
<li class="page-item disabled">
  <%= link_to raw(t 'views.pagination.truncate'), '#', class: 'page-link' %>
</li>
```

* [/app/views/kaminari/_last_page.html.erb](https://github.com/oasis-forever/perfect-ruby-on-rails/blob/master/app/views/kaminari/_last_page.html.erb)

```html
<li class="page-item">
  <%= link_to_unless current_page.last?, raw(t 'views.pagination.last'), url, remote: remote, class: 'page-link' %>
</li>
```

* [/app/views/kaminari/_next_page.html.erb](https://github.com/oasis-forever/perfect-ruby-on-rails/blob/master/app/views/kaminari/_next_page.html.erb)

```html
<li class="page-item">
  <%= link_to_unless current_page.last?, raw(t 'views.pagination.next'), url, rel: 'next', remote: remote, class: 'page-link' %>
</li>
```

* [/app/views/kaminari/_page.html.erb](https://github.com/oasis-forever/perfect-ruby-on-rails/blob/master/app/views/kaminari/_page.html.erb)

```html
<% if page.current? %>
  <li class="page-item active">
    <%= content_tag :a, page, data: { remote: remote }, rel: page.rel, class: 'page-link' %>
  </li>
<% else %>
  <li class="page-item">
    <%= link_to page, url, remote: remote, rel: page.rel, class: 'page-link' %>
  </li>
<% end %>
```

* [/app/views/kaminari/_paginator.html.erb](https://github.com/oasis-forever/perfect-ruby-on-rails/blob/master/app/views/kaminari/_paginator.html.erb)

```html
<%= paginator.render do %>
  <nav>
    <ul class="pagination">
      <%= first_page_tag unless current_page.first? %>
      <%= prev_page_tag unless current_page.first? %>
      <% each_page do |page| %>
        <% if page.left_outer? || page.right_outer? || page.inside_window? %>
          <%= page_tag page %>
        <% elsif !page.was_truncated? %>
          <%= gap_tag %>
        <% end %>
      <% end %>
      <%= next_page_tag unless current_page.last? %>
      <%= last_page_tag unless current_page.last? %>
    </ul>
  </nav>
<% end %>
```

* [/app/views/kaminari/_prev_page.html.erb](https://github.com/oasis-forever/perfect-ruby-on-rails/blob/master/app/views/kaminari/_prev_page.html.erb)

```html
<li class="page-item">
  <%= link_to_unless current_page.first?, raw(t 'views.pagination.previous'), url, rel: 'prev', remote: remote, class: 'page-link' %>
</li>
```

### 6-3. Apply Pagination to Views

[Render `shared/pagination_header`](https://github.com/oasis-forever/perfect-ruby-on-rails/blob/03c5b208eefd13e818136a3434c8157a7df46bec/app/views/welcome/index.html.erb#L2) to show the pagination header and call [paginate](https://github.com/oasis-forever/perfect-ruby-on-rails/blob/03c5b208eefd13e818136a3434c8157a7df46bec/app/views/welcome/index.html.erb#L13) method to show the pagination footer.

```html
<h1>イベント一覧</h1>
<%= render 'shared/pagination_header', objects: @events %>
<div class="list-group">
  <% @events.each do |event| %>
    <%= link_to(event, class: 'list-group-item list-group-item-action') do %>
      <h5 class="list-group-item-heading"><%= event.name %></h5>
      <p class="mb-1"><%= "#{l(event.start_at, format: :long)} - #{l(event.end_at, format: :long)}" %></p>
    <% end %>
  <% end %>
</div>

<div class="pagination-content float-right mt-4">
  <%= paginate(@events) %>
</div>
```

## 7. Assets

Define styles for the pagination header and footer in `/app/assets/stylesheets/pagination.css.scss`.

```css
.pagination-field {
  display: table;
  width: 100%;
  margin-top: 10px;
  .pagination-wrapper {
    display: table-row;
    .entry-content {
      display: table-cell;
      width: 220px;
      vertical-align: middle;
    }
    .pagination-content {
      display: table-cell;
      vertical-align: middle;
    }
  }
}
```

## 8. Deliverables

<img src="https://user-images.githubusercontent.com/37478830/194869325-d500f1f7-0ea7-4d53-987a-a3c5b548395a.png" alt="Pagination Header" />

***

<img src="https://user-images.githubusercontent.com/37478830/194869330-735f7911-4f86-47cb-9ee9-093d3ccf5fd6.png" alt="Pagination Footer" />
