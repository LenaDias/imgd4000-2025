## Tutorial 2: Player movement and input
Goal: Create a very small world with a floating mesh that represents the player using an `SpherePawn` subclass. Configure a `PlayerController`, `SphereController`, to use WASD keys for movement using C++ and the `EnhancedInputSubsystem`. Assign spacebar to “Fire” other instances of a separate `Projectile` class. Use a `UPROPERTY` value to configure speed of projectiles. Connect to Git!

Basically; let's make a player character that can move around and add player input to move it!

### Visual Studio setup

Use [these instructions](https://dev.epicgames.com/documentation/en-us/unreal-engine/setting-up-visual-studio-development-environment-for-cplusplus-projects-in-unreal-engine) to configure Visual Studio for Unreal. If you're using a lab computer for this and get an error on startup, talk to me.


### Creating the Project / Level
We’re going to spend another day starting “from scratch” in Unreal. Although you will probably start your final project using a template, I’m hopeful that going through this process will help give you a better understanding of how everything is tied together.

1. Create a new blank game project that uses C++ and includes the starter content pack.
2. Create a new Basic level.
3. From the Place Actors panel (Open via `Window > Place Actors` if you don't see it), drag a `Basic > Plane` mesh into your scene. Increase its X & Y scale to 20. Add a material of your choosing to it (I’ll use “Chrome”)
4. Drag a Lights > Directional Light into the scene.
5. Remember how that breaks? Adjust your DirectionalLight's Forward Shading Priority. Change it to 1 to make your own DirectionalLight the most important one in the scene for determining certain kinds of shadows. Alternatively just delete the existing DirecionalLight already in the Outliner.
6. We should already have one, but if you don't, drag a `Visual Effects > Sky Atmosphere` into the scene

OK, you should now have a lit scene ready to go… next thing to do is to map some inputs.


### Setup for EnhancedInputSubsystem: InputMappingContext and InputAction

Now, we need to create a couple of assets for managing player input. 

1. In the Content Drawer, right click and create an `Input > Input Mapping Context` and name it `IMC`.

2. Double click on the IMC to open its editor window.

This is an `InputMappingContext`, and it lets you map player inputs. For example; say you have a game where the player can Run, Drive, and Swim. As your player transitions between these different "modes," their control scheme might change. *This* is what an InputMappingContext is useful for; grouping and mapping player inputs. 

Using an `InputMappingContext` will also allow us to use the `EnhancedInputSubsystem`, Unreal's new way of managing user input. This saves us all sorts of effort, like programming input mode-switching manually.

We don't need this level of functionality for our tutorial- we could just use the deprecated input system- but it's useful to know how to make an `InputMappingContext` in case you have complex input.

3. Press the + button next to `Mappings`

4. Tap the `Input Action` dropdown, and press `Create New Asset > Input Action`. Name it `Move`

This is an `Input Action`. It's a `Data Asset` that represents an abstract action that your player might take in the game, like moving, swimming, jumping, firing...etc.

5. In the main Unreal Editor window, find your new InputAction asset and double-click it.

In this window, we can associate a player action with a value that can be passed to code. Under `Value Type`, change the dropdown to `Axis2D(Vector2D)`, since our movement will be 2-dimensional. 

6. Make a few more control mappings for S, A, and D. You can click on the keyboard to type it in. 
![{30738A73-B508-43FA-85A6-A30499F642FC}](https://github.com/user-attachments/assets/9457b52e-f3f0-41c1-8585-3893ec458f38)

7. Now, give each of these a `Trigger`, which determines when the input event fires. Set each trigger to `Down`.

8. Let's add some modifiers. Give `S` and `A` a `Negate` modifier. The `Negate` modifier will change the Axis2D(Vector2D) we get in the code when the button is pressed to a negative value . 

9. We'll also add a `Swizzle Input Axis Values` modifier to the `W` and `S` buttons. Give them an `Order` of `YXZ`.

We need to do this since, by default, the `EnhancedInputSubsystem` fills input values into the `Y`, `X`, and `Z` of the `Axis2D(Vector2D)` in that order; first Y, then X, then Z. With no modifiers, if `W` is held, our `MoveAction` will take our current `Axis2D(Vector2D)`...
`X:0, Y:0, Z:0`
...and fill it in with a 1 from our `Axis2D(Vector2D)` as...
`X:1, Y:0, Z:0`

But we're trying to move the player forward/back along the Y axis, not the X axis! With `Swizzle Input Axis Values` set to `YXZ`, we swap the order values are filled in along `X` and `Y`. This makes it so that when the `W` key is pressed, our `Axis2D(Vector2D)` is assembled as:
`X:0, Y:1, Z:0`

`A` and `D` control us along the X axis; they don't need a swizzle.

All in all, your IMC editor should look like this:
![{09BB6102-B0D9-45BB-ACFD-F8AB4A29EE50}](https://github.com/user-attachments/assets/6995605c-64e3-45c6-a0bd-627c0849c8c5)
![{FA91E4A5-AD1B-48B1-AEC6-EEA652896D1A}](https://github.com/user-attachments/assets/cd6ce50e-fbc9-43f2-b98f-fa109776c5d2)

10. Save and exit these editor windows.


### Creating our player Pawn: header

In Unreal, the `Pawn` class is a specialized type of `Actor` that is generally used to represent the player and any NPCs / AI in the game world. It can be "possessed" by the player and controlled with input.

Unreal also has the `PlayerController` class. As the [Unreal documentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/player-controllers-in-unreal-engine) describes it, "the `PlayerController` essentially represents the human player's will." Usually, it's used for more abstract things, like receiving and processing player input. 

Generally, it's probably a better idea to do input binding in a `PlayerController`, and leave the physical representation of the game world to a player `Pawn`. Consider segmenting functionality in each based on your own game's needs- again, see this [Unreal documentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/player-controllers-in-unreal-engine).

So, let's start by making our representation of the player in the game world; the player `Pawn`:

1. Choose `Tools > New C++ class`, and then select `Pawn`. Name the class "SpherePawn" and make it public, so other classes can see its member functions.
2. Visual Studio 2022 should open, with two files in the window; `SpherePawn.cpp`, and `SpherePawn.h`.
3. It's possible your Error List is on screen flagging hundreds of errors. These are false flags caused by Intellisense- change the 'Show issues generated' dropdown from 'Build+Intellisense' to 'Build Only'. 
![{CD9011F6-34E5-483F-8779-39541E286397}](https://github.com/user-attachments/assets/0ed29c47-ca45-4f10-b50a-4eb464615cec)
4. Let's start in SpherePawn.h. Add a `UPROPERTY` named `Mesh` that will hold the mesh representing our player. Make the property public in the new pawns header file:
```c++
public:
	ASpherePawn();

	virtual void Tick(float DeltaTime) override;
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

	//U prefix denotes a class
	UPROPERTY(EditAnywhere) class UStaticMeshComponent* Mesh;
```
By using the UPROPERTY macro decorator we’ll be able to define this value from within the UE5 editor; it also ensures the mesh will be added to UE5's memory management. 

`EditAnywhere` allows you to edit the property from Unreal Editor; you can do this, for example, in the `Details` window of an instance of our `SpherePawn` class. That'd be useful, for instance, if you wanted to drop 3 different-colored copies of an enemy in the game world; you could simply have a `UPROPERTY` that stores the color in the C++, and then use the `Details` pane to change their color with a dropdown in-Editor. It's also helpful to designers and artists to not have to edit the code.

You can [heavily customize how each UPROPERTY can be edited](https://romeroblueprints.blogspot.com/2020/10/the-uproperty-macro.html).

If you are receiving an MSB3073 code 8 or Visual Studio is screaming at you, you may have the wrong class declaration. If `TEST_API` is present in your header files as part of the class declaration, you might have copy-pasted from my code and it is creating a conflict. This code is auto-generated after you make a new C++ class in Unreal Editor and will differ based on your project name. For example, if your project is named playerMovementInput, your `SpherePawn` class header should look like...
```c++
UCLASS()
class PLAYERMOVEMENTINPUT_API ASpherePawn : public APawn
```

5. We'll also add a protected mmember component that adds movement to our `SpherePawn`. 
```c++
protected:

	virtual void BeginPlay() override;
	UFloatingPawnMovement* PawnMovement;

```
6. To get that last step to resolve, we need either some `include`s or a *forward-declaration*. Either way will get the compiler the assurance it needs that the classes you are referencing do indeed exist. Forward-declarations are a little more C++ linker-bullshit resistant, from my understanding, and will thus save you some debugging when you can use them. Your `SpherePawn.h` should now look like this:
```c++
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Pawn.h"
#include <GameFramework/FloatingPawnMovement.h>
#include "SpherePawn.generated.h"

struct FInputActionValue;

UCLASS()
class TEST_API ASpherePawn : public APawn
{
	GENERATED_BODY()

public:

	ASpherePawn();
	virtual void Tick(float DeltaTime) override;
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

	//U prefix denotes a class
	UPROPERTY(EditAnywhere) class UStaticMeshComponent* Mesh;

	void Move(const FInputActionValue& InputActionValue);

protected:

	virtual void BeginPlay() override;
	UFloatingPawnMovement* PawnMovement;

};

```

7. A note about compilation; hitting the `Recompile` button in the bottom right of Unreal Editor only hot reloads your C++ code into Unreal; it does not save it to the solution. For this reason. I recommend you `Build > Compile` in Visual Studio at least once every 30 minutes. Hit `CTRL` + `F5` to open your unreal project after doing so, if you need to. 

That's all we need in the header for now. Remember to save.


### Creating our player Pawn: implementation:

Now, the implementation for our `SpherePawn`, in `SpherePawn.cpp`.

1.We _forward-declared_ our `FInputActionValue` in our header, which means we need to include it in our `.cpp` file _below_ its `SpherePawn` include:
```c++
#include "SpherePawn.h"
#include "InputActionValue.h"
```

2.  The first thing we need here is the implementation for our `Move` method. This method will be called by our `PlayerController` (when we make it), and handles movement of our `SpherePawn`. Most of these lines are self-explanatory.
```c++
void ASpherePawn::Move(const FInputActionValue& InputActionValue)
{
	
	const FVector2D InputAxisVector = InputActionValue.Get<FVector2D>();

	//Get control rotation
	const FRotator ControlRotation = GetControlRotation();
	
	//Isolate the yaw
	const FRotator YawRotation(0, ControlRotation.Yaw, 0);
	//Get forward vector	
	const FVector ForwardDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);
	//Get right vector
	const FVector RightDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);
	


	//Move the character
	if (Mesh)
	{
		AddMovementInput(ForwardDirection, InputAxisVector.Y);
		AddMovementInput(RightDirection, InputAxisVector.X);
	}
}
```

3. And in our constructor, instantiate our `PawnMovement` and `Mesh` components:
```c++
// Sets default values
ASpherePawn::ASpherePawn()
{
 	// Set this pawn to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;
	PawnMovement = CreateDefaultSubobject<UFloatingPawnMovement>("PawnMovement");
	Mesh = CreateDefaultSubobject<UStaticMeshComponent>("Mesh");

	static ConstructorHelpers::FObjectFinder<UStaticMesh>SphereMeshAsset(TEXT("StaticMesh'/Engine/BasicShapes/Sphere.Sphere'"));
	Mesh->SetStaticMesh(SphereMeshAsset.Object);
	
	RootComponent = SphereMesh;

}
```	

4. All in all, our `SpherePawn.cpp` should look like this:
```c++
// Fill out your copyright notice in the Description page of Project Settings.

#include "SpherePawn.h"
#include "GameFramework/FloatingPawnMovement.h"
#include "InputActionValue.h"

// Sets default values
ASpherePawn::ASpherePawn()
{
 	// Set this pawn to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;
	PawnMovement = CreateDefaultSubobject<UFloatingPawnMovement>("PawnMovement");
	Mesh = CreateDefaultSubobject<UStaticMeshComponent>("Mesh");

}

// Called when the game starts or when spawned
void ASpherePawn::BeginPlay()
{
	Super::BeginPlay();
	
}

// Called every frame
void ASpherePawn::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

// Called to bind functionality to input
void ASpherePawn::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

}

void ASpherePawn::Move(const FInputActionValue& InputActionValue)
{
	
	const FVector2D InputAxisVector = InputActionValue.Get<FVector2D>();

	//Get control rotation
	const FRotator ControlRotation = GetControlRotation();
	
	//Isolate the yaw
	const FRotator YawRotation(0, ControlRotation.Yaw, 0);
	//Get forward vector	
	const FVector ForwardDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);
	//Get right vector
	const FVector RightDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);
	


	//Move the character
	GEngine->AddOnScreenDebugMessage(0, 3.0f, FColor::Green, FString(Mesh->GetName()));
	if (Mesh)
	{
		AddMovementInput(ForwardDirection, InputAxisVector.Y);
		AddMovementInput(RightDirection, InputAxisVector.X);
	}
}
```



### Creating Our PlayerController subclass: header

Now, We’ll create a subclass of `PlayerController` and setup movement control via the WASD keys. Remember: the `PlayerController` just handles player control (like inputs).

1. Choose `Tools > New C++ class`, and then select `PlayerController`. Name the class "SphereController" and make it public, so other classes can see its member functions.
2. Visual Studio 2022 should open, with two files in the window; `SphereController.cpp`, and `SphereController.h`.
3. Now, we're gonna break from the old tutorial and use the Enhanced Input System. If in Unreal you go to `Edit > Project Settings > Engine > Input`, you can see (and use) the old input system. It's deprecated, so we're using the Enhanced Input System for this.

2. Now, in `SphereController.h`, add some forward declarations and includes for EnhancedInput functionality. The order of includes is important here; they must be above the `SphereController.generated.h` include:
```c++
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/PlayerController.h"
#include "EnhancedInputSubsystems.h"
#include "SphereController.generated.h"

class UEnhancedInputComponent;
class UInputMappingContext;
class UInputAction;
struct FInputActionValue;

/**
 * 
 */
UCLASS()
class TEST_API ASphereController : public APlayerController...
```

3. Next, we want to change the `BeginPlay()` function that comes standard with the `PlayerController` class. Add it as a protected method to the header file with an `override` keyword so C++ knows we're overriding default functionality:
```c++
UCLASS()
class TUT2_ENHANCEDINPUT_API ASphereController : public APlayerController
{
	GENERATED_BODY()

protected:
	virtual void BeginPlay() override;

};
```

4. Add a custom function of our own called `SetupInputComponent`:
```c++
private:
	void Move(const FInputActionValue& InputActionValue);
	void SetupInputComponent();
```	

5. Also in `SphereController.h`, add a protected `MoveAction` pointer. Give it a `UPROPERTY` decorator:

The `UPROPERTY` decorator will allow it to be editable in its `Details` pane in Unreal Editor, like the `Material` dropdown we used in the first tutorial.

```c++
protected:
	virtual void BeginPlay() override;

	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Inputs")
	UInputAction* MoveAction;


```
This `UPROPERTY` is where we will pass in our `InputMappingContext` from a Blueprint....when the time comes. We set it to `EditDefaultsOnly`, so only the *default* value of this `MoveAction` property can be edited from the `Details` window. That's useful if you want every instance of the C++ class to have a default `Move` action associated with it.

The `U` denotes that whatever we get as a parameter needs to be a subclass of an exiting Unreal class.

You may have also noticed that there are two kinds of pointers here: a standard `*` C++ pointer and an Unreal-specific `TObjectPtr` (the `T` stands for template). Both pointers  work, but basically, [Unreal prefers the TObjectPtr](https://forums.unrealengine.com/t/why-should-i-replace-raw-pointers-with-tobjectptr/232781/3).


6. Also add a `UPROPERTY`'d pointer to a `UInputMappingContext'. 
```c++
protected:
	virtual void BeginPlay() override;

	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Inputs")
	UInputAction* MoveAction;

	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Inputs")
	UInputMappingContext* InputMappingContext;
```
7. Next, we need another private method. We're gonna make one called `Move` that gets bound to when the player makes a "Move" input. 

It takes a reference to the Struct (that's the `F`) `FInputActionValue` we'll be creating. The reference makes sure it's non-null.
```c++
private:
	void Move(const FInputActionValue& InputActionValue);
```

8. All in all, our `SphereController.h` should look like this:
```c++
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/PlayerController.h"
#include "EnhancedInputSubsystems.h"
#include "SphereController.generated.h"

class UEnhancedInputComponent;
class UInputMappingContext;
class UInputAction;
struct FInputActionValue;

/**
 * 
 */
UCLASS()
class TEST_API ASphereController : public APlayerController
{
	GENERATED_BODY()

protected:
	virtual void BeginPlay() override;

	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Inputs")
	UInputAction* MoveAction;

	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Inputs")
	UInputMappingContext* InputMappingContext;

private:
	void Move(const FInputActionValue& InputActionValue);
	void SetupInputComponent();
	
};

```

Nice! Let's give our `SphereController` an implementation.


### Creating Our PlayerController subclass: Implementation

Now, let's move on to the `SphereController.cpp` implementation file. But before you go there:
1. Right click on one of your methods you declared in `SphereController.h`. Select `Quick Actions and Refactorings... > Create declaration/definition`. This should save you the trouble of copying over the method signature.
   
2. SAVE your Unreal Editor project. There's a chance this code crashes Unreal.
   
3. Let's start working on our `BeginPlay()` function, which runs whenever the game starts.

We're gonna ensure that the prerequisite `Data Asset`s for `EnhancedInput` are present with some `checkf` statements. 

Then, we find the EnhancedInput subsystem and connect it to the `InputMappingContext` we made earlier:
```c++
void ASphereController::BeginPlay()
{
	//Call the superclass method for BeginPlay() so Unreal knows to include this as part of the list of objects doing stuff
	Super::BeginPlay();

	//checkf makes sure the IMC and MoveAction are set in the Details pane in Unreal Editor. If not, it'll (intentionally) crash Unreal at this line.
	checkf(InputMappingContext, TEXT("InputMappingContext is not set in %s"), *GetNameSafe(this));
	checkf(MoveAction, TEXT("MoveAction is not set in %s"), *GetNameSafe(this));

	//Get a pointer to the EnhancedInputSubsystem
	TObjectPtr<UEnhancedInputLocalPlayerSubsystem> EnhancedInputSubsystem = GetLocalPlayer()->GetSubsystem<UEnhancedInputLocalPlayerSubsystem>();
	//Add the IMC to the EnhancedInputSubsystem
	if (EnhancedInputSubsystem) {
		EnhancedInputSubsystem->AddMappingContext(InputMappingContext, 0);
	}

}
```
4. Our next function is `ASphereController::SetupInputComponent()`. This one will connect the `EnhancedInputComponent`, which enables `EnhancedInputSubsystem` functionality in our `PlayerController`. We use a `CastChecked` to ensure there's not a null pointer being returned.

Additionally, we need to bind the `MoveAction` Data Asset we made to `ASphereController::Move`, so that function gets called whenever the `MoveAction` is `Triggered`:
```c++
//Set up player input
void ATestPlayerController::SetupInputComponent()
{
	//Access super class to connect input
	Super::SetupInputComponent();

	//Get a pointer to the EnhancedInputComponent our PlayerController now has
	UEnhancedInputComponent* EnhancedInputComponent = CastChecked<UEnhancedInputComponent>(InputComponent);

	//Bind the MoveAction to the Move method
	EnhancedInputComponent->BindAction(MoveAction, ETriggerEvent::Triggered, this, &ASphereController::Move);

}
```

5. One last function. This one simply transfers the responsibility of moving the player `SpherePawn` to the instance of our `SpherePawn`.
```c++
//Passes input to the SpherePawn so that it moves itself
void ASphereController::Move(const FInputActionValue& InputActionValue)
{
	if (ASpherePawn* ControlPawn = GetPawn<ASpherePawn>())
	{
		//GEngine->AddOnScreenDebugMessage(0, 3.0f, FColor::Green, FString(ControlPawn->GetName()));
		ControlPawn->Move(InputActionValue);
	}
}
```

6. And don't forget to add some includes:
```c++
#include "SphereController.h"
#include <SpherePawn.h>
#include "EnhancedInputComponent.h"
```

7. Your `SphereController.cpp` should look like this:
```c++
// Fill out your copyright notice in the Description page of Project Settings.


#include "SphereController.h"
#include <SpherePawn.h>
#include "EnhancedInputComponent.h"

void ASphereController::BeginPlay()
{
	//Call the superclass method for BeginPlay() so Unreal knows to include this as part of the list of objects doing stuff
	Super::BeginPlay();

	//checkf makes sure the IMC and MoveAction are set in the Details pane in Unreal Editor. If not, it'll (intentionally) crash Unreal at this line.
	checkf(InputMappingContext, TEXT("InputMappingContext is not set in %s"), *GetNameSafe(this));
	checkf(MoveAction, TEXT("MoveAction is not set in %s"), *GetNameSafe(this));

	//Get a pointer to the EnhancedInputSubsystem
	TObjectPtr<UEnhancedInputLocalPlayerSubsystem> EnhancedInputSubsystem = GetLocalPlayer()->GetSubsystem<UEnhancedInputLocalPlayerSubsystem>();
	//Add the IMC to the EnhancedInputSubsystem
	if (EnhancedInputSubsystem) {
		EnhancedInputSubsystem->AddMappingContext(InputMappingContext, 0);
	}

}


//Set up player input
void ASphereController::SetupInputComponent()
{
	GEngine->AddOnScreenDebugMessage(0, 3.0f, FColor::Green, FString(TEXT("SphereController SetupInputComponent")));
	//Access super class to connect input
	Super::SetupInputComponent();

	//Get a pointer to the EnhancedInputComponent our PlayerController now has
	UEnhancedInputComponent* EnhancedInputComponent = CastChecked<UEnhancedInputComponent>(InputComponent);

	//Bind the MoveAction to the Move method
	EnhancedInputComponent->BindAction(MoveAction, ETriggerEvent::Triggered, this, &ASphereController::Move);
}

void ASphereController::Move(const FInputActionValue& InputActionValue)
{
	GEngine->AddOnScreenDebugMessage(0, 3.0f, FColor::Green, FString(TEXT("SphereConntroller move")));
	if (ASpherePawn* ControlPawn = GetPawn<ASpherePawn>())
	{
		GEngine->AddOnScreenDebugMessage(0, 3.0f, FColor::Green, FString(ControlPawn->GetName()));
		ControlPawn->Move(InputActionValue);
	}
}

```

### Make a blueprint from your SphereController.cpp

Phew! That's all the code for basic input and some movement. There are a few things we need to do before getting our player to actually spawn in the world in Unreal Editor.

Now; surprise, we're actually gonna use Blueprints. We need them to set the default `UPROPERTY` values that we set up earlier in our `SphereController`.

1. In the Content Drawer, right click on your C++ Class for `SphereController`.
2. Select `Create Blueprint class based on SphereController`. Name it `BP_SphereController`.
3. Open your Blueprint. This Blueprint is a subclass of the C++ `SphereController` class that you made, and Unreal Engine knows it. Thus, any changes to your C++ file will also automatically perpetuate to this Blueprint through C++ inheritance.
4. In the `Details` pane, set the class defaults for our `MoveAction` and `InputMappingContext`:
![{F9754086-AACA-427F-9F61-D42F7AA12D2D}](https://github.com/user-attachments/assets/3a2d5dbd-6bff-41cb-becd-9c5db762986f)
5. Save, compile, and exit this editor window.


### Configuring the GameMode 

1. Make a new `GameModeBase` as below and name it `SphereGameMode`. The `GameMode` determines what scenario the game is being run in, including its rules; it also spawns the player character.
![{4034D6ED-6EA6-45D9-80B0-F2E6B878D16E}](https://github.com/user-attachments/assets/d4ac1a93-9093-4043-b58b-d323fd92aa98)

2. Edit your new `GameMode` so that the `Player Controller Class` is `BP_SphereController` (not `SphereController`, since we want the subclass with the defaults we set).
3. Set the `Default Pawn Class` to `SpherePawn`. This will make the game spawn our custom pawn with our custom controller.

![{33BFFAE8-A0E0-4A2C-A74F-DA84BF044C8A}](https://github.com/user-attachments/assets/0304695e-fc24-452f-8860-2f52597840cb)

(My `SpherePawn` is called `MyPawn` here)


### Try it out!!!
Now, for the moment of truth...

1. Hit Play in Unreal Editor. You should see two `Actor`s get added to the `Outliner` window; `BP_SphereController_0` and `SpherePawn_0`. This means they are successfully being created as part of the game world!
2. Try using the WASD keys to move around!

Yay! You did it. Take a break here if you haven't already. And be sure to save your solution in Visual Studio, not Unreal Editor.


### Creating our player camera
We need to create a camera that will follow our player at a certain offset in order to be able to see the player. 
1. Add our camera as a protected member to the header of our `SpherePawn` class.
```c++
UPROPERTY(EditAnywhere)
class UCameraComponent * Camera;
```
2. We _forward-declared_ our `UCameraComponent` in our header, which means we need to include it in our `.cpp` file _below_ its `SphereController` include:
```c++
#include "Camera/CameraComponent.h"
```
3. Add our camera instantiation / setup the `SpherePawn` constructor:
```c++
ASpherePawn::ASpherePawn(){
	PrimaryActorTick.bCanEverTick = true;
	PawnMovement = CreateDefaultSubobject<UFloatingPawnMovement>("PawnMovement");

	Mesh = CreateDefaultSubobject<UStaticMeshComponent>("Mesh");
	Camera = CreateDefaultSubobject<UCameraComponent>("CameraComponent");

	Camera->SetRelativeLocation(FVector(-500.f,0.f,0.f));
	Camera->SetupAttachment(Mesh);
}
```
This assigns an offset (which can be change in the UE5 editor under the Camera details of `SphereController`) and tells the camera to follow our mesh.
4. Now, we need to make sure that the default Mesh is actually getting fed to the SpherePawn on creation. You can modify your constructor function in SpherePawn by following the answer [here](https://stackoverflow.com/questions/61965061/how-to-add-the-value-of-a-static-mesh-component-from-c-instead-of-the-unreal-e). You can also make a Blueprint subclass of your C++ and change the class defaults there.
5. Go ahead and hit Compile (bottom right of Unreal Editor) and then Play in UE5:
![{7A39FD7F-E5E8-4F96-98C6-01BD300F7BE5}](https://github.com/user-attachments/assets/a07b2973-f232-4076-b260-f34daedc16ea)

You can also hit `Build > Compile` in Visual Studio after closing Unreal.

You should see that the camera follows the mesh now.
![{0B04337A-B5DA-455B-AFE3-2079956F0417}](https://github.com/user-attachments/assets/570285f6-f72b-4f12-956e-296875df6a28)


### Connecting to Git

Finally, let's do some version control. Use GitHub Desktop to designate where your game is stored as a Git repository. Then, follow the steps here under "Configure Unreal Engine": https://www.anchorpoint.app/blog/git-with-unreal-engine-5

And yay, we're done! If you read this, send me a picture of a cute bunny on Discord and celebrate.


### Useful troubleshooting resources
Wow, that was a lot! Here are some useful tips if you get stuck.

Try to save your work every 30 minutes or less. Unreal has a habit of crashing.

If you get a `The Project 'ProjName' has been modified outside the environment.` just hit `Reload All`. This is Visual Studio detecting that your solution changed and that it needs to load the new stuff.

Here is what a successful VS compile (Build > Compile) looks like. You may need to use this if Unreal Editor will not open due to uncompiled C++ code and asks you to build in your IDE.
![{A5C5F578-D22E-4E43-8B4F-7AD5C97E0237}](https://github.com/user-attachments/assets/0bfa0a54-5f71-4d98-92c6-f7ffd925f312)

Disabling Live Coding sometimes helps with compilation problems, especially MSB3073 code 6. Go to Edit > Editor Preferences > General > Live Coding > untick "Enable Live Coding"

Error MSB3073 code 8? You may have the wrong class declaration. If `TEST_API` is present in your header files as part of the class declaration, you might have copy-pasted from my code and it is creating a conflict. This code is auto-generated after you make a new C++ class in Unreal Editor and will differ based on your project name. For example, if your project is named playerMovementInput, your `SphereController` class header should look like...
```c++
UCLASS()
class PLAYERMOVEMENTINPUT_API ASphereController : public APlayerController...
```

Getting weird UClass errors? Make sure your includes are in the right order. https://www.reddit.com/r/unrealengine/comments/6lrryt/uclass_declaration_has_no_storage_class_or_type/ 
