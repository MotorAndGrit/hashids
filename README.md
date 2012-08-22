
# hashids

A small PHP class to generate YouTube-like hashes from one or many numbers. Use **hashids** when you do not want to expose your database ids to the user.

[http://www.hashids.org/php/](http://www.hashids.org/php/)

## What's it for?

**hashids** creates short, unique, decryptable hashes from unsigned integers.

It was designed for websites to use in URL shortening, tracking stuff, or making pages private (or at least unguessable).

This algorithm tries to satisfy the following requirements:

1. Hashes must be unique and decryptable.
2. They should be able to contain more than one integer (so you can use them in complex or clustered systems).
3. You should be able to specify minimum hash length.
4. Hashes should not contain basic English curse words (since they are meant to appear in public places - like the URL).

Instead of showing items as `1`, `2`, or `3`, you could show them as `U6dc`, `u87U`, and `HMou`.
You don't have to store these hashes in the database, but can encrypt + decrypt on the fly.

All integers need to be greater than or equal to zero.

## lib folder

- Use `lib/hashids.php-5-3.php` if you have __PHP 5.3.*__
- Use `lib/hashids.php` if you have __PHP 5.4.*__ or higher

Examples below assume you have PHP 5.4 and above:

## Sample Usage

#### Encrypting one number

You can pass a unique salt value so your hashes differ from everyone else's. I use "**this is my salt**" as an example.

```php
<?php

require_once('lib/hashids.php');
$hashids = new hashids('this is my salt');

$hash = $hashids->encrypt(1234);
```

`$hash` is now going to be:
	
	x7yR
	
#### Decrypting

Notice during decryption, same salt value is used:

```php
<?php

require_once('lib/hashids.php');
$hashids = new hashids('this is my salt');

$numbers = $hashids->decrypt('x7yR');
```

`$numbers` is now going to be:
	
	array(1) {
		[0]=>
		int(1234)
	}

#### Decrypting with different salt

Decryption will not work if salt is changed:

```php
<?php

require_once('lib/hashids.php');
$hashids = new hashids('this is my pepper');

$numbers = $hashids->decrypt('x7yR');
```

`$numbers` is now going to be:
	
	array(0) {
	}
	
#### Encrypting several numbers

```php
<?php

require_once('lib/hashids.php');
$hashids = new hashids('this is my salt');

$hash = $hashids->encode(683, 94108, 123, 5);
```

`$hash` is now going to be:
	
	z8pFrxLyCRah7
	
#### Decrypting is done the same way

```php
<?php

require_once('lib/hashids.php');
$hashids = new hashids('this is my salt');

$numbers = $hashids->decrypt('z8pFrxLyCRah7');
```

`$numbers` is now going to be:
	
	array(4) {
		[0]=>
		int(683)
		[1]=>
		int(94108)
		[2]=>
		int(123)
		[3]=>
		int(5)
	}
	
#### Encrypting and specifying minimum hash length

Here we encode integer 1, and set the minimum hash length to **17** (by default it's **0** -- meaning hashes will be the shortest possible length).

```php
<?php

require_once('lib/hashids.php');
$hashids = new hashids('this is my salt', 17);

$hash = $hashids->encode(1);
```

`$hash` is now going to be:
	
	KR9AIXzUMGcR9AIXz
	
#### Decrypting

```php
<?php

require_once('lib/hashids.php');
$hashids = new hashids('this is my salt', 17);

$numbers = $hashids->decrypt('KR9AIXzUMGcR9AIXz');
```

`$numbers` is now going to be:
	
	array(1) {
		[0]=>
		int(1)
	}
	
#### Specifying custom hash alphabet

Here we set the alphabet to consist of only four letters: "abcd"

```php
<?php

require_once('lib/hashids.php');
$hashids = new hashids('this is my salt', 0, 'abcd');

$hash = $hashids->encode(1, 2, 3, 4, 5);
```

`$hash` is now going to be:
	
	adcdacddcdaacdad
	
## Randomness

The primary purpose of hashids is to obfuscate ids. It's not meant or tested to be used for security purposes or compression.
Having said that, this algorithm does try to make these hashes unguessable and unpredictable:

#### Repeating numbers

```php
<?php

require_once('lib/hashids.php');
$hashids = new hashids('this is my salt');

$hash = $hashids->encode(5, 5, 5, 5);
```

You don't see any repeating patterns that might show there's 4 identical numbers in the hash:

	GkIpCxSL

Same with incremented numbers:

```php
<?php

require_once('lib/hashids.php');
$hashids = new hashids('this is my salt');

$hash = $hashids->encode(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
```

`$hash` will be :
	
	z7FXtaiRSkIpFrtYirSo
	
### Incrementing number hashes:

```php
<?php

require_once('lib/hashids.php');
$hashids = new hashids('this is my salt');

var_dump($hashids->encode(1)); // MG
var_dump($hashids->encode(2)); // eL
var_dump($hashids->encode(3)); // o7
var_dump($hashids->encode(4)); // 4R
var_dump($hashids->encode(5)); // ag
```

## Speed

Even though speed is an important factor of every hashing algorithm, primary goal here was encoding several numbers at once and making the hash unique and random.

On a *2.7 GHz Intel Core i7 with 16GB of RAM*, it takes roughly **0.37 seconds** to:

1. Encrypt 1000 hashes consisting of 1 integer `$hashids->encrypt(12);`
2. And decrypt these 1000 hashes back into integers `$hashids->decrypt($hash);` while ensuring they are valid

If we do the same with 3 integers, for example: `$hashids->encrypt(10, 11, 12);`
-- the number jumps up to **0.56 seconds** on the same machine.

On a *2.26 GHz Intel Core 2 Duo with 8GB of RAM*, it takes about **0.75 seconds** to do the same with 1 integer, and **1.15 seconds** for 3 integers.

*Sidnote: The numbers tested with were relatively small -- if you increase them, the speed will obviously decrease.*

#### What you could do to speed it up

Usually people either encrypt or decrypt one hash per request, so the algorithm should already be fast enough for that.
However, there are still several things you could do:

1. Wrap this class in your own, and cache hashes/numbers in static variables - so that per lifetime of a request, they would be remembered by PHP and hashids wouldn't have to recalcuate them.
2. Use [Memcache](http://memcached.org/) or [Redis](http://redis.io/).
3. You could also decrease the length of your alphabet. Your hashes will become longer, but calculating them will be faster.

## Bad hashes

I wrote this class with the intent of placing these hashes in visible places - like the URL. If I create a unique hash for each user, it would be unfortunate if the hash ended up accidentally being a bad word. Imagine auto-creating a URL with hash for your user that looks like this - `http://example.com/user/a**hole`

Therefore, this algorithm tries to avoid generating most common English curse words with the default alphabet. This is done by never placing the following letters next to each other:
	
	c, C, s, S, f, F, h, H, u, U, i, I, t, T
	
## Changelog

**0.1.2 - Current Stable**

	Warning: If you are using 0.1.1 or below, updating to this version will change your hashes.

- Minimum hash length can now be specified
- Added more randomness to hashes
- Added unit tests
- Added example files
- Changed warnings that can be thrown
- Renamed `encode/decode` to `encrypt/decrypt`
- Consistent shuffle does not depend on md5 anymore
- Speed improvements

**0.1.1**

- Speed improvements
- Bug fixes

**0.1.0**
	
- First commit