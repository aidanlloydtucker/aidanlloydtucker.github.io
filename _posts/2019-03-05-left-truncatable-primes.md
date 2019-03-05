---
layout: post
title: Left-truncatable Primes
---

I was recently doing an interview when I was asked the question: given a constant time `isPrime(x)` and `truncateLeft(x)` functions, find all truncatable primes up to n. Now first, I created a `isTruncatablePrime(x)` function:

{{ "{% highlight java " }}%}
boolean isTruncatablePrime(int x) {
    if (x < 2) {
        return false;
    }
    if (x < 10) {
        return isPrime(x);
    }
    return isPrime(x) && isTruncatablePrime(truncateLeft(x));
}
{{ "{% endhighlight " }}%}

This is a simple recursive function that runs in O(N) time where N is the length of x. Thus, N=floor(log10(x)).

Now to get all left-truncatable primes under n. The obvious solution would be to iterate through all numbers and check if `isTruncatablePrime(x)` is true. But that will take `O(nlog10(n))`. We can do it in O(n). The next solution that could work is to initialize an array of left-truncatable primes and then go top down fron n to 2. For each left-trucatable prime we encounter, we know that it's left-truncated version will also be a prime, and so on. So, every time we run into a lt prime, we mark it in the array and mark its left-truncatable child/grandchild/ancestors. This will take O(n) time, but also O(n) space.

We can do better. What if we do a bottom up approach. We know that every two-digit lt prime must have a one-digit prime when it is left-truncated. Thus, we know every n-digit lt prime has a (n-1)-digit lt prime. This means its definition is recursive. So, let's start at the bottom. We know every lt prime must have either a 2, 3, 5, or 7 in the last digit. So, lets start with those 4 numbers (really 3 numbers as any number not equal to 2 but has a 2 at the end is not a prime). We then can go up from these 4 numbers and check it against all 9 numerals it could start with (1 through 9). We continue this building of the lt primes while the constructed primes are less than or equal to n. Once they are greater than n, we ignore them and stop that particular building pattern. As we build, every time we reach a new number, we check if it's prime. If it is, we add it to the set S of lt primes. We then check the same for all of its parents, meaning all numbers [1..9] concatenated before the number. If it is not a prime, we stop the construction. Thus, we will get a full set of lt primes up to n by the end. The code is shown below.

{{ "{% highlight java " }}%}
Set constructLeftTruncatablePrimes(int n) {
    Set<Integer> s = new Set<>();
    int[] firstPrimes = {2, 3, 5, 7};
    for (int p : firstPrimes) {
        recursiveConstruct(p, 1, s, n);
    }
    return s;
}

// x is the number, N is the digits in x, s is the set of all
// lt primes, max is the maximum value allowed.
void recursiveConstruct(int x, int N, Set s, int max) {
    if (x > max) {
        return;
    }
    if (!isPrime(x)) {
        return;
    }

    // It's below max and prime, so it's left truncatable!
    s.add(x);

    // This is the power to concat i to the left of x
    int leftAddNum = Math.pow(10, N);

    // Call for all possible lt primes built off of this lt prime
    for (int i = 1; i < 10; i++) {
        recursiveConstruct((leftAddNum * i) + x, N+1, s, max);
    }
}
{{ "{% endhighlight " }}%}

It's easy to figure this runs in O(n) time. We have (let's round) 10 options for each digit and N=log10(n) digits maximum, so this is 10^N = 10^(log10(n)) = O(n) time. But this isn't the tightest bound we could get. First, we use 9 options. Second, we use only 4 or 3 options at the start. Third, we only progress if the number is prime. The distribution of primes is around n/ln(n). This is an approximate function, but with it, we can tighten the bound. See my next post to on how I figure out how to do this.