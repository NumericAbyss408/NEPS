# NEPS
A Non-Euclidean Portal system made on the Roblox Studio Platform
##  
Hello People!  
Today, I will be showcasing my own very own non-euclidean portal system, very reminiscent of the portals seen in Valve's <a href=https://www.thinkwithportals.com>Portal series. <a/>

## What it can do  
This portal system allows players to look through and travel through a portal seamlessly.
[examples here]  

## How i made it  
to make the portal system, i considered the problem and broke it into 2 main problems:  
- visualizing the portal
- Positioning objects that cross it  

In addition, before starting i made sure <a href=https://github.com/NumericAbyss408/CMV6-White-Hole> CMv7 <a/> was at a stage where it was ready for use. this was so that this portal system would become a reusable component, to be used in later projects. Another module that was used in this implementation was the Utilities module i made to package with ScryptTools (a suite of modules that make development a bit easier. *made by yours truly, read about it <a href = blank>here<a/>.*)  

### The visual problem
#### 1. Capturing the world
The portal needed to render the world in front of the exit portal, and it needed to do so fast enough not to hinder the portal creation process.
To do so i started by creating a 5x5x3 grid of very large parts that act as the "frustum", encompassing area in front of the portal.
I would then run `workspace:getpartsinpart()` on each cell and get all assemblies in the grid which are stored in a WorldModel for later use.
#### 2. Viewing the world
(this is where the fun begins.)
most implementations of the portal system would follow the logic described in <a href=https://youtu.be/_SmPR5mvH7w> DigiDigger's video (up to 4:30)<a/>:
- position the camera relative to the exit portal
- have that virtual camera follow our camera's rotation
- get full image of the virtual camera
- paste it onto the desired face.

*roblox doesn't like working that way.* if we are to use a surfaceGUI to render our portal, it is not possible to crop this virtual image to our liking, since masking is not really a thing on roblox studio.

so what we do instead is simulate the straight-on perspective *only*, moving and scaling the virtual FOV and the image to simulate perspective changes, in order to get a believable effect.

##### 0 - Preliminaries
we first need some setup. namely:
- the portal is rendered on the `back` face of the part, rather than the front.
this is so that our perpendicular relative distance from the portal is positive (since forward is -Z in roblox.) (this is to simplify later calculations)
- we get the camera offset, relative to the portal (obtained using `CFrame:pointToObjectSpace`).
this will return a vector 3 in the form (rightOffset, UpOffset, PerpendicularDistance)
- the `SurfaceGUI` we are going to use is going to have it's `SizingMode` set to `pixelsPerStud`. consider maxing out the `PixelsPerStud` property.
ensure that it is is a descendant of the `playerGui` folder. `ViewportFrames` don't play nice in workspace for some reason.
- the `ViewportFrame` we use is going to have its `AnchorPoint` set to `Vector2.one * 0.5`. this is so that scaling is uniform.

it can be useful to turn off the `SurfaceGUI`'s `ClipDescendants` property, to visualize how the image changes.

The following steps happen every frame, so consider enclosing them in a `heartbeat` function.
##### 1 - Centering the view
the portal's viewport has to be placed such that when we look at it straight on, it is centered to our view.
```lua
local Camera = workspace.CurrentCamera -- the default roblox camera
	
	local Offset = Entrance.CFrame:PointToObjectSpace(
		Camera.CFrame.Position
	)
	
	VFrame.Position = UDim2.new(
		0.5, Offset.X * SurfaceGUI.PixelsPerStud,
		0.5, -Offset.Y * SurfaceGUI.PixelsPerStud
	)
```
-- the result:  
<img width="557" height="356" alt="image" src="https://github.com/user-attachments/assets/69f88e59-db91-47a2-8dbd-e2432443d55b" />

##### 2 - Filling the pool
Now, notice how because we centered the ViewportFrame, the part is not filled properly. we can't move the frame back, so instead, we are going to scale in order to fill the part.
```lua
-- the new width and height of the viewport
-- we use 2 * |offset| because we want the image to remain centered
	local VWidth = (2*math.abs(Offset.X)) + Entrance.Size.X
	local VHeight = (2*math.abs(Offset.Y)) + Entrance.Size.Y
	
	-- since we are scaling, we are going to get ratios
	local VRRatioX = VWidth/Entrance.Size.X
	local VRRatioY = VHeight/Entrance.Size.Y
```
-- the result:  
<img width="492" height="314" alt="image" src="https://github.com/user-attachments/assets/855f7dee-7215-49a0-9075-720f37e40ccb" />

##### 3 - Manipulating the camera
the `Camera` is now ready to be positioned.
```lua
-- place the camera relative to the exit.
	-- it faces directly out of the exit. 
	-- this ensures that only our camera changes rotation.
	local FinalCFrame = CFrame.fromMatrix(
		TargetCFrame:PointToWorldSpace(Offset), 
		TargetCFrame.RightVector,
		TargetCFrame.LookVector
	)
```

now that the camera is in position, we have to correct its field of view.  
<img width="837" height="769" alt="image" src="https://github.com/user-attachments/assets/6bf41088-275f-4745-bf8b-45ddc81d7417" />  
The new Field of view can be derived by our PerpendicularOffset, and the stud half-height of the viewportframe (refer to step 2 - filling the pool).
```lua
local PerpDistance = Offset.Z
	local CorrectedFOV = math.deg(
		2 * math.atan((VHeight/2)/PerpDistance)
	)
	VCam.FieldOfView = CorrectedFOV
```
At this point, the portal effect is complete for the most part. however, when we come closer, the effect breaks because roblox imposes a 120* FOV limit. **This is where we are going to introduce some trickery.**  
Roblox allows non-orthogonal CFrames (so long as the magnitude of each axis of the 3x3 basis does not exceed ~1.2, otherwise the basis gets normalized).
something interesting that happens is that when the camera's CFrame is multiplied by a matrix
$`\begin{bmatrix}
n&0&0\\
0&n&0\\
0&0&1
\end{bmatrix}`$
, where $n < 1$, the camera's image becomes a percentage of a new image. for example, for n=0.8, you find that the camera's original image is now 80% of the new image.  
OriginalImage
<img width="1521" height="726" alt="image" src="https://github.com/user-attachments/assets/b0e22bc6-e3aa-4cea-8f21-4cfc1a01240a" />

ScaledImage
<img width="1531" height="717" alt="image" src="https://github.com/user-attachments/assets/e7e2e5a6-3535-4278-9980-5e1c4315afff" />

we can use this to break through the 120* FOV limit:
```Luau
if CorrectedFOV > 120 then
		local MaximumHeight = math.tan(math.pi/6) * PerpDistance
		local ImageScaleFactor = MaximumHeight/VHeight
		FinalCFrame *= CFrame.fromMatrix(
			Vector3.zero, -- prevents unwanted position change
			Vector3.xAxis * ImageScaleFactor,
			Vector3.yAxis * ImageScaleFactor,
			Vector3.zAxis -- to break orthogonality, applying scale
		)
end

-- finally, apply the CFrame
VCam.CFrame = FinalCFrame
```

#### Some Limitations
The main limitation of this solution is resolution. at extreme angles and closeups, the viewportframe starts getting very large. this results un lower resolution, which can look unsightly with large portals (which already suffer from lower resolution due to having to fill such a large frame.)

## Next Steps
- Implement Portal Movement
- Release a module that handles both the visual and mechanical aspects of the portal

### Sources used in my work
while a lot of this work ended up being trial and improvemnt, notable sources include:
- <a href=https://youtu.be/_SmPR5mvH7w>DigiDigger's video (up to 4:30)<a/>
- <a href=https://devforum.roblox.com/t/re-creating-a-portal-effect/337159>This DevForum thread <a/> in particular, `EgoMoose`'s contributions.

