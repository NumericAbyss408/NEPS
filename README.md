# NEPS
A Non-Euclidean Portal system made on the Roblox Studio Platform
##  
Hello People!  
Today, I will be showcasing my own very own non-euclidean portal system, very reminiscent of the see in Valve's <a href=google.com> Portal series. <a/>

## What it can do  
This portal system allows players and objects to travel to different areas of the map, adjusting for offset and retaining momentum. The Look-Through effect looks very convincing in most scenarios, only being dulled by roblox's internal limit.  
[examples here]  

## How i made it  
to make the portal system, i considered the problem and broke it into 2 main problems:  
- visualizing the portal
- Positioning objects that cross it  

In addition, before starting i made sure <a href=https://github.com/NumericAbyss408/CMV6-White-Hole> CMv6 <a/> was at a stage where it was ready for use. this was so that this portal system would become a reusable component, to be used in later projects (particularly DELTA SPEED, a passion project made on the same Roblox platform). Another module that was used in this implementation was the Utilities module i made to package with ScryptTools (a suite of modules that make development a bit easier. *made by yours truly, read about it <a href = blank>here<a/>.*)  

### The visual problem
I used a combination of SurfaceGUI and Viewportframes to solve this problem.
the first step is capturing the world the portal should see. i had to capture a possibly large amount of parts in the workspace, very quickly so as to not hinder the speed at which the visuals are generated.  
i landed on the following:
Create a 5x5x3 grid of very large parts that act as the "visible" area in front of the portal
run workspace:getpartsinpart() on each part
store these in a WorldModel for later use
