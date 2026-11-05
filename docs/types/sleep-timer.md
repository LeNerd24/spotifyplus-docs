---
sidebar_label: 'Sleep timer types'
---

# Sleep timer types

These types describe the rows returned by `SpotifyPlus.Platform.SleepTimer`.

```ts
type SleepTimerUnit = 'minutes' | 'hours';

interface SleepTimerEntry {
    id: string;
    kind: 'timer' | 'button' | 'end-of-content';
    minutes: number;
    title: string | null;
    scriptId: string | null;
}
```

`id` identifies a row for `remove()` and `move()`. `minutes` is the duration for timer rows; custom buttons and the end-of-content row use zero. `title` is supplied for custom buttons. `scriptId` identifies an extension-owned row and is `null` for defaults.

Default rows have IDs `default:5`, `default:10`, `default:15`, `default:30`, `default:45`, `default:60`, and `default:end-of-content`. Use `list()` to get the current order after extensions have customized it.
