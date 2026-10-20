# Skill invocation counts

`skill_invocation` analytics events report detected skill use, not successful task
completion. Each event has `skill_id`, `skill_name`, and `event_params` containing
`thread_id`, `turn_id`, and `invoke_type` (`explicit` or `implicit`).

An explicit injection and an implicit access can emit separate events for the same
skill in one turn. The analytics reducer suppresses repeated implicit resource
invocations by resource ID within a turn. Host invocations do not use this reducer
rule. Shell-based implicit detection has separate per-turn deduplication before
submission to analytics.

For example, these inputs use one thread and a distinct resource ID for each skill.

| Turn   | Skill ID | Invocation type | Analytics event emitted |
| ------ | -------- | --------------- | ----------------------- |
| turn-1 | doc-id   | explicit        | Yes                     |
| turn-1 | doc-id   | implicit        | Yes                     |
| turn-1 | doc-id   | implicit        | No                      |
| turn-1 | lint-id  | implicit        | Yes                     |
| turn-2 | doc-id   | implicit        | Yes                     |

The example produces four invocation events. A count of unique turns using each
skill counts distinct `(event_params.thread_id, event_params.turn_id, skill_id)`
tuples and excludes `invoke_type` from the key. The example has three such tuples,
two for `doc-id` and one for `lint-id`. A turn using two skills contributes to both
skills' counts, so summing per-skill counts does not give an overall unique-turn count.

The backend report exposes `invocation_counts`, but its aggregation implementation
is outside this repository. The distinct-tuple definition above is a separate
reporting unit.

[OTEL skill invocation logs](../otel/README.md#skill-invocation-logs) are a separate
stream without analytics `skill_id`. Skill-tool logs are emitted before analytics
reducer deduplication, so repeated reads can produce repeated logs.
