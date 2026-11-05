---
sidebar_label: 'API result types'
---

# API result types

These types describe the results of search, library, playlist, queue, Connect, and user methods.

## SearchOptions

```ts
interface SearchOptions { 
  limit?: number; 
  locale?: string;
}
```

## SearchResponse

```ts
interface SearchResponse { 
  query: string; 
  items: SearchResult[];
}

interface SearchResult { 
  uri: string; 
  text: string;
}
```

## PageOptions

```ts
interface PageOptions { 
  offset?: number; 
  limit?: number;
}
```

## Page

```ts
interface Page<T> { 
  items: T[]; 
  offset: number; 
  limit: number; 
  total: number;
}
```

## LibraryItem

```ts
interface LibraryItem { 
  uri: string; 
  name: string; 
  imageUri: string; 
  pinned: boolean; 
}
```

## PlaylistPage

```ts
interface PlaylistItem { 
  uri: string; 
  rowId: string; 
  name: string; 
  addedAt: number; 
}

interface PlaylistPage extends Page<PlaylistItem> {
  uri: string; 
  name: string; 
  description: string; 
  ownedBySelf: boolean;
  imageUri?: string;
}
```

`rowId` identifies one occurrence of a track in a playlist. Use it for `removeTracks()` and `moveTracks()`.

## QueueSnapshot

```ts
interface QueueSnapshot {
  revision: string; 
  current: QueueEntry | null;
  next: QueueEntry[]; 
  previous: QueueEntry[];
}

interface QueueEntry { 
  uri: string; 
  uid: string; 
  metadata: Record<string, string>; 
}
```

`QueueSnapshot.next` contains manually queued entries. `UpNextSnapshot.next` contains upcoming context entries. Both share the current track and playback history. Index edits read the latest queue internally; you do not pass a snapshot. `UpNext.replace()` uses the revision explicitly.

## UpNextSnapshot

```ts
interface UpNextSnapshot extends QueueSnapshot {}
```

## ExpandedUpNextSnapshot

```ts
interface ExpandedUpNextSnapshot extends UpNextSnapshot {
    snapshotId: string;
    contextUri: string;
    requestedLimit: number;
    completeness: 'unknown' | 'limit-reached';
}
```

`snapshotId` is an opaque handle for an expanded replacement. It expires after 30 seconds or a successful replacement, and can be evicted when the host retains newer snapshots. `completeness` does not confirm that the whole context has been returned.

## ConnectDevice

```ts
interface ConnectDevice {
  id: string; 
  name: string; 
  type: string; 
  isActive: boolean;
  isLocal: boolean; 
  isDisabled: boolean; 
  supportsVolume: boolean;
  volume: number; // Can be a value between 0 and 65535
}
```

## SpotifyUser

```ts
interface SpotifyUser {
  username: string; 
  displayName: string; 
  uri: string;
  images: Array<{ url: string; width: number; height: number }>;
}
```
