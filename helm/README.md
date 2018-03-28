# reddit Helm styleguide

## Chart Requirements (for dependencies)

Helm has a built-in notion of [Chart Requirements](https://docs.helm.sh/chart_best_practices/#requirements-files). For example, if my service needs a cache to operate, my Chart might optionally require the memcached Chart. If I make the requirement optional, we can do things like swap in an ElastiCache instance instead of running the cache in-cluster (this may be desirable in production).

Reddit Services have a complex web of dependencies, so we have opted for a more "flat" structure in that *Reddit Service Charts should not add other Reddit Service Charts as requirements*. For example, reddit-service-listing might require reddit-service-thing to operate, but we do not set it as a requirement in charts.yaml.

If we bake in all of our requirements to all of our Reddit Service Charts, we lose the ability to run arbitrary numbers of releases of each of our services (ie: for canaries, experiments, or traffic segmenting). It also gets to be very complex to manage the deeply nested Chart values that would result.

To summarize this with some examples:

* If your chart requires a non-Reddit system like a database, cache, broker, etc, feel free to include it as an *optional* requirement (see [Conditions and Tags](https://docs.helm.sh/chart_best_practices/#requirements-files)). It needs to be optional so that we can swap in something else in production and/or staging.
* If your chart requires another Reddit service to operate, do not include said service as a requirement. Do add a value that can be used to point at a hostname and port combo, which we'll pair with a [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/).
