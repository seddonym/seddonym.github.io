---
layout: post
title: The trouble with transaction.atomic
description: "Django's transaction.atomic context manager is an important tool for maintaining data integrity.
  But its guarantees are frequently misunderstood."
image: trouble-atomic.jpg
featured: true
weight: 1
tags: [django]
---

Something about Django has bothered me for some time. It's the context manager at the heart of its API for handling
database transactions: `django.db.transaction.atomic`. It offers a neat design to help us write robust code, but also
plenty of opportunity to write programs that are not as robust as they appear. Before we look into why, here's a
refresher on `transaction.atomic`.

## What transaction.atomic does

By default, each Django ORM query is automatically committed to the database. Sometimes, however, we need to
group multiple operations, so that they happen either altogether, or not at all. The property of queries grouped
indivisibly like this is known as _atomicity_: the 'A' in the well known acronym
[ACID](https://en.wikipedia.org/wiki/ACID).

Take this example from a fictional banking application, in which a transfer between two accounts requires the creation of two
different rows in a table.

{% highlight python %}
from django.db import transaction


def transfer(source: Account, destination: Account, amount: int) -> None:
    with transaction.atomic():
        BalanceLine.objects.create(account=source, amount=-amount)
        BalanceLine.objects.create(account=destination, amount=amount)

{% endhighlight %}

Because we've wrapped the queries in `transaction.atomic`, we know that whatever happens, the two balance lines will
either both make it into the database together, or not at all. This is a very good thing, because if only the first
balance line was written, the books wouldn't balance.

How does it work? Under the hood, Django begins a database transaction when `transaction.atomic` is entered, and does not commit it
until the context exits without an exception being raised. If, for some reason, the second ORM operation does raise an
exception, the database will roll back the transaction. Corrupt data is avoided. We call this operation 'atomic'
because, like an atom, it's indivisible. (For the sake of the metaphor, let's ignore particle accelerators, shall we?)

### Nested atomic blocks

`transaction.atomic` also supports _nesting_:

{% highlight python %}
with transaction.atomic():
    ...
    ...
    with transaction.atomic():
        ...
        with transaction.atomic():
            ...
            ...
    with transaction.atomic():
        ...
    ...
{% endhighlight %}

The details of how this works are a little harder to wrap one's head around. In many database engines, such as
PostgreSQL, there's no such thing as a nested transaction. So instead Django implements this using a single outer
transaction, and then a series of database savepoints:

{% highlight python %}
with transaction.atomic():  # Begin transaction
    ...
    ...
    with transaction.atomic():  # Savepoint 1
        ...
        with transaction.atomic():  # Savepoint 2
            ...
            ...
    with transaction.atomic():  # Savepoint 3
        ...
    ...
... # Transaction will now be committed.
{% endhighlight %}

All the inner atomic blocks behave slightly differently to the outer one: in the event of an exception, rather than
rolling back the whole transaction, it rolls back to the savepoint set when it entered the context manager.

The crucial thing to realize here is that the transaction is _only committed_ when we exit the outer block. This means
that any database operations executed in inner blocks are still subject to rollback.

## Atomic may not mean what you think

I'm not criticizing this design: it's powerful and well thought out. But it does suffer from a serious gotcha.

Let's look at another example, this time involving a third party API call as well as a database operation. What we're
aiming for is to ensure that our database accurately reflects whether or not the money actually left our real bank
account.

{% highlight python %}
import banking_api


def transfer_to_other_bank(account: Account, bank_details: BankDetails, amount: int) -> None:
    with transaction.atomic():
        # Store the movement in our database.
        balance_line = BalanceLine.objects.create(account=account, amount=-amount, bank_details=bank_details)
        # Log the external transfer, generating a unique reference.
        external_transfer = ExternalTransfer.objects.create(
            account=account,
            amount=amount, 
            bank_details=bank_details,
        )
        # Instruct third party API to move money.
        banking_api.make_payment(account, bank_details, amount, reference=external_transfer.reference)
{% endhighlight %}

At first glance, the `transaction.atomic` is well placed: if the call to the third party API fails, we roll back the
transaction, meaning that our database will only record the rows in the database if the money actually moved.

Or will it?

What happens if the function above is invoked, elsewhere in the application, like this?

{% highlight python %}

with transaction.atomic():
    transfer_to_other_bank(account, bank_details, amount)
    do_something_unreliable()
{% endhighlight %}

If the unreliable function fails, it will roll back the database work done by `transfer_to_other_bank`, _but not the API
call_. We end up having lost the record of having moved the money, despite our transfer function apparently doing
its best to preserve the correctness of the data.

The nub of the problem is that `transaction.atomic` behaves very differently depending on its context. Sometimes it represents
a database transaction that gets safely committed to the database; other times, merely a savepoint in an uncommitted transaction.
These very different behaviours _look exactly the same_ if you look purely at the code in isolation.

Even if we check that the calling code is not wrapping it in a transaction, there is nothing to stop a future developer from adding
a well-intentioned `transaction.atomic` higher up the call stack.

Note that ``transaction.atomic`` _is_ living up to its promises. Atomicity is still guaranteed. But it turns out that's not actually
what we want here. Instead, we want the guarantee that once we've made the API call, the record is written reliably to the database.

We don't need to make up a name for this - like atomicity, it's one of the four transactional properties
found in the well known acronym [ACID](https://en.wikipedia.org/wiki/ACID).  We want _durability_.

## Guaranteeing durability isn't so hard

Wouldn't it be nice if we could be sure of the behaviour of ``transaction.atomic`` without needing outside context? Wouldn't it be nice
if we could check that there was no transaction in progress, guaranteeing that as we exit that ``atomic`` block,
the transaction is durably committed?

I have good news. It turns out to be fairly straightforward, providing you're happy to use an API that is
[described in Django's source code as private](https://github.com/django/django/blob/3.1.3/django/db/transaction.py#L16).
Assuming our code is running on the default database connection, we can do this:

{% highlight python %}
from django.db import transaction


def transfer_to_other_bank(account: Account, bank_details: BankDetails, amount: int) -> None:
    if transaction.get_connection().in_atomic_block:
      raise RuntimeError("Function should not be called within an atomic block.")

    with transaction.atomic():
        # As before.
{% endhighlight %}

(Disclaimer: if your application uses multiple connections, you may need to think a bit more carefully about this.)

This would be enough, if we never wanted to test our code. But, for performance reasons, it's standard practice (and
indeed the default behaviour of ``django.test.TestCase``) to
[wrap each test in an atomic block](https://docs.djangoproject.com/en/3.1/topics/testing/tools/#django.test.TestCase).
Any such tests exercising this code will run headlong into the durability check (precisely because the code in them
_isn't_ durable). Hmm.

Now, we could opt instead for the confusingly named `django.test.TransactionTestCase` (in which the test _isn't_
wrapped). The advantage of these tests is that they run the code in a more production-like context, and can catch
programming errors that the wrapped test cases will miss (such as a `SELECT FOR UPDATE` outside a transaction). The
trouble is, they're much, much slower. We generally want to keep the majority of our tests wrapped in transactions.

We can work around this by making the check possible to turn off. The addition of a `DISABLE_DURABILITY_CHECKING`
setting allows us to run this code within transaction-wrapped tests:

{% highlight python %}
from django.db import transaction
from django.conf import settings


def transfer_to_other_bank(account: Account, bank_details: BankDetails, amount: int):
    if not settings.get("DISABLE_DURABILITY_CHECKING") and transaction.get_connection().in_atomic_block:
      raise RuntimeError("Function should not be called within an atomic block.")

    with transaction.atomic():
        # As before.
{% endhighlight %}

It's a simple matter to set `DISABLE_DURABILITY_CHECKING` to `True` in our test configuration, or control it on
individual tests using
[`override_settings`](https://docs.djangoproject.com/en/3.1/topics/testing/tools/#django.test.override_settings).
 
### The @durable decorator 

We use this approach extensively at [my workplace ](https://octopus.energy/), and have created a decorator named
`durable` to cut down on the boilerplate:

{% highlight python %}
@durable
def transfer_to_other_bank(account: Account, bank_details: BankDetails, amount: int):
    with transaction.atomic():
        # As before.
{% endhighlight %}

You can see the code for `@durable` in [this Gist](https://gist.github.com/seddonym/170c62d0a694f0ccb794c9ad5569ee20).
If you decide to use it, bear in mind that it's only been tested on PostgreSQL, uses an undocumented API that may
change, and may need to be adapted if you use multiple databases.

## When to use durable

Unlike with `transaction.atomic`, `durable` functions cannot be nested. This means you need to be more sparing of when you use
them. In our architecture, we limit its use to certain higher-level functions (called 'use cases') which sit between
Django views and business logic code.

## Conclusion

We must be careful, when using `transaction.atomic`, not to mistake its atomicity guarantee for a guarantee of
durability. Doing so can lead to missing data, particularly in code which is subject to intermittent failure (such as
calls over a network).

Fortunately, durability guarantees are fairly easily achieved with a little custom code. Using this in well-chosen
places will make your applications more robust.
