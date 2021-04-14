---
layout: post
title: Using vertices and triangles in Unity
date: 2021-04-13 11:10:00 -07:00
categories: [ coding, writeup ]
---

As part of my experimentation with [GuaGAN](http://nvidia-research-mingyuliu.com/gaugan/) I thought I'd try to recreate something like [this](https://www.youtube.com/watch?v=y8kw8g1_JdY). My goal is to apply the AI filter to a generated world in a Unity environment. For this, I needed procedurally generated terrain. I experimented with using the `Terrain` component, but I found it didn't grant enough control. I decided on implementing a custom mesh which turned out to be more difficult than I imagined.

#### Part 1: Three vertices, one triangle

Like most game engines, Unity grants us access to the raw vertices and triangles passed to the CPU. These are arrays, one containing a series of points and the other containing instructions for how to connect them into triangles. For example, let's say we want to draw the triangle below:

![triangle](/assets/img/2021-04-13-custom-grids-unity/triangle.png)

This triangle consists (like most) of three vertices. For the sake of simplicity, I've only shown the X and Z coordinates. We will assume that they are all on the same Y plane. We can initialize the array of vertices in Unity like follows:

```c#
Vector3[] vertices = new Vector3[] {
    new Vector3(0, 0, 0),   // vert 0
    new Vector3(0, 0, 1),   // vert 1
    new Vector3(1, 0, 0)    // vert 2
};
```

Now the GPU has the positional data, but it doesn't know how to connect the dots. This is the purpose of the `triangles` array--which describes in which order the GPU should connect the dots.

```c#
int[] triangles = new int[] {
    0, 1, 2
};
```

Each integer in the triangles array is simply a reference to a vertice. A value of 0 would reference `vertices[0]`, a value of 1 would reference `vertices[1]` etc. To pass this data into a `Mesh` component in Unity, we can do the following:

```c#
public class TerrainGenerator : MonoBehaviour
{
    Vector3[] vertices;
    int[] triangles;
    Mesh mesh;

    void Start() {
        mesh = new Mesh();
        GetComponent<MeshFilter>().mesh = mesh;

        vertices = new Vector3[] {
            new Vector3(0, 0, 0),   // vert 0
            new Vector3(0, 0, 1),   // vert 1
            new Vector3(1, 0, 0)    // vert 2
        };

        triangles = new int[] {
            0, 1, 2
        };
    }

    void Update() {
        mesh.Clear();
        mesh.vertices = vertices;
        mesh.triangles = triangles;
        mesh.RecalculateNormals();
    }
}
```

As expected, this produces the triangle we initially sketched (despite lacking a texture). Since we added the mesh update code to the `Update()` function, changes we make to `vertices` and `triangles` will be visible in real-time. It's also worth noting that the `GameObject` this script is attached to requires an empty mesh filter and a mesh renderer.

![the first of many](/assets/img/2021-04-13-custom-grids-unity/first.png)

#### Part 2: Lots of vertices

Now we know how to render a triangle, we're going to need to create a grid of vertices like below.

![the vertices](/assets/img/2021-04-13-custom-grids-unity/grid1.png)

Each point is labeled with its index in the `vertices` array. To calculate the index from a given `X, Z` position, we can simply do `(Z * width) + X`, since for every increment on the Z-axis, we add `width` offset to the X position. This will also be important later on.

```c#
vertices = new Vector3[width * height];
for (int z = 0; z < height; z++) {
    for (int x = 0; x < width; x++) {
        // calculate vertices index using (Z * width) + X
        vertices[z * width + x] = new Vector3(x, 0, z);
    }
}
```

We can also enable "Gizmo" rendering in the scene viewer to give us the following view--proving our code is working correctly.

```c#
private void OnDrawGizmos() {
    if (vertices == null) return;

    for (int i = 0; i < vertices.Length; i++)
        Gizmos.DrawSphere(vertices[i], 0.1f);
}
```

![success](/assets/img/2021-04-13-custom-grids-unity/viewer1.png)

#### Part 3: Lots of triangles

Time to connect the dots. Initially, I thought this was going to be a huge pain but once I sketched it out it was simplified into basic math. The important thing to note here is backface culling. If a triangle is drawn in a clockwise direction, it will be rendered--if not, it will be culled. This means we have to be careful when ordering our vertices in the `triangles` array.

![determines culling](/assets/img/2021-04-13-custom-grids-unity/direction.png)

To fill in the triangles, I came up with the following algorithm. Assuming Q is the vertice index of the bottom left vertice in a square, we can label each corner of the square as follows.

![determines culling](/assets/img/2021-04-13-custom-grids-unity/vertices.png)

Which translates into:

```c#
triangles[0] = Q;
triangles[1] = Q + width;
triangles[2] = Q + 1;

triangles[3] = Q + 1;
triangles[4] = Q + width;
triangles[5] = Q + width + 1;
```

When we add a for-loop and iterate over `triangles` to fill out the grid--success!

```c#
triangles = new int[(width - 1) * (height - 1) * 6];
int i = 0;
for (int z = 0; z < height - 1; z++) {
    for (int x = 0; x < width - 1; x++) {
        // calculate vertices index using (Z * width) + X
        int quad = x + width * z;

        // triangle 1
        triangles[i + 0] = quad;
        triangles[i + 1] = quad + width;
        triangles[i + 2] = quad + 1;

        // triangle 2
        triangles[i + 3] = quad + 1;
        triangles[i + 4] = quad + width;
        triangles[i + 5] = quad + width + 1;

        // increment by 6 for every 2 triangles drawn
        i += 6;
    }
}
```

![shaded result](/assets/img/2021-04-13-custom-grids-unity/viewer3.png)

![wireframe close up](/assets/img/2021-04-13-custom-grids-unity/viewer4.png)

#### Part 4: Perlin noise

Perlin noise is a simple noise function that can generate natural(ish) looking heightmaps. In reality, you would want to layer multiple octaves of perlin noise at different frequencies to create better-looking terrain but for this example, we'll just use a single layer.

![perlin noise](/assets/img/2021-04-13-custom-grids-unity/perlin.png)

If we revisit our vertice generation, we'll remember that the Y value was always zero. Instead, let's replace it with `CalculateHeight(x, z)` which can return a heightmap value generated using perlin noise.

```c#
// calculate vertices index using (Z * width) + X
vertices[z * width + x] = new Vector3(x, CalculateHeight(x, z), z);
```

Let's implement the function, and assign a value of 20 to `scale` and `depth`:

```c#
float CalculateHeight(int x, int z) {
    return Mathf.PerlinNoise(
        (float) x / scale,
        (float) z / scale
    ) * depth;
}
```

![wavy](/assets/img/2021-04-13-custom-grids-unity/wavy.png)

Success, 3D "Terrain"! This needs some work--but for now, it's a good example of creating custom meshes in Unity. The full code can be found [here](https://gist.github.com/wg4568/484dce199d38819f1b27fb42ee8f04b7).
