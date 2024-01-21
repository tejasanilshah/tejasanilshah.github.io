---
layout: single
classes: wide
title:  "Terraform conundrums"
collection: code
tags: aws terraform grafana
excerpt: "When one `terrform apply` fails, try two!"
---

I'm currently working on a project where I'm building and deploying a data monitoring dashboard. As there was an idea of monitoring data drift and data quality parameters we first toyed with the idea of using [Evidently](https://www.evidentlyai.com/) as a solution for this problem. However after working through a few `Hello, World!` examples of Evidently, it quickly became clear to me that it wouldn't work.

Firstly, Evidently expects data to be served in data frames, and most of our data was in a postgres database. Now, although querying a database and converting the result to a data frame isn't that big an overhead. The problem was setting up a DAG in Airflow to ensure that the metrics are observed on a schedule. Now, the metrics that we wanted to track could be achieved with simple SQL queries, so Evidently just felt like the wrong tool for the job.

After a quick chat with the lead, we agreed that our best bet was to ditch Evidently and work with [Grafana](https://grafana.com/) instead. A day later, I had the dashboard ready and working locally with a docker container, the next step was to deploy.

This is where [Terraform](https://www.terraform.io/) comes into the picture. One of the interesting problems that I came upon was routing. The state in which I found the project was something like this:

```hcl
resource "aws_lb_listener_rule" "mlflow" {  
  ...
  priority     = 2
  condition {  
    path_pattern {  
      values = [  
        "*",  
      ]  
    }  
  }  
  ... 
}  
resource "aws_lb_listener_rule" "airflow" {  
  ...
  priority     = 1
  condition {  
    path_pattern {  
      values = [  
        "/airflow",  
        "/airflow/*",
      ]  
    }  
  }  
  ...
}
```

In the current state of the project we were already serving the Airflow dashboard on `http://ip.address.of.project/airflow/` and MLFlow on `http://ip.address.of.project/`. 

My objective was to serve the Grafana dashboard that I built from `http://ip.address.of.project/grafana/`.  So, following the pattern and just the right amount of `CTRL-C`ing, `CTRL-V`ing, and replacing later. I got myself to the following code.

```hcl
resource "aws_lb_listener_rule" "mlflow" {  
  ...
  priority     = 2
  condition {  
    path_pattern {  
      values = [  
        "*",  
      ]  
    }  
  }  
  ... 
}  
resource "aws_lb_listener_rule" "airflow" {  
  ...
  priority     = 1
  condition {  
    path_pattern {  
      values = [  
        "/airflow",  
        "/airflow/*",
      ]  
    }  
  }  
  ...
}
resource "aws_lb_listener_rule" "grafana" {  
  ...
  condition {  
    path_pattern {  
      values = [  
        "/grafana",  
        "/grafana/*",
      ]  
    }  
  }  
  ...
}
```

I skipped the priority value, since I read that the priority would be auto-assigned. This meant that `aws_lb_listener_rule.grafana.priority` was set to `3`.

This was my first mistake. As, this would mean that the `path_pattern` of MLFlow; `*`, which essentially is a catch all, would end up catching everything that wasn't caught by the `path_pattern` of Airflow. This made the Grafana dashboard unreachable.

So, I went from that piece of code to:

```hcl
resource "aws_lb_listener_rule" "mlflow" {  
  ...
  priority     = 3  
  ... 
}  
resource "aws_lb_listener_rule" "airflow" {  
  ...
  priority     = 1  
  ...
}
resource "aws_lb_listener_rule" "grafana" {  
  ...
  priority     = 2
  ...
}
```

This should've fixed the problem of Granfana being unreachable. With high hopes, I started do a `terraform plan`, the plan would go through quite well, however, it would fail on apply complaining that the priority values were already taken. So in the end, in order to get to the desired state, I had to do apply terraform twice. First taking the priorities to some random values, and then to the desired values.

Ultimately, we got the dashboard working, and I learnt a fair amount about AWS and Terraform.