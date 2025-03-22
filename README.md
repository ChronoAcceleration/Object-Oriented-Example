# Object-Oriented Programming in Luau

## Introduction

This documentation provides a comprehensive guide to implementing object-oriented programming (OOP) in Luau, the scripting language used in Roblox. Luau extends Lua with additional features, type checking, and performance improvements that make OOP implementation more robust.

## Table of Contents

- [Fundamentals of OOP in Luau](#fundamentals-of-OOP-in-luau)
- [Creating Classes](#creating-classes)
- [Inheritance](#inheritance)
- [Encapsulation](#encapsulation)
- [Polymorphism](#polymorphism)
- [Working with Metatables](#working-with-metatables)
- [Best Practices](#best-practices)
- [Examples](#examples)
- [Resources](#resources)

## Fundamentals of OOP in Luau

Luau doesn't have built-in class syntax like languages such as Java or C#, but we can implement OOP concepts using tables and metatables. The core concepts include:

- **Classes**: Templates for creating objects
- **Objects**: Instances of classes
- **Methods**: Functions that operate on objects
- **Properties**: Data stored within objects

## Creating Classes

### Basic Class Structure

```lua
-- Class definition
local MyClass = {}
MyClass.__index = MyClass

-- Constructor
function MyClass.new(initialValue)
    local self = setmetatable({}, MyClass)
    self.value = initialValue or 0
    return self
end

-- Method
function MyClass:getValue()
    return self.value
end

-- Method with parameters
function MyClass:setValue(newValue)
    self.value = newValue
end

-- Usage
local instance = MyClass.new(10)
print(instance:getValue()) -- Outputs: 10
instance:setValue(20)
print(instance:getValue()) -- Outputs: 20
```

## Inheritance

Inheritance allows a class to reuse code from a parent class while potentially extending or modifying its behavior.

```lua
-- Parent class
local Animal = {}
Animal.__index = Animal

function Animal.new(name)
    local self = setmetatable({}, Animal)
    self.name = name
    return self
end

function Animal:speak()
    return "Some generic animal sound"
end

-- Child class
local Dog = {}
Dog.__index = Dog
setmetatable(Dog, Animal) -- Inherit from Animal

function Dog.new(name, breed)
    local self = setmetatable(Animal.new(name), Dog)
    self.breed = breed
    return self
end

function Dog:speak()
    return "Woof!"
end

-- Usage
local myDog = Dog.new("Rex", "Golden Retriever")
print(myDog.name)   -- "Rex" (inherited property)
print(myDog.breed)  -- "Golden Retriever"
print(myDog:speak()) -- "Woof!" (overridden method)
```

## Encapsulation

Encapsulation involves hiding implementation details and exposing only necessary interfaces.

```lua
local Counter = {}
Counter.__index = Counter

function Counter.new(initialValue)
    local private = {
        count = initialValue or 0
    }
    
    local self = setmetatable({}, Counter)
    
    function self:increment()
        private.count = private.count + 1
        return self
    end
    
    function self:getCount()
        return private.count
    end
    
    return self
end

-- Usage
local counter = Counter.new(5)
counter:increment():increment() -- Method chaining
print(counter:getCount()) -- 7
-- print(counter.count) -- This would be nil; count is private
```

## Polymorphism

Polymorphism allows different classes to be treated as instances of the same class through inheritance.

```lua
local Shape = {}
Shape.__index = Shape

function Shape.new()
    return setmetatable({}, Shape)
end

function Shape:calculateArea()
    error("Method must be implemented by subclass")
end

-- Circle subclass
local Circle = {}
Circle.__index = Circle
setmetatable(Circle, Shape)

function Circle.new(radius)
    local self = setmetatable(Shape.new(), Circle)
    self.radius = radius
    return self
end

function Circle:calculateArea()
    return math.pi * self.radius * self.radius
end

-- Rectangle subclass
local Rectangle = {}
Rectangle.__index = Rectangle
setmetatable(Rectangle, Shape)

function Rectangle.new(width, height)
    local self = setmetatable(Shape.new(), Rectangle)
    self.width = width
    self.height = height
    return self
end

function Rectangle:calculateArea()
    return self.width * self.height
end

-- Polymorphic function
local function printArea(shape)
    print("Area:", shape:calculateArea())
end

-- Usage
local myCircle = Circle.new(5)
local myRectangle = Rectangle.new(4, 6)

printArea(myCircle)     -- Area: 78.53981633974483
printArea(myRectangle)  -- Area: 24
```

## Working with Metatables

Metatables allow you to customize the behavior of tables, which is essential for OOP in Luau.

```lua
local Vector2D = {}
Vector2D.__index = Vector2D

function Vector2D.new(x, y)
    return setmetatable({x = x or 0, y = y or 0}, Vector2D)
end

-- Overload the + operator
function Vector2D.__add(a, b)
    return Vector2D.new(a.x + b.x, a.y + b.y)
end

-- Overload the - operator
function Vector2D.__sub(a, b)
    return Vector2D.new(a.x - b.x, a.y - b.y)
end

-- Overload tostring
function Vector2D:__tostring()
    return "Vector2D(" .. self.x .. ", " .. self.y .. ")"
end

-- Usage
local v1 = Vector2D.new(1, 2)
local v2 = Vector2D.new(3, 4)
local v3 = v1 + v2
print(v3) -- Vector2D(4, 6)
```

## Best Practices

1. **Use the colon operator** for methods that operate on the object (`self`).
2. **Initialize all properties** in the constructor for clarity.
3. **Follow naming conventions**:
   - Use PascalCase for class names: `Animal`, `Person`
   - Use camelCase for methods and properties: `getName()`, `firstName`
4. **Implement proper encapsulation** by using closures for private data.
5. **Document your classes** with clear comments describing purpose and usage.
6. **Check types** using Luau's type system for better reliability.

## Examples

### Complete Game Entity System

```lua
-- Base Entity class
local Entity = {}
Entity.__index = Entity

function Entity.new(id, position)
    local self = setmetatable({}, Entity)
    self.id = id
    self.position = position or Vector3.new(0, 0, 0)
    self.components = {}
    return self
end

function Entity:addComponent(componentName, component)
    self.components[componentName] = component
    return self
end

function Entity:getComponent(componentName)
    return self.components[componentName]
end

function Entity:update(dt)
    for _, component in pairs(self.components) do
        if component.update then
            component:update(dt)
        end
    end
end

-- Health Component
local HealthComponent = {}
HealthComponent.__index = HealthComponent

function HealthComponent.new(maxHealth)
    local self = setmetatable({}, HealthComponent)
    self.maxHealth = maxHealth
    self.currentHealth = maxHealth
    return self
end

function HealthComponent:takeDamage(amount)
    self.currentHealth = math.max(0, self.currentHealth - amount)
    return self.currentHealth <= 0
end

function HealthComponent:heal(amount)
    self.currentHealth = math.min(self.maxHealth, self.currentHealth + amount)
end

-- Movement Component
local MovementComponent = {}
MovementComponent.__index = MovementComponent

function MovementComponent.new(speed)
    local self = setmetatable({}, MovementComponent)
    self.speed = speed
    self.velocity = Vector3.new(0, 0, 0)
    return self
end

function MovementComponent:update(dt)
    -- Update entity position based on velocity and dt
    if self.entity then
        self.entity.position = self.entity.position + self.velocity * self.speed * dt
    end
end

function MovementComponent:setEntity(entity)
    self.entity = entity
end

-- Usage
local player = Entity.new("player1", Vector3.new(0, 10, 0))
local healthComp = HealthComponent.new(100)
local movementComp = MovementComponent.new(5)

movementComp:setEntity(player)

player:addComponent("health", healthComp)
      :addComponent("movement", movementComp)

-- Game loop
local function gameLoop(dt)
    player:update(dt)
    
    -- Check player health
    local health = player:getComponent("health")
    if health.currentHealth <= 0 then
        print("Player defeated!")
    end
end
```

## Resources

- [Luau Documentation](https://luau-lang.org/)
- [Roblox Developer Hub](https://developer.roblox.com/en-us/learn-roblox/coding-scripts)
- [Object-Oriented Programming in Lua](http://www.lua.org/pil/16.html)
- [Metatables and Metamethods](http://www.lua.org/pil/13.html)
