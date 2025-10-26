+++
title = "Creational Design Patterns in Golang"
date = 2025-10-26T08:00:19+05:30
draft = false
showToc = true
cover.image = 'images/creational_design.jpg'
tags = ['Golang', 'Tech']
+++

This is one the first blog in the design pattern series, and in this one, I'll be focusing on the creational design patterns. Creational Design pattern is one of the most intuitive design pattern if you ask me, dealing with the fundamental problem of _how do we create objects the right way_ .

## But what is Creational Design Pattern and how does it help me with my work?

Creational design pattern is basically an encapsulation hidden from the end user, hiding the implementation details, since for the user, they're only concerned with getting their object instance. This branch of design pattern try to give the user ready-to-user objects instead of asking for their creation, which could be complex in cases when the application gets bigger and complex.

The more you use these design patterns, you'll get to observe they follow two basic themes:

1. All of them encapsulate the knowledge around which classes(structs in the case of golang) gets used.
2. They hide how the classes are created and put together, all the end users know is the interfaces they have to use to get the job done.

## That's cool and all but what's the point? Why can't we just _create_ object as we like?

Imagine this scenario:
You're building a first person shooter game, with lots of characters, weapon designs, in-game objects. Each instance of these objects would be different than the other. Let's say you write the object creation code each time you want it, i.e. duplicate the code a lot of times, you'll get screwed pretty easily if you decide to change the creation logic at a later point of your development cycle.

This is where the Creational Design patterns come in. They _streamline_ and _centralise_ the object creations for you! so you don't have to worry about implementation details when creating your application objects.

## Factory Design Pattern

Factory pattern is one of the most known design patterns. The purpose of this pattern is to abstract the user from the knowledge of the struct they need to achieve a specific purpose, such as retrieving some value, maybe from a web service or a database.

Let's take the example of Hotwheels, i.e. a die-cast car manufacturing company. If you're not aware, whenever Hotwheels designers come up with a new design, they don't just build it the same for all the units. Instead, using the design as the base model, they make multiple variations of that model — with different colors, details, wheels, and sometimes even special editions.

Now, not every car that rolls off the line is the same. Some are regular releases, some are Treasure Hunts (rare finds with special paint jobs), and a few are Super Treasure Hunts — the holy grail for collectors, featuring premium wheels and even more detailed finishes.

And for that, they can't just create brand-new designs for every variation, because that would be redundant.
This is where the Factory Design Pattern comes in handy.

Let’s look at how the code would look without this design pattern and then with it:

```go
type Car struct {
    Model string
    Variant string
    Color string
}

func main() {
    car1 := Car{Model: "Mustang", Variant: "Regular", Color: "Red"}
    car2 := Car{Model: "Mustang", Variant: "Treasure Hunt", Color: "Gold"}
    car3 := Car{Model: "Mustang", Variant: "Super Treasure Hunt", Color: "Chrome"}

    fmt.Println(car1, car2, car3)
}
```

Now, as you can imagine, this gets messy real fast. Every time you introduce a new variant or a model, you'd have to rewrite the creation logic all over again.

Here's how the same idea looks like using Creational Design Pattern:

```go
package main

import "fmt"

type Car interface {
	Name() string
	Details() string
}

type RegularCar struct{ Model, Color string }

func (c RegularCar) Name() string   { return "Regular Edition" }
func (c RegularCar) Details() string {
	return fmt.Sprintf("%s %s - basic design, mass produced", c.Color, c.Model)
}

type TreasureHuntCar struct{ Model, Color string }

func (c TreasureHuntCar) Name() string   { return "Treasure Hunt" }
func (c TreasureHuntCar) Details() string {
	return fmt.Sprintf("%s %s - rare find with special paint and hidden logo", c.Color, c.Model)
}

type SuperTreasureHuntCar struct{ Model, Color string }

func (c SuperTreasureHuntCar) Name() string   { return "Super Treasure Hunt" }
func (c SuperTreasureHuntCar) Details() string {
	return fmt.Sprintf("%s %s - ultra-rare with rubber tires and metallic finish", c.Color, c.Model)
}

type HotWheelsFactory struct{}

func (HotWheelsFactory) CreateCar(variant, model, color string) Car {
	switch variant {
	case "Regular":
		return RegularCar{Model: model, Color: color}
	case "Treasure Hunt":
		return TreasureHuntCar{Model: model, Color: color}
	case "Super Treasure Hunt":
		return SuperTreasureHuntCar{Model: model, Color: color}
	default:
		return nil
	}
}

func main() {
	factory := HotWheelsFactory{}

	cars := []Car{
		factory.CreateCar("Regular", "Mustang", "Red"),
		factory.CreateCar("Treasure Hunt", "Corvette", "Gold"),
		factory.CreateCar("Super Treasure Hunt", "Camaro", "Chrome"),
	}

	for _, car := range cars {
		fmt.Printf("[%s] → %s\n", car.Name(), car.Details())
	}
}
```

### Okay but how is this different than manually creating the cars?

Let’s say you’re working on a large Hot Wheels catalog system, an API, or even a simulation game.
You’re not just creating a handful of cars anymore — you’re creating hundreds, each with their own attributes and variations.

If you create cars manually everywhere, you’ll start to see problems like:

1. Repetitive code – you’d be repeating the same “creation” logic all over.

2. Tight coupling – your main code now depends on which struct to create and how it’s initialized.

3. No central control – if tomorrow, “Super Treasure Hunt” cars require an extra property (say, a serialNumber or rarityScore), you’d have to update every place where they’re created.

In short,

| Without Factory                                       | With Factory                                      |
| ----------------------------------------------------- | ------------------------------------------------- |
| You manually instantiate structs across the codebase. | All creation logic is encapsulated in one place.  |
| Every new variant requires code changes everywhere.   | New variants require updates only in the factory. |
| Code is harder to maintain and test.                  | Code is modular, testable, and easy to extend.    |

## Abstract Factory

If the **Factory Pattern** is like having a single workstation that can produce one specific type of car (say **Regular**, **Treasure Hunt**, or **Super Treasure Hunt**),  
then the **Abstract Factory Pattern** is like owning an **entire production unit** — where each workstation (factory) specializes in producing a _family_ of related cars.

It’s one level higher.  
Instead of asking _“make me a car of type X”_, you say _“give me a factory that makes Hot Wheels cars”_ — and that factory itself knows how to produce **Regular**, **Treasure Hunt**, and **Super Treasure Hunt** variants.

### Extending Our Hot Wheels Factory

Imagine Hot Wheels has two main product lines:

1. **Basic Series** — your regular mainline cars found in most stores, basically toys, meant for children.
2. **Premium Series** — high-end cars with metal bodies, rubber tires, and collector packaging, for serious collectors.

Each series can still have **Regular**, **Treasure Hunt**, and **Super Treasure Hunt** cars —  
but the difference lies in the _series-level behavior_ — how they’re **built**, **packaged**, and **presented**.

### Why Abstract Factory Pattern?

This is where the **Abstract Factory Pattern** shines:  
it lets you create entire _families of related objects_ (like **cars** and their **packages**) without specifying their concrete classes.

In our case:

- The **Basic Series Factory** knows how to build and package cars for the mainline series.
- The **Premium Series Factory** knows how to build and package high-end collector editions.

You just pick the factory — it handles the rest.

Let's look at the code example for this :

```go
package main

import "fmt"


type Car interface {
	Name() string
	Details() string
}

type Packaging interface {
	BoxType() string
}


type BasicRegularCar struct{ Model string }
type BasicTreasureHuntCar struct{ Model string }
type BasicSuperTreasureHuntCar struct{ Model string }

func (c BasicRegularCar) Name() string          { return "Regular" }
func (c BasicRegularCar) Details() string       { return fmt.Sprintf("%s - Standard mainline car", c.Model) }
func (c BasicTreasureHuntCar) Name() string     { return "Treasure Hunt" }
func (c BasicTreasureHuntCar) Details() string  { return fmt.Sprintf("%s - Hidden logo, rare paint", c.Model) }
func (c BasicSuperTreasureHuntCar) Name() string { return "Super Treasure Hunt" }
func (c BasicSuperTreasureHuntCar) Details() string {
	return fmt.Sprintf("%s - Metallic finish, rubber tires", c.Model)
}

type BasicPackaging struct{}

func (BasicPackaging) BoxType() string { return "Standard blister pack" }


type PremiumRegularCar struct{ Model string }
type PremiumTreasureHuntCar struct{ Model string }
type PremiumSuperTreasureHuntCar struct{ Model string }

func (c PremiumRegularCar) Name() string          { return "Regular (Premium)" }
func (c PremiumRegularCar) Details() string       { return fmt.Sprintf("%s - Metal body, glossy paint", c.Model) }
func (c PremiumTreasureHuntCar) Name() string     { return "Treasure Hunt (Premium)" }
func (c PremiumTreasureHuntCar) Details() string  { return fmt.Sprintf("%s - Collector decals and detailing", c.Model) }
func (c PremiumSuperTreasureHuntCar) Name() string { return "Super Treasure Hunt (Premium)" }
func (c PremiumSuperTreasureHuntCar) Details() string {
	return fmt.Sprintf("%s - Elite collectible, limited edition packaging", c.Model)
}

type PremiumPackaging struct{}

func (PremiumPackaging) BoxType() string { return "Collector box with display case" }


type HotWheelsFactory interface {
	CreateRegular(model string) Car
	CreateTreasureHunt(model string) Car
	CreateSuperTreasureHunt(model string) Car
	CreatePackaging() Packaging
}


type BasicSeriesFactory struct{}

func (BasicSeriesFactory) CreateRegular(model string) Car           { return BasicRegularCar{Model: model} }
func (BasicSeriesFactory) CreateTreasureHunt(model string) Car      { return BasicTreasureHuntCar{Model: model} }
func (BasicSeriesFactory) CreateSuperTreasureHunt(model string) Car { return BasicSuperTreasureHuntCar{Model: model} }
func (BasicSeriesFactory) CreatePackaging() Packaging               { return BasicPackaging{} }

type PremiumSeriesFactory struct{}

func (PremiumSeriesFactory) CreateRegular(model string) Car           { return PremiumRegularCar{Model: model} }
func (PremiumSeriesFactory) CreateTreasureHunt(model string) Car      { return PremiumTreasureHuntCar{Model: model} }
func (PremiumSeriesFactory) CreateSuperTreasureHunt(model string) Car { return PremiumSuperTreasureHuntCar{Model: model} }
func (PremiumSeriesFactory) CreatePackaging() Packaging               { return PremiumPackaging{} }


func main() {
	var factory HotWheelsFactory

	factory = BasicSeriesFactory{}
	displaySet(factory, "Mustang")

	fmt.Println()
	factory = PremiumSeriesFactory{}
	displaySet(factory, "Mustang")
}

func displaySet(factory HotWheelsFactory, model string) {
	fmt.Println("=== Producing", model, "Set ===")
	c1 := factory.CreateRegular(model)
	c2 := factory.CreateTreasureHunt(model)
	c3 := factory.CreateSuperTreasureHunt(model)
	p := factory.CreatePackaging()

	fmt.Println(c1.Name(), "→", c1.Details())
	fmt.Println(c2.Name(), "→", c2.Details())
	fmt.Println(c3.Name(), "→", c3.Details())
	fmt.Println("Packaging:", p.BoxType())
}

```

## Builder Design Pattern

Builder design pattern, as the name suggests, tries to _build_ the object step-by-step.

The **Builder** is responsible for **assembling** an object, and you can control the process by setting attributes one by one. Instead of pasing all parameters in a constructor, you pass only the ones you care about, and the builder takes care of the rest.

If the Factory Deisng pattern decides what type of car to build, Builder Design Pattern decides "how to assemble it".

Let's take a look at the code for this design pattern :

```go
package main

import "fmt"

type HotWheelsCar struct {
	Model        string
	Color        string
	WheelType    string
	Decals       string
	TreasureType string
}

func (c *HotWheelsCar) ShowSpecs() {
	fmt.Printf("Model: %s\nColor: %s\nWheels: %s\nDecals: %s\nEdition: %s\n\n",
		c.Model, c.Color, c.WheelType, c.Decals, c.TreasureType)
}

type CarBuilder interface {
	SetModel(model string) CarBuilder
	SetColor(color string) CarBuilder
	SetWheels(wheelType string) CarBuilder
	SetDecals(decals string) CarBuilder
	SetTreasureType(tType string) CarBuilder
	Build() HotWheelsCar
}

type HotWheelsCarBuilder struct {
	car HotWheelsCar
}

func NewHotWheelsCarBuilder() *HotWheelsCarBuilder {
	return &HotWheelsCarBuilder{}
}

func (b *HotWheelsCarBuilder) SetModel(model string) CarBuilder {
	b.car.Model = model
	return b
}

func (b *HotWheelsCarBuilder) SetColor(color string) CarBuilder {
	b.car.Color = color
	return b
}

func (b *HotWheelsCarBuilder) SetWheels(wheelType string) CarBuilder {
	b.car.WheelType = wheelType
	return b
}

func (b *HotWheelsCarBuilder) SetDecals(decals string) CarBuilder {
	b.car.Decals = decals
	return b
}

func (b *HotWheelsCarBuilder) SetTreasureType(tType string) CarBuilder {
	b.car.TreasureType = tType
	return b
}

func (b *HotWheelsCarBuilder) Build() HotWheelsCar {
	return b.car
}

type CarDirector struct {
	builder CarBuilder
}

func NewCarDirector(b CarBuilder) *CarDirector {
	return &CarDirector{builder: b}
}

func (d *CarDirector) BuildRegularCar() HotWheelsCar {
	return d.builder.
		SetModel("Hot Wheels Racer").
		SetColor("Red").
		SetWheels("Standard Chrome").
		SetDecals("Flame Decal").
		SetTreasureType("Regular").
		Build()
}

func (d *CarDirector) BuildTreasureHuntCar() HotWheelsCar {
	return d.builder.
		SetModel("Hot Wheels Racer").
		SetColor("Green").
		SetWheels("Rubberized").
		SetDecals("Hidden Flame Logo").
		SetTreasureType("Treasure Hunt").
		Build()
}

func (d *CarDirector) BuildSuperTreasureHuntCar() HotWheelsCar {
	return d.builder.
		SetModel("Hot Wheels Racer").
		SetColor("Spectraflame Gold").
		SetWheels("Real Riders").
		SetDecals("TH Logo").
		SetTreasureType("Super Treasure Hunt").
		Build()
}

func main() {
	builder := NewHotWheelsCarBuilder()
	director := NewCarDirector(builder)

	regular := director.BuildRegularCar()
	treasure := director.BuildTreasureHuntCar()
	superTreasure := director.BuildSuperTreasureHuntCar()

	fmt.Println("Hot Wheels Collection:")
	regular.ShowSpecs()
	treasure.ShowSpecs()
	superTreasure.ShowSpecs()
}

```

## Prototype Design Pattern

There will come a time when you would need to copy an already existing object with a very slight modification. In this case, creating a new object from scratch using the same constructor values would become redundant and inefficient.

Think about it like this — Hot Wheels has already built a limited-edition Super Treasure Hunt car, but they want to make a few modified versions with slight tweaks (say a new color or special wheels) for a convention release.

Instead of rebuilding everything from scratch, they can just clone the existing model and modify the necessary parts.

That’s the Prototype Design Pattern in a nutshell —
it lets you duplicate an existing object rather than creating a new instance manually, which can save time, memory, and effort, especially when the creation logic is expensive.

Let's take a look how it will look in Go:

```go
package main

import "fmt"

type CarPrototype interface {
	Clone() CarPrototype
	Show()
}

type HotWheelsCar struct {
	Model        string
	Color        string
	WheelType    string
	Decals       string
	TreasureType string
}

func (c *HotWheelsCar) Clone() CarPrototype {
	clone := *c
	return &clone
}

func (c *HotWheelsCar) Show() {
	fmt.Printf("[%s] %s | Wheels: %s | Decals: %s | Edition: %s\n",
		c.Color, c.Model, c.WheelType, c.Decals, c.TreasureType)
}

func main() {
	superTH := &HotWheelsCar{
		Model:        "Ferrari F50",
		Color:        "Spectraflame Gold",
		WheelType:    "Real Riders",
		Decals:       "TH Logo",
		TreasureType: "Super Treasure Hunt",
	}

	fmt.Println("Original:")
	superTH.Show()

	clone := superTH.Clone().(*HotWheelsCar)
	clone.Color = "Spectraflame Blue"
	clone.Decals = "Convention Edition Logo"

	clone.Show()
}
```

And this would be the output:

```md
Original:
[Spectraflame Gold] Hot Wheels Racer | Wheels: Real Riders | Decals: TH Logo | Edition: Super Treasure Hunt

Cloned and Modified:
[Spectraflame Blue] Hot Wheels Racer | Wheels: Real Riders | Decals: Convention Edition Logo | Edition: Super Treasure Hunt
```

### Why use Prototype??

Because sometimes, copying is smarter than recreating.

Instead of going through all the steps of the Builder again, the Prototype pattern lets you take a fully-configured object, make a few tweaks, and you’re done. It’s especially helpful when the object initialization involves deep structures, database calls, or heavy setup logic.

So in essence:

- You use Factory when you need new objects frequently.
- You use Builder when the creation is step-by-step and complex.
- You use Prototype when you already have a configured object and just need variations.

## Singleton Design Pattern

Finally, let’s talk about the one pattern that everyone seems to know — the Singleton Pattern.

The Singleton ensures that only one instance of a particular struct exists throughout the entire program.
You use this pattern when a global point of access is required — for example, a database connection, a logger, or a configuration manager.

To bring back our Hot Wheels analogy — imagine there’s only one Master Production Controller overseeing the entire manufacturing process. Every department can access it, but there can only ever be one.

That’s exactly what Singleton achieves.

Let's see how it will look in golang:

```go
package main

import (
	"fmt"
	"sync"
)

type MasterController struct {
	PlantName string
}

var instance *MasterController
var once sync.Once

func GetInstance() *MasterController {
	once.Do(func() {
		instance = &MasterController{PlantName: "Hot Wheels Main Factory - Malaysia"}
	})
	return instance
}

func main() {
	controller1 := GetInstance()
	controller2 := GetInstance()

	fmt.Println("Controller 1:", controller1.PlantName)
	fmt.Println("Controller 2:", controller2.PlantName)

	if controller1 == controller2 {
		fmt.Println("Both controllers point to the same instance.")
	}
}
```

### Why Singleton?

Because sometimes, you only need one of something —
and having multiple instances could cause chaos.

For example:

- Having multiple loggers might fragment your logs.
- Having multiple DB connections might overload your system.
- Having multiple production controllers… well, you can imagine the confusion at the Hot Wheels plant.

## TLDR;

Creational Design Patterns, in essence, help you control how your objects come to life.
They give structure to the otherwise chaotic process of object creation — keeping your code flexible, modular, and scalable as your application grows.

| Pattern              | Core Idea                                 | Hot Wheels Analogy                              |
| -------------------- | ----------------------------------------- | ----------------------------------------------- |
| **Factory**          | Centralize object creation                | One workstation producing car variants          |
| **Abstract Factory** | Create families of related objects        | Separate factories for Basic and Premium series |
| **Builder**          | Build complex objects step-by-step        | Assembling each car with fine control           |
| **Prototype**        | Clone existing objects with modifications | Copying a rare car for special editions         |
| **Singleton**        | Only one instance globally                | The single master production controller         |

Next in this series, we'lll take a look at Behavioral design pattern
