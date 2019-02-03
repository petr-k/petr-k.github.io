---
layout: page
title: "Kubernetes tips and tricks, part&nbsp;2: inspecting logs"
snippet: |
  This time we'll focus on a simple, yet suprisingly powerful kubectl feature:
  inspecting container logs.
tags: [kubernetes]
comments-issue-id: 2
---

Chances are your Kubernetes setup includes a log collection pipeline, perhaps
leveraging solutions such as the ELK stack (or
"[EFK](https://blog.ptrk.io/how-to-deploy-an-efk-stack-to-kubernetes/)". As with
almost anything in Kubernetes, when it comes to logs, there's many ways to skin
the cat. Still, it's useful to get to grips with barebones log-related features
of `kubectl`, which are surprisingly powerful and sometimes downright necessary
when debugging, if all else fails.

Having said that, this installment will be just a super-quick refresher on
`kubectl logs`. We'll assume that your cluster is
[set up](https://kubernetes.io/docs/concepts/cluster-administration/logging/) in
a "standard" way so that `stdout` and `stderr` of running containers are
captured.

## 1. Single vs multi-container pods

This one's so simple it doesn't really count as a tip. If there's more than one
container within a `Pod`, `kubectl logs` will let you specify the intended one
via the `-c` flag (also covers _init containers_). Otherwise, it will pick the
first running container.

Should you want to inspect logs from _all_ containers of a `Pod`, you can use
the `--all-containers` flag.

## 2. Previous logs

When a `Pod` is restarted because of a container crash, you will need to see
logs of the _previous_ container, not the _current_ one, to examine what
actually happened.

This is what the `-p`/`--previous` flag is for:

```shell
kubectl logs <pod-name> -p
```

Note that log storage retention is subject to
[kubelet garbage collection](https://kubernetes.io/docs/concepts/cluster-administration/kubelet-garbage-collection/)
and might differ from cluster to cluster.

## 3. Since when?

If you don't specify otherwise, `kubectl logs` will attempt to show logs since
the container startup. Sometimes this can amount to a huge number of log entries
while you're usually most interested in what's been happening only recently.
Luckily, `kubectl logs` has a few options to control this behavior:

- `--tail`, e.g. `--tail 50`, showing the last 50 lines.
- `--since`, e.g. `--since 5m`, showing logs for the last 5 minutes.
- `--since-time` which takes a RFC3339 datetime, e.g.
  `--since 2019-01-12T21:35:57+00:00`. The datetime format makes this option a
  little less convenient for casual entry from the command line, but can still
  be useful in scripts.

## 4. Live logs

If you sometimes find yourself repeatedly invoking `kubectl logs` on the same
container, use the `--follow`/`-f` flag. Usually makes sense when coupled with
one of the options from the previous section:

```shell
kubectl logs <pod-name> --tail 10 --follow
```

## 5. Logs from multiple Pods at once

Running an application in multiple instances poses a slight problem when
inspecting logs conveniently. But since all `Pod`s based on a `Deployment` spec
should have the same label ([see previous
post]({% post_url 2019-01-06-kubernetes-tips-and-tricks-part-1-kubectl %})), you
can combine `kubectl logs` with the label selector `-l` flag:

```shell
kubectl logs --tail 10  -l app=nginx-deployment
```

It'll give you logs across multiple `Pod`s, but still has a couple of drawbacks:

- You can't easily tell where logs of one container end and of another one
  begin.
- You can't `--follow` logs at the same time.

There are some 3rd-party tools striving to address the above limitations,
though: check out [kubetail](https://github.com/johanhaleby/kubetail) or
[stern](https://github.com/wercker/stern). The former will even give you a fancy
color-coded output.

## 6. Conclusion

As always, be sure to take a good look at the myriad of options `kubectl`
subcommands offer - such as the reference for
[kubectl logs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs),
which we've touched on in this post. You're likely to stumble upon few useful
gems.

Next time, we'll get into something a little meatier: inspecting `Pod`s live
traffic using Wireshark.
