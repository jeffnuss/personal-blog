---
layout: post
title: "Why Factoring Huge Numbers Is a Big Deal"
date: 2014-01-30 01:17:26 -0700
comments: true
categories: crypto
---

A few months ago, there was a [post on hacker news](https://news.ycombinator.com/item?id=6506120) about the fact that [RSA-210](https://en.wikipedia.org/wiki/RSA_numbers#RSA-210) had been factored. I've always been interested in crypto and have a basic understanding of some of the concepts, but I wanted to take a deeper dive into why the ability to do [prime factorization](http://en.wikipedia.org/wiki/Prime_factorization) of really big numbers is a big deal, especially when those numbers are [semiprimes](https://en.wikipedia.org/wiki/Semiprimes) or numbers that are the product of 2 primes.

For this post, I'm going to focus on [RSA](https://en.wikipedia.org/wiki/RSA_\(algorithm\)) because it's [the most commonly used implementation of public-key encryption](https://developer.mozilla.org/en-US/docs/Introduction_to_Public-Key_Cryptography#Public-Key_Encryption) and it's used in several different applications such as SSL/TLS and SSH. Also, the algorithm is actually pretty simple so you can do an example that very clearly illustrates why factoring large semiprimes is important. Know that I'm not a cryptographer so please leave a comment if I've gotten something wrong.

So off we go.

Basically [public-key cryptography](https://en.wikipedia.org/wiki/Public_key) works by using some cool math that allows me to take a message and encrypt it using your public key which you can then decrypt using your private key. These keys are basically just a couple of big numbers that have a special mathematical relationship.

To see how this actually works, let's [run through an example taken from wikipedia](https://en.wikipedia.org/wiki/RSA_\(algorithm\)#A_worked_example). I'm going to convert their steps to python 3 code so if you want to run the code yourself, feel free. If you want a copy of all the code together, check out [this gist](https://gist.github.com/jeffnuss/8682623) on github.

1. Choose two distinct prime numbers, such as 61 and 53.

        p = 61
        q = 53

2. Compute n = pq which is 3233.

        n = p * q

3. Compute the [totient](http://en.wikipedia.org/wiki/Totient) of the product as φ(n) = (p − 1)(q − 1) which is 3120.

        phi = (p - 1) * (q - 1)

4. Choose any number 1 < e < 3120 that is [coprime](http://en.wikipedia.org/wiki/Coprime) to 3120. If we choose a prime number for e, then in order to verify that it is indeed coprime is to simply check that e is not a divisor of 3120.

        e = 17

5. This is the trickiest step. Compute d, the [modular multiplicative inverse](http://en.wikipedia.org/wiki/Modular_multiplicative_inverse) of e (mod φ(n)) which is 2753. In order to do this, I grabbed some code [from here](http://rosettacode.org/wiki/Modular_inverse#Python).

        def extended_gcd(aa, bb):
            lastremainder, remainder = abs(aa), abs(bb)
            x, lastx, y, lasty = 0, 1, 1, 0
            while remainder:
                lastremainder, (quotient, remainder) = remainder, divmod(lastremainder, remainder)
                x, lastx = lastx - quotient*x, x
                y, lasty = lasty - quotient*y, y
            return lastremainder, lastx * (-1 if aa < 0 else 1), lasty * (-1 if bb < 0 else 1)

        def modinv(a, m):
            g, x, y = extended_gcd(a, m)
            if g != 1:
                raise ValueError
            return x % m

        d = modinv(e, phi)

6. So our public key is (n = 3233, e = 17). Let's say we want to encrypt m. We do m^e [mod](http://en.wikipedia.org/wiki/Modulo_operation) n = 2790, which gives us c, our [ciphertext](https://en.wikipedia.org/wiki/Ciphertext).

        m = 65
        c = (m ** e) % n

7. Our private key is (n = 3233, d = 2753). To decrypt our ciphertext c, we do c^d mod n, which gives us back m.

        m_decrypted = (c ** d) % n
        print(m == m_decrypted)

In reality, much larger prime numbers are used for p and q as is a technique called [padding](https://en.wikipedia.org/wiki/RSA_\(algorithm\)#Padding). There are also some optimizations used to speed things up, but that is basically how RSA works.

# Why factoring huge semiprimes is a big deal

As I said before, the keys are basically just a special set of numbers with a relationship to each other. That relationship is why RSA can even work to encrypt and decrypt messages. However, it does present a possible problem. If you look at our public key, you'll notice that n is one of the numbers in the key. Remember, I make my public key available to whomever wants it (that's why it's called a public key) so they can send me encrypted messages. However, recall that n = p * q. In this case, it is not that difficult to factor 3233 back to the primes p and q. Once we have p and q, we can then calculate d, the other piece of our private key. If an attacker has your private key, there goes your secrecy.

As I said before, in real life, the primes used are much much bigger. And this is where we come to our main point. Because RSA is used in so many places, if computers were able to factor the huge semiprimes used in RSA, RSA would be in trouble. And not just RSA, but most of modern cryptography because there are other algorithms and areas of crypto that also rely on huge prime numbers. As such, using RSA keys that are too small is considered insecure, but as long as your keys are large enough (2048 bits or 617 digits), it'll take roughly [6.4 quadrillion years, which is many many times longer than the life of the universe](http://www.digicert.com/TimeTravel/math.htm), to factor them and break the encryption using standard desktop hardware. Even if you factor in [Moore's law](http://en.wikipedia.org/wiki/Moore's_law) and were to use cutting edge hardware, those cat pictures you secretly uploaded to your server aren't going to get intercepted anytime soon.

# Quantum computing

Which brings us to our next point. One of the reasons why [quantum computing](http://en.wikipedia.org/wiki/Quantum_computing) is such a big deal is because basically that whole 6.4 quadrillion years things goes out the window if you have a quantum computer. This is because of something called [Shor's algorithm](http://en.wikipedia.org/wiki/Shor%27s_algorithm). Shor's algorithm is an algorithm that allows you to find the prime factors of a number in polynomial time on a quantum computer as opposed to the fastest known factoring algorithm for a classical computer, the [general number field sieve](http://en.wikipedia.org/wiki/General_number_field_sieve), which works in sub-exponential time. Without diving into tons of detail about [algorithmic efficiency](http://en.wikipedia.org/wiki/Algorithmic_efficiency) and talking about things like [P=NP](http://www.codinghorror.com/blog/2009/06/the-girl-who-proved-p-np.html), know that once quantum computers become a reality, we'll need a new crypto standard because it will be much much faster to do prime factorization and thus derive the corresponding private key from a public key.

So there you have it. Now you know what all the fuss is about when people talk about factoring huge numbers and quantum computers. And you know that once we have quantum computers, you'll need to find a new way to securely upload your cat photos.