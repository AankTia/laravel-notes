# Logic in Laravel Blade templates

In Laravel Blade templates, you can incorporate PHP logic in several ways. Here are the main approaches:

## Basic PHP Expressions

You can use `{{ }}` to echo a PHP variable or simple expression:

```php
{{ $variable }}
{{ count($items) }}
{{ $price * 1.2 }}
```

## Raw PHP Blocks

For more complex PHP logic, you can use `@php` directive:

```php
@php
    $total = 0;
    foreach ($items as $item) {
        $total += $item->price;
    }
@endphp

Total: ${{ $total }}
```

## Control Structures

Blade provides various directives for common control structures:

### Conditionals
```php
@if ($condition)
    This will display if condition is true
@elseif ($anotherCondition)
    This will display if another condition is true
@else
    This will display if all conditions are false
@endif

@unless ($user->isAdmin())
    You are not an admin.
@endunless
```

### Loops
```php
@foreach ($users as $user)
    <p>{{ $user->name }}</p>
@endforeach

@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@while ($condition)
    <p>I'm looping until the condition is false.</p>
@endwhile
```

### Switch Statements
```php
@switch($case)
    @case(1)
        First case...
        @break
    @case(2)
        Second case...
        @break
    @default
        Default case...
@endswitch
```

## Including Other Views

```php
@include('view.name', ['variable' => 'value'])

@includeIf('view.name', ['variable' => 'value'])

@includeWhen($boolean, 'view.name', ['variable' => 'value'])

@includeFirst(['custom.admin', 'admin'], ['variable' => 'value'])
```

## Extending Layouts

```php
@extends('layouts.app')

@section('content')
    Content goes here
@endsection
```

Is there any specific Blade functionality you'd like me to elaborate on?