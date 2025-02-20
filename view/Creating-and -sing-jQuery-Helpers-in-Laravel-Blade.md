# Creating and Using jQuery Helpers in Laravel Blade

This guide demonstrates how to create reusable jQuery helper functions and integrate them with Laravel Blade templates.

## Step 1: Set Up jQuery in Your Laravel Project

First, make sure jQuery is included in your project. You can use npm or include it directly.

### Option 1: Using NPM

```bash
npm install jquery
```

Then in your `resources/js/app.js`:

```javascript
import $ from 'jquery';
window.$ = window.jQuery = $;
```

### Option 2: Direct CDN Inclusion in Layout

```php
{{-- resources/views/layouts/app.blade.php --}}
<head>
    <!-- ... other head elements ... -->
    <script src="https://code.jquery.com/jquery-3.7.1.min.js" integrity="sha256-/JqT3SQfawRcv/BIHPThkBvs0OEvtFFmqPF/lYI/Cxo=" crossorigin="anonymous"></script>
</head>
```

## Step 2: Create jQuery Helper File

Create a dedicated helper file to contain your reusable jQuery functions:

```javascript
// resources/js/jquery-helpers.js

/**
 * jQuery UI Helpers
 * Collection of UI-related jQuery helper functions
 */
const UIHelpers = {
    /**
     * Fade in an element with custom settings
     * @param {string|jQuery} selector - Element selector or jQuery object
     * @param {number} duration - Animation duration in ms
     * @param {Function} callback - Callback function after animation
     */
    fadeInElement: function(selector, duration = 400, callback = null) {
        $(selector).fadeIn(duration, callback);
    },
    
    /**
     * Fade out an element with custom settings
     * @param {string|jQuery} selector - Element selector or jQuery object
     * @param {number} duration - Animation duration in ms
     * @param {Function} callback - Callback function after animation
     */
    fadeOutElement: function(selector, duration = 400, callback = null) {
        $(selector).fadeOut(duration, callback);
    },
    
    /**
     * Show a notification that automatically disappears
     * @param {string} message - Message to display
     * @param {string} type - Notification type (success, error, warning, info)
     * @param {number} duration - How long to show notification (ms)
     */
    showNotification: function(message, type = 'info', duration = 3000) {
        // Create notification element if it doesn't exist
        if ($('#notification-container').length === 0) {
            $('body').append('<div id="notification-container"></div>');
        }
        
        // Create unique ID for this notification
        const id = 'notification-' + Date.now();
        
        // Create and append notification
        const $notification = $(`
            <div id="${id}" class="notification notification-${type}">
                <div class="notification-content">${message}</div>
            </div>
        `);
        
        $('#notification-container').append($notification);
        
        // Show notification with animation
        $notification.fadeIn(300);
        
        // Auto-remove after duration
        setTimeout(function() {
            $notification.fadeOut(300, function() {
                $(this).remove();
            });
        }, duration);
        
        return id;
    }
};

/**
 * jQuery Form Helpers
 * Collection of form-related jQuery helper functions
 */
const FormHelpers = {
    /**
     * Serialize form data to JSON object
     * @param {string|jQuery} formSelector - Form selector or jQuery object
     * @returns {object} Form data as JSON object
     */
    serializeFormToJson: function(formSelector) {
        const formData = {};
        const $form = $(formSelector);
        
        $form.serializeArray().forEach(function(field) {
            formData[field.name] = field.value;
        });
        
        return formData;
    },
    
    /**
     * Validate form fields using specified rules
     * @param {string|jQuery} formSelector - Form selector or jQuery object
     * @param {object} rules - Validation rules
     * @returns {boolean} Whether form is valid
     */
    validateForm: function(formSelector, rules) {
        const $form = $(formSelector);
        let isValid = true;
        
        // Clear previous errors
        $form.find('.error-message').remove();
        $form.find('.is-invalid').removeClass('is-invalid');
        
        // Check each rule
        Object.entries(rules).forEach(([fieldName, fieldRules]) => {
            const $field = $form.find(`[name="${fieldName}"]`);
            const value = $field.val();
            
            // Required check
            if (fieldRules.required && !value.trim()) {
                this._showFieldError($field, fieldRules.required === true ? 
                    'This field is required' : fieldRules.required);
                isValid = false;
                return;
            }
            
            // Minimum length check
            if (fieldRules.minLength && value.length < fieldRules.minLength) {
                this._showFieldError($field, `Minimum length is ${fieldRules.minLength} characters`);
                isValid = false;
                return;
            }
            
            // Email format check
            if (fieldRules.email && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
                this._showFieldError($field, 'Please enter a valid email address');
                isValid = false;
                return;
            }
            
            // Custom validation function
            if (fieldRules.custom && typeof fieldRules.custom.validate === 'function') {
                if (!fieldRules.custom.validate(value)) {
                    this._showFieldError($field, fieldRules.custom.message || 'Invalid input');
                    isValid = false;
                    return;
                }
            }
        });
        
        return isValid;
    },
    
    /**
     * Private method to show field error
     * @param {jQuery} $field - Field element
     * @param {string} message - Error message
     * @private
     */
    _showFieldError: function($field, message) {
        $field.addClass('is-invalid');
        $field.after(`<div class="error-message">${message}</div>`);
    },
    
    /**
     * Submit form via AJAX
     * @param {string|jQuery} formSelector - Form selector or jQuery object
     * @param {object} options - AJAX options
     */
    ajaxSubmit: function(formSelector, options = {}) {
        const $form = $(formSelector);
        const formData = this.serializeFormToJson($form);
        const defaultOptions = {
            url: $form.attr('action'),
            method: $form.attr('method') || 'POST',
            data: formData,
            dataType: 'json',
            beforeSend: function() {
                $form.find('button[type="submit"]').prop('disabled', true).addClass('loading');
            },
            complete: function() {
                $form.find('button[type="submit"]').prop('disabled', false).removeClass('loading');
            },
            success: function(response) {
                UIHelpers.showNotification('Form submitted successfully', 'success');
            },
            error: function(xhr) {
                UIHelpers.showNotification('An error occurred', 'error');
                
                // Handle Laravel validation errors
                if (xhr.status === 422 && xhr.responseJSON && xhr.responseJSON.errors) {
                    Object.entries(xhr.responseJSON.errors).forEach(([field, errors]) => {
                        const $field = $form.find(`[name="${field}"]`);
                        FormHelpers._showFieldError($field, errors[0]);
                    });
                }
            }
        };
        
        // Merge defaults with user options
        const ajaxOptions = $.extend({}, defaultOptions, options);
        
        // Execute the AJAX request
        $.ajax(ajaxOptions);
    }
};

/**
 * jQuery Data Helpers
 * Collection of data-handling jQuery helper functions
 */
const DataHelpers = {
    /**
     * Load data via AJAX and populate elements
     * @param {string} url - URL to fetch data from
     * @param {object} options - Configuration options
     */
    loadData: function(url, options = {}) {
        const defaults = {
            selector: '#data-container',
            method: 'GET',
            dataType: 'json',
            template: null, // Function that returns HTML string
            loadingTemplate: '<div class="loading">Loading...</div>',
            errorTemplate: '<div class="error">Failed to load data</div>',
            emptyTemplate: '<div class="empty">No data available</div>',
            beforeLoad: null,
            afterLoad: null
        };
        
        const settings = $.extend({}, defaults, options);
        const $container = $(settings.selector);
        
        // Show loading state
        if (settings.loadingTemplate) {
            $container.html(settings.loadingTemplate);
        }
        
        // Call beforeLoad callback if provided
        if (typeof settings.beforeLoad === 'function') {
            settings.beforeLoad($container);
        }
        
        // Make AJAX request
        $.ajax({
            url: url,
            method: settings.method,
            dataType: settings.dataType,
            success: function(response) {
                // Handle empty data
                if (!response || (Array.isArray(response) && response.length === 0)) {
                    $container.html(settings.emptyTemplate);
                    return;
                }
                
                // Use template function if provided
                if (typeof settings.template === 'function') {
                    $container.html(settings.template(response));
                } else {
                    // Default handling for JSON data
                    let html = '';
                    
                    if (Array.isArray(response)) {
                        response.forEach(item => {
                            html += `<div class="data-item">${JSON.stringify(item)}</div>`;
                        });
                    } else {
                        html = `<div class="data-item">${JSON.stringify(response)}</div>`;
                    }
                    
                    $container.html(html);
                }
                
                // Call afterLoad callback if provided
                if (typeof settings.afterLoad === 'function') {
                    settings.afterLoad($container, response);
                }
            },
            error: function(xhr) {
                $container.html(settings.errorTemplate);
                console.error('Error loading data:', xhr);
            }
        });
    },
    
    /**
     * Format a number as currency
     * @param {number} amount - Amount to format
     * @param {string} locale - Locale code (default: 'en-US')
     * @param {string} currency - Currency code (default: 'USD')
     * @returns {string} Formatted currency string
     */
    formatCurrency: function(amount, locale = 'en-US', currency = 'USD') {
        return new Intl.NumberFormat(locale, {
            style: 'currency',
            currency: currency
        }).format(amount);
    }
};

// Export helpers
export { UIHelpers, FormHelpers, DataHelpers };
```

## Step 3: Register jQuery Helpers Globally

In your `resources/js/app.js`:

```javascript
import $ from 'jquery';
window.$ = window.jQuery = $;

// Import jQuery helpers
import { UIHelpers, FormHelpers, DataHelpers } from './jquery-helpers';

// Make helpers globally available
window.UIHelpers = UIHelpers;
window.FormHelpers = FormHelpers;
window.DataHelpers = DataHelpers;

// Any other app initialization code...
```

## Step 4: Include in Your Laravel Layout

```php
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <title>@yield('title', 'My Laravel App')</title>
    
    <!-- Styles -->
    @vite(['resources/css/app.css'])
    
    <!-- Add some basic notification styling -->
    <style>
        #notification-container {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 9999;
        }
        .notification {
            margin-bottom: 10px;
            padding: 15px;
            border-radius: 4px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
            display: none;
        }
        .notification-success { background: #d4edda; color: #155724; }
        .notification-error { background: #f8d7da; color: #721c24; }
        .notification-warning { background: #fff3cd; color: #856404; }
        .notification-info { background: #d1ecf1; color: #0c5460; }
        
        .error-message {
            color: #dc3545;
            font-size: 0.875rem;
            margin-top: 0.25rem;
        }
        .is-invalid {
            border-color: #dc3545;
        }
        .loading:after {
            content: "...";
            animation: dots 1.5s steps(5, end) infinite;
        }
        @keyframes dots {
            0%, 20% { content: "."; }
            40% { content: ".."; }
            60%, 100% { content: "..."; }
        }
    </style>
</head>
<body>
    <header>
        @include('partials.nav')
    </header>
    
    <main>
        @yield('content')
    </main>
    
    <footer>
        &copy; {{ date('Y') }} My Laravel App
    </footer>
    
    <!-- CSRF Token for AJAX Requests -->
    <script>
        $.ajaxSetup({
            headers: {
                'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
            }
        });
    </script>
    
    <!-- Scripts -->
    @vite(['resources/js/app.js'])
    @stack('scripts')
</body>
</html>
```

## Step 5: Practical Examples in Blade Templates

### Example 1: Contact Form with jQuery Validation

```php
{{-- resources/views/contact.blade.php --}}
@extends('layouts.app')

@section('title', 'Contact Us')

@section('content')
<div class="container">
    <h1>Contact Us</h1>
    
    <form id="contact-form" action="{{ route('contact.submit') }}" method="POST">
        @csrf
        
        <div class="form-group">
            <label for="name">Name</label>
            <input type="text" id="name" name="name" class="form-control">
        </div>
        
        <div class="form-group">
            <label for="email">Email</label>
            <input type="email" id="email" name="email" class="form-control">
        </div>
        
        <div class="form-group">
            <label for="subject">Subject</label>
            <input type="text" id="subject" name="subject" class="form-control">
        </div>
        
        <div class="form-group">
            <label for="message">Message</label>
            <textarea id="message" name="message" rows="5" class="form-control"></textarea>
        </div>
        
        <button type="submit" class="btn btn-primary">Send Message</button>
    </form>
    
    <div id="form-response"></div>
</div>

@push('scripts')
<script>
    $(document).ready(function() {
        // Define validation rules
        const validationRules = {
            name: {
                required: true,
                minLength: a3
            },
            email: {
                required: true,
                email: true
            },
            subject: {
                required: true,
                minLength: 5
            },
            message: {
                required: true,
                minLength: 10,
                custom: {
                    validate: function(value) {
                        // Custom validation: Message shouldn't just be all caps
                        return value !== value.toUpperCase();
                    },
                    message: 'Please don\'t shout. Turn off caps lock.'
                }
            }
        };
        
        // Form submission
        $('#contact-form').on('submit', function(e) {
            e.preventDefault();
            
            // Validate form
            if (FormHelpers.validateForm(this, validationRules)) {
                // Submit form via AJAX
                FormHelpers.ajaxSubmit(this, {
                    success: function(response) {
                        $('#form-response').html('<div class="alert alert-success">Message sent successfully!</div>');
                        $('#contact-form')[0].reset();
                    },
                    error: function(xhr) {
                        if (xhr.status !== 422) { // Not validation error
                            $('#form-response').html('<div class="alert alert-danger">Failed to send message. Please try again.</div>');
                        }
                    }
                });
            }
        });
        
        // Example of using UI Helper for notifications
        $('#contact-form input, #contact-form textarea').on('focus', function() {
            if ($(this).hasClass('is-invalid')) {
                UIHelpers.showNotification('This field contains errors', 'warning', 2000);
            }
        });
    });
</script>
@endpush
@endsection
```

### Example 2: Product Listing with AJAX Loading

```php
{{-- resources/views/products/index.blade.php --}}
@extends('layouts.app')

@section('title', 'Our Products')

@section('content')
<div class="container">
    <h1>Our Products</h1>
    
    <div class="filters mb-4">
        <div class="row">
            <div class="col-md-3">
                <select id="category-filter" class="form-control">
                    <option value="">All Categories</option>
                    @foreach($categories as $category)
                        <option value="{{ $category->id }}">{{ $category->name }}</option>
                    @endforeach
                </select>
            </div>
            <div class="col-md-3">
                <select id="sort-filter" class="form-control">
                    <option value="newest">Newest First</option>
                    <option value="price_low">Price: Low to High</option>
                    <option value="price_high">Price: High to Low</option>
                    <option value="best_selling">Best Selling</option>
                </select>
            </div>
            <div class="col-md-6">
                <div class="input-group">
                    <input type="text" id="search-filter" class="form-control" placeholder="Search products...">
                    <div class="input-group-append">
                        <button id="search-btn" class="btn btn-primary">Search</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
    
    <div id="products-container" class="row"></div>
    
    <div id="load-more-container" class="text-center mt-4">
        <button id="load-more" class="btn btn-secondary">Load More</button>
    </div>
</div>

@push('scripts')
<script>
    $(document).ready(function() {
        // Product data state
        let currentPage = 1;
        let hasMorePages = true;
        
        // Product template function
        const productTemplate = function(data) {
            if (!data.products || data.products.length === 0) {
                return '<div class="col-12 text-center">No products found.</div>';
            }
            
            let html = '';
            
            data.products.forEach(product => {
                html += `
                <div class="col-md-4 mb-4">
                    <div class="card product-card">
                        <img src="${product.image}" class="card-img-top" alt="${product.name}">
                        <div class="card-body">
                            <h5 class="card-title">${product.name}</h5>
                            <p class="card-text">${product.description.substring(0, 100)}${product.description.length > 100 ? '...' : ''}</p>
                            <div class="d-flex justify-content-between align-items-center">
                                <span class="price">${DataHelpers.formatCurrency(product.price)}</span>
                                <button class="btn btn-sm btn-primary add-to-cart" data-product-id="${product.id}">Add to Cart</button>
                            </div>
                        </div>
                    </div>
                </div>
                `;
            });
            
            return html;
        };
        
        // Initial load
        const loadProducts = function(append = false) {
            const url = "{{ route('api.products') }}";
            const params = {
                page: currentPage,
                category: $('#category-filter').val(),
                sort: $('#sort-filter').val(),
                search: $('#search-filter').val()
            };
            
            DataHelpers.loadData(`${url}?${$.param(params)}`, {
                selector: '#products-container',
                loadingTemplate: '<div class="col-12 text-center"><div class="loading">Loading products</div></div>',
                template: productTemplate,
                beforeLoad: function($container) {
                    if (!append) {
                        $container.empty();
                    }
                    $('#load-more').prop('disabled', true);
                },
                afterLoad: function($container, data) {
                    // Update pagination state
                    hasMorePages = data.has_more_pages;
                    $('#load-more').prop('disabled', !hasMorePages);
                    
                    // Add animation to newly loaded products
                    if (append) {
                        $container.find('.product-card').slice(-data.products.length).hide().fadeIn(500);
                    } else {
                        $container.find('.product-card').hide().fadeIn(500);
                    }
                    
                    // Show result count notification
                    UIHelpers.showNotification(`Showing ${data.products.length} products`, 'info', 2000);
                }
            });
        };
        
        // Event handlers
        $('#category-filter, #sort-filter').on('change', function() {
            currentPage = 1;
            loadProducts();
        });
        
        $('#search-btn').on('click', function() {
            currentPage = 1;
            loadProducts();
        });
        
        // Enter key in search field
        $('#search-filter').on('keypress', function(e) {
            if (e.which === 13) {
                currentPage = 1;
                loadProducts();
            }
        });
        
        $('#load-more').on('click', function() {
            if (hasMorePages) {
                currentPage++;
                loadProducts(true);
            }
        });
        
        // Add to cart functionality (event delegation)
        $(document).on('click', '.add-to-cart', function() {
            const productId = $(this).data('product-id');
            const $button = $(this);
            
            $button.prop('disabled', true).text('Adding...');
            
            $.ajax({
                url: "{{ route('cart.add') }}",
                method: 'POST',
                data: { product_id: productId, quantity: 1 },
                dataType: 'json',
                success: function(response) {
                    $button.text('Added!').addClass('btn-success').removeClass('btn-primary');
                    
                    setTimeout(function() {
                        $button.text('Add to Cart').addClass('btn-primary').removeClass('btn-success').prop('disabled', false);
                    }, 2000);
                    
                    // Show notification
                    UIHelpers.showNotification('Product added to cart', 'success');
                    
                    // Update cart count (assuming you have a cart badge in the header)
                    $('#cart-count').text(response.cart_count);
                },
                error: function() {
                    $button.prop('disabled', false).text('Add to Cart');
                    UIHelpers.showNotification('Failed to add product to cart', 'error');
                }
            });
        });
        
        // Initial load
        loadProducts();
    });
</script>
@endpush
@endsection
```

### Example 3: Modal Confirmation Dialog Helper

```php
{{-- resources/views/admin/users/index.blade.php --}}
@extends('layouts.app')

@section('title', 'User Management')

@section('content')
<div class="container">
    <h1>User Management</h1>
    
    <table class="table">
        <thead>
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Email</th>
                <th>Created</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            @foreach($users as $user)
            <tr>
                <td>{{ $user->id }}</td>
                <td>{{ $user->name }}</td>
                <td>{{ $user->email }}</td>
                <td>{{ $user->created_at->format('M d, Y') }}</td>
                <td>
                    <button class="btn btn-sm btn-primary edit-user" data-user-id="{{ $user->id }}">Edit</button>
                    <button class="btn btn-sm btn-danger delete-user" data-user-id="{{ $user->id }}" data-user-name="{{ $user->name }}">Delete</button>
                </td>
            </tr>
            @endforeach
        </tbody>
    </table>
</div>

@push('scripts')
<script>
    $(document).ready(function() {
        /**
         * Create a dialog helper
         */
        const DialogHelper = {
            /**
             * Show a confirmation dialog
             * @param {object} options - Dialog options
             * @returns {Promise} Resolves when confirmed, rejects when canceled
             */
            confirm: function(options) {
                const defaults = {
                    title: 'Confirm Action',
                    message: 'Are you sure you want to proceed?',
                    confirmText: 'Confirm',
                    cancelText: 'Cancel',
                    confirmButtonClass: 'btn-primary',
                    size: 'modal-md'
                };
                
                const settings = $.extend({}, defaults, options);
                const dialogId = 'confirm-dialog-' + Date.now();
                
                // Create modal HTML
                const modal = `
                <div class="modal fade" id="${dialogId}" tabindex="-1" role="dialog" aria-hidden="true">
                    <div class="modal-dialog ${settings.size}" role="document">
                        <div class="modal-content">
                            <div class="modal-header">
                                <h5 class="modal-title">${settings.title}</h5>
                                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                    <span aria-hidden="true">&times;</span>
                                </button>
                            </div>
                            <div class="modal-body">
                                ${settings.message}
                            </div>
                            <div class="modal-footer">
                                <button type="button" class="btn btn-secondary cancel-btn" data-dismiss="modal">${settings.cancelText}</button>
                                <button type="button" class="btn ${settings.confirmButtonClass} confirm-btn">${settings.confirmText}</button>
                            </div>
                        </div>
                    </div>
                </div>
                `;
                
                // Append to body and initialize
                $('body').append(modal);
                const $modal = $(`#${dialogId}`);
                
                return new Promise((resolve, reject) => {
                    $modal.modal('show');
                    
                    // Handle confirmation
                    $modal.find('.confirm-btn').on('click', function() {
                        $modal.modal('hide');
                        resolve(true);
                    });
                    
                    // Handle cancellation
                    $modal.find('.cancel-btn, .close').on('click', function() {
                        reject(false);
                    });
                    
                    // Clean up after hiding
                    $modal.on('hidden.bs.modal', function() {
                        $modal.remove();
                    });
                });
            }
        };
        
        // Delete user button handler
        $('.delete-user').on('click', function() {
            const userId = $(this).data('user-id');
            const userName = $(this).data('user-name');
            const $row = $(this).closest('tr');
            
            DialogHelper.confirm({
                title: 'Delete User',
                message: `Are you sure you want to delete user <strong>${userName}</strong>? This action cannot be undone.`,
                confirmText: 'Delete User',
                confirmButtonClass: 'btn-danger'
            })
            .then(() => {
                // User confirmed deletion
                $.ajax({
                    url: `/admin/users/${userId}`,
                    method: 'DELETE',
                    dataType: 'json',
                    beforeSend: function() {
                        // Show loading state
                        UIHelpers.showNotification('Deleting user...', 'info');
                    },
                    success: function(response) {
                        // Fade out and remove the table row
                        UIHelpers.fadeOutElement($row, 400, function() {
                            $row.remove();
                            UIHelpers.showNotification('User deleted successfully', 'success');
                        });
                    },
                    error: function() {
                        UIHelpers.showNotification('Failed to delete user', 'error');
                    }
                });
            })
            .catch(() => {
                // User canceled - do nothing
            });
        });
        
        // Edit user button handler
        $('.edit-user').on('click', function() {
            const userId = $(this).data('user-id');
            
            // Show loading notification
            UIHelpers.showNotification('Loading user data...', 'info');
            
            // Load user data
            DataHelpers.loadData(`/admin/users/${userId}/edit`, {
                selector: '#edit-user-container',
                afterLoad: function($container, data) {
                    // Assuming data contains user object
                    UIHelpers.showNotification('User data loaded', 'success');
                    
                    // Example of form initialization with loaded data
                    $('#edit-user-form').each(function() {
                        // Populate form fields
                        $('#edit-name').val(data.name);
                        $('#edit-email').val(data.email);
                        
                        // Handle form submission
                        $(this).on('submit', function(e) {
                            e.preventDefault();
                            
                            // Validate form
                            const validationRules = {
                                name: { required: true, minLength: 3 },
                                email: { required: true, email: true }
                            };
                            
                            if (FormHelpers.validateForm(this, validationRules)) {
                                FormHelpers.ajaxSubmit(this, {
                                    method: 'PUT',
                                    url: `/admin/users/${userId}`,
                                    success: function(response) {
                                        UIHelpers.showNotification('User updated successfully', 'success');
                                        // Reload page after short delay
                                        setTimeout(() => window.location.reload(), 1000);
                                    }
                                });
                            }
                        });
                    });
                }
            });
        });
    });
</script>
@endpush
@endsection
```

## Step 