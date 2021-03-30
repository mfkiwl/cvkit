
USING CVKIT
===========

The package contains the tools:

* `sv`:     For viewing and analyzing 2D images.
* `plyv`:   For viewing and analyzing 3D models.
* `imgcmd`: For scaling, cropping and converting images.
* `plycmd`: For converting depth images into PLY format.

All tools print a help text if they are called without any parameters. `imgcmd`
and `plycmd` are pure shell programs. `sv` and `plyv` use a simple window for
visualization. All functions are available through mouse clicks, moves or keys.
Images and 3D models must be provided as arguments when starting the tools.
Press 'h' for getting help.

Windows users should open images and PLY files through the Windows Explorer.
The installer will add file associations automatically.

See example directory.

TILED IMAGES
------------

All tools can read a special tiled image format. All tiles must be of the
same size. The file name determines the position of the tile in the image.
The format is `<prefix>_<row number>_<column number>_<suffix>`. Tiles can be
missing. The filename for loading a tiled image is `<prefix>:<suffix>`. For
example, lets assume 3 files in directory image_dir:

    > ls image_dir

    scene1_00_00_rgb.ppm
    scene1_00_01_rgb.ppm
    scene1_01_01_rgb.ppm

The following command would load the tiled image. The pixels of the missing
tile are set to 0 (i.e. black):

    sv image_dir/scene1:rgb.ppm

If a file `<prefix>.hdr` or `<prefix>_param.txt` exists and if it contains the
line `border=<n>`, the specified number is interpreted as border size. The
actual tile width and height is 2*n smaller and the border area is blended
while loading a tiled image.

LOADING DISPARITY IMAGES AS 3D MODEL
------------------------------------

A disparity or height image can be loaded as 3D model. plyv converts the
images on the fly to a 3D colored or shaded mesh. This requires additional
information, which is appended in a comma separated list to the name of the
image. The following properties can be given in a `<key>=<value>` format:

* `p=<file>`:
  Specifies a parameter file. The contents of the file is described
  below. This property can be given multiple times. All parameter
  files are merged. If no parameter files are specified, the tools
  will try to find them automatically (see below).

* `i=<file>`:
  Image that is used as texture. It must be of the same size as the
  disparity or height image. If no image is specified, the tools
  will try to find it automatically (see below).

* `ds=<f>`:
  Integer factor for down sampling of the disparity or height image.
  Default is 1.

* `x=<x>`
  `y=<y>`
  `w=<w>`
  `h=<h>`:
  Loads only the specified part (given in pixel after down sampling)
  of the image. Default is to load the whole image.

* `s=<step>`:
  Maximum step size between neighboring pixels for connecting them
  with a triangle. Default is 1.

--- Parameter files ---

The parameter files contain information about the geometric properties of
the camera for reconstruction from disparity or height images. The files are
given in ASCII in a `<key>=<value>` format.

For perspective disparity images, the camera matrix as well as the factor rho
must be given by:

    camera.A=[<fx> <s> <u>; 0 <fy> <v>; 0 0 1]
    rho=<value>

fx and fy are the focal length in x and y direction in pixel. s is the skew
factor and u and v are the position of the principle point in pixel. For
rectified images with a principle point in the middle of the image, fx=fy=f,
s=0, u=width/2 and v=height/2. rho is the factor for computing the distance
from the disparity. For rectified images rho=f*t with t as length of the
baseline.

Optionally, the pose of the camera may be given. This is useful for loading
several models. The transformation from a camera point Pc to a world point
Pw is defined by Pw = R*Pc + T:

    camera.R=[1 0 0; 0 1 0; 0 0 1]
    camera.T=[0 0 0]

Disparity values are normally expected as absolute values that are stored in a
floating point format (e.g. PFM or TIFF). Infinity is used to mark invalid
values. For also supporting integer formats, the following optional values can
be used:

    disp.scale=1
    disp.offset=0
    disp.inv=inf

For orthogonal height images, the following parameters must be given instead
camera.A and rho:

    resolution=<r>
    depth.resolution=<dr>

The conversion between image coordinates (i.e. column i and row k) and
img(i, k) to local coordinates x, y, z is defined by `x=<r>*i`, `y=-<r>*k`,
`z=<dr>*img(i, k)`.

Optionally, a transformation can be given, which is added to the local
coordinates:

    origin.T=[<x> <y> <z>]

Additionally, for supporting the disparity images of the Middlebury benchmark,
if the name of the disparity starts with `disp<x>...`, then the file
`calib.txt` is loaded from the same directory and the information for camera x
is converted accordingly.

Furthermore, the maximum disparity step between neighboring pixels can be
defined for triangulation. The default is 1.

    step=<step>

--- Automatically finding the parameter files and image ---

If the parameters p and i are not given, then the tools try to find the
parameter files and image automatically. This works by first deriving a
list of possible prefix from the name of the depth image. A prefix is the
part of the name up to ':' (for supporting tiled images), up to the last '.'
or up to every '_', i.e.:

    <prefix>:*
    <prefix>.* (last dot)
    <prefix>_* (for every '_' in the filename)

The prefixes are optionally combined with the search path which is given
with the command line parameter -spath or the environment variable
CVKIT_SPATH. This effectively increases the number of possible prefixes.

For finding parameter files, all prefixes are combined with the suffixes
.txt and _param.txt, i.e.:

    <prefix>.txt
    <prefix>.TXT
    <prefix>_param.txt
    <prefix>_PARAM.TXT

For finding texture images, the files must be readable images. The size must
be the same than the size of the disparity or height image. It is also
permitted that the texture image is by an integer factor bigger than the
disparity or depth image. All files with all possible
prefixes are tested, i.e.:

    <prefix>*
    <prefix>:*

If there is more than one file that could be the texture, one of it is taken
by chance. Color images are preferred in this case.

Additionally, for supporting the disparity images of the Middlebury benchmark,
if the name of the disparity starts with `disp<x>...`, then the file `im<x>.*`
is loaded as texture.

Lets consider the following files in two different directories:

    dir1/image.jpg
    dir1/image.txt (containing camera.A, camera.R and camera.T)

    dir2/image_disp.pfm
    dir2/image_param.txt (containing rho)

This can be visualized by explicitely specifying all files:

    cd dir2
    plyv image_disp.pfm,p=image_param.txt,p=dir1/image.txt,i=image.jpg

Alternatively the same can be visualized by automatically finding the
associated parameter files and images:

    cd dir2
    plyv -spath dir1 image_disp.pfm

VISUALIZING CAMERAS IN PLYV
---------------------------

Camera definitions can be loaded into plyv and are shown as pyramids. Links
between cameras can be visualized as well. All files with the suffix txt are
loaded as camera definitions. The files must contain at least the camera matrix
(see above) and image size:

    camera.A=[<fx> <s> <u>; 0 <fy> <v>; 0 0 1]
    camera.width=<image width>
    camera.height=<image height>

Optional properties with default values are:

    camera.R=[1 0 0; 0 1 0; 0 0 1]
    camera.T=[0 0 0]

Cameras with overlapping views that are used for stereo matching are given
as comma separated list of camera definition files:

    camera.match=<camera 0>,<camera 1>, ...

The depth range of the scene can also be visualized for each camera. It is
defined by:

    camera.zmin=<min distance>
    camera.zmax=<max distance>

SUPPORTED PLY PROPERTIES
------------------------

There is sufficient documentation about the PLY format online on several web
pages. cvkit supports PLY in ASCII or binary with little and big endian
encoding. This sections describes the properties that are understood by plyv.
Unsupported properties are ignored when reading files. An origin can be
defined as comment. This can be useful for geo-referenced models. The
format is:

    comment Origin <x> <y> <z>

Additionally, the pose of the initial camera can be defined by the rotation
matrix Rc and the translation vector Tc, such that Pw=Rc*Pc+Tc with Pc as a
point in camera coordinates and Pw as point in world coordinates:

    comment Camera [<r00> <r01> <r02>; <r10> <r11> <r12>; <r20> <r21> <r22>] [<x> <y> <z>]

### Point Clouds

The following properties are supported in the 'vertex' element:

- x, y, z

Coordinates of a vertex.

- nx, ny, nz

Optional per vertex normal vector. The normal vector is calculated on the
fly if a mesh is defined (see below)

- diffuse_red, diffuse_green, diffuse_blue or
- red, green, blue

Optional per vertex color.

- scan_size

Optional real world size of a vertex. The size is typically derived from the
sampling distance of the scan. It is used by plyv for increasing the size
of vertices if the viewer comes closer.

- scan_error, scan_conf

Optionally, the scan_error defines the expected geometric real world error
of the vertex in scan direction (see sx, sy and sz below). scan_conf is the
confidence about the existence of the vertex in the range of 0 to 1. Both
properties are currently not used by plyv, but will be supported in future.

- sx, sy, sz

Optionally, the location from which the vertex has been measured. This can
be used for interpreting the direction of the scan_error and as a hint that
the space between this location and the vertex should be free. The
properties are currently unused by plyv, but will be supported in future.

### Mesh

The following properties are supported in the 'face' element:

- vertex_indices or
- vertex_index

Only the first three indices of the list are read and interpreted as
triangle. Remaining indices (if any) are skipped without warning. The
indices are starting with 0. The triangles must be numbered anti-clockwise
when seen from outside.

#### Per-Vertex Texture Coordinates

Instead of per-vertex color, texture coordinates can be defined in the
'vertex' element:

- u, v or
- texture_u, texture_v or
- s, t

The coordinates must be in the range of 0 to 1, which is scaled to the
width and height of the texture image. The name of the texture file is
usually given in a comment, i.e.:

    comment TextureFile <filename>

If this comment does not exist, but there are per-vertex texture
coordinates, then an image with the same name as the PLY file is sought,
i.e. the suffix ply is removed and the first file with the same base name and
a different suffix is used, if it can be opened as image.

#### Per Triangle Texture Coordinates

Additionally, it is also possible to define the texture coordinates per
triangle. This permits to use multiple textures per PLY file. The properties
of the 'face' element are:

- texcoord

This property must be a list with 6 values, i.e. two texture coordinates per
triangle corner.

- texnumber

This property defines the number of the texture for each triangle, starting
with 0. Multiple textures are given by defining multiple 'TextureFile'
comments. The order defines the number of each texture image.
