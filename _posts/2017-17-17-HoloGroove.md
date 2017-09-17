# HOLO GROOVE
3D point clouds encoded in 2D vector graphics.

# Abstract
Scratch holography or 'Highlight holography' converts three-dimensional computer models into mechanical 'holograms' machined onto reflective surfaces. 
The surface consists of small grooves or 'scratches' each of which produces a highlight when illuminated by a directional light.
This highlight appears in different places for different view positions. 
Grooves with a particular path, can lend  the changing position of the highlight the correct binocular and motion parallax corresponding to a virtual 3D point position.

The pipeline presented here begins with a 3D computer model and a desired view position.
We then sample the model to generate points that depict its features accurately.
For each of these representative points we compute a location on the reflective surface that corresponds to the location of the desired highlight for that point and that view position.
We then move the view position, and repeat the computation to deterimine the highlight location again.

Each point in our cloud therefore has a corresponding highlight location on the reflective surface for each desired viewing position. 

We then draw a bezier curve that passes through each of these highlight locations, in this way smoothing the individual point's highlight location between view positions.

Finally we machine these paths as grooves onto a surface - using a diamond-tipped engraving bit to trace the paths over the surface of a reflective material.

This method offers two important advantages and adds to the work described in the prior art.

  1. It is relatively trivial to use raycasting to determine if a point should not be visible for a given view position. This gives our holographic image opaque characteristics.
  2. As easily as we can smooth between changing viewing positions we can smooth between changing point positions, potentially we can have animated holograms that change their shape and features as the viewing position changes. 

# Related Work

Abrasion holograms have been popular projects amognst hobbyists for some time.
William J Beaty desribed much of what is possible with this technique in 1995 - http://amasci.com/amateur/holo1.html

While this project from  2008 demonstrates software made by Mike Miller a student at the time at Milwakee School of Engineering.
https://www.youtube.com/watch?v=JaGZ651U4j4

Similarly, Matthew Brand a researcher and artist has evidentenly developed advanced proprietarty software and manufacturing processes for this type of Hologram as well as indicationg further work (litho, etc...)
http://www.zintaglio.com/

Generally  there have been incremental developments in the process and novel applications for the effect.
For example artist Tristam Duke http://www.infinitylightscience.com/ added an abrasion hologram to Jack White's albumn 'Lanzarotto' https://www.youtube.com/watch?v=YGH2m5sRylM

And in 2016 Disney researchers patented a method for computational generated abrasion holograms. https://github.com/gl2748/hologroove/blob/master/HighlightHolography.pdf - Although I have not been able to reproduce their results. 


# Setup
Considering a single groove __q__ on the hologram surface, which focuses diretional incident light to produce an image at the 3D point __p__ when viewer is at __v__.

As shown in Figure 1 the center of the groove __i__ is found by casting a ray from the central view position __v__ through the sample point __p__ and intersecting it with the hologram plane __r__.
Moving the viewer position __v__ gives another point of intersection on the plane __r__. Joining these two points of intersection gives us a path. 

$$ i = v - \left(\frac{v_y}{p_y - v_y}\right)(p-v) $$

******************************************************************
*                                                                *
*                                                                *
*                                                                *
*                                                                *
*                                                                *
*                                                      \         *
*                                                  o v (+ eye    *
*                                                 /    /         *
*                                                /               *
*                                  c+-------+   /                *
*                                  /|      /|  /                 *
*                                 +-------+ | /                  *
*                                 | |     | |/                   *
*                                 | +-----| /                    *
*                                 |/      |/                     *
*                      lo         +-------o                      *
*                        \               / p                     *
*                         \             /                        *
*                          \           /                         *
*                           \         /                          *
*                            \       /                           *
*                             \     /                            *
*                       +------\---/------+                      *
*                      /        v /        \                     *
*                 y           ---*--- q     \                    *
*                 ^  ^x           i          \                   *
*                 | /                         \                  *
*                 |/                           \                 *
*                 +---> z ----------------------+                *
*                                                r               *
*                                                                *
*                                                                *
******************************************************************
[Figure [diagram]: Setup for derivation of the reflective point corresponding to one virtual point __p__. The viewer position is centered at __v__, and parallel light rays are incident in direction __l__.]

Zooming in on the groove (Figure 2), we see how the incident light interacts with the interior of the groove, passing through the virtual point __p__ and onto the viewer at __v__. 

******************************************************************
*                                                                *
*                                                                *
*                             l|             \                   *
*                              v         o   (+ eye              *
*                              |        / v  /                   *
*                              |       /                         *
*                 y            |      /                          *
*                 ^  ^x        |     o                           *
*                 | /          |    / p                          *
*                 |/           |   /                             *
*                 +---> z --+  |  /   +----------                *
*                            \ | /   /           r               *
*                             \|/   /                            *
*                              +   /                             *
*                               \ /                              *
*                                *                               *
*                                 i                              *
*                                                                *
******************************************************************
[Figure [diagram]: Detailed view of the cross-sectional profile of a groove for one point]

The length of the groove is determined by the change in viewer position. The viewer moves position and the desired highlight location is re-calculated.
To create a groove a path between the two locations on the surface is drawn.

Because we do not have total control over the groove's cross-sectional profile beyond the tooling we confine viewer movement to a circular orbit.
This generates circular grooves that create the desired motion paralax for a rotating hologram plane __r__. 

As a certain point is occluded by some other point or surface in the virtual scene we interupt the continuous path and resume it when the point comes back into view. 

Later on we can possible create multiple grooves for each point, enforcing the intensity of the virtual point at __p__ .
An alternative aproach is to machine the inerior of the groove to be a parabaloid, thereby removing the need for curved paths. 


// In actual fact the incident light is reflected in many directions. 

To optimise  the initial setup  we take into account a few things.
1. The interior shape of the groove surface created by the available tools. For example using a 120o diamond tip:
We setup the desired view position to fall within the regular path of light reflected from inside the groove cut by the available tool.


## Pseudo code.

In Pseudo Code this is what the software does:

Generate a point cloud __c__ that represents the surface details of some 3d object.

Add a plane __r__ to represent the reflective surface we will be engraving the hologram onto.

Add an viewer position __v__ , which is used to calculate, for each point( __p__ in the point cloud __c__ :
 - When the point __p__  is occulded by opaque elements in the scene.
 - The point of intersection __i__ of a line that passes through the viewer position __v__ , the visible point __p__ and the plane __r__.
 
Either, move the viewer position __v__  and recalculate this point of intersection __i__ on the plane, drawing a continuous line(l) through all of the points to reflect the point's movement over time.

Or, move the eyeball(e) and recalculate the point of interstection(q) on the plane, drawing a continuous bezier curve(l) through all the points to reflect the point's relative movement over time.

Having generated an individual bezier curve(l) for each point(i) as it remains visible through whatever motion cast these to an svg file.


## Actual code.

Please see: 

We leverage Blender's Raycasting method to determine the visibility of a point.

The interesting part is drawing lines through all the points of intersection.

To do this refer to this line: 




For each point in the point cloud 

We initalize the project with an oject named 'Cube' in Blender's default start-up space.

We generate a particle system that attaches an even, random distribution of particles over the surface (faces) of the object.
Later we can optimize this distribution with something from here. https://bost.ocks.org/mike/algorithms/ For the time being we use Blender's baked in algorithm. (HERE).

We also consider the vertices of the opject points, because it visually helps establish the structure of the object. 

We also use weight painting to increase the density of points at the edges of the object, or where more fidelity is required.

With the particle system  in place we setup a plane. This represents the planar surface our hologram will be machined onto.
Furthermore, the plane's location relative to the 'Cube' object represents how the virtual image of the 'Cube'  will appear... for example it can appear floating above or below the plane surface.

We now add a camera object.  This represents the viewer's eyeball.
It is used to determine the point at which a line, drawn from the eyeball through a particular point in our point cloud intersects with the plane. 


## Results. 

<!-- Markdeep: --><style class="fallback">body{visibility:hidden;white-space:pre;font-family:monospace}</style><script src="markdeep.min.js"></script><script src="https://casual-effects.com/markdeep/latest/markdeep.min.js"></script><script>window.alreadyProcessedMarkdeep||(document.body.style.visibility="visible")</script>
