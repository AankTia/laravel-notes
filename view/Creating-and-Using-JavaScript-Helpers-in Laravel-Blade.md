# Creating and Using JavaScript Helpers in Laravel Blade

This guide demonstrates how to create reusable JavaScript helper functions and integrate them with Laravel Blade templates.

## Step 1: Set Up Your JavaScript Helper File

First, create a dedicated helper file to contain your reusable functions:

```javascript
// resources/js/helpers.js

/**
 * Collection of utility functions for DOM manipulation
 */
export const DOMHelpers = {
    /**
     * Selects an element and adds a class to it
     * @param {string} selector - CSS selector
     * @param {string} className - Class to add
     */
    addClass(selector, className) {
        const element = document.querySelector(selector);
        if (element) element.classList.add(className);
    },
    
    /**
     * Creates an element with attributes and content
     * @param {string} tag - HTML tag name
     * @param {object} attributes - Element attributes
     * @param {string} content - Inner text content
     * @returns {HTMLElement} The created element
     */
    createElement(tag, attributes = {}, content = '') {
        const element = document.createElement(tag);
        
        // Set attributes
        Object.entries(attributes).forEach(([key, value]) => {
            element.setAttribute(key, value);
        });
        
        // Set content if provided
        if (content) element.textContent = content;
        
        return element;
    }
};

/**
 * Collection of formatting utility functions
 */
export const FormatHelpers = {
    /**
     * Format a number as currency
     * @param {number} amount - Amount to format
     * @param {string} locale - Locale code (default: 'en-US')
     * @param {string} currency - Currency code (default: 'USD')
     * @returns {string} Formatted currency string
     */
    currency(amount, locale = 'en-US', currency = 'USD') {
        return new Intl.NumberFormat(locale, {
            style: 'currency',
            currency: currency
        }).format(amount);
    },
    
    /**
     * Format a date
     * @param {string|Date} date - Date to format
     * @param {object} options - Intl.DateTimeFormat options
     * @returns {string} Formatted date string
     */
    date(date, options = { dateStyle: 'medium' }) {
        return new Intl.DateTimeFormat('en-US', options).format(new Date(date));
    }
};

/**
 * Collection of validation utility functions
 */
export const ValidateHelpers = {
    /**
     * Validates an email address
     * @param {string} email - Email to validate
     * @returns {boolean} Whether the email is valid
     */
    isValidEmail(email) {
        const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return regex.test(email);
    },
    
    /**
     * Checks if a string has a minimum length
     * @param {string} str - String to check
     * @param {number} minLength - Minimum required length
     * @returns {boolean} Whether the string meets the minimum length
     */
    hasMinLength(str, minLength) {
        return str.length >= minLength;
    }
};
```

## Step 2: Make Helpers Available to Your Application

### Option 1: Register Globally via app.js

```javascript
// resources/js/app.js
import { DOMHelpers, FormatHelpers, ValidateHelpers } from './helpers';

// Register helpers in global window object
window.DOMHelpers = DOMHelpers;
window.FormatHelpers = FormatHelpers;
window.ValidateHelpers = ValidateHelpers;

// Other app initialization code...
```

### Option 2: Use Module Imports Directly (Modern Approach)

Use ES modules directly in your templates (for browsers that support module scripts).

## Step 3: Include in Your Laravel Layout

```php
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('title', 'My Laravel App')</title>
    
    <!-- Styles -->
    @vite(['resources/css/app.css'])
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
    
    <!-- Scripts -->
    @vite(['resources/js/app.js'])
    @stack('scripts')
</body>
</html>
```

## Step 4: Practical Examples in Blade Templates

### Example 1: Product Card Component

```php
{{-- resources/views/components/product-card.blade.php --}}
<div class="product-card" data-product-id="{{ $product->id }}" data-price="{{ $product->price }}">
    <img src="{{ $product->image_url }}" alt="{{ $product->name }}">
    <h3>{{ $product->name }}</h3>
    <p class="description">{{ $product->description }}</p>
    <div class="price"></div>
    <button class="add-to-cart">Add to Cart</button>
</div>

@once
@push('scripts')
<script>
    document.addEventListener('DOMContentLoaded', () => {
        // Format all product prices on the page
        document.querySelectorAll('.product-card').forEach(card => {
            const price = parseFloat(card.dataset.price);
            const priceElement = card.querySelector('.price');
            priceElement.textContent = FormatHelpers.currency(price);
            
            // Truncate long descriptions
            const description = card.querySelector('.description');
            if (description.textContent.length > 100) {
                description.setAttribute('title', description.textContent);
                description.textContent = description.textContent.substring(0, 100) + '...';
            }
            
            // Add event listener to cart button
            card.querySelector('.add-to-cart').addEventListener('click', () => {
                // Add active class to button
                DOMHelpers.addClass('.add-to-cart', 'active');
                
                // Create notification
                const notification = DOMHelpers.createElement('div', {
                    class: 'notification',
                    'data-product-id': card.dataset.productId
                }, 'Added to cart');
                
                document.body.appendChild(notification);
                
                // Remove notification after 3 seconds
                setTimeout(() => {
                    notification.remove();
                }, 3000);
            });
        });
    });
</script>
@endpush
@endonce
```

### Example 2: Contact Form Validation

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
            <input type="text" id="name" name="name" required>
            <span class="error" id="name-error"></span>
        </div>
        
        <div class="form-group">
            <label for="email">Email</label>
            <input type="email" id="email" name="email" required>
            <span class="error" id="email-error"></span>
        </div>
        
        <div class="form-group">
            <label for="message">Message</label>
            <textarea id="message" name="message" rows="5" required></textarea>
            <span class="error" id="message-error"></span>
        </div>
        
        <button type="submit">Send Message</button>
    </form>
    
    <div id="submission-date" class="mt-4"></div>
</div>

@push('scripts')
<script>
    document.addEventListener('DOMContentLoaded', () => {
        const form = document.getElementById('contact-form');
        
        // Display current date using our formatter
        const submissionDateEl = document.getElementById('submission-date');
        submissionDateEl.textContent = `Today is ${FormatHelpers.date(new Date(), {
            weekday: 'long',
            year: 'numeric',
            month: 'long',
            day: 'numeric'
        })}`;
        
        form.addEventListener('submit', function(event) {
            event.preventDefault();
            
            // Reset errors
            document.querySelectorAll('.error').forEach(el => el.textContent = '');
            
            // Validate form using our helpers
            let isValid = true;
            
            // Validate name
            const nameInput = document.getElementById('name');
            if (!ValidateHelpers.hasMinLength(nameInput.value, 3)) {
                document.getElementById('name-error').textContent = 'Name must be at least 3 characters';
                isValid = false;
            }
            
            // Validate email
            const emailInput = document.getElementById('email');
            if (!ValidateHelpers.isValidEmail(emailInput.value)) {
                document.getElementById('email-error').textContent = 'Please enter a valid email address';
                isValid = false;
            }
            
            // Validate message
            const messageInput = document.getElementById('message');
            if (!ValidateHelpers.hasMinLength(messageInput.value, 10)) {
                document.getElementById('message-error').textContent = 'Message must be at least 10 characters';
                isValid = false;
            }
            
            // Submit form if valid
            if (isValid) {
                // Create loading indicator using DOM helper
                const loadingEl = DOMHelpers.createElement('div', {
                    class: 'loading-indicator'
                }, 'Sending message...');
                
                form.appendChild(loadingEl);
                
                // Normally would submit form here
                // form.submit();
                
                // For demo, simulate submission
                setTimeout(() => {
                    loadingEl.remove();
                    form.innerHTML = '<div class="success-message">Thank you for your message!</div>';
                }, 1500);
            }
        });
    });
</script>
@endpush
@endsection
```

### Example 3: Direct Module Import (Modern Approach)

```php
{{-- resources/views/dashboard.blade.php --}}
@extends('layouts.app')

@section('content')
<div class="dashboard">
    <h1>Dashboard</h1>
    
    <div class="stats-container">
        <div class="stat" id="revenue-stat" data-value="{{ $monthlyRevenue }}"></div>
        <div class="stat" id="orders-stat" data-value="{{ $orderCount }}"></div>
        <div class="stat" id="last-order" data-date="{{ $lastOrderDate }}"></div>
    </div>
</div>

@push('scripts')
<script type="module">
    // Direct import without going through global window object
    import { FormatHelpers, DOMHelpers } from "{{ Vite::asset('resources/js/helpers.js') }}";
    
    // Format revenue
    const revenueStat = document.getElementById('revenue-stat');
    const revenue = parseFloat(revenueStat.dataset.value);
    revenueStat.innerHTML = `
        <h3>Monthly Revenue</h3>
        <p class="value">${FormatHelpers.currency(revenue)}</p>
    `;
    
    // Format order count
    const ordersStat = document.getElementById('orders-stat');
    const orderCount = parseInt(ordersStat.dataset.value);
    ordersStat.innerHTML = `
        <h3>Total Orders</h3>
        <p class="value">${orderCount}</p>
    `;
    
    // Format date
    const lastOrderStat = document.getElementById('last-order');
    const lastOrderDate = lastOrderStat.dataset.date;
    lastOrderStat.innerHTML = `
        <h3>Last Order</h3>
        <p class="value">${FormatHelpers.date(lastOrderDate, { 
            dateStyle: 'full', 
            timeStyle: 'short' 
        })}</p>
    `;
    
    // Create a refresh button
    const refreshButton = DOMHelpers.createElement('button', {
        id: 'refresh-stats',
        class: 'refresh-button'
    }, 'Refresh Stats');
    
    document.querySelector('.stats-container').after(refreshButton);
    
    // Add click event
    refreshButton.addEventListener('click', () => {
        // Show loading indicator
        refreshButton.textContent = 'Loading...';
        refreshButton.disabled = true;
        
        // Simulate API call
        setTimeout(() => {
            refreshButton.textContent = 'Refresh Stats';
            refreshButton.disabled = false;
            
            // Show notification
            const notification = DOMHelpers.createElement('div', {
                class: 'notification success'
            }, 'Stats refreshed successfully');
            
            document.body.appendChild(notification);
            
            setTimeout(() => notification.remove(), 3000);
        }, 1500);
    });
</script>
@endpush
@endsection
```

## Step 5: Using with Alpine.js (Optional)

If your project uses Alpine.js, you can integrate your helpers:

```php
{{-- resources/views/components/pricing-table.blade.php --}}
<div x-data="pricingComponent()" class="pricing-table">
    <div class="plans">
        <template x-for="plan in plans" :key="plan.id">
            <div class="plan" :class="{'selected': selectedPlan === plan.id}">
                <h3 x-text="plan.name"></h3>
                <p class="price" x-text="formatPrice(plan.price)"></p>
                <ul>
                    <template x-for="feature in plan.features" :key="feature">
                        <li x-text="feature"></li>
                    </template>
                </ul>
                <button @click="selectPlan(plan.id)">Select Plan</button>
            </div>
        </template>
    </div>
</div>

@push('scripts')
<script>
    // Import helpers only where needed
    import { FormatHelpers } from "{{ Vite::asset('resources/js/helpers.js') }}";

    function pricingComponent() {
        return {
            plans: [
                {
                    id: 1,
                    name: 'Basic',
                    price: 9.99,
                    features: ['5 Projects', '2GB Storage', 'Email Support']
                },
                {
                    id: 2,
                    name: 'Pro',
                    price: 19.99,
                    features: ['Unlimited Projects', '10GB Storage', 'Priority Support', 'Advanced Analytics']
                },
                {
                    id: 3,
                    name: 'Enterprise',
                    price: 49.99,
                    features: ['Unlimited Everything', 'Dedicated Account Manager', 'Custom Integration', '24/7 Phone Support']
                }
            ],
            selectedPlan: null,
            formatPrice(price) {
                // Use our imported helper
                return FormatHelpers.currency(price, 'en-US', 'USD');
            },
            selectPlan(planId) {
                this.selectedPlan = planId;
                // Additional selection logic...
            }
        };
    }
</script>
@endpush
```

## Conclusion

By organizing your JavaScript code in helper modules, you create a more maintainable codebase with reusable functionality across your Laravel Blade templates. This approach keeps your views clean while providing powerful JavaScript functionality.
