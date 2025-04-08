# Adding a GUI
In this tutorial we'll learn the basics of adding a GUI (Graphical User Interface), linking it to our `GameModeBase` subclass, and linking the GUI to a C++ `Pawn` class. We'll start with a brand new, blank project.

## Setup
1. Start by creating a new `Blank` C++ project. Set it to `Scalable` and include the starter content.
2. Create a new `Basic` level.
3. Go ahead and make a new `Pawn` C++ subclass (under `Tools`), and name it `SpherePawn`.
4. Add two public properties to the `SpherePawn` C++ header:

```c++
	UPROPERTY(EditAnywhere)
	class UStaticMeshComponent * Mesh;
	
	UPROPERTY(EditAnywhere, BlueprintReadOnly)
	float Health;
```

5. We're going to create a simple progress bar that will track the `Health` property of our `SpherePawn` instance. Go ahead and edit the `.cpp` file. We want to change the constructor and the tick method.

```c++
ASpherePawn::ASpherePawn()
{
 	// Set this pawn to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
  PrimaryActorTick.bCanEverTick = true;
  Mesh = CreateDefaultSubobject<UStaticMeshComponent>("Mesh");
  Health = 1.f;
}

void ASpherePawn::Tick(float DeltaTime)
{
  Super::Tick(DeltaTime);
  Health -= .001f;
}
```

With these two methods in place, our `Health` will decrease on every frame of gameplay. 

6. Hit `Build > Build [ProjectName]` in VS.
7. Compile your code in Unreal Editor:
![image](https://github.com/user-attachments/assets/79e621ea-2ddd-4036-964b-c37ff0f1a19d)
 
8. Drag in an instance of your `SpherePawn` class to the level.
9. Go ahead and assign mesh of your choice for the instance in the `Details` pane.
10. Set `SpherePawn -> Pawn -> Auto Possess` to `Player 0`. 

This is a different way of getting the player to "possess" a pawn and control it. You might recall that we set up player pawn possession through the `GameModeBase` `Class defaults` last time; that method is more useful for a player pawn that is used across the game. 

The "create an instance > details pane > auto possess" method is more useful for an individual, one-time use player pawn. For example, your game might normally use a `ThirdPersonCharacter` (that you set up with `Class defaults`) in most levels. But, if you have one gimmick level where you control a car, you can use the `auto-possess` method to just possess the new car pawn for that one level.

Your `Tick` method will not be called until the `SpherePawn` has possessed the player, so this is an important step.

## Modifying our build file
Next, we want to modify the build file for our project so that we can add the necessary UI libraries to the engine. 

1. In Visual Studio, go ahead and open the C# build file for your project, located with the rest of your project source code under `Games/[ProjectName]/Source/[ProjectName]/`. It should be called something like `[ProjectName].Build.CS`

This file configures many of the default libraries that are included in your build. Add `"UMG"` (for Unreal Motion Graphics) to the first dependency module configuration, so it reads:

```c++
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "UMG" });
```

2. Next, uncomment the `Slate` UI line immediately beneath the one you just edited. This includes everything we need to start doing GUI development in our project. Like `EnhancedInput` is a framework for input processing, `Slate` is a framework used for user interface processing.

## Editing our GameMode subclass
As we learned last time, the GameMode is responsible for broadly managing the game. This includes displaying relevant statistics via the HUD, managing win/lose conditions, and more. This is the class we’ll be using to display the GUI we create using the Unreal Motion Graphics editor (UMG).

1. Make a new subclass of `GameModeBase`. I named mine `GUIGameModeBase`. You can find that option here after hitting `Tools > New C++ Class`:
![image](https://github.com/user-attachments/assets/8b7b5508-59be-46eb-b1e6-a492c79d94c9)

2. Go ahead and open up your project’s `GameModeBase` subclass header;  the Unreal project template will have made this for you automatically. 

3. Add the following public properties to your class:

```c++
public:
	UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category="Health")
	TSubclassOf<class UUserWidget> HUDWidgetClass;
	
	UPROPERTY()
	class UUserWidget * CurrentWidget;

	virtual void BeginPlay() override;
```
We add a new `UPROPERTY` named `HUDWidgetClass` where we'll feed in the type of widget our widget will be.

The new class we see here, a `UserWidget`, is a type of Unreal Engine `Widget` that includes functionality for talking between Blueprints and C++. A `Widget` is basically just a piece of the UI that shows up on screen. Your phone, for example, also has "Widgets" you can add to its home screen that give you calendars, weather information, etc. that you can interact with without having to open a separate app. Same thing here. 

3. Next, override the `BeginPlay` method by adding it in the C++ file. Also include the `UserWidget` header file. Make sure to change your project name where appropriate, and don’t remove the A prefix in the class name that denotes this an Actor:	

```c++
#include "GUIGameModeBase.h"
#include "Blueprint/UserWidget.h"

void AGUIGameModeBase::BeginPlay() {
    Super::BeginPlay();

    CurrentWidget = CreateWidget<UUserWidget>(GetWorld(), HUDWidgetClass);
    //Add our widget to the game display
    CurrentWidget->AddToViewport();
}
```
Notice how after creating it, we add our new `Widget` to the game's `Viewport`. The `Viewport` is a layer that the game is seen through; this will add our UI `Widget` on top of the visualization of the game world when the player plays the game.

## Create a Blueprint to display our Health.
Let's make a Blueprint to finalize the `Health` display. 

1. Returning to Unreal Editor, go to the `Content Drawer`. Right-click and choose `User Interface > Widget Blueprint`. Select `User Widget` and name it `HealthHUD`.
2. Click on your new Blueprint and- wow! A new type of editor window should open! This the UI Widget editor. It has a lot of features that allow for precise positioning of UI elements. You'd make one of these to compile all your HUD elements into one place and arrange them.
3. This editor also has some pre-built elements in the `Palette`. Drag out a `Panel > Canvas Panel`. A `Canvas Panel` has some useful settings for anchoring, centering, etc. that allows you to arrange the widgets placed inside it. 

Notice how the `Hierarchy` window has had your new widget added to it. Notice, also, that this is called a `Hierarchy`; your widgets have levels of nesting, just like `Actors` in the `Outliner`. You can use this to group elements and reference the groups in code.
4. Next, lay out a simple `Progress Bar` widget. Make it bigger!
5. Place an accompanying `Text` widget. Adjust it have a larger `Font > Size` and make the `Content > Text` say `Health`.
6. With the progress bar you just added selected, choose `Details > Progress > Percent > Bind > Create Binding` (it may be slightly off-screen). This will create a new binding for your element that will enable you to update your progress bar in a Blueprint.

In the resulting Blueprint graph view, create a Blueprint like this. We get the player pawn, access its `Health` member, and feed it into the `Return Node` to print it to the widget:
![image](https://github.com/user-attachments/assets/11c6d52d-51e3-485a-856d-0bea5329eb6b)


(Notice that your editor has switched over to `Blueprint` view. You can return to the `Designer` view in the top right)
7. `Save` and `Compile` this blueprint.

## Link everything together

1. Back in the main editor, create a Blueprint subclass of our `GameModeBase` by right clicking on it in the `Content Drawer` under the `C++` folder and selecting `Create Blueprint Class Based On GUIGameModeBase`. Name it `BP_GameMode`.
2. Double click on it and open the `Class Defaults` pane. Set the `Health > HUDWidgetClass` to be our `HealthHUD`.
3. In the main editor window, choose `Edit > Project Settings > Maps and Modes > Default Modes > Default GameMode` and set it to be `BP_GameMode` using the associated dropdown menu.
4. Remember to `Build [ProjectName]` in VS if you haven’t done so in a while. Return to Unreal, `Compile`, and then run Play. You should see a health bar counting down!

You could tie this health bar element to all sorts of things (it was the base for the battery mechanic in Slapsticklers) through code. This editor is the one you'll be using for UI elements- remember that! 

## Well done!

Congrats on completing tutorial 4! If you got here, send me...a frog :)
