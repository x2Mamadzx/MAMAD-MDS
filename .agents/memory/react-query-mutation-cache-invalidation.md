---
name: React Query mutation cache invalidation for list-derived UI
description: A mutation that changes a field shown via a list query must invalidate that list's query key on success
---

When a component renders a controlled input (e.g. a `<select>`) whose value
comes from an item inside a list fetched via a React Query `useQuery` hook,
and a sibling mutation (e.g. `useUpdateX`) changes that same field, the
mutation must invalidate (or optimistically update) the list query's cache on
success.

**Why:** Without invalidation, the mutation succeeds server-side but the
list query cache still holds the pre-mutation value, so the controlled input
appears to "revert" until an unrelated refetch happens. This was caught by
code review on a Kanban-style admin dashboard where changing a status via a
`<select>` looked like it silently failed.

**How to apply:** Pass `mutation: { onSuccess: () => queryClient.invalidateQueries({ queryKey: getXQueryKey() }) }`
(orval-generated hooks accept this via their `options.mutation`) whenever a
mutation affects data also shown through a separate list/query hook.
