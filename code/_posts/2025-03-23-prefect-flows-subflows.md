---
layout: single
classes: wide
title:  "Composing Prefect flows with sub-flows"
collection: code
tags: prefect
excerpt: "Composing flows inside flows"
---

A common use case when working with workflow orchestration tools is running multiple instances of a certain flow with the change
of a few parameters. It could be getting satellite data for multiple locations, or retrieving specific files from different 
sources and delivering them to specific locations. In the interest of keeping the code DRY, of course, the natural way to
approach this problem is to define the flow function once, and change the parameters it gets.

Now to run through all the parameters, you'd then need to create a parent flow which sets up and runs the flows with all the
different parameters that you want to run. This is where Prefect starts getting a little tricky.

This blog post will highlight a quick gotcha when working with parent-flow and child-flows. (Or flows and sub-flows)

It is important to keep in mind how flow errors are handled in Prefect.

> The parent-flow errors out and halts execution upon the first child-flow hitting an error.

This might not be the behaviour that you desire from your flows.

## A minimal example

### Default behaviour

Here is a simple example of a parent flow, with 5 child flows, where `child_flow_2` always fails

```python
from time import sleep
from prefect import flow, serve

@flow(log_prints=True, flow_run_name="child_flow_{flow_param}")
def child_flow(flow_param: int):
    sleep(flow_param)
    print(f"In flow {flow_param}")
    if flow_param==2:
        raise ValueError("child_flow_2 always fails")


@flow(log_prints=True)
def parent_flow():
    for i in range(0,5):
        child_flow(i)

if __name__ == "__main__":
    parent_flow_deployment = parent_flow.to_deployment(name="parent_flow_dev_deployment")
    serve(parent_flow_deployment)
```
![default-behaviour.png](/assets/images/code/2025-03-23-prefect-flows-subflows-default-behaviour.png)

### Desired behaviour

Simply by wrapping the call to `child_flow` in a `try-except` statement and handling the exception 
we can continue on with the rest of the child-flows.

In Prefect there is an annotation called `allow_failure`, but there is [no documentation](https://reference.prefect.io/prefect/?h=allow_failure#prefect.allow_failure) on how to use it.
So currently the `try-except` block is a simple way to get the desired behaviour.

```python
from time import sleep
from prefect import flow, serve

@flow(log_prints=True, flow_run_name="child_flow_{flow_param}")
def child_flow(flow_param: int):
    sleep(flow_param)
    print(f"In flow {flow_param}")
    if flow_param==2:
        raise ValueError("child_flow_2 always fails")


@flow(log_prints=True)
def parent_flow():
    for i in range(0,5):
        try:
            child_flow(i)
        except Exception as e:
            print(f"Exception in child_flow_{i}: {e}")

if __name__ == "__main__":
    parent_flow_deployment = parent_flow.to_deployment(name="parent_flow_dev_deployment")
    serve(parent_flow_deployment)
```

![desired-behaviour.png](/assets/images/code/2025-03-23-prefect-flows-subflows-desired-behaviour.png)


In essence, a successful parent flow is one which brings the child-flows into existence and gives them the parameters
necessary to run. A child flow succeeding or failing is the child-flow's business.