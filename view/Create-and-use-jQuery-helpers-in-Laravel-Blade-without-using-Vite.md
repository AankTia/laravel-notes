# Create and use jQuery helpers in Laravel Blade without using Vite

## Step 1: Include jQuery in your Layout

First, add jQuery to your main Blade layout file (typically `resources/views/layouts/app.blade.php`):

```php
<!DOCTYPE html>
<html>
<head>
    <!-- Other head elements -->
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    @yield('content')

    <!-- Your custom JS will go here -->
    @yield('scripts')
</body>
</html>
```

## Step 2: Create a Custom jQuery Helpers File

Create a new JavaScript file in your public directory, for example, `public/js/jquery-helpers.js`:

```javascript
// jQuery Helper Functions
(function ($) {
  // Example 1: Form data to JSON helper
  $.fn.serializeJSON = function () {
    var formData = {};
    this.serializeArray().forEach(function (item) {
      formData[item.name] = item.value;
    });
    return formData;
  };

  // Example 2: Flash message helper
  $.flashMessage = function (message, type = "success", duration = 3000) {
    var alert = $(
      '<div class="alert alert-' + type + '">' + message + "</div>"
    ).css({
      position: "fixed",
      top: "20px",
      right: "20px",
      "z-index": "9999",
      display: "none",
    });

    $("body").append(alert);
    alert
      .fadeIn(300)
      .delay(duration)
      .fadeOut(300, function () {
        $(this).remove();
      });
  };

  // Example 3: AJAX form submission helper
  $.fn.ajaxSubmit = function (options) {
    var defaults = {
      method: "POST",
      dataType: "json",
      beforeSend: function () {},
      success: function () {},
      error: function () {},
    };

    var settings = $.extend({}, defaults, options);
    var $form = this;

    return $form.on("submit", function (e) {
      e.preventDefault();

      $.ajax({
        url: $form.attr("action"),
        type: settings.method,
        data: $form.serialize(),
        dataType: settings.dataType,
        beforeSend: settings.beforeSend,
        success: settings.success,
        error: settings.error,
      });
    });
  };
})(jQuery);
```

## Step 3: Include Your Helpers in Layout

Add your custom helpers file after jQuery in your layout:

```html
<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
<script src="{{ asset('js/jquery-helpers.js') }}"></script>
```

## Step 4: Using the Helpers in Blade Views

Now you can use your helpers in any Blade view. For example:

```php
@extends('layouts.app')

@section('content')
<div class="container">
    <form id="contactForm" action="/contact" method="POST">
        @csrf
        <div class="form-group">
            <label for="name">Name</label>
            <input type="text" name="name" class="form-control">
        </div>
        <div class="form-group">
            <label for="email">Email</label>
            <input type="email" name="email" class="form-control">
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
    </form>
</div>
@endsection

@section('scripts')
<script>
    $(document).ready(function() {
        // Using the ajaxSubmit helper
        $('#contactForm').ajaxSubmit({
            beforeSend: function() {
                // Show loading indicator
            },
            success: function(response) {
                // Using the flash message helper
                $.flashMessage('Form submitted successfully!');

                // Using the serializeJSON helper
                console.log($('#contactForm').serializeJSON());
            },
            error: function(xhr) {
                $.flashMessage('Error submitting form', 'danger');
            }
        });
    });
</script>
@endsection
```

# Creating Your Own Helpers

To create your own jQuery helpers, follow this pattern in your `jquery-helpers.js` file:

```javascript
// For jQuery object methods (called on a jQuery object)
$.fn.yourHelperName = function (options) {
  // 'this' refers to the jQuery object
  return this.each(function () {
    // Implementation
  });
};

// For utility functions (called directly on jQuery)
$.yourUtilityFunction = function (options) {
  // Implementation
};
```
