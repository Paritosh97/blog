# Differentiable Rendering

As I’ve started to pay more attention to machine learning, differentiable rendering is one topic that caught my attention and has been popping up with some frequency. My first thought was, “cooooool is this a new system for generating pixels that somehow can leverage machine learning?” After digging into the topic, this was immediately followed by disappointment, but the disappointment was ultimately replaced by a realistic excitement for the actual practical applications. So, what is differentiable rendering?

Inverse graphics attempts to take sensor data and infer 3D geometry, illumination, materials, and motions such that a graphics renderer could realistically reproduce the observed scene. Renderers, however, are designed to solve the forward process of image synthesis. To go in the other direction, we propose an approximate differentiable renderer (DR) that explicitly models the relationship between changes in model parameters and image observations.
-OpenDR: An Approximate Differentiable Renderer (paper, tech talk)

OpenDR can take color and vertices as input to produce pixels in an image and from those pixels, it retains derivatives such that it can determine exactly what inputs contributed to the final pixel colors. In this way, it can “de-render” an image back into colors and vertices.

OpenDR is “approximate” because there are discontinuities in rasterization, for example due to occlusion. This is just one form of differentiable render engine (a rasterizer), but other forms of DR exist including ray marching, point-based techniques, or even a single shaded surface. The single shaded surface case (imagine a full-screen quad) is perhaps easiest to understand, as it only requires propagating through the lighting and BRDF to get back to the inputs. So how is this useful?

One use case for differentiable rendering is to compute a loss when training a machine learning model. For example, in the SVBRDF reconstruction paper, the network produces four output texture maps (diffuse albedo, specular albedo, roughness, normal), but computing the loss in those four spaces individually was insufficient. The problem was that a comparison between the target normals (for example) and the inferred normals did not capture the perceptual loss that is visible when the texture is actually rendered as a lit surface. A differentiable renderer was used to compute the loss in rendered image space; the loss was then propagated back to the four texture inputs and from there, back propagation was applied to train the network.

DR has a similar application in Real-Time Height Map Fusion using Differentiable Rendering. The goal is to robustly reconstruct a height map from a single monocular camera. Using a triangle mesh and DR, both efficiency and robustness were improved.
An important element of our method which enables highly efficient operation is a differentiable renderer implemented within the standard OpenGL computer graphics pipeline. Given a current surface model and a camera pose it can render a predicted image and depth for each pixel together with the derivatives of these quantities with respect to the model parameters at almost no extra computational cost.
The initial depth estimation is converted into a height map and then rendered as a triangle mesh with vertex displacements to produce a new depth map which can then be used to compute loss. The entire process is differentiable, meaning the loss can be tracked back through the depth map, through the triangle mesh, back through the height map, back to the original depth estimation and back through the network for training.

Conclusion & Additional Reading
This was a quick overview of differentiable rendering, which I hope was useful if, like me, you’ve found yourself wondering about the definition and uses of DR. The following is a short list of additional papers related to differentiable rendering. I’m not making any claims about quality of these papers, these are just related works which I found while writing this post.
RenderNet: A deep convolutional network for differentiable rendering from 3D shapes
Inverse Problems in Computer Vision Using Adversarial Imagination Priors
Differentiable Image Parameterizations
Neural Scene De-rendering