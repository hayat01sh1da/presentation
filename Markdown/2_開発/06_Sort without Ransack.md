[![hackmd-github-sync-badge](https://hackmd.io/YIWaeRG_SriNtTCLaNXx-w/badge)](https://hackmd.io/@hayat01sh1da/sort-without-ransack)

<img src="https://hackmd.io/_uploads/r1p5_FHOA.png" alt="Ruby on Rails" />

## 1. Environment

* Ubuntu 20.04.5 LTS
* ruby 3.1.2
* Rails 7.0.4

## 2. Requirements

* Sort data by `学生コード (student_code)`, `学生名 (name)`, `学年 (grade_id)` in `students` table
* Sustain the search result
* Properly modularize the function to apply to other pages in the future

## 3. Custom Helpers

### 3-1. StudentHelper

Implement custom helpers for each model.

```ruby
module StudentHelper
  def student_sort_link(column)
    search_params = search_students_params.empty? ? {} : { search_students_form: search_students_params }
    sort_link(Student.human_attribute_name(column), column, search_params)
  end
end
```

* Take column as argument
* Return an empty hash if search form parameter is empty or return a hash including key and value of the search result
* Pass column to the common method `sort_link`
  * 1st argument: String with a link for sorting localised in Japanese
  * 2nd argument: Generate a query string `column=[column]`
  * 3rd argument: A hash of key and value of search from parameter

### 3-2. ApplicationHelper

Implement a common procedure.

```ruby
module ApplicationHelper
  def sort_link(column_name, column, **additional_params)
    if params[:column] != column.to_s || params[:direction].nil?
      link_to column_name, { column: column, direction: :asc, **additional_params }
    elsif params[:column] == column.to_s && params[:direction] == 'asc'
      link_to "#{column_name} ▲", { column: column, direction: :desc, **additional_params }
    elsif params[:column] == column.to_s && params[:direction] == 'desc'
      link_to "#{column_name} ▼", { column: column, direction: :asc, **additional_params }
    end
  end
end
```

* Receive the search result in 3rd argument as a variable one
* Here are three conditions
  * The sorted column is not equal to the one you try to sort, or no column is sorted
      * => Sort the column in asc
  * The sorted column is equal to the one you try to sort, and the column is sorted in asc
      * => Sort the column in desc and `▲` appears
  * The sorted column is equal to the one you try to sort, and the column is sorted in desc
      * => Sort the column in asc and `▼` appears

## 4. Controllers

Run SQL with the query string.

```ruby
def index
  @students = Student.search(@search_form).order(sort_column => sort_direction).preload(:grade)
end
...
private

def sort_column
  params[:column].in?(%w(student_code name grade_id)) ? params[:column] : :id
end

def sort_direction
  params[:direction].in?(%w(asc desc)) ? params[:direction] : :asc
end
```

* `order` receives `column to sort => asc/desc` as a hash
* `sort_column`: If the column exists in the white list, it is assigned to the key of `order`. If not,  `id` is allotted.
* `sort_direction`: If the sorting direction is in the white list, it is assigned to the value of `order`. If not,  `asc` is allotted.

## 5. Views

Take symbols as arguments as follows.

```html
<th class="col-sm-2"><%= student_sort_link(:user_code) %></th>
<th class="col-sm-3"><%= student_sort_link(:name) %></th>
<th class="col-sm-2"><%= student_sort_link(:grade_id) %></th>
```

## 6. Sort by student_code

* Default: No query string

<img src="https://user-images.githubusercontent.com/37478830/194869307-94ff8e33-5850-44d7-b490-1b4472170e25.png" alt="Default" />

***

* Asc: `?column=user_code&direction=asc`

<img src="https://user-images.githubusercontent.com/37478830/194869311-1cf9e6f8-4b78-4d23-be00-f201f658b588.png" alt="Ascendimg Order" />

***

* Desc: `?column=user_code&direction=desc`

<img src="https://user-images.githubusercontent.com/37478830/194869316-15fb9bdd-460e-4d55-8a5a-888240277123.png" alt="Descendimg Order" />
