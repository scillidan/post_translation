# [When Is Insurance Worth It?](https://entropicthoughts.com/when-is-insurance-worth-it)

TL;DR: If you want to know whether getting insurance is worth it, [use the Kelly Insurance Calculator](https://xkqr.org/insurance). If you want to know why or how, read on.

## Misunderstandings about insurance

People online sometimes ask if they should get some insurance, and then other people say incorrect things, like[^1]

> This is a philosophical question; my spouse and I differ in views.

or

> Technically no insurance is ever worth its price, because if it was then no insurance companies would be able to exist in a market economy.

or

> Get insurance if you need it to sleep well at night.

or

> Instead of getting insurance, you should save up the premium you would have paid and get compounding market return on it. The money you end up with is on average going to be more than whatever you’ll end up claiming on the insurance.

or

> If you love your children, you should get disability insurance for them.

or

> You should insure only what you cannot afford to lose.

These are the things I would say in response.

- It is not a philosophical question, it is a mathematical one.
- Technically, some insurance is worth its price, _even when_ the insurance company makes a profit.
- Whether or not to get insurance should have nothing to do with what makes one sleep – again, it is a mathematical decision with a correct answer.[^2]
- Saving up the premium instead of getting insurance is making the mistake of conflating an ensemble average with a time average.
- Love does not make insurance a mathematically appropriate decision. Running the numbers does.

The last quote (“things you cannot afford to lose”) is the closest to being true, but it doesn’t define exactly what it means to afford to lose something, so it ends up recommending a decision based on vibes anyway, which is wrong.

In order to be able to make the insurance decision wisely, we need to know what the purpose of insurance really is. Most people do not know this, even when they think they do.

## The purpose of insurance

The purpose if insurance is _not_ only to help us pay for things that we literally do not have enough money to pay for. It does help in that situation, but the purpose of insurance is much broader than that. What insurance does is help us avoid large drawndowns on our accumulated wealth, in order for our wealth to gather compound interest faster.

Think about that. _Even though insurance is an expected loss for the insured_, it helps us earn more money in the long run. This comes back to the [Kelly criterion](https://entropicthoughts.com/the-misunderstood-kelly-criterion.html), which teaches us that the compounding effects on wealth can make it worth paying a little up front to avoid a potential large loss later.[^3]

_This_ is the hidden purpose of insurance. It’s great at protecting us against losses which we literally cannot cover with our own money, but it also protects us against losses which set our wealth back far enough that we lose out on significant compounding effects.

To determine where the threshold for large enough losses is, we need to calculate.

## Computing when insurance is worth it

The Kelly criterion is not just a general idea, but [a specific mathematical relationship](https://entropicthoughts.com/the-misunderstood-kelly-criterion.html). We can use this to determine when insurance is worth it. We need to know some numbers:

- What is our current wealth WW? To keep things simple, this might be any liquid funds we can access within a few days.
- How much is the premium PP? For this, use the total cost of the insurance over the period insured.[^4]

Then we need to estimate the probability distribution of the bad events that could occur.[^5] In other words, for each bad event ii we can think of, we estimate

- What is the probability pip\_i that this event happens? and
- If it does happen, what would be the uninsured cost cic\_i?

We’re going to ignore the deductible for now because it makes the equation more complicated, but we’ll get back to it. We plug these numbers into the equation[^6] for the value VV of the insurance to someone in our situation:

![](https://scillidan.github.io/image_post/when-is-insurance-worth-it_01.webp)  
```tex
V \;=\;\log\bigl(W - P\bigr)
\;-\;\bigl(1 - \sum p_i\bigr)\,\log W
\;-\;\sum\bigl[p_i\,\log\bigl(W - c_i\bigr)\bigr]
```

If this number is positive, then the insurance is worth it. If it is negative, we would do better to pay the costs out of our own pockets.

## Motorcycle insurance

In a concrete example, let’s say that our household wealth is $25,000, and we’ve just gotten a motorcycle with some miles on it already. Insuring this motorcycle against all repairs would cost $900 per year. We might think of two bad events:

- It ends up needing expensive maintenance due to its age. We expect this to happen once in the next three years, meaning there’s roughly a 33% probability it’s needed the next year. This costs maybe $2000.
- We end up riding irresponsibly and wreck it, or it gets stolen, or somehow needs to be replaced completely. Maybe there’s a 1/40 risk of this any given year, and it would cost $8000.

Assuming no deductible, would this be worth it? Yes! Solving the equation – or entering the parameters into the [Kelly insurance calculator](https://xkqr.org/insurance) – we see that we should be willing to pay a premium of up to $912 in this situation. If our wealth had been $32,000 instead, the insurance would no longer have been worth it – in that situation, we should not spend more than $899 on it, but the premium offered is $900.

## The effect of the deductible

In the same example as above, now set a fixed deductible of $500 for both events, and watch the value of the insurance plummet! Under those terms, we should only accept the insurance if our wealth is less than $10,000.

We can put a fixed deductible dd into the equation as such:

![](https://scillidan.github.io/image_post/when-is-insurance-worth-it_02.webp)  
```tex
V = \bigl(1 - \sum p_i\bigr)\,\log(W - P)
\;+\;\Bigl(\sum p_i\Bigr)\,\log\bigl(W - P - d\bigr)
\;-\;\bigl(1 - \sum p_i\bigr)\,\log W
\;-\;\sum\bigl[p_i\,\log\bigl(W - c_i\bigr)\bigr]
```

If the deductible varies based on the event ii, the sum in the second term can be taken over the appropriate probability-weighted logarithm instead.

## Helicopter hovering exercise

To test your knowledge, we’ll run with one more example.

Let’s say you get the opportunity to try to hover a helicopter close to ground, for whatever reason. There’s a real pilot next to you who will take control when you screw up (because hovering a helicopter is hard!) However, there’s a small (2 %) chance you will screw up so bad the other pilot won’t be able to recover control and you crash the helicopter. You will be fine, but you will have to pay $10,000 to repair the helicopter, if that happens.

You can get insurance before you go, which will cover $6,000 of helicopter damage (so even with insurance, you have to pay $4,000 in addition to the insurance premium if you crash), but cost you $150 up front. Do you take it?

You probably know by now: it depends on your wealth! There’s a specific number of dollars in the bank you need to have to skip the insurance. Whipping out the [Kelly insurance calculator](https://xkqr.org/insurance), we figure it out to be $34,700. Wealthier than that? Okay, skip the insurance. Have less than that? It’s wise to take the offer up.

## It’s not that hard

I am surprised not more people are talking about this. Everyone goes around making insurance decisions on vibes, even as these decisions can be quite consequential and involve a lot of money. There’s just a general assumption that insurance decisions are incalculable – but the industry has calculated with them for at least seventy years! Are people not a little curious how they do it?

More specifically: until now, there has been no insurance calculator that actually uses the Kelly criterion. All others use loose heuristics. Who thinks that leads to better decisions?

## Appendix A: Anticipated and actual criticism

I think there are two major points of disagreement possible in the description above:

1. The Kelly criterion is bad, and
2. The probability distribution of bad events is unknown.

Both of these points are technically true, but not as meaningful as their proponents seem to think.

Yes, the Kelly criterion is too aggressive for most people, who do not value maximum growth over all else. Most people want to trade off some growth against security. The correct response here is not to throw the baby out with the bathwater and ignore Kelly entirely – the correct response is to use a fractional Kelly allocation. This can be done quite easily by entering a lower wealth in the [Kelly insurance calculator](https://xkqr.org/insurance). See [the Kelly article](https://entropicthoughts.com/the-misunderstood-kelly-criterion.html) for more discussion on this.

The probability distribution of _anything_ is unknown, but this is not a problem. Good forecasters estimate accurate probabilities all the time, and nearly [anyone can learn to do it](https://entropicthoughts.com/improving-a-gut-feeling-forecast.html).

But, perhaps most fatally, the people who oppose the method suggested in this article have not yet proposed a better alternative. They tend to base their insurance decisions on one of the incorrect superstitions that opened this article.

## Appendix B: How insurance companies make money

The reason all this works is that the insurance company has way more money than we do. If we enter the motorcycle example with no deductible into the Kelly insurance calculator again, and increase our wealth by a factor of ten, we see the break-even point moves down to $863. This is the point where the insurance _starts_ being worth offering for someone with 10× our wealth!

In other words, when someone with 10× our wealth meets us, and we agree on motorcycle insurance for $900, _we_ have made a $12 profit _and_ the insurer has made a $37 profit.

It sounds crazy, but that’s the effect of the asymmetric nature of differential capital under compounding. This is the beauty of insurance: deals are struck at premiums that profit _both_ parties of the deal.

## Appendix C: The relativity of costs

The clever reader will also see that if we set the deductible to be event-dependent, and create a virtual event for when nothing bad happens (this event has a deductible and cost of zero), a lot of the terms are similar and can be combined. Indeed, the equation can then be given as

![](https://scillidan.github.io/image_post/when-is-insurance-worth-it_03.webp)  
```tex
V \;=\;\sum\bigl[p_i\,\log\!\frac{\,W - P - d_i\,}{\,W - c_i\,}\bigr]
```

This, perhaps, makes it clear that it is not the absolute size of the wealth that matters, but its size in proportion to the premium, deductible, and cost of events.

[^1]: These are real quotes from just one forum discussion. Some answers were even worse, but I’ve picked out a representative sample.
[^2]: I’ve gotten pushback on this item. People claim that if you need insurance to sleep at night, it might be worth paying for even when not mathematically appropriate because sleep itself is valuable. I agree. Look at this argument more as a “if you plan your insurance decisions wisely, you might discover you don’t need as much insurance as you think to sleep well at night – or maybe you need more of it.”
[^3]: The typical example is how it takes as long to go from $2,000 to $10,000 as it does from $10,000 to $50,000. This means that if we are forced to pay $8,000 out of our $10,000 wealth, we will end up with $10,000 again at the same time as we would have ended up with $50,000 if we had not been forced to pay. Losing $8,000 at one point is equal to a $40,000 loss later on, once compounding is taken into account. No wonder Einstein coined compounding the eighth wonder of the world. This effect is huge. Having to shell out 20% of our wealth for an unexpected accident is so bad – even if the accident is improbable – that we may want to chuck out a guaranteed 0.5% of our wealth to get out of that risk.
[^4]: If you’re signing a long-term insurance such as a life insurance bound over a few decades, you may need to compute this cost using compounding maths.
[^5]: In what follows we assume these events ii form a partition, i.e. that they are mutually exclusive. If we think multiple bad things can happen at once, we’ll have to model that by including some virtual events ii that are compositions of other events.
[^6]: For information on how this is derived, [see the previous article on the Kelly criterion](https://entropicthoughts.com/the-misunderstood-kelly-criterion.html).
