## Tutorial 5: soundscape
Goal: Learn how to use the Soundscape plugin to create spatialized sound and manage sound settings across multiple `Actor`s.

[Another good tutorial in Unreal]([https://docs.unrealengine.com/5.1/en-US/soundscape-in-unreal-engine/](https://docs.google.com/forms/d/1V4i_d5ktBFxxr8eaezJqX3fT6pC9wBKrD3VtIgqh2Cg/edit?usp=drive_web )

1. Create a new first or third person project
2. Turn the Soundscape plugin on (in `Edit > Plugins`). You’ll have to restart the editor.
3. Go ahead and find a few different audiofiles / metasounds to use. Drag them into the `Content` folder. You can also use the ones include in the starter content (real ones will use Explosion01.wav)
4. Drag your audio files into the scene, creating an `AmbientSound` `Actor` in the game world.
5. For each audiofile / metasound, make sure you specify an attenuation class under `Attenuation Settings`. Otherwise your sound won’t be spatialized (made to play from a physical location in the game world). You will need to create a new `Sound Attenuation` asset from this menu.

You might _not_ want a sound to be spatialized, for example, if it's a sound that "comes from" the UI, or is a disembodied deity's voice, etc. But those sounds should probably be played in the Blueprints for those respective things rather than placed in the scene.
6. Open your new `Sound Attenuation` asset from the `Content Drawer`. A `Sound Attenuation` is basically just a bunch of settings for how the sound should play relative to player distance. The nice thing about a `Sound Attenuation` being a special kind of `Data Asset` is that settings can be reused between multiple `Actor`s. There are a ton of options in here for the audio-inclined to pore over, but for now, let's just close this window.
6. For each sound, right click in the `Content Drawer` and create a new `Soundscape Color` (`Audio > Soundscape > Soundscape Color`). Each one of these will represent one of the sounds in your final soundscape palette; they contain various (reusable!) settings for sound playback.
7. Open up the new `Soundscape Color`. 
8. Choose the `Sound` (your audio file)
9. Make sure to choose the `randomize pitch` option, to provide some listening variety; players will get tired of any noise if they hear it often enough. Set these numbers far enough apart to have the randomization be perceptible; `0.95` and `1.2` will suffice. 
. Also check `Spawn Behavior > Continuously Respawn` so that the sound will be repeatedly triggered. Experiment with other settings as you see fit.
10. Make `Sound Palette` (`Audio > Soundscape > Sound Palette`) in the `Content Drawer`.  `Sound Color`s control the nature of the sound playback, but `Sound Palette`s control when those sound play via a tag system.
11. Open it up. For now, leave the `Playback Conditions` blank, but go ahead and add in all your various `Color`s.
12. Soundscapes are triggered by setting `Gameplay` states, aka `Tags`. Open your project settings and select `Project > GameplayTags`. Then open the `Gameplay Tag Manager` by clicking `Manage Gameplay Tags`. Click the + button to add a new tag, and call it `ACTIVE`. Set the Source to be `DefaultGamePlayTags.ini`, and press the Add New Tag button.
13. While you’re in your project settings, let’s add the `Soundscape Palette` to the project. Do a search for `Soundscape`, and then add your palette to the `Soundscape Palette Collection`. This step is important for getting soundscapes to be a dynamic, considered part of the game world as managed by the engine. Close your project settings when you’re done.
14. Now we’ll setup your sound palette to be triggered by the `ACTIVE` tag. Open your `Soundscape Palette` again and click the `Soundscape Palette Playback Condition`. Edit it; set the `Root Expression` to `Any Tags Match`. Under `Tags`, choose `ACTIVE`. This makes the game switch to this `Sound Palette` whenever a `Tag` is given to the `Soundscape Subsystem` that matches the name `ACTIVE`. Save and exit.
15. OK, almost done! Open your level Blueprint. Add an `Event BeginPlay` node, and then a `Get Soundscape Subsystem` node.
16. Drag the output pin from the `Get Soundscape Subsystem` node to serve as a parameter for a `Set State` node. Choose your `ACTIVE` tag from the `Soundscape State`.
17. Connect the `Event BeginPlay` output to your `Set State` trigger. Save and compile your [level blueprint](./level.png) Save and compile.
18. Test your level! You should hear your soundscape in action. If you used explosions, it probably sounds like a warzone, or perhaps like Lancer Deltarune showed up.

Yay! Sound! That changes based on where you are! Hopefully, this gives some idea of how to more precisely and centrally manage sound in Unreal. Send me...a pic of your favorite trash-eating animal if you got here.
