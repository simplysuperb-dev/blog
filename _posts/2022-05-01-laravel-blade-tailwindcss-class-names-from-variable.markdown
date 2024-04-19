---
layout: post
title:  "Handling class names derived from variables with Blade and TailwindCSS"
date:   2022-05-01 12:29:10 +0200
permalink: 2022-05-01-laravel-blade-tailwindcss-class-names-from-variable
author: itodorova
---

First, you need to have TailwindCSS already setup with Laravel. To do so, you can follow the [official documentation at TailwindCSS](https://tailwindcss.com/docs/guides/laravel).

Since TailwindCSS requires classes to be present in whatever file you have specified inside your `content` array in `tailwindcss.config.js`, during your development you might experience missing styles due to the usage of variables inside your `*.blade.php` files (for example).

Consider the following case, you have few buttons that look _exactly_ the same except for their background color:

{% highlight html %}
<!-- button for default operations: eg. login -->
<button type="button" class="bg-blue-700 hover:bg-blue-800 focus:ring-blue-300 focus:ring-4 text-white font-medium rounded-lg text-sm px-5 py-2.5 mr-2 mb-2">Default</button>

<!-- button for potentially dangerous operations: eg. disabling 2FA -->
<button type="button" class="bg-orange-700 hover:bg-orange-800 focus:orange-orange-300 focus:ring-4 text-white font-medium rounded-lg text-sm px-5 py-2.5 mr-2 mb-2">Warning</button>

<!-- button for dangerous operations: eg. deleting -->
<button type="button" class="bg-red-700 hover:bg-red-800 focus:ring-red-300 focus:ring-4 text-white font-medium rounded-lg text-sm px-5 py-2.5 mr-2 mb-2">Danger</button>
{% endhighlight %}

What changes in each button is the css utility classes applied for background color. Now since the markup is the same and the styling for text also matches each button, let's create an [Anonymous Blade Component](https://laravel.com/docs/9.x/blade#anonymous-components) button:

{% highlight php %}
// ./resources/views/components/button.blade.php
@props(['color' => 'blue'])

<button type="button" class="bg-{$color}-700 hover:bg-{$color}-800 focus:ring-{$color}-300 focus:ring-4 text-white font-medium rounded-lg text-sm px-5 py-2.5 mr-2 mb-2">
    {%raw%}{{$slot}}{%endraw%}
</button>
{% endhighlight %}

And use it in our view as follows:

{% highlight php %}
<x-button color="blue"> Default </x-button>
{% endhighlight %} 

However, since the class names use php variable (`bg-{$color}-700`), TailwindCSS isn't able to extract them (don't forget to run `npm run watch` / `npm run dev`) and your buttons won't look as expected (almost invisible, white text on white background _yikes_).

To solve this, let's create several new components, representing the state of our button and one base component containing the common styles between them, in a new folder at `views/components/button`:

{% highlight php %}
// ./resources/views/components/button/base.blade.php
<button type="button" {%raw%}{{$attributes->merge([class="text-white font-medium rounded-lg text-sm px-5 py-2.5 mr-2 mb-2"])}}{%endraw%}>
    {%raw%}{{$slot}}{%endraw%}
</button>
{% endhighlight %}

Now let's create the default "state" of our button:

{% highlight php %}
// ./resources/views/components/button/default.blade.php
<x-button.base class="bg-blue-700 hover:bg-blue-800">
{%raw%}{{$slot}}{%endraw%}
</x-button.base>
{% endhighlight %}

And we can use the `Default` button in our views as follows:

{% highlight php %}
<x-button.default>Default</x-button>
{% endhighlight %} 

If you run `npm run dev`, you will see that the button now have background color, you can do the same thing for all button states that you have and tailwindcss will be able to extract those classes with no problem.

<p class="text-right">
<strong><em>Stay humble</em></strong>,<br/>
<em>Iv</em>
</p>