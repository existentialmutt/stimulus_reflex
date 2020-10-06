# StimulusReflex Cheat Sheet

## Calling Reflexes

### via Data Attributes

`app/views/pages/index.html.erb`

```html
<a
  href="#"
  data-reflex="click->CounterReflex#increment"
  data-step="1"
  data-count="<%= @count.to_i %>"
  >Increment <%= @count.to_i %></a
>
```

`app/reflexes/counter_reflex.rb`

```ruby
class CounterReflex < StimulusReflex::Reflex
  def increment
    @count = element.dataset[:count].to_i + element.dataset[:step].to_i
  end
end
```

### from Stimulus.js Controller

`app/views/pages/index.html.erb`

```erb
<a href="#"
  data-controller="counter"
  data-action="click->counter#increment"
>Increment <%= @count %></a>
```

`app/javascript/controllers/counter_controller.js`

```javascript
import { Controller } from "stimulus";
import StimulusReflex from "stimulus_reflex";

export default class extends Controller {
  connect() {
    StimulusReflex.register(this);
  }

  increment(event) {
    event.preventDefault();
    this.stimulate("Counter#increment", 1);
  }
}
```

`app/reflexes/counter_reflex.rb`

```ruby
class CounterReflex < StimulusReflex::Reflex
  def increment(step = 1)
    session[:count] = session[:count].to_i + step
   end
end

```

`app/controllers/pages_controller.rb`

## Morphs

### Scoping Page Morphs with `data-reflex-root`

{% tabs %}
{% tab title="index.html.erb" %}

```text
<div data-reflex-root="[forward],[backward]">
  <input type="text" value="<%= @words %>" data-reflex="keyup->Example#words">
  <div forward><%= @words %></div>
  <div backward><%= @words&.reverse %></div>
</div>
```

{% endtab %}

{% tab title="example\_reflex.rb" %}

```ruby
  def words
    @words = element[:value]
  end
```

{% endtab %}
{% endtabs %}

### Permanent Elements

Add data-reflex-permanent to any element in your DOM, and it will be left unchanged by full-page Reflex updates and morph calls that re-render partials.

{% code title="index.html.erb" %}

```markup
<div data-reflex-permanent>
  <iframe src="https://ghbtns.com/github-btn.html?user=hopsoft&repo=stimulus_reflex&type=star&count=true" frameborder="0" scrolling="0" class="ghbtn"></iframe>
  <iframe src="https://ghbtns.com/github-btn.html?user=hopsoft&repo=stimulus_reflex&type=fork&count=true" frameborder="0" scrolling="0" class="ghbtn"></iframe>
</div>
```

{% endcode %}

### Selector Morphs

{% tabs %}

{% tab title="show.html.erb" %}

```markup
<header data-reflex="click->Example#change">
  <%= render partial: "path/to/foo", locals: {message: "Am I the medium or the massage?"} %>
</header>
```

{% endtab %}

{% tab title="\_foo.html.erb" %}

```markup
<div id="foo">
  <span class="spa"><%= message %></span>
</div>
```

{% endtab %}

{% tab title="app/reflexes/example\_reflex.rb" %}

```ruby
class ExampleReflex < ApplicationReflex
  def change
    morph "#foo", "Your muscles... they are so tight."
  end
end
```

{% endtab %}

{% endtabs %}

### Nothing Morph

Use `morph :nothing` in reflexes that do something on the server without updating the client.

{% code title="app/reflexes/example\_reflex.rb" %}

```ruby
class ExampleReflex < ApplicationReflex
  def change
    LongRunningJob.perform_later
    morph :nothing
  end
end
```

{% endcode %}

## Helpful Tips and Tricks

### Inheriting data-attributes from parent elements

You can use the `data-reflex-dataset="combined"` directive to scoop all data attributes up the DOM hierarchy and pass them as part of the Reflex payload.

{% tabs }
{% tab title="app/views/comments/new.html.erb" %}

```markup
<div data-post-id="<%= @post.id %>">
  <div data-category-id="<%= @category.id %>">
    <button data-reflex="click->Comment#create" data-reflex-dataset="combined">Create</button>
  </div>
</div>
```

{% endtab %}

{% tab title="app/reflexes/comments_reflex.rb" %}

```ruby
class CommentReflex < ApplicationReflex
  def create
    puts element.dataset["post-id"]
    puts element.dataset["category-id"]
  end
end
```

{% endtab %}

{% endtabs %}

### Aborting a Reflex

{% tab title="app/reflexes/comments_reflex.rb" %}

```ruby
class CommentReflex < ApplicationReflex
  def create
    puts element.dataset["post-id"]
    puts element.dataset["category-id"]
  end
end
```

{% endtab %}

[Aborting a Reflex](https://docs.stimulusreflex.com/reflexes#aborting-a-reflex)

data-reflex-permanent

Lots of good stuff here [Useful Patterns - StimulusReflex](https://docs.stimulusreflex.com/patterns)
logging, I18n, spinners, autofocus

## Callbacks

server + client side
