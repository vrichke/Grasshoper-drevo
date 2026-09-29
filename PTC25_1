"""
GhPython — Random Low-Poly Tree DELA!!!!!!!!!!!!!

Inputs:
    Seed     : int
    trunk_h  : float
    trunk_w  : float
    levels   : int
    canopy_r : float
    segs     : int
    lean     : float
    jitter   : float

Outputs:
    Tree  : Mesh
    Debug : list[str]
"""

import random, math
from Rhino.Geometry import Point3d, Mesh


def _d(v, d):
    return d if v is None else v

# --- del kode ki že avtomatsko zgenerira drevo --------------
Seed     = _d(Seed, 123)
trunk_h  = _d(trunk_h, 6.0)
trunk_w  = _d(trunk_w, 0.6)
levels   = int(_d(levels, 3))
canopy_r = _d(canopy_r, 2.0)
segs     = max(3, int(_d(segs, 8)))
lean     = _d(lean, 0.4)    # 0 = straight up, 0.6 = noticeably tilted
jitter   = _d(jitter, 0.25) # 0 = perfect polygon, 0.4 = jaggier "low poly"

random.seed(Seed)

# --- obroč ---------------------------------------------------
def ring_points(n, radius, z, jitter_amount):
    pts = []
    for i in range(n):
        t = 2.0 * math.pi * i / n
        # small random radial jitter -> "low-poly roughness"
        r = radius * (1.0 + (random.random() * 2 - 1) * jitter_amount)
        x = r * math.cos(t)
        y = r * math.sin(t)
        pts.append(Point3d(x, y, z))
    return pts

def add_vertices(mesh, pts):
    idx = []
    for p in pts:
        idx.append(mesh.Vertices.Add(p))
    return idx

def cap_from_center(mesh, center_idx, ring_indices, flip=False):
    n = len(ring_indices)
    for i in range(n):
        a = ring_indices[i]
        b = ring_indices[(i + 1) % n]
        if flip:
            mesh.Faces.AddFace(center_idx, a, b)
        else:
            mesh.Faces.AddFace(center_idx, b, a)

# --- DEBLO -------------------------------------------
def prism_trunk(n, width, height):
    m = Mesh()
    rad = width * 0.5

    bot_pts = ring_points(n, rad, 0.0, 0.0)
    top_pts = [Point3d(p.X, p.Y, p.Z + height) for p in bot_pts]

    ibot = add_vertices(m, bot_pts)
    itop = add_vertices(m, top_pts)

    cbot = m.Vertices.Add(Point3d(0, 0, 0))
    ctop = m.Vertices.Add(Point3d(0, 0, height))

    # sides (quads)
    for i in range(n):
        a = ibot[i]
        b = ibot[(i + 1) % n]
        d = itop[i]
        c = itop[(i + 1) % n]
        m.Faces.AddFace(a, b, c, d)

    # caps (triangles)
    cap_from_center(m, cbot, ibot, flip=True)
    cap_from_center(m, ctop, itop, flip=False)
    return m

# --- klele mamo krošnje piramidalne oblike ---------------
def pyramid_canopy_level(n, base_radius, base_z, height, lean_amount, jitter_amount):
    m = Mesh()
    base = ring_points(n, base_radius, base_z, jitter_amount)

    # random nagib
    ang = random.random() * 2.0 * math.pi
    lx = math.cos(ang) * lean_amount * height
    ly = math.sin(ang) * lean_amount * height
    apex = Point3d(lx, ly, base_z + height)

    ibase = add_vertices(m, base)
    iapex = m.Vertices.Add(apex)
    cbase = m.Vertices.Add(Point3d(0, 0, base_z))

    # sides (triangles)
    for i in range(n):
        a = ibase[i]
        b = ibase[(i + 1) % n]
        m.Faces.AddFace(iapex, b, a)

    # bottom cap (optional but keeps mesh closed)
    cap_from_center(m, cbase, ibase, flip=False)
    return m

# --- NARED DREVO --------------------------------------------------------
trunk = prism_trunk(segs, trunk_w, trunk_h)

tree = Mesh()
tree.Append(trunk)

z = trunk_h
scale = 1.0
levels = max(1, levels)
for i in range(levels):
    # da se malo randomizira
    r = canopy_r * scale * (0.9 + random.random() * 0.2)
    h = r * (0.8 + random.random() * 0.6)
    level_mesh = pyramid_canopy_level(segs, r, z, h, lean, jitter)
    tree.Append(level_mesh)

    # da se krošnje prekrivajo
    z += h * 0.5
    scale *= 0.8

tree.Normals.ComputeNormals()
tree.Compact()

Tree = tree
Debug = ["verts: %d  faces: %d" % (tree.Vertices.Count, tree.Faces.Count)]
