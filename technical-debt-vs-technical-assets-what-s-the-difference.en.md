# [Technical debt vs technical assets: What-s the difference?](https://liblab.com/blog/accruing-technical-assets-vs-paying-off-technical-debt)

## What is Technical Debt?[](https://liblab.com/blog/accruing-technical-assets-vs-paying-off-technical-debt#what-is-technical-debt "Direct link to What is Technical Debt?")

Some people see technical debt as a list of missing features, but it should be a list of the problems that you know you have to vs want to solve. The list can vary but should include known bugs and errors in your code. It should also include how readable your code is and any bloat you might carry. Slowness in build and execution time needs consideration as well.

The point of figuring out your technical debt ahead of time is that you need to be honest with yourself. The more problems we have to solve, the less we get to work on problems we want to solve. So we treat technical debt the same as financial debt, by paying it down while avoiding more.

The rule of thumb should always be to pay off your debts first, then start accumulating assets because you don’t want to try to pay for assets when you have debts holding you down.

## Why do we get into Technical Debt?[](https://liblab.com/blog/accruing-technical-assets-vs-paying-off-technical-debt#why-do-we-get-into-technical-debt "Direct link to Why do we get into Technical Debt?")

What is Debt? We tend to think of financial debt more than other kinds of debts, like social or technical. Debt is a tool that's used to leverage your ability to do more in less time, it's not a bad thing unless it's abused. It can have destructive consequences if ignored as it compounds on itself over time. Too much leverage can be a bad thing.

All debt must get paid off by someone in time, when you accumulate technical debt, it may be the developer that comes after you who must pay it off. This perverse incentive is why debt can be dangerous in any setting. As the old Levantine Proverb says **“The Debtor is slave to the Lender”** which means you will lose power over your code.

The mantra **"Move fast and break things"** is a popular saying in our industry because it helps us move forward. It's more akin to **"Put it on the credit card"** if you think of it in technical terms because what you're saying is fix it later. In the meantime, every break has a cost that needs payment and so the more you break it the more you pay for it so to speak.

> **“Procrastination is the souls attempt to rebel against entrapment” - Nassim Taleb**

Wanting to have something done now vs waiting until later is a big reason people get into debt. They ignore what they know they need to do in place of what they want, or they procrastinate to feel good. They don't want to feel trapped by their responsibilities.

Humans also have a tendency to overestimate their own abilities, we are not good at estimating risk. The reason we underestimate risks is that otherwise, indecision would paralyze us. So putting off problems we know we need to solve for later makes us assume that we can solve those problems later. Worse, it makes us assume we can solve those problems in a much more limited timeframe than if we had done it sooner.

So as developers we are always adding technical debt to make it easier on ourselves now. We don't realize we are punishing either our future selves or those that come into the project after us.

## What is a Technical Asset?[](https://liblab.com/blog/accruing-technical-assets-vs-paying-off-technical-debt#what-is-a-technical-asset "Direct link to What is a Technical Asset?")

A Technical Asset does not mean future-proofing or building a feature early, but rather an early problem solved. It's an investment into a known problem that from experience you know that you, or those that come after you, will have. The only way to know if it's not future-proofing is through experience, if you've had the problem then it's an asset.

In the same way experience will tell you whether a financial asset will make you money. It's a bet. Not every investment into an asset will pay off in the way you want it to, or for your own personal benefit. The point is not to stop investing in assets, but rather to teach people how to spot good from bad investments.

The accumulation of financial assets gives you freedom in life. In much the same way technical assets give you the freedom to code without the fear of regression. You get to choose what problems you want to solve when you invest in technical assets upfront.

> **“A society grows great when old men plant trees in whose shade they shall never sit” - Greek Proverb**

You're building technical assets for others more than for yourself and this is why it's hard to do. In software development, we like to get to the root of the answer quickly and so we make life easier for ourselves now. We tend to not think about our future selves or others who will take over the project after us.

Yet many of the greatest stories in technology development come from the opposite.

-   Amazon AWS was an asset that came at the very end of an expensive mono codebase debt payoff.
-   Facebook made investments in React and Open Compute to fix scalability.
-   Google Instant reinvented caching as an investment into what seemed impossible.

They had to first pay off their technical debt before they could build assets. These were not core products or features, but they prioritized a known problem ahead of time.

## Investing in an SDK is a Technical Asset[](https://liblab.com/blog/accruing-technical-assets-vs-paying-off-technical-debt#investing-in-an-sdk-is-a-technical-asset "Direct link to Investing in an SDK is a Technical Asset")

At liblab we are in the asset building business.

If you build API’s as an organization then our job is to help you to automate your [SDK build process](https://liblab.com/blog/why-do-i-need-to-build-an-sdk) and provide you with technical assets that you can release to your customers.

You do the part of documenting your API’s correctly using [OpenAPI spec](https://liblab.com/blog/why-your-open-api-spec-sucks), Postman collections or [GraphQL](https://liblab.com/blog/using-github-graphql-api-with-github-actions) schema and we do the hard work of building long term technical assets that you don’t have to manage and instead can promote to your customers.

## How to find other Technical Assets to Invest in?[](https://liblab.com/blog/accruing-technical-assets-vs-paying-off-technical-debt#how-to-find-other-technical-assets-to-invest-in "Direct link to How to find other Technical Assets to Invest in?")

Much like picking any investment asset, you have to plan and discuss it like investors. Present a prospectus about what the investment would look like and the tradeoffs. In the software world, we would write a Request for Comment or RFC to break down the idea in a way that can be peer reviewed.

You want to instill a process that creates forcing functions that limit the scope of work. Don't let your investment go off the rails without first enforcing tests and coverage. The earlier you are at enforcing linting for code readability the more proactive you will be. Building out automated CI/CD and managed deploys ensure less pain for your company long term.

Avoid survivor bias in determining a good investment by remembering the Lindy Effect:

> **_“If a book has been in print for forty years, I can expect it to be in print for another forty years. But, and that is the main difference, if it survives another decade, then it will be expected to be in print another fifty years. This, simply, as a rule, tells you why things that have been around for a long time are not ‘aging’ like persons, but ‘aging’ in reverse. Every year that passes without extinction doubles the additional life expectancy. This is an indicator of some robustness. The robustness of an item is proportional to its life!”_**

Just like any bad real world investment might force companies and organizations out of the market entirely, a bad technical investment might do the same for tech companies. So although it may be counter intuitive in the software industry to choose stuff that’s been around for a while, the best approach might be to invest in implementations that have proven themselves sturdy over time.

In conclusion, you have to pick the processes that have been proven effective over time and across companies. Adopting those successful processes as technical assets allows you to accrue them for yourselves but not at the expense of paying off your technical debts beforehand. Paying your debts off early will give you the freedoms you need to work on code you want to work on instead of stuff you know you have to. This discipline helps you avoid the “put it on the credit card” mindset earlier in your process.
