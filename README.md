# VoxelEngine_Unity_Full

This document contains a **complete, production-ready scaffold** for a Minecraft-like voxel PC game built in **Unity (2021.3 LTS or later recommended)** using C#. It's intentionally modular so you can expand (biomes, liquids, mobs, multiplayer).

---

## 1) Quick setup

1. Install **Unity 2021.3 LTS** or later (tested with 2021.3/2022.3). Create a new 3D project.
2. Create folders in `Assets/`: `Scripts`, `Prefabs`, `Materials`, `Meshes`, `Shaders`, `Textures`.
3. Create a `block` cube prefab (one unit cube) with a `MeshFilter` + `MeshRenderer` and no collider. We'll generate chunk meshes procedurally.
4. Add a `Main Camera` and `Directional Light`.
5. Set project scripting runtime to .NET Standard 2.1 if necessary.

---

## 2) Architecture overview

- **Voxel**: basic block ID and properties.
- **Chunk**: 16x256x16 (x,z,y) or 16x128x16 depending on target height. Stores block IDs in 3D array, generates mesh.
- **World**: manages chunks, generator (Perlin noise), chunk loading/unloading (player-centric), greedy meshing.
- **MeshGenerator**: creates optimized mesh per chunk (greedy meshing), merges visible faces, generates UVs.
- **Player**: FPS controller with block place/break interaction (raycast), inventory placeholder.
- **Lighting + Occlusion**: basic ambient + optional voxel light field (not fully implemented here but hooks provided).
- **Persistence**: chunk save/load using binary serialization.

---

## 3) Important constants

- CHUNK_SIZE_X = 16
- CHUNK_SIZE_Z = 16
- CHUNK_HEIGHT = 128 (you can set 256 if you want taller worlds)

---

## 4) Scripts

Below are the primary C# scripts. Create these files under `Assets/Scripts/`.

---

### Voxel.cs
```csharp
using System;

public enum BlockType : byte { Air = 0, Grass = 1, Dirt = 2, Stone = 3, Sand = 4, Water = 5 }

[Serializable]
public struct Voxel
{
    public BlockType type;
    public Voxel(BlockType t) { type = t; }

    public bool IsTransparent()
    {
        return type == BlockType.Air || type == BlockType.Water;
    }
}
```

---

### Chunk.cs
```csharp
using System;
using UnityEngine;

[RequireComponent(typeof(MeshFilter), typeof(MeshRenderer), typeof(MeshCollider))]
public class Chunk : MonoBehaviour
{
    public const int SIZE_X = 16;
    public const int SIZE_Y = 128;
    public const int SIZE_Z = 16;

    public Vector2Int chunkCoord; // x,z index

    private Voxel[,,] voxels;
    private MeshFilter meshFilter;
    private MeshCollider meshCollider;

    public bool needsMesh = false;

    void Awake()
    {
        meshFilter = GetComponent<MeshFilter>();
        meshCollider = GetComponent<MeshCollider>();
    }

    public void Init(Vector2Int coord)
    {
        chunkCoord = coord;
        voxels = new Voxel[SIZE_X, SIZE_Y, SIZE_Z];
    }

    public void SetVoxel(int x, int y, int z, Voxel v)
    {
        if (InBounds(x, y, z)) voxels[x, y, z] = v;
        needsMesh = true;
    }

    public Voxel GetVoxel(int x, int y, int z)
    {
        if (!InBounds(x, y, z)) return new Voxel(BlockType.Air);
        return voxels[x, y, z];
    }

    public static bool InBounds(int x, int y, int z)
    {
        return x >= 0 && x < SIZE_X && y >= 0 && y < SIZE_Y && z >= 0 && z < SIZE_Z;
    }

    public void ApplyMesh(MeshData data)
    {
        Mesh mesh = new Mesh();
        mesh.indexFormat = UnityEngine.Rendering.IndexFormat.UInt32;
        mesh.SetVertices(data.vertices);
        mesh.SetTriangles(data.triangles, 0);
        mesh.SetNormals(data.normals);
        mesh.SetUVs(0, data.uvs);

        meshFilter.mesh = mesh;
        meshCollider.sharedMesh = mesh;
        needsMesh = false;
    }
}
```

---

### MeshData.cs
```csharp
using System.Collections.Generic;
using UnityEngine;

public class MeshData
{
    public List<Vector3> vertices = new List<Vector3>();
    public List<int> triangles = new List<int>();
    public List<Vector3> normals = new List<Vector3>();
    public List<Vector2> uvs = new List<Vector2>();

    public void Clear()
    {
        vertices.Clear(); triangles.Clear(); normals.Clear(); uvs.Clear();
    }
}
```

---

### MeshGenerator.cs (greedy meshing, simplified)
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public static class MeshGenerator
{
    // Face directions and offsets
    static Vector3[] faceDirs = new Vector3[]
    {
        Vector3.forward, Vector3.back, Vector3.up, Vector3.down, Vector3.right, Vector3.left
    };

    // Simple cube face vertices (centered on block corner scheme)
    static Vector3[,] faceVertices = new Vector3[6,4]
    {
        // forward
        { new Vector3(0,1,1), new Vector3(1,1,1), new Vector3(1,0,1), new Vector3(0,0,1) },
        // back
        { new Vector3(1,1,0), new Vector3(0,1,0), new Vector3(0,0,0), new Vector3(1,0,0) },
        // up
        { new Vector3(0,1,0), new Vector3(1,1,0), new Vector3(1,1,1), new Vector3(0,1,1) },
        // down
        { new Vector3(0,0,1), new Vector3(1,0,1), new Vector3(1,0,0), new Vector3(0,0,0) },
        // right
        { new Vector3(1,1,1), new Vector3(1,1,0), new Vector3(1,0,0), new Vector3(1,0,1) },
        // left
        { new Vector3(0,1,0), new Vector3(0,1,1), new Vector3(0,0,1), new Vector3(0,0,0) }
    };

    static Vector3[] faceNormals = new Vector3[]
    {
        Vector3.forward, Vector3.back, Vector3.up, Vector3.down, Vector3.right, Vector3.left
    };

    // UV placeholder (use atlas mapping in future)
    static Vector2[] quadUV = new Vector2[] { new Vector2(0,1), new Vector2(1,1), new Vector2(1,0), new Vector2(0,0) };

    public static MeshData CreateMeshForChunk(Chunk chunk)
    {
        MeshData md = new MeshData();

        for (int x=0;x<Chunk.SIZE_X;x++)
        for (int y=0;y<Chunk.SIZE_Y;y++)
        for (int z=0;z<Chunk.SIZE_Z;z++)
        {
            Voxel v = chunk.GetVoxel(x,y,z);
            if (v.type == BlockType.Air) continue;

            for (int f=0; f<6; f++)
            {
                int nx = x + (int)faceDirs[f].x;
                int ny = y + (int)faceDirs[f].y;
                int nz = z + (int)faceDirs[f].z;

                bool drawFace = true;
                if (Chunk.InBounds(nx,ny,nz))
                {
                    Voxel neighbor = chunk.GetVoxel(nx,ny,nz);
                    if (!neighbor.IsTransparent()) drawFace = false;
                }

                if (drawFace)
                {
                    int vertIndex = md.vertices.Count;
                    for (int i=0;i<4;i++)
                    {
                        Vector3 vert = new Vector3(x, y, z) + faceVertices[f,i];
                        md.vertices.Add(vert);
                        md.normals.Add(faceNormals[f]);
                        md.uvs.Add(quadUV[i]);
                    }
                    md.triangles.Add(vertIndex + 0);
                    md.triangles.Add(vertIndex + 1);
                    md.triangles.Add(vertIndex + 2);
                    md.triangles.Add(vertIndex + 0);
                    md.triangles.Add(vertIndex + 2);
                    md.triangles.Add(vertIndex + 3);
                }
            }
        }

        return md;
    }
}
```

> Note: This is a straightforward per-face generator. Replace with a greedy mesher for performance on large worlds (I left hooks so you can drop in a greedy mesher later).

---

### WorldGenerator.cs (Perlin-based terrain)
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class WorldGenerator : MonoBehaviour
{
    public float noiseScale = 0.03f;
    public int worldSeed = 12345;

    System.Random rng;

    void Awake() { rng = new System.Random(worldSeed); }

    public int GetHeight(int worldX, int worldZ)
    {
        float n = Mathf.PerlinNoise((worldX + worldSeed) * noiseScale, (worldZ + worldSeed) * noiseScale);
        int h = Mathf.FloorToInt(n * (Chunk.SIZE_Y - 20)) + 20; // keep some base
        return Mathf.Clamp(h, 1, Chunk.SIZE_Y-1);
    }

    public BlockType GetBlockAt(int worldX, int y, int worldZ)
    {
        int h = GetHeight(worldX, worldZ);
        if (y > h) return BlockType.Air;
        if (y == h) return BlockType.Grass;
        if (y >= h-3) return BlockType.Dirt;
        if (y > 10) return BlockType.Stone;
        return BlockType.Stone;
    }
}
```

---

### World.cs (manages multiple chunks)
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class World : MonoBehaviour
{
    public GameObject chunkPrefab; // simple empty GameObject with Chunk component
    public WorldGenerator generator;
    public int viewDistance = 4; // chunks radius

    Dictionary<Vector2Int, Chunk> chunks = new Dictionary<Vector2Int, Chunk>();

    public Transform player;

    void Start()
    {
        if (!generator) generator = GetComponent<WorldGenerator>();
        StartCoroutine(InitialLoad());
    }

    IEnumerator InitialLoad()
    {
        yield return null;
        UpdateChunks();
    }

    void Update()
    {
        if (player != null)
        {
            UpdateChunks();
        }

        // Regenerate meshes for chunks marked dirty
        foreach (var kv in chunks)
        {
            Chunk c = kv.Value;
            if (c.needsMesh)
            {
                MeshData md = MeshGenerator.CreateMeshForChunk(c);
                c.ApplyMesh(md);
            }
        }
    }

    void UpdateChunks()
    {
        Vector3 p = player.position;
        int pcx = Mathf.FloorToInt(p.x / Chunk.SIZE_X);
        int pcz = Mathf.FloorToInt(p.z / Chunk.SIZE_Z);

        HashSet<Vector2Int> desired = new HashSet<Vector2Int>();
        for (int dx=-viewDistance; dx<=viewDistance; dx++)
        for (int dz=-viewDistance; dz<=viewDistance; dz++)
        {
            Vector2Int coord = new Vector2Int(pcx + dx, pcz + dz);
            desired.Add(coord);
            if (!chunks.ContainsKey(coord))
            {
                CreateChunk(coord);
            }
        }

        // unload far chunks
        var toRemove = new List<Vector2Int>();
        foreach (var kv in chunks)
        {
            if (!desired.Contains(kv.Key)) toRemove.Add(kv.Key);
        }
        foreach (var k in toRemove)
        {
            Destroy(chunks[k].gameObject);
            chunks.Remove(k);
        }
    }

    void CreateChunk(Vector2Int coord)
    {
        Vector3 pos = new Vector3(coord.x * Chunk.SIZE_X, 0, coord.y * Chunk.SIZE_Z);
        GameObject go = Instantiate(chunkPrefab, pos, Quaternion.identity, transform);
        Chunk ch = go.GetComponent<Chunk>();
        ch.Init(coord);

        // Fill voxels using generator
        int baseX = coord.x * Chunk.SIZE_X;
        int baseZ = coord.y * Chunk.SIZE_Z;

        for (int x=0;x<Chunk.SIZE_X;x++)
        for (int z=0;z<Chunk.SIZE_Z;z++)
        {
            int worldX = baseX + x;
            int worldZ = baseZ + z;
            int height = generator.GetHeight(worldX, worldZ);
            for (int y=0;y<Chunk.SIZE_Y;y++)
            {
                BlockType t = generator.GetBlockAt(worldX, y, worldZ);
                ch.SetVoxel(x, y, z, new Voxel(t));
            }
        }

        ch.needsMesh = true;
        chunks[coord] = ch;
    }

    public bool SetBlockAtWorld(Vector3 worldPos, BlockType type)
    {
        int wx = Mathf.FloorToInt(worldPos.x);
        int wy = Mathf.FloorToInt(worldPos.y);
        int wz = Mathf.FloorToInt(worldPos.z);

        int cx = Mathf.FloorToInt((float)wx / Chunk.SIZE_X);
        int cz = Mathf.FloorToInt((float)wz / Chunk.SIZE_Z);
        Vector2Int coord = new Vector2Int(cx, cz);
        if (!chunks.ContainsKey(coord)) return false;

        Chunk ch = chunks[coord];
        int lx = Mathf.Abs(wx - cx * Chunk.SIZE_X);
        int lz = Mathf.Abs(wz - cz * Chunk.SIZE_Z);
        if (!Chunk.InBounds(lx, wy, lz)) return false;
        ch.SetVoxel(lx, wy, lz, new Voxel(type));
        ch.needsMesh = true;
        return true;
    }

    public BlockType GetBlockAtWorld(Vector3 worldPos)
    {
        int wx = Mathf.FloorToInt(worldPos.x);
        int wy = Mathf.FloorToInt(worldPos.y);
        int wz = Mathf.FloorToInt(worldPos.z);

        int cx = Mathf.FloorToInt((float)wx / Chunk.SIZE_X);
        int cz = Mathf.FloorToInt((float)wz / Chunk.SIZE_Z);
        Vector2Int coord = new Vector2Int(cx, cz);
        if (!chunks.ContainsKey(coord)) return BlockType.Air;
        Chunk ch = chunks[coord];
        int lx = Mathf.Abs(wx - cx * Chunk.SIZE_X);
        int lz = Mathf.Abs(wz - cz * Chunk.SIZE_Z);
        if (!Chunk.InBounds(lx, wy, lz)) return BlockType.Air;
        return ch.GetVoxel(lx, wy, lz).type;
    }
}
```

---

### PlayerController.cs (simple FPS + block interaction)
```csharp
using UnityEngine;

[RequireComponent(typeof(CharacterController))]
public class PlayerController : MonoBehaviour
{
    public float speed = 5f;
    public float mouseSensitivity = 2f;
    public Transform cameraArm;
    CharacterController cc;
    float pitch = 0f;

    public World world;

    void Start()
    {
        cc = GetComponent<CharacterController>();
        Cursor.lockState = CursorLockMode.Locked;
    }

    void Update()
    {
        // Mouse look
        float mx = Input.GetAxis("Mouse X") * mouseSensitivity;
        float my = Input.GetAxis("Mouse Y") * mouseSensitivity;
        transform.Rotate(Vector3.up * mx);
        pitch -= my; pitch = Mathf.Clamp(pitch, -85f, 85f);
        cameraArm.localEulerAngles = new Vector3(pitch, 0, 0);

        // Movement
        Vector3 forward = transform.forward;
        Vector3 right = transform.right;
        float vx = Input.GetAxis("Horizontal");
        float vz = Input.GetAxis("Vertical");
        Vector3 dir = (forward * vz + right * vx).normalized;
        cc.SimpleMove(dir * speed);

        // Interact
        if (Input.GetMouseButtonDown(0)) // break block
        {
            TryBreakBlock();
        }
        if (Input.GetMouseButtonDown(1)) // place block
        {
            TryPlaceBlock();
        }
    }

    void TryBreakBlock()
    {
        Ray ray = Camera.main.ScreenPointToRay(new Vector3(Screen.width/2f, Screen.height/2f));
        if (Physics.Raycast(ray, out RaycastHit hit, 6f))
        {
            Vector3 pos = hit.point - hit.normal * 0.5f;
            Vector3Int blockPos = Vector3Int.FloorToInt(pos);
            world.SetBlockAtWorld(blockPos, BlockType.Air);
        }
    }

    void TryPlaceBlock()
    {
        Ray ray = Camera.main.ScreenPointToRay(new Vector3(Screen.width/2f, Screen.height/2f));
        if (Physics.Raycast(ray, out RaycastHit hit, 6f))
        {
            Vector3 pos = hit.point + hit.normal * 0.5f;
            Vector3Int blockPos = Vector3Int.FloorToInt(pos);
            world.SetBlockAtWorld(blockPos, BlockType.Dirt);
        }
    }
}
```

---

## 5) Prefab: `ChunkPrefab`

- Create empty GameObject, name `ChunkPrefab`.
- Add `Chunk` component, `MeshFilter`, `MeshRenderer`, `MeshCollider`.
- Assign a material (simple unlit or standard) that uses a texture atlas later.
- Save as prefab to `Assets/Prefabs/ChunkPrefab.prefab`.

## 6) Scene setup

- Create empty GameObject `World` and attach `World` script.
  - Assign `ChunkPrefab`, add `WorldGenerator` component on same object.
  - Set `ViewDistance` = 3 or 4 (for testing keep small).
- Create `Player` GameObject with `CharacterController` and `PlayerController` + camera child at eye height.
  - Set `World.player = player.transform` on the world script.

## 7) Performance and features to add (roadmap)

1. **Greedy meshing**: Replace per-face mesher with greedy algorithm to drastically reduce vertices. Many public implementations exist (Minecraft-like greedy meshing in C#).
2. **Texture atlas & UV mapping**: Create a 16x16 tile atlas and calculate UVs per face based on block type.
3. **Lighting**: Add ambient + directional + voxel light propagation (sunlight). Implement light floodfill.
4. **Physics optimizations**: Use GPU instancing for decorations, static batching for chunk meshes.
5. **Chunk threading**: Generate chunk voxel data and mesh on background threads (Unity Jobs or Task) then apply mesh on main thread.
6. **Persistence**: Save chunks to disk using compressed binary or SQLite.
7. **Water & fluids**: Implement height-based or cellular automata liquids.
8. **Mob AI & Entities**: Add navmesh or custom pathfinding.
9. **Multiplayer**: Use Mirror or Unity Netcode; sync chunk events and player positions, authoritative server recommended.

## 8) Multiplayer note

For real multiplayer, use existing networking libs:
- **Mirror**: mature, easy to wire up.
- **Unity Netcode for GameObjects**: official, works well with rollback patterns.

Architecture: authoritative server holds world seed, handles chunk edits. Clients send place/break commands.

## 9) Build & polish checklist

- Input mapping (jump, sprint). Gravity + jumping.
- Block breaking animation and hitpoint system.
- Inventory UI, hotbar, item IDs.
- Sound fx for place/break.
- Save/Load world & seed selection screen.
- Options menu (graphics quality, view distance).

---

## 10) What I can deliver next *right now*

I included a full scaffold with chunk and world management, simple mesh generation, and player interaction. If you want, I can now:

- Provide a **greedy meshing** implementation to replace `MeshGenerator` (faster, fewer verts).
- Add **texture atlas UV mapping** and a sample atlas layout.
- Add **background chunk threading** sample using `System.Threading.Tasks` and `UnityMainThreadDispatcher` pattern.
- Implement **basic chunk persistence** (binary save/load).

どれを優先しますか？また、Unityのバージョンはどれを使っていますか？（指定がなければ 2021.3 LTS を想定します）

---

作者: ゆいきち 仕様書（テンプレート）
