# RFC <issue#> - <YYYY-MM-DD> - <title>

*One paragraph description of the change.*

This RFC proposes a rate limiting scheme for `File` source in order to prioritize log collection using a group based approach.

## Context

*- Link to any previous issues, RFCs, or briefs (do not repeat that context in this RFC).*
- Issue: [Controlling log flow through group based collection](https://github.com/vectordotdev/vector/issues/10182)
- Pull Request: [enhancement(kubernetes_logs source): Add group based log collection using event metadata](https://github.com/vectordotdev/vector/pull/10652)
## Cross cutting concerns

*- Link to any ongoing for future work relevant to this change.*
## Scope

### In scope

*- List work being directly addressed with this RFC.*
- Rate limit log collection from `File` based sources:
    - File
    - Kubernetes_logs

### Out of scope

*- List work that is completely out of scope. Use this to keep discussions focused. Please note the "future changes" section at the bottom.*
- Non-file source based components
## Pain

- What internal or external *pain* are we solving?
- Do not cover benefits of your change, this is covered in the "Rationale" section.

## Proposal

### User Experience

- Explain your change as if you were describing it to a Vector user. We should be able to share this section with a Vector user to solicit feedback.
- Does this change break backward compatibility? If so, what should users do to upgrade?

### Implementation

- Explain your change as if you were presenting it to the Vector team.
- When possible, demonstrate with pseudo code not text.
- Be specific. Be opinionated. Avoid ambiguity.

## Rationale

*- Why is this change worth it?*
*- What is the impact of not doing this?*
*- How does this position us for success in the future?*

In the case of a big cluster with high log volume, we need to have a data flow control in the log collection process so that we can limit the collection rate based on the priority of application workloads in the cluster.

## Drawbacks

- Why should we not do this?
- What kind on ongoing burden does this place on the team?

## Prior Art

- List prior art, the good and bad.
- Why can't we simply use or copy them?

## Alternatives

- What other approaches have been considered and why did you not choose them?
    1. Currently, `Kubernetes_logs` and `File` sources have `max_read_bytes` which controls the amount of data that can be read from a single file at a given time. This parameter is useful for controlling the log flow in your data pipeline. However, `max_read_bytes` will give equal opportunity to all files included in the source. 

    2. To control the flow of logs through our pipeline, we could add a group-based rate-limiting approach where each file member will be a part of a group with a group line limit (`limit`) and a time period (`rate_window_secs`) in which that limit can be satisfied. 
    For example, in Kubernetes_logs, files could be grouped on the basis of namespace alone, pod name alone or using both namespace and pod names.
    ```
        [sources.kube]
        type = "kubernetes_logs"
        rate_window_secs = 30

        # group using only namespace
        [[sources.kube.rule]]
            namespace = "logging"
            limit = 20000
            # 20000 line limit in 30s

        # group using only podname
        [[sources.kube.rule]]
            pod = "frontend"
            limit = 100
            # 100 line limit in 30s

        # group using both namespace and podname
        [[sources.kube.rule]]
            namespace = "website"
            pod = "backend"
            limit = 1500
            # 1500 line limit in 30s
    ```
    However, as explained in [PR#10652/comment](https://github.com/vectordotdev/vector/pull/10652), we can achieve this configuration by chaining `route` and `throttle` transforms. Modifying the way Vector reads a file will require a lot of information to be propagated back to `File` source. 

- How about not doing this at all?

Adding this feature in Vector will allow us to collect logs efficiently and help in downstream observability tasks. Having this feature in our source component will add more *noisy* logs from low priority applications.
## Outstanding Questions

- List any remaining questions.
- Use this to resolve ambiguity and collaborate with your team during the RFC process.
- *These must be resolved before the RFC can be merged.*

## Plan Of Attack

Incremental steps to execute this change. These will be converted to issues after the RFC is approved:

- [ ] Submit a PR with spike-level code _roughly_ demonstrating the change.
- [ ] Incremental change #1
- [ ] Incremental change #2
- [ ] ...

Note: This can be filled out during the review process.

## Future Improvements

- List any future improvements. Use this to keep your "plan of attack" scope small and project a sound design.
