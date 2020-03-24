# kubernetes-logging
A repository which will deploy a managed logging stack onto the cluster with some sensible defaults

It uses Elastic Search to store all the log data, kibana to provide a nice searable interface and
dashboard to that data, fluent-bit to retrieve the logs from each node and push them into Elastic Search
and Curator to keep your data to a particular size. So your node doesn't run out of HDD space and
you have to thrash the disks around frantically to stop containers from dying whilst you're trying
to figure out what to do with all this (now useless) log data you obviously didn't need.

# Motivation
I decided to split this into it's own repository because the main kubernetes-cluster repository was
starting to become a bit of a "all kinds of stuff" repository instead of having focus on individual 
elements that people wanted. Not everybody wants to deploy ES/Kibana/Curator onto their cluster.

So by splitting this into it's own repository, I let you decide whether you want or need this and add
it as you see fit.

Curator will delete all the indices which expire past your cut off point, keeping your logs down to a
minimum size. You can keep more logs if you wish. I choose to keep 3 months cause to be honest, anything
more than 3 months old is something I obviously don't care that much about.

# Installation
1. Tag a node as where you're going to run Elastic Search. I did this with: 
    - `kubectl label nodes s2 kube-logging=true`

2. Really easy, just run: `kubectl apply -f .` in the directory and if you're following the a similar pattern than I am, it will deploy all the components that
you need.

If you want to adjust the number of days to withhold logs before deleting the old ones, look at the following lines
in the "curator" yaml file:

```
unit: days
unit_count: 90
```

I think three months is a good figure to have here, but you might prefer to have something different. Feel free to change
that and apply it to your cluster.

# Conclusion

By running this against your server, you'll create the following

1. A kube-logging namespace for all resources
2. A fluent-bit daemon onto every node to collect logs from the node's docker log directory
2. An elastic search cluster allocating about 1GB of ram on the node that you tagged with `kube-logging`
3. A kibana installation pointing towards the Elastic Search cluster
4. A curator cron job which will delete all indices from the cluster older than the set number of days

# Notes
- I use a hostpath volume of 20GB to store the elastic search data in. Volumes don't really respect 
the allocated size cause I allocated 20GB and found 148GB of data stored in it. So obviously it's 
more of a suggestion than a rule.
- I set 1GB of ram to elasticsearch because with less, curator would fail to run on large data sets.
 
