# Seed Stage

**Number of Employees**: 5 - 10

Your Jaffle Shop is the talk of the town. You've been featured by a few food bloggers and influencers and your sales are growing rapidly.
You now have a constant stream of new users signing up on the website, in addition to your existing users who keep coming back for more jaffles.

You've added a new feature that allows users to leave reviews for the jaffles they've purchased.
You've also added even more jaffles to your menu, including "The Breakfast Jaffle" and "The Dessert Jaffle" bringing the total number of jaffles to 5.

Your website is starting to slow down and you're starting to see some errors when users try to place an order, but you don't have the time or expertise to fix them.
You also need a full time customer support person to handle all the questions and complaints that are coming in, as well as an operations person to help you manage the inventory and shipping.

You decide to raise some money to help you grow the business.

## Data Requirements

### Automating KPIs

You no longer have the time to update your spreadsheet every weekend, so you decide to automate this process.

### Basic Operations

You need to make sure you have the raw ingredients to make jaffles, so you set up a simple dashboard that keeps track of
your inventory levels so you can place an order if you're running low.

Your customer support person also needs a way to fetch the details of an order if a user has a question or complaint,
so you set up a simple interface that allows them to search for an order by order number.

## Data Investments - Minimal

At this stage you can still get by with a simple SQL database and a cron job that runs every night to update your spreadsheet.
The amount of data you're processing is quite small so you're still running queries against your production database.
