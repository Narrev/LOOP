# LOOP
Lua object-oriented programming.
Work was primarily driven by three constraints; in no particular order:

* Memory usage
* Speed
* The desire to not write Lua code while writing Lua code (read: convenience).

None of these can be attained optimally without sacrificing at least one of the others.  This implementation is a tradeoff that suits my purposes: It's fast, avoids function wrappers for the sake of speed and simplicity, and minimizes unnecessary overhead wherever possible.


##Spec

Class bodies can include metamethods, static members, methods, and a constructor.

```lua
local Person = Class{
	Say = function(self, phrase) -- Member
		print(self.name .. ':', phrase)
	end;
}

local BagelEater = Class{
	EatBagel = function(self)
		self.bagelCount = self.bagelCount - 1
	end;
	HasBagel = function(self)
		return self.bagelCount > 0
	end;
}
```

Inherited classes are specified in parenthesis at the class declaration.  Inheritance is implemented as contcatenating the parent class members in the order of the first to last Class argument, and finally with the members of the child class.

Member shadowing occurs when a member of one inherited class overrides another.  Member shadowing can occasionally be undesireable (e.g. diamond inheritance graphs), so it can be disabled or enabled at the top of the script.


The name of the constructor is `Init` by default, although one can change it at the top of the script.  Instance members should typically be initialized in the constructor.  Best practice is to initialize class members with `rawset`, especially if `class.__newindex` is defined.

```lua
local PersonWhoEatsBagels = Class(Person, BagelEater){ -- Inherits from Person & BagelEater
	Init = function(self, name, bagelCount)
		rawset(self, 'name', name)
		rawset(self, 'bagelCount', bagelCount)
	end;
}
```

To create an instance of the class, simply call the class name:

```lua
local bob = PersonWhoEatsBagels('bob', 999)
bob:Say'Hi' --> Bob: Hi
bob:EatBagel()
print(bob:HasBagel()) --> true
```

Member names matching any metamethods from Lua 5.1 are treated as metamethods for class instances.

```lua
local Number = Class{
	Init = function(value)
		self.value = value
	end;
	__add = function(op0, op1)
		return op0.value + op1.value
	end;
	__tostring = function(self)
		return tostring(self.value)
	end;
}

print(Number(1) + Number(2)) --> 3
```

This system has several benefits to more traditional Lua approaches to OOP.

* Able to override `__index`
* Reduction in boilerplate
* More expressive syntax
* Built-in type checking
* Clean and straightforward polymorphism
