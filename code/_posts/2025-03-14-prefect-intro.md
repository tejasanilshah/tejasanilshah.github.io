---
layout: single
classes: wide
title:  "Prefect Intro"
collection: code
tags: prefect
excerpt: "What was Airflow again?"
---

# Prefect - Airflow's hot cousin?

I got my first taste of task scheduling with plain old cron jobs.
I used them to setup notifications with new and interesting word from a long text file on my Ubuntu machine.
At the time I didn't ever think about how important and ridiculously difficult it is to make sure that a specific script run
at a specific time.

A few adventures in Israel, and Estonia later, and also completely oblivious to this problem I ended up working as a Data Engineer at STACC.
That's when I was reintroduced to the whole ecosystem of workflow orchestration.
Every platform has its own way of defining the problem that it tries to solve.
However, at the heart of it, the problem that they solve is this:

> 1. Run a specific program at a specific time, in a specific environment
> 2. Based on the result of step 1, repeat step 1 for a possibly different program, different time and different environment 

Did I mention I like loops?

Anyway, getting back to the problem at hand.
There are a plethora of tools and platforms out there which have their own way of solving this problem.
The biggest and the most established of which is [Airflow](https://airflow.apache.org/).

Now for those who already know about Airflow and have worked with it a fair amount,
I've gotta say [Prefect](https://www.prefect.io/) feels so much better.

Now, I wasn't born this morning, and I am fully aware that Prefect is just another tool.
Having said that it comes with its own set of good, bad, and ugly.

# The Good

## Plain Python*
With Prefect, you simply define the order in which you want to run your programs/functions the way you'd
normally execute the code, and then sprinkle the necessary decorators.
That's it. You don't have to really worry about a DAG as such.

With Airflow however, you've to define DAGs inorder to achieve the same result.

## Free scheduler in the cloud*
When working with Airflow, you had to ~~worry~~ setup at least three things before you even begin running flows:
1. The scheduler
2. The worker 
3. The webserver for the UI 

With Prefect, you get the scheduler, and webserver for "free". The free tier allows you to run 5000 flow runs a day.
Which is pretty generous and so far I haven't run into a problem with this.

The UI is also pretty slick compared to the prehistoric UI of Airflow.

## Timezone aware
One of the things I remember worrying about the entire time when working with Airflow was that it wasn't timezone aware.
With Estonia still observing Daylight Savings Time (DST), it was always a mess trying to schedule even slightly complicated
DAGs. 

With Prefect however, you simply set the timezone and that's it. The schedules work the way you'd expect in local time.

## No dependency hell
A project that I got stuck on when working with Airflow was that of upgrading from Airflow 1.x to Airflow 2.x.
I was stuck on that project for more than a month. Now of course, the project was a lot more complicated that simply
upgrading Airflow. It also included upgrading the Python version from 3.6 to 3.9. Given that it was a fairly large legacy project,
I still take pride in the fact that I kept chipping away at it little by little and eventually got everything working.

With Prefect, no such complications so far. It is one package, and a few more optional dependencies that you can
list in one line of your `requirements.txt` file. No BS of listing constraints and then praying to the Airflow gods that
it works.

# The Bad

## Unclear pricing
The free tier of Prefect is quite generous, and allows 5000/flow runs per day.
Once your scheduling needs go over the free tier, it isn't very clear on what the move to a paid tier would cost.

The website simply asks you to contact the team. Of course, with a product like Prefect which can do a million things
pricing it isn't very straightforward. I get that, however it would be nice to have some indicative prices without having to
reach out for a quote.


# The Ugly

## deploy, serve, workers, agents, what?
With Prefect, there are some concepts that just end up confusing me a lot.
You can use a function called `.deploy` to create a deployment, and you can also serve a flow with `.serve`, but then you actually create
a deployment. I remember with one of these functions, Prefect also created a docker image for you and uploaded it dockerhub.

Now that's one function doing too many things.

## Sketchy documentation at times
As part of my take home task for my current position, I built an ETL pipeline on AWS, where the worker machine would be
spawned on the trigger of a run. Although there were docs for something like this, I could build the full project without
help from a Sales Engineer at Prefect. Eventually the problem simply was that I had to meddle with some settings
from the Prefect UI.

This is where I believe Airflow shines.
Sure, there is so much documentation that finding what you need might be a bit challenging, but at the very least you will
find some help from the docs.

# The verdict
The way I see it, Prefect is another tool in a Data Engineer's tool chest. It excels at certain things, and leaves you
wanting on others. Need a quick way to orchestrate some Python, give Prefect a go, and you'll be surprised at how quickly you'll
be up and running.