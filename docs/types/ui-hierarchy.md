---
sidebar_label: 'UI hierarchy types'
---

# UI hierarchy types

`UI.captureHierarchy()` returns a snapshot of Spotify's UI. Use this to find resource IDs and understand the visible layout.

```ts
interface UIHierarchyNode {
    id: number;
    parent: number;
    window: number;
    kind: 'view' | 'compose-semantics' | 'compose-layout' | 'accessibility';
    className: string;
    path: string;
    boundsPx: [number, number, number, number];
    properties: Record<string, unknown>;
    children: number[];
}

interface UIHierarchySnapshot {
    schemaVersion: 1;
    activity: string;
    spotifyVersion: string;
    capturedAt: number;
    composeMode: 'layout' | 'unmerged-semantics';
    truncated: boolean;
    warnings: string[];
    nodes: UIHierarchyNode[];
}
```

`parent` and `children` refer to snapshot node IDs. `window` identifies the containing window. `properties` contains information available for that node; check the actual values rather than assuming every node has the same fields. `boundsPx` contains its bounds in pixels. Snapshots may be truncated or carry warnings when part of the hierarchy cannot be captured.
