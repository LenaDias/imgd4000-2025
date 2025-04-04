## Tutorial 3: Projectile spawning, Collision tests, Blueprints & C++, Particles
Goal: First, we'll add a projectile the player can shoot, which will teach us how to spawn and manage actors. 

Then, we'll extend our C++ code from last class with a `hitscan` laser projectile. We'll learn about `Collision Presets` as a way to manage types of "blocking" along the way.

After that, we'll augment this by attaching a fire particle effect to the struck object. This will
take advantage of Blueprints. Part of the goal of this class is to spend a bit more time in the various environments Blueprints gives us, while also connecting Blueprints to C++.

Then, we'll destroy the struck object!


## Adding a projectile

### Creating Our Projectile subclass
We’ll make a simple Actor to define our projectiles. These Actors will then be spawned (inside of our SpherePawn code) whenever the spacebar is pressed.

1, Create a new C++ class and make it an `Actor`. Name it `ProjectileActor`.
We only need to add two members to the header file. Make them both public, although we’ll probably only access the `Speed` parameter externally in this demo:
```c++
public:	
	AProjectileActor();
	class UStaticMeshComponent * SphereMesh;
	float Speed;
```
2. There’s some kinda tricky code to instantiate a new Sphere. We’ll do that in our constructor, in addition to initializing our Speed property.
```c++
AProjectileActor::AProjectileActor()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;
	SphereMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("SphereMesh"));

	static ConstructorHelpers::FObjectFinder<UStaticMesh>SphereMeshAsset(TEXT("StaticMesh'/Engine/BasicShapes/Sphere.Sphere'"));
	SphereMesh->SetStaticMesh(SphereMeshAsset.Object);

	Speed = 10.f;

	RootComponent = SphereMesh;
}
```
Note the last line… by specifying the `SphereMesh` as our `RootComponent` (a property every `Actor` has) we ensure that this mesh will be the primary representation of our `Actor` and be placed at the top of the component hierarchy. You can see this hierarchy by viewing an `Actor`'s `Details` window in Unreal Editor.

3. Next let’s define a `Tick` method for our `Projectile`. This will control how they move once they have been spawned.

The `Tick` function runs every frame, so it's good for things that need constant updating (like our `Actor`'s movement). `DeltaTime` represents the amount of time that has passed between frames:
```c++
void AProjectileActor::Tick(float DeltaTime){
	Super::Tick(DeltaTime);
	FVector forward = GetActorForwardVector();
	SetActorLocation( GetActorLocation() + forward * Speed );
}
```
Hopefully this is fairly straightforward (hah, pun). <--------------roberts wrote this   We get the forward vector (the direction our `Projectile` is facing… this will be set when the `Projectile` is spawned) and then move it in that direction, scaled by our `Speed` property.

4. All done! Make sure it compiles correctly (`Build > Compile` at the top of VS), and then move on to firing it from our `SpherePawn` player.


### Mapping the `Fire` input
Now, we need to add the `FireAction` input. Remember how last time we set up `EnhancedInput` using an `InputMappingContext` in the `SphereController` class? We'll basically be doing that again. Now is a great time to test your knowledge on input mapping if you like!

1. In the `Content Drawer`, create a new `Input > Input Action`. Call it `IA_Fire`.
2. Open `IA_Fire`. Make sure it outputs a `Value Type` of `Digital (bool).`
3. In the `Content Drawer`, find your `Input Mapping Context` (probably called `IMC`). Open it.
4. Add a new `Mapping` and wire it to your `IA_Fire` `Input Action`. Map it to the `Space Bar` and set a `Trigger` for a `Down` input.
![image](https://github.com/user-attachments/assets/0e5a061b-847e-4c5f-b88b-c140b9c96ef1)

5. Now, back to C++. Open your `SphereController.h` header file.
6. Add a protected, `UPROPERTY`-decorated member that stores the `FireAction` (we'll feed this our `IA_Fire` `Move Action` later.)
7. Also add another private method:
```c++
	void Fire(const FInputActionValue& InputActionValue);
```

8. Open your `SphereController.cpp` file.
9. At the bottom of the constructor, bind the input to your `SphereController`'s `Fire` method as you did with `Move`:
```c++
//Bind player actions to the SphereController methods
EnhancedInputComponent->BindAction(MoveAction, ETriggerEvent::Triggered, this, &ASphereController::Move);
EnhancedInputComponent->BindAction(FireAction, ETriggerEvent::Triggered, this, &ASphereController::Fire);
```
10. And implement the `Fire` method such that it passes on the message that the player is attempting to fire to your `SpherePawn`:
```c++
void ASphereController::Fire(const FInputActionValue& InputActionValue)
{
	GEngine->AddOnScreenDebugMessage(0, 3.0f, FColor::Green, FString(TEXT("SphereController fire")));
	if (AMyPawn* ControlPawn = GetPawn<AMyPawn>())
	{
		GEngine->AddOnScreenDebugMessage(0, 3.0f, FColor::Green, FString(ControlPawn->GetName()));
		ControlPawn->Fire(InputActionValue);
	}
}
```

### Firing our `Projectile` subclass from the `SpherePawn`
1. First, we want to create a `Fire` method for our player (represented by the `SpherePawn`) that'll do the actual projectile creation and firing. In our `SpherePawn.h` header, just add the following as a protected method:
```c++
void Fire(const FInputActionValue& InputActionValue);
```
2. We also want to add a `public` declaration in our `SpherePawn` header that gives our `SpherePawn` its own `Projectile`-launching speed that we can edit in Unreal Editor. This is good practice for created easily-tweaked, parameterized variables:
```c++
	UPROPERTY(EditAnywhere)
	float ProjectileSpeed;
```
3. Add an `include` statement in `SpherePawn.h` to let the compiler know we're using that implementation of `ProjectileActor`: 
```c++
#include "CoreMinimal.h"
#include "GameFramework/Pawn.h"
#include <GameFramework/FloatingPawnMovement.h>
#include "ProjectileActor.h"
#include "MyPawn.generated.h"
```
3. Add initialization for our `ProjectileSpeed` property to the end of our `SpherePawn` constructor:
```c++
  ProjectileSpeed = 10.f;
```
4. Last but not least, we add our `Fire` method to our implementation:
```c++
void ASpherePawn::Fire() {
	FVector loc = GetActorLocation();
	loc.Z += 100.f;
	loc.Y += 50.f;
	
	AProjectileActor *a = GetWorld()->SpawnActor<AProjectileActor>( loc, GetActorRotation() );

	a->Speed = ProjectileSpeed;
}
```
6. Make sure this drop-down is set to `UnrealBuildTool`.

![image](https://github.com/user-attachments/assets/8b80808a-446c-49fa-a86b-77c1e71f0ec9)

7. Perform a `Build > Clean`, then a `Build > Build`.
8. Remember your `BP_SphereController` subclass you made last class? Let's open it up again. Notice how it automatically updated to reflect the changes you made, like the new `UPROPERTY` to `SphereController.cpp` and `SphereController.h`, since it is a subclass of those.
9. Set the `Fire Action` to your new `IA_Fire` `Input Action`.
10. Save and compile.
11. Back in Unreal Editor's main window., hit `Play`. You should be able to fire via the `Space Bar` now.  You can also adjust the speed of the `Projectile` using the `SpherePawn` `Details` view. Yay!

## New types of projectiles

### Concepts for hit tests
So, we made a simple example of a projectile that flew off into space. However, for many types of of "shots" it’s better to be near-instantaneous... think of a laser, for example. In these cases, we want to simply see if a hit occurs at the moment the shot is fired. 

The solution for this is usually referred to in games as "hitscan". We _scan_ to see if a hitbox overlaps with something in the world, and then the game can react instantly to see if it _hit_. The other kind of projectile, like the one we made, is simply called a "projectile"; it's a physical object moving through the world (good for things you want to be affected by the physics engine or travel slowly- maybe you're modeling a bullet's wind drift).
[Visual explanation](https://www.youtube.com/watch?v=mduj4C9ZuhU)

Unreal enables us to do a "hitscan" shot like this with `traces`. A trace in Unreal is the equivalent of a `Raycast` object in other game systems. We’ll project a line from one point to another, see if any objects are struck in the process, and return the first one we hit that `Blocks` (does not allow objects to pass through it). Although you can [define custom filters for the types of objects you want to perform hit tests with](https://www.unrealengine.com/en-US/blog/collision-filtering) in this case we’ll use one of UE5’s built-in filters, which checks against all objects that are visible. 

### Firing a hitscan: Unreal Editor
So, let's wire up the ability for the player to fire a laser. It will use "hitscan" hit detection and print a message on hit.

1. Just as you did before for `Fire`, go through your `IMC` and add a new `Input Action` to our `InputMappingContext`. Name it `IA_FireLaser` on the... `L` key. For Laser.

But hold up- this is a "Fire" action, right? Why aren't we mapping this under the same `IA_Fire` action as before? 

Well, `Input Action`s are simply a abstraction layer. They represent something your player can do; they don't care about the technical details of what key was pressed to make that happen. That's the `Input Mapping Context`'s job. This means the `Input Mapping Context` is also more useful for mapping "many buttons" to "one action" than it is mapping "one button" to "many actions", since it just provides a way to map as many inputs you want onto a function through binding. This would be really useful, say, if you had a game that allows for keyboard, gamepad, and touchscreen control.

If you look in `SphereController.cpp`, you'll see that `BindAction` doesn't provide a way to check which button was pressed. It simply looks for what trigger to listen for, and what function to run when it is triggered:
```c++
	EnhancedInputComponent->BindAction(FireAction, ETriggerEvent::Triggered, this, &ASphereController::Fire);
```

You may want to read [this](https://gamedev.stackexchange.com/questions/189461/how-to-see-which-mouse-button-was-pressed-unity-input-system) for why that is. It explains it much better than I can.

TLDR: An `Input Action` is just a hook to tell a function to run whenever any arbitrary set of inputs is detected! It is _not_ an input map itself- that's the `Input Mapping Context`. It is _not_ the logic following those inputs- that's your `PlayerController`'s job.

3. Also wire up the inputs in your `SphereController` and `SpherePawn`. `FireLaser` should be a new method.
4. Build (Pause `Live Coding`/close Unreal Editor if you get a MSB3073 code 6) and then return to Unreal Editor
5. Remember to go into your `BP_SphereController` and set the `Fire Laser Action` to `IA_FireLaserAction`.


### Firing a hitscan: Code
We will primarily be implementing the `SpherePawn::FireLaser` method.

1. But first, we need to include a guide that will let us see where we’re shooting, so that we can visually identify if hits should occur. Include the following at the top of `SpherePawn.h`:

```c++
#include "CoreMinimal.h"
#include "GameFramework/Pawn.h"
#include <GameFramework/FloatingPawnMovement.h>
#include "ProjectileActor.h"
#include "DrawDebugHelpers.h"
#include "MyPawn.generated.h"
```

This gives us a variety of different shapes that we can dynamically draw in our world at different locations and with different orientations. In this case we’ll be using lines. 

2. Now go to our `FireLaser` implementation. We need to define a start point and an endpoint for our line. 	Our code already defines a starting location at the beginning of our fire method, let’s also augment its X component:

```c++
FVector start = GetActorLocation();
start.Z += 100.f;
start.Y += 100.f;
start.X += 150.f;
```

3. Next, we want to define an end position. This will simply be our start + (actorForward * distance):

```c++
float distance = 1000.f;
FVector fv = GetActorForwardVector();
FVector end = ((fv * distance) + start);
```

4. Now let’s draw a line using a function from our draw debug helpers:

```c++
DrawDebugLine(GetWorld(), start, end, FColor::Red, false, 2.f, 0, 5.f);
```

The first four parameters are probably self-explanatory. The fifth(currently `false`) dictates whether the line is persistent. The sixth is the amount of time the line is on screen, then depth priority, and last but not least line thickness.

Ideally, we'd make this new kind of projectile another file (maybe a `LaserActor`) but I am very sleeby.

5. If you `Build` and `Play` now, you should have some shooting lines that appear when you hit the `L` key.


### The hit test

OK, let’s find out if something is hit. 

1. We need to create a couple of public structs in our `SpherePawn.h` (those start with `F`, remember?) to pass to our hit test.

```c++
FCollisionQueryParams cqp;
FHitResult hr;
```

2. We’ll use the default query parameters that our `cqp` struct gives us, while our `hr` struct will be populated with the results of our hit test.

The call to our hit test, in `SpherePawn::FireLaser`, is:

```c++
GetWorld()->LineTraceSingleByChannel(hr, start, end, ECC_Visibility, cqp);
```

...where `ECC_Visibility` defines that trace filter that we want to apply to our collision test; in this case, is the object visible. Again, the results of our hit test will be place in the `FHitResult` struct, `hr`.

3. A blocking interaction is a type of collision that occurs when an object set to `Block` hits an object that it is allowed to `Block`. 

That sounds kind of vague; if you select our `Cube` in the `Outliner`, the `Details` window will show that it has a `Collision > Collision Presets` of `Default`, meaning it'll use the collision settings that were applied to the `Static Mesh` in the `Static Mesh Editor`:
![image](https://github.com/user-attachments/assets/bc5005a8-9945-4eee-be08-2c6bb14574e3)

For `Shape_Cube` `Actor`s, the `CollisionPreset` in the `Static Mesh Editor` is set to `BlockAll`, as you can see by opening `Starter Content > Shape_Cube`:
![image](https://github.com/user-attachments/assets/d6d2e38d-9f40-45db-b156-2f3d7f411fb4)

This means that, by default, a `Shape_Cube` that you drop into the scene will `Block` all other objects. There are all sorts of ways to manage interactions between different `Collision Preset`s here (and you'd want to do that, if, say, you wanted objects that are immune to certain types of blocking, but for now, this is all we need for our hitscan test.

If you'd like to know more about collision responses, [read this](https://dev.epicgames.com/documentation/en-us/unreal-engine/collision-in-unreal-engine---overview).

Now let’s look and see if a hit was found on a blocking object:

```c++
if( hr.bBlockingHit == true ) {	
    if( hr.GetActor() != this ) {
        UE_LOG(LogTemp, Warning, TEXT("HIT! %s"), *hr.GetActor()->GetName() );
        hr.GetActor()->Destroy();
    }
}
```


4. OK! If you hit `Build` and then `Play`, we should be getting the lines drawn, but no other results. Go ahead and drag a Cube actor (or any other shape) out into the world in a place where the rays will be able to strike it. Try it again, and you should see the instance name of the object presented in the message console. The cube should also be destroyed.

5. Instead of using `UE_LOG` we can also draw our debugging messsages to the screen. Change the above block of code to the following to try this out:

```c++
if( hr.bBlockingHit == true ) {	
    if( hr.GetActor() != this ) {
        GEngine->AddOnScreenDebugMessage(-1, 5.f, FColor::White, FString::Printf( TEXT("HIT! %s"), *hr.GetActor()->GetName() ) );

        hr.GetActor()->Destroy();
    }
}
```

This is a bit quicker to look at, but doesn't save the message to a log file. Of course, there's nothing to stop you from using both `UE_LOG` and `AddOnScreenDebugMessage` to get the best of both worlds.

## Adding a particle system

### Creating a “hit” event for Blueprints
We’re going to edit / dynamically place our particle emitter using Blueprints, but before we can do that we need to create 
an event that will tell us when our hit test actually strikes the enemy cube. 

1. We’ll add this `HitSomething` function to the header for our `SpherePawn` class:

```c++
UFUNCTION(BlueprintNativeEvent)
void HitSomething(class UStaticMeshComponent *m);
void HitSomething_Implementation(class UStaticMeshComponent *m);
```

Important (and bizzaro) things to point out, that all relate to the bindings between these C++ classes, the editor, and the reflection engine:

 1. The `BlueprintNativeEvent` specifier will enable us to create a node in Blueprints that outputs an event trigger.
 2. We need to create two separate functions. One generates our Blueprints event, and one contains an implementation for the event. 
 3. You must name the implementation the same as the Blueprints event + `_Implementation` or the compiler will throw an error. This is a very specific way to create bindings between C++ / Blueprints / the editor. This method will be called if the event is not used in the Blueprint. Since we know we are going to use the event in our Blueprint we could basically leave this blank, however, you MUST include the function in the header/C++ files or your class will not compile.

2. OK, with that out of the way, we can all our `HitSomething` method from our `FireLaser` method (assuming we do actually hit something). In the `SpherePawn` C++ file, switch the following line in the `FireLaser` method:

```c++
hr.GetActor()->Destroy();
```

 to the following:
```c++
HitSomething( Cast<UStaticMeshComponent>(hr.GetActor()->GetRootComponent()) );
```

3. For our implementation, we'll just leave it with a log statement:

```c++
void ASpherePawn::HitSomething_Implementation(class UStaticMeshComponent *meshThatWasHit ) {
UE_LOG(LogTemp, Warning, TEXT("HIT SOMETHING CALLED"));
}
```

Note that calling `HitSomething` (which we defined as a UFUNCTION with the `BlueprintNativeEvent` specifier) is what actually triggers the event in Blueprints. We have to include that call.

4. Compile your C++ code and then move on to creating / editing a Blueprint subclass of it. DO NOT TEST IT AT THIS STAGE OR YOU WILL CRASH THE ENGINE.

### Into the Blueprint
OK, so now we want to make a Blueprint “subclass” out of our `SpherePawn` C++ class, so that we can access the Blueprint event we created. 

1. Return to Unreal Editor.
2. Go ahead and delete any previous instance of the C++ `SpherePawn` class that you had placed in the scene. 
3. Next, right-click on the `SpherePawn` C++ class in your `Content Browser`
and choose “Create Blueprint class based on SpherePawn”.  Name the new blueprint `BP_SpherePawn`.

4. Open up the resulting Blueprint.
5. Click on the “Event Graph” tab to open the visual programming environment, or just select the corresponding tab if it's already open.
6. We’re not going to use any of the events they give us, instead we’ll use the one we just added in C++... go ahead and delete the other events with `Backspace`.

7. Right-click in any of the blank space and then type “Hit” to search through the available nodes. 
You should see our custom `Event HitSomething` event appear in the dropdown menu; select it. Great! We now have access to the C++ event we created.

The arrow at the top of this node is the control flow for event processing, while the blue output marked `M` is the mesh that has been hit.

9. Now let’s spawn our particle emitter. Right-click on the Blueprints canvas again and search for `Spawn Emitter Attached`. Click on it.
10. Connect the topmost control flow output from our `Event HitSomething` node to the topmost control flow input in our `Spawn Emitter Attached` node. 

11. Next, we need to select the particle system that will be spawned. Click on the `Emitter Template` input for the spawn node, and then select the `P_Fire` that we migrated to this project earlier. If you don't see this as an option, you probably need to import the `Starter Content` into your project.

For custom particles, check out [this tutorial](https://dev.epicgames.com/community/learning/tutorials/b8yy/unreal-engine-unreal-particle-effects-tutorial).
12. Almost there! Connect the “M” output to the “Attach to Component” input of our spawn node.
13. Set “Location Type” on the spawn node to be “Snap to Target, Including Scale”. These will attach our particles to whatever ends up getting hit.
Your `BP_SpherePawn` `Event Graph` should look like this:

![image](https://github.com/user-attachments/assets/8db9752f-f906-49e9-af76-0077beddb262)


14. Last step: Edit your `SphereGameModeBase` to spawn the new `BP_SpherePawn` subclass as the player `Pawn`.
![image](https://github.com/user-attachments/assets/92a654d0-0833-4492-a63b-c3d2285122c5)
![image](https://github.com/user-attachments/assets/dcfbc65d-75db-4292-aa1a-e7a37d087cfe)


16. OK! Hit `Save`,`Compile` your Blueprint,  and then hit `Play`. You should see the particle system placed when you hit the cube with your line trace. It might help to make the cube a bigger target (by scaling it) so it’s easier to hit.


### The cube has burned and must be removed <- roberts wrote this too??
Let's now setup our blueprint so that after the fire has burned for a few seconds our target is removed from the scene. We'll also set it up so that our fire continues to burn for a second or two afterwards.

1. In order to create delays in our Blueprints we can use the `Delay` node. Back in your `BP_SpherePawn` event graph, Drag a pin from the empty white control node at the top of `Spawn Emitter Attached` and then search for `Delay`. Use `3.0` or whatever else you want for our delay duration.

![image](https://github.com/user-attachments/assets/98925bc1-e0fe-4601-886d-747a7e60bad3)

2. At the end of these three seconds we want to set out cube visibility to 0. While we could simply destroy the cube actor, our particles have been attached to it and would immediately also disappear if we did this. By changing the visibility of the mesh to 0, other components remain visible.

Drag a pin from the `Completed` control flow output of our `Delay` node, and search for `Set Visibility`. You may need to disable `Context Sensitive`. 

3. Connect the `M` output from our event to the `Target` output of our new `Set Visibility` node. Make sure the `New Visibility` argument is unchecked (false).

![image](https://github.com/user-attachments/assets/25a0e111-6747-469b-909d-99701c73a6d3)

4. If you save/compile and test your blueprint now, you should see the cube vanish after three seconds while the fire remains.

5. Next we'll drag a pin from the control flow output of `Set Visibility` and create a second `Delay` node; set the duration for `2.0` seconds.
6. Drag a pin from the `Completed` control flow output and choose the `Destroy Actor` node.

7. Last but not least we need the actor we want to destroy. Drag a pin from the `M` output of our event; remember that this represents our `StaticMeshComponent`. To get the `Actor` that owns the mesh, we can make a `Get Owner` node.
![image](https://github.com/user-attachments/assets/3dc553e2-1053-4f3a-a32d-679623efc2d2)

8. Take the return value of `Get Owner` and connect it to the `Target` input of our `Destroy Actor` node. Your final blueprint should look something like this, minus all the formatting and comments:
![image](https://github.com/user-attachments/assets/4911e4db-e951-4309-93a0-753ed77678ba)

9. Save and compile the blueprint, and then test your scene to see the results. There's no nice fade transition or anything, but it works!


## Yay! All done!

And congrats! That's all for this tutorial. We learned a lot about connecting C++ and Blueprints, managing `Actor`s, collision types, and even dipped into the particle system. Hopefully it was also a good refresh on what we learned last time. Send me a gif of a ferret if you finished this one!

