# Laravel — Bester Practices

This is a commentary on https://github.com/alexeymezenin/laravel-best-practices 

When you're learning to code, the first step is to make code work. To accomplish something at all.
But once you create a slightly larger app, the code becomes either into a huge blob called the Ball of Mud.
Or, if you've already picked up some techniques to redirect the flow (e.g. functions or classes) and the DRY concept, you can even tangle it into some Spaghetti code!

The second step is to learn how to organize your code so that your project isn't total mess.
For that you usually learn various techniques like DRY, SOLID and so on.
At this point you should also learn how and when to apply these techniques.
Sadly, most of the time instead of teaching the tools along with their uses, you just get a huge toolbox with a "BEST PRACTICES" label.
The result? Devs at this point in their journey make their code either into a nearly infinite stack of layers (Lasagna code) or split it into a thousand isolated microclasses — Ravioli code.
The Ravioli is the ultimate application of the Single Responsibility principle: each dumpling is only responsible for one thing — invoking the next dumpling.

The reality is that the third step is to learn how and when to apply the techniques you've learned as a junior.
And I'll try to provide brief tips on that here.
Nothing comprehensive, I'll just give a small commentary on each of the [best practices](https://github.com/alexeymezenin/laravel-best-practices) making them even bester.

Are the statements in this article correct? No, these are also opinions. Some of them are intentionally contrary to the originals.
My goal here is to highlight that the best practices are not dogmas and you are allowed to think about their use. Make your own decisions.

> [!NOTE]  
> I only knew Spaghetti and Ravioli, but I asked ChatGPT for the rest of the terms so they must be correct.
> Even if they weren't before this.
> It also suggested me other great names like Goulash code (a mix of all techniques) and Casserole code (rewritten so much it has become unrecognizable).

## Contents

- [Single responsibility principle](#single-responsibility-principle)
- [Methods should do just one thing](#methods-should-do-just-one-thing)
- [Fat models, skinny controllers](#fat-models-skinny-controllers)
- [Validation](#validation)
- [Business logic should be in service class](#business-logic-should-be-in-service-class)
- [Don't repeat yourself (DRY)](#dont-repeat-yourself-dry)
- [Prefer to use Eloquent over using Query Builder and raw SQL queries. Prefer collections over arrays](#prefer-to-use-eloquent-over-using-query-builder-and-raw-sql-queries-prefer-collections-over-arrays)
- [Mass assignment](#mass-assignment)
- [Do not execute queries in Blade templates and use eager loading (N + 1 problem)](#do-not-execute-queries-in-blade-templates-and-use-eager-loading-n--1-problem)
- [Chunk data for data-heavy tasks](#chunk-data-for-data-heavy-tasks)
- [Comment your code, but prefer descriptive method and variable names over comments](#prefer-descriptive-method-and-variable-names-over-comments)
- [Do not put JS and CSS in Blade templates and do not put any HTML in PHP classes](#do-not-put-js-and-css-in-blade-templates-and-do-not-put-any-html-in-php-classes)
- [Use config and language files, constants instead of text in the code](#use-config-and-language-files-constants-instead-of-text-in-the-code)
- [Use standard Laravel tools accepted by community](#use-standard-laravel-tools-accepted-by-community)
- [Follow Laravel naming conventions](#follow-laravel-naming-conventions)
- [Convention over configuration](#convention-over-configuration)
- [Use shorter and more readable syntax where possible](#use-shorter-and-more-readable-syntax-where-possible)
- [Use IoC container or facades instead of new Class](#use-ioc-container-or-facades-instead-of-new-class)
- [Do not get data from the `.env` file directly](#do-not-get-data-from-the-env-file-directly)
- [Store dates in the standard format. Use accessors and mutators to modify date format](#store-dates-in-the-standard-format-use-accessors-and-mutators-to-modify-date-format)
- [Do not use DocBlocks](#do-not-use-docblocks)

### **Single responsibility principle**

A class should have only one responsibility. But you decide what that responsibility is. You decide the scope.
The scope should be inversely proportional to complexity. If there's a lot to do, you split it.
If handling the request can be done in 4 expressions, don't obfuscate it into other dumplings. Just create the entry and return a response.

But what's even more important is to at least accomplish that single responsibility.
The example in [the original](https://github.com/alexeymezenin/laravel-best-practices/tree/2d958de71af0beaab1b2ea14d5b5352daad6477d?tab=readme-ov-file#single-responsibility-principle) suggests handing off the `$request->events` array to a `logService`.
But what's the responsibility of a controller in MVC?

If you want to isolate the logging responsibility, you must still parse the request inside your controller and hand off prepared data to the service.
Your services should not care about the structure of the arrays in requests and parse the dates. That's exactly the responsibility of your controller.

> [!NOTE]  
> If you look up the definition and explanations by Robert C. Martin, SRP requires the class to be responsible to a single stakeholder.
> I.e. your class can do a lot of logic, but it should all belong to the same business needs.

> Gather together the things that change for the same reasons. Separate those things that change for different reasons.
> https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html

### **Methods should do just one thing**

Same principle, different scope. Same considerations:

- Is there a lot of mess in the method or can you just perceive it all as is?
- Will it be enough to see the method names instead of logic? Or will you always have to look for the code of methods?
- Will you reuse these methods?

My rule of thumb is to split off a method when there's something I wouldn't need to know to understand the rest of the flow.
In [the original](https://github.com/alexeymezenin/laravel-best-practices/tree/2d958de71af0beaab1b2ea14d5b5352daad6477d?tab=readme-ov-file#methods-should-do-just-one-thing) it's the check of being verified.
It's messy, it's custom and you can the gist faster by just reading the method name. The rest is obvious as is. So something like this would be optimal:

```php
public function getFullNameAttribute(): string
{
    if ($this->isVerifiedClient()) {
        return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
    } else {
        return $this->first_name[0] . '. ' . $this->last_name;
    }
}

public function isVerifiedClient(): bool
{
    return auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified();
}
```

### **Fat models, skinny controllers**

Don't go into extremes ok? :)

Querying logic can often go into the scopes of your models. But not necessarily everything has to bloat the model.
Sometimes you need to load additional stuff for a particular request and it's fine to do that where you need it. Be it a service or a controller or a job.

In addition, you should usually avoid invoking "final" methods like `->get()` and `->first()` in the model. It breaks DRYness.

Good:

```php
class Client extends Model
{
    public function getWithNewOrders(): Collection
    {
        return $this->verified()
            ->with(['orders' => function ($q) {
                $q->where('created_at', '>', Carbon::today()->subWeek());
            }])
            ->get();
    }
}
```

Gooder:

```php
class Client extends Model
{
    public function scopeWithNewOrders(Builder $clients): void
    {
        $clients->verified()->with(['orders' => function ($q) {
            $q->where('created_at', '>', Carbon::today()->subWeek());
        }]);
    }
}
```

Why is that better? Because that can be consumed by the weekly job (`$processables = Client::withNewOrders()->get()` or `->chunk(500)`) as well as the dashboard controller (`Client::withNewOrders()->count()`)
and the new order controller (`Client::withNewOrders()->paginate(10)`).
Believe me, your code will not become cleaner by creating `getWithNewOrders()`, `countWithNewOrders()` and `getWithNewOrdersPaginated(int $pageSize)` in your model or repository.
It's ok if the model defines a reusable scope and the consumers decide what to get using it.

### **Validation**

Sure, move validation from controllers to Request classes.

But it's also ok to do the validation inside the controller if the handler is small, the validation cares about a couple of fields and isn't really reused.
It's not like you want a directory of 380 small Request classes each with a single use, right?

Let's rethink the SRP. One stakeholder. One reason to change. You know what?
Most of the time when you have a reason to change the validation (e.g. add a new field), that same reason will also cause you to change the controller.

I'm not saying using form requests is wrong. If the validation mess makes your controller harder to read, put it away.
Even if it's a single-use class. I'm just saying you shouldn't use form requests just because you can.

### **Business logic should be in service class**

The statement is correct. The [example](https://github.com/alexeymezenin/laravel-best-practices/tree/2d958de71af0beaab1b2ea14d5b5352daad6477d?tab=readme-ov-file#business-logic-should-be-in-service-class) is bad.

Do you remember that the controller has a responsibility too? Let it check whether there is an uploaded file or not.

Besides the handling in the example is so simple, it can be done in the controller without any issues. 

### **Don't repeat yourself (DRY)**

Yeah, reuse when you can. Use and reuse scope, partial templates and so on.

I will just add that there are some limits and exceptions.
If the reuse needs make the reusable too customizable and complex, just don't do it. Don't make a function with 5 boolean params for every case.
And don't DRY unrelated things. Even if two branches of your code look very similar, you don't have to put them together just based on similarity.
If they are expected to change in unrelated ways, let them live separately.

### **Prefer to use Eloquent over using Query Builder and raw SQL queries. Prefer collections over arrays**

Yes.

This was perfectly stated thanks to the "prefer" qualifier. Sticking to default conventions by default will make your code understandable to any Laravel dev.

There will be times when query builder will be handier than defining a model, when SQL will be more readable than Eloquent and when foreaching an array will be more performant than calling a chain of methods on a collection. But these should be exceptions when necessary, not preferences.

### **Mass assignment**

I wouldn't call direct assignment "bad", but mass assignment is indeed simpler most of the time.
You should be familiar with both approaches to choose the appropriate one.

And don't forget to validate if you're mass-assigning!

### **Do not execute queries in Blade templates and use eager loading (N + 1 problem)**

Eager loading is a no-brainer, surely do it whenever it allows you to easily avoid the 1+N.

But don't be that dogmatic about queries in views. It's not always that useful to query a dozen of simple enums in the controller and pass around the variables.
Sometimes this ir just perfect:

```blade
<select name=venue_type>
@foreach (VenueType::all() as $type)
    <option value={{$type->id}}>{{$type->name}}
@endforeach
</select>
```

### **Chunk data for data-heavy tasks**

Yes. Once again having a qualifier makes the point great. Tasks with a lot of entries is the use case for chunking.

### **Prefer descriptive method and variable names over comments**

Yes. Prefer the most readable option. Most of the time having a great method/variable name will be enough.

Just don't spend half of your day to come up with the perfect naming. Let someone suggest it in the CR if your mind is blanking.

### **Do not put JS and CSS in Blade templates and do not put any HTML in PHP classes**

Why not?

There's nothing wrong with putting simple JS or CSS snippets inside your templates.
Not everything has to be put away, especially if it's only relevant to that one view.

Notably many of Laravel templates have `onclick="event.preventDefault(); this.closest('form').submit();"` on their logout button.
Perfectly readable and you would gain nothing by adding this event listener via `addEventListener` in a separate JS file.

### **Use config and language files, constants instead of text in the code**

Yeah, most of the time you should strive to define the strings once so you can update them in a single place.

### **Use standard Laravel tools accepted by community**

A [huge list](https://github.com/alexeymezenin/laravel-best-practices/tree/2d958de71af0beaab1b2ea14d5b5352daad6477d?tab=readme-ov-file#use-standard-laravel-tools-accepted-by-community), but...

Sometimes the third party is the standard. Sometimes the listed "standard" just isn't enough...

For example the built-in localization includes translation features, but it doesn't handle setting the language based on request or localizing routes.
The policies provide a place to check permissions, but it does not answer how you assign those permissions to users in the first place.
Packages like `barryvdh/laravel-debugbar` or `spatie/laravel-medialibrary` are 3rd party, but they are also the standard tools for Laravel devs.

Don't worry. You'll get familiar with the ecosystem over time. Just search and read when you need something.

### **Follow Laravel naming conventions**

Having consistency in your code base is good. Following a particular convention is not necessary.
If your project usees camelCased URI paths since 2014, so be it.

What's even more important is to not spend a lot of time arguing about these and to let automated tools take care of it.

### **Convention over configuration**

Yup, and you will have to be very familiar with Laravel's (especially Eloquent's) conventions to know when you need something custom and when you can just use a convention and it will work using the defaults.

Read docs, bro. Or the framework code.
I can never remember if `$table->foreignIdFor(User::class)` will do exactly what I need or something more or something less.
And will it work with a differently named primary key or a string one?

### **Use shorter and more readable syntax where possible**

Redability matters. Shortness? Not so much.
In the [original example](https://github.com/alexeymezenin/laravel-best-practices/tree/2d958de71af0beaab1b2ea14d5b5352daad6477d?tab=readme-ov-file#use-shorter-and-more-readable-syntax-where-possible) looking up stuff on the `$request` can be justified as it makes mocking much easier.

Laravel provides you with a bunch of ways to call a tool — facade, helper, injection, `$request`, `$this` on controllers and so on.
You know what's not useful and productive? Starting a holy war about which of them are good and which are bad. If one falls short in a particular case, change it to another one. That's why you have them.

### **Use IoC / Service container instead of new Class**

Indeed, using `new` is rarely useful.

However, creating model instances is one of the cases where it makes sense. There's nothing wrong in this:

```php
$user = new User($request->validated());
```

### **Do not get data from the `.env` file directly**

First of all, remember that the `env` helper does not necessarily get data from the `.env` file.
You are getting it from the environment. The `.env` file is just a way to populate the environment.
But in production it might be set by your server or the web server. Those could be params of your docker container or your AWS instance.
Those values can even be set by a `putenv` call in an another PHP thread.

So you decide. Do you need the value from the actual environment at this moment?
Or do you need the config value that was populated from the environment when it was cached?

### **Store dates in the standard format. Use accessors and mutators to modify date format**

I agree. Carbon is a great tool. Use it.

Just don't `->copy()` if an instance of `CarbonImmutable` would be more appropriate. Look it up ;)

### **Do not use DocBlocks**

Totally use native typehints and good method names. Do not repeat the info in a docblock.

When the native tooling is not enough, don't be afraid to use a docblock. Explain what you need to explain.
