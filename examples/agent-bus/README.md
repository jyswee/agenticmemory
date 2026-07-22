# Agent bus — two agents, one queue

A producer agent pushes work; a worker agent long-polls and processes it. No webhook server, no message broker, no human relaying tasks.

## Setup

Both agents share one space and one API key (or two keys scoped to the same space):

```bash
agmry space create "team-bus" team-bus
```

## Producer — send work

```bash
SPACE=team-bus
agmry queue $SPACE jobs push '{"type":"review","pr":42,"repo":"acme/api"}'
agmry queue $SPACE jobs push '{"type":"deploy","env":"staging","sha":"a3f2c1d"}'
```

## Worker — process until empty

`pop` exits with code 2 when the queue is empty, so a plain shell loop is the whole worker:

```bash
#!/bin/bash
SPACE=team-bus
while JOB=$(agmry queue $SPACE jobs pop --wait 25 --json); do
  echo "processing: $JOB"
  if ! process "$JOB"; then
    # don't drop failures — dead-letter them with a reason
    agmry queue $SPACE jobs dlq push "$JOB" --reason "process failed $(date -u +%FT%TZ)"
  fi
done
echo "queue empty, worker done"
```

`--wait 25` long-polls up to 25 seconds per attempt, so the loop reacts to new work in near-realtime without hammering the API.

## Inspect without consuming

```bash
agmry queue team-bus jobs            # length + head envelope
agmry queue team-bus jobs dlq list   # what failed, when, and why
```

## Same bus from MCP

MCP-connected agents use the same queues through the `queue_push`, `queue_pop`, and `queue_peek` tools — a CLI worker and an MCP producer interoperate on the same queue.
