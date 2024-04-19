---
layout: post
title:  "Laravel 10+: Unit testing custom validation rules"
date:   2024-04-19 11:59:10 +0300
permalink: /2024-04-19-laravel-10-plus-unit-test-custom-validation-rule
author: itodorova
---

This post covers one of the ways you can test custom validation rules in Laravel. It assumes you're running a Laravel version no older than 10.x. I prefer this way of testing as it's the cleanest and more concise of others you can find online.

I will be using the custom rule provided in the official documentation:

{% highlight php linenos %}
// ./app/Rules/Uppercase.php
<?php
 
namespace App\Rules;
 
use Closure;
use Illuminate\Contracts\Validation\ValidationRule;
 
class Uppercase implements ValidationRule
{
    /**
     * Run the validation rule.
     */
    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        if (strtoupper($value) !== $value) {
            $fail('The :attribute must be uppercase.');
        }
    }
}
{% endhighlight %}

This is a very simple and straightforward rule that tests a field to see if it contains only uppecase letters. It doesn't take into account any other data in the request (or elsewhere). If you've worked with older Laravel versions, you will notice that the old `passes` and `message` have been replaced by a single `validate()` method's that doesn't return anything (but voidness).

A test class for the code above looks like this:

{% highlight php linenos %}
// ./tests/Unit/Rules/Uppercase.php

<?php

namespace Tests\Unit\Rules;

use Tests\TestCase;
use App\Rules\Uppercase;

class UppercaseTest extends TestCase {

    const VALID_STRINGS = [
        'I1M UP3RC4SE',
        'I AM UPPERCASE',
        '0TH3R_UPP3RC4S3'
    ];

    const INVALID_STRINGS = [
        'I MiGhT BE UPP3R',
        'bUt I am Not'
    ];

    /**
    * @test
    */
    public function validation_passes_for_valid_strings() {
        $rulew = new Uppercase();
        
        foreach(self::VALID_STRINGS as $string) {
            $rule->validate('name', $string, function () use ($string) {
                $this->assertTrue(
                    false,
                    sprintf("String is not uppercase only: %s", $string)
                );
            });
        }

        $this->asserTrue(true);
    }

    /**
    * @test
    */
    public function validation_fails_for_invalid_strings() {
        $rulew = new Uppercase();
        $timesFailed = 0;

        foreach(self::INVALID_STRINGS as $string) {
            $rule->validate('name', $string, function () use ($string, &$timesFailed) {
                $timesFailed++;
            });
        }

        $this->assertEquals(
            count(self::INVALID_STRINGS),
            $timesFailed,
            'All strings must fail validation'
        );
    } 
}

{% endhighlight %}

The test **_validation_passes_for_valid_strings()_** creates an instance of our custom rule and then iterates all strings inside the `VALID_STRINGS` constant. On each iteration we call the `Uppercase::validate` method and (here comes the cool part) as a closure we're telling our rule assert an always failing comparison and log our failing `$string`.

Additionally, we are asserting a seemingly redundant check `true===true` to avoid getting this message in phpunit:

> _! validation passes for valid strings → This test did not perform any assertions  0.50s_  
> 
> _Tests:    1 risky (0 assertions)_

The other test **_validation_fails_for_invalid_strings()_** counts the number of fails and asserts the number is equal to the total strings inside `INVALID_STRINGS`.

And that's it!

<p>
<strong><em>Stay humble</em></strong>,<br/>
<em>Iv</em>
</p>
