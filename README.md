# KD Tree

`com.unigame.kdtree` is a Unity package for spatial queries over a `float3` point cloud. It provides a k-d tree implementation optimized for nearest-neighbor, radius, and AABB interval searches.

## Features

- build a `KDTree` from an array or list of points;
- rebuild the tree after point positions change;
- supported query types:
  - nearest point via `ClosestPoint()`;
  - `k` nearest points via `KNearest()`;
  - all points inside a radius via `Radius()`;
  - all points inside an axis-aligned box via `Interval()`;
- reusable `KDQuery` object with internal buffers to reduce allocations;
- dynamic point array growth via `SetCount()`;
- debug visualization of visited nodes via `DrawLastQuery()`.

## Requirements

- Unity 2020.2+
- `Unity.Mathematics`

## Package structure

- `Datastructures/KDTree` — tree, nodes, bounds, and query implementations;
- `Datastructures/Heap` — internal heap data structures used by queries;
- `Datastructures/Tests` — usage examples, test scenes, and benchmark components.

Primary namespace:

```csharp
using UnioGame.KDTree;
using Unity.Mathematics;
```

## Installation

### Option 1. Git URL

Add the package to `Packages/manifest.json`:

```json
{
  "dependencies": {
    "com.unigame.kdtree": "https://github.com/UnioGame/kd.tree.git"
  }
}
```

### Option 2. Local package

Copy the `kd.tree` folder into `Packages`, or reference it as a local package through the Unity Package Manager.

## Quick start

### Build a tree

```csharp
using System.Collections.Generic;
using UnioGame.KDTree;
using Unity.Mathematics;

public class KDTreeSample
{
    private KDTree _tree;
    private KDQuery _query;
    private List<int> _indices = new List<int>();
    private List<float> _distances = new List<float>();

    public void Initialize(float3[] points)
    {
        _tree = new KDTree(points, maxPointsPerLeafNode: 32);
        _query = new KDQuery();
    }
}
```

### Rebuild after moving points

If you modify coordinates in `tree.points`, rebuild the tree:

```csharp
for (int i = 0; i < _tree.Count; i++)
{
    _tree.points[i] += new float3(0f, 0.01f, 0f);
}

_tree.Rebuild();
```

## API

### `KDTree`

#### Constructors

```csharp
var tree = new KDTree();
var tree = new KDTree(maxPointsPerLeafNode: 32);
var tree = new KDTree(points, maxPointsPerLeafNode: 32);
```

#### Core methods

- `Build(float3[] newPoints, int maxPointsPerLeafNode = -1)`
- `Build(float3[] newPoints, int pointCount, int maxPointsPerLeafNode = -1)`
- `Build(List<float3> newPoints, int maxPointsPerLeafNode = -1)`
- `Rebuild(int maxPointsPerLeafNode = -1)`
- `SetCount(int newSize)`

#### Useful fields

- `points` — internal point array;
- `permutation` — internal index array reordered during tree construction;
- `Count` — number of active points;
- `RootNode` — root tree node;
- `maxPointsPerLeafNode` — leaf size threshold.

### `KDQuery`

`KDQuery` is designed to be created once and reused. Internally it keeps pooled buffers and heap structures that grow when needed and are reused on later queries.

> A single `KDQuery` instance should only be used from one thread. For multithreaded usage, create one query object per thread.

#### Nearest point

```csharp
_indices.Clear();
_distances.Clear();

_query.ClosestPoint(_tree, new float3(0f, 0f, 0f), _indices, _distances);

int closestIndex = _indices[0];
float squaredDistance = _distances[0];
```

#### `k` nearest points

```csharp
_indices.Clear();
_distances.Clear();

_query.KNearest(_tree, new float3(0f, 0f, 0f), 8, _indices, _distances);
```

#### Radius search

```csharp
_indices.Clear();

_query.Radius(_tree, new float3(0f, 0f, 0f), 5f, _indices);
```

#### AABB interval search

```csharp
_indices.Clear();

float3 min = new float3(-1f, -1f, -1f);
float3 max = new float3( 1f,  1f,  1f);

_query.Interval(_tree, min, max, _indices);
```

## Best practices

### 1. Reuse `KDQuery`

Do not create a new `KDQuery` for every query. Its internal buffers are meant to be reused.

### 2. Clear result lists yourself

Query methods append to the provided lists and do not clear them automatically. Call `Clear()` before each query.

### 3. Call `Rebuild()` after modifying points

If point positions change, the tree must be rebuilt before the next query.

### 4. Do not rely on result order

For `Radius()` and `Interval()`, result order depends on tree traversal. For `KNearest()`, if you need a strictly sorted result by distance, sort it explicitly after the query.

### 5. Grow dynamic clouds with `SetCount()`

If your point count grows over time, increase capacity and fill `tree.points` manually:

```csharp
int oldCount = _tree.Count;
_tree.SetCount(oldCount + 100);

for (int i = oldCount; i < _tree.Count; i++)
{
    _tree.points[i] = new float3(i, 0f, 0f);
}

_tree.Rebuild();
```

## Debugging

After a query, you can visualize the visited nodes:

```csharp
private void OnDrawGizmos()
{
    if (_query == null) return;

    _query.DrawLastQuery();
}
```

This method uses `Gizmos` and is intended for editor/debug scenes.

## Included examples

The package already contains example scripts and scenes:

- `Datastructures/Tests/KDTreeQueryTests.cs` — demonstrates all query modes;
- `Datastructures/Tests/KDTreeQueryResizeTests.cs` — shows a growing point cloud scenario;
- `Datastructures/Tests/KDTreeBenchmark.cs` — basic construction and query benchmarks;
- `Datastructures/Tests/testScene.unity` and `Datastructures/Heap/heapTestScene.unity` — test scenes.

## Notes and limitations

- the tree operates on `float3` coordinates;
- the package indexes points, but does not store custom payload objects;
- if you need to associate points with gameplay data, keep a parallel array or list using the same indices;
- duplicate-coordinate support is enabled in source via the `KDTREE_DUPLICATES` define;
- `ClosestPoint()` returns a squared distance when `resultDistances` is provided.

## License

The package is distributed under the MIT license. Original license and copyright notices from the base implementation are preserved in the source files.
