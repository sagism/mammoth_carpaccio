# Mammoth Carpaccio

This is a sample solution to a feature slicing exercise based on the classic Elephant Carpaccio exercise: https://docs.google.com/document/u/1/d/1TCuuu-8Mm14oxsOnlk8DqfZAA1cvtYu9WGv67Yj_sSk/pub 

## Exercise Goals
Experience the importance of getting feedback early and often by building a product incrementally and iteratively.

## Exercise Instructions
Please form teams of 2. You will be building a small application for our sales force.
This application helps a sales person provide a price quote to a customer.

The application calculates the total price for items, including US state tax and a quantity discount (see table) and provides a bottom-line price. Each item has a price and quantity. Would be nice to be able to log and retrieve quotes and to see how far the order is from hitting the next discount bracket.

| State   | Tax Rate     |
|:-------:|-------------:|
| NV      | 8.32%        |
| CA      | 8.66%        |
| OR      | 0%           |
| AZ      | 8.40%        |
| UT      | 7.18%        |
| ID      | 6.03%        |

| Total Amount   | Discount    |
|--------------:|-------------:|
| $1,000        | 3%           |
| $5,000        | 5%           |
| $7,000        | 7%           |
| $10,000       | 10%          |
| $50,000       | 15%          |




Take 15 minutes, spec-out the solution (what you intend to build) and its design (technology, distribution..). 

Estimate the amount of time it will take you.

Build in small slices to get early feedback

Each slice is not complete till you get feedback on it (you cannot proceed to next slice)

Feedback = person from another team, or coaches (which act as customer representatives)

Each slice should be visibly different from the previous one (bring new value)

## Installing the Sample Solution

Requires a recent version of Node.

Install the required packages:

```bash
npm i
```


## Running

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

