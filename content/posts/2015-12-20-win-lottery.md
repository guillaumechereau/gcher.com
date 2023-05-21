---
categories:
  - quantum
  - multiple
  - universe
aliases: /win-lottery.html
subtitle: 0.000005 percent of the time, it works 100 percent of the time
title: How I made my parallel self win the lottery
date: 2015-12-20
---


While I was in France recently I decided to make one of my parallel self
a millionaire.  It's very easy and you too can do it if you want.

How to do it?  First you have to check how the national lottery of the country
where you are works.  In France, a lotto ticket costs 2 euro, you have to
pick five different numbers from 1 to 49 (included), and one 'lucky' number
from 1 to 10 (included).  To win the big price, all your numbers must match the
winning numbers.  If you do, you can earn up to a few millions.

We are going to play this -otherwise pretty stupid- game in such a way that we
are sure that we are going to win in at least one parallel universe.

To pick your numbers, go on one of the numerous websites that offer live
stream of quantum random numbers, for example the ANU Quantum Random Number
Server (http://qrng.anu.edu.au).  From there pick a long enough random number.
In my case I got this one (in hexadecimal):

    1a7bc82fe1393178549ec0959b4b862d52d7f5ea02f16f5c848f15a1c68cedd789e2f

You should start to see where this is going.  If the multi worlds
interpretation of quantum physics is correct, and assuming the number really
was computed using a pure quantum process, then there exists one parallel
universe for each possible values of this random number.

Now that the universe has been separated into multiple equi-probable versions,
only differentiated by this number, we can start to pick our lottery numbers.
I used a simple python script that would generate the 5 + 1 numbers from the
initial random number I got.  The algorithm is not very important, the only
difficulty is to make sure that we don't get two times the same number, and to
ensure that each possible set of values has an equal chance to appears.

In my case the numbers I played are:

    [28, 31, 20, 12, 1] 8

And I know for sure that all the other possible numbers have been picked in
different universes.  Think about it: there are universes where you are right
now reading the same web page, with the only difference being that those
numbers (and the initial random number) are different.

Now, assuming that there is no cross correlation between the ANU Quantum Random
Number Server and the French lottery, I can be sure that in at least one
universe I got the numbers:

    [09, 20, 26, 27, 47] 07

Which happens to be the winning numbers of the day I bought the ticket.

I can know rest assured that somewhere in the multiverse, one of my parallel
self did actually win the lottery.
