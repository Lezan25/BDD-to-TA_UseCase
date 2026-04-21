# Simplified I4.0 Use Case — BDD Scenarios (Concrete)

## Model

```
model SimplifiedI4Factory
```

---
## Entities

```
entity user {
	actions:
	states: idle, waiting, ordering
	properties: ordered amount = 0
}

entity raw storage {
  actions: 
  states: idle, empty
  properties: level = 100
}

entity dough maker {
  actions: 
  states: idle, requesting, processing
}

entity dough shaper {
  actions: 
  states: idle, shaping
  properties: queue = 0
}

entity baking oven {
  actions: 
  states: idle, baking
  properties: queue = 0
}

entity cooling rack {
  actions: 
  states: idle, cooling
  properties: queue = 0
}

entity packaging machine {
  actions: 
  states: idle, packaging
  properties: queue = 0
}

entity product storage {
  actions:
  states: idle
  properties: level = 0
}

entity restocking system {
  actions:
  states: idle, restocking
  properties:
}
```

---
## Scenarios

### User

```
Scenario: User starts
	Given the user is idle
		And the dough maker is idle
	When 0 seconds pass
	Then the user is waiting
		And the dough maker is requesting
```

```
Scenario: User starts ordering
	Given the user is waiting
	When 30 seconds pass
	Then the user is ordering
		And the ordered amount is between 1 and 8
```

```
Scenario: User orders products #1
	Given the user is ordering
		And the ordered amount is greater than 0
		And the ordered amount is less or equal to the level of product storage
		And the dough maker is not idle
	Whenever 5 seconds pass
	Then the level of the product storage is decreased by the ordered amount
		And the ordered amount is between 1 and 8
```

```
Scenario: User orders products #2
	Given the user is ordering
		And the ordered amount is greater than 0
		And the ordered amount is less or equal to the level of product storage
		And the dough maker is idle
		And the level of the product storage is greater than 50
	Whenever 5 seconds pass
	Then the level of the product storage is decreased by the ordered amount
		And the ordered amount is between 1 and 8
```

```
Scenario: User orders products #3
	Given the user is ordering
		And the ordered amount is greater than 0
		And the ordered amount is less or equal to the level of product storage
		And the dough maker is idle
		And the level of the product storage is less or equal to 50
	Whenever 5 seconds pass
	Then the level of the product storage is decreased by the ordered amount
		And the dough maker is requesting
		And the ordered amount is between 1 and 8
```

```
Scenario: User orders products #4
	Given the user is ordering
		And the ordered amount is greater than 0
		And the ordered amount is greater than the level of product storage
	When 5 seconds pass
	Then the user is waiting
```

### Raw Storage

### Dough Maker

```
Scenario: Request raw ingredients #1
	Given the level of the raw storage is between 6 and 20
		And the dough maker is requesting
	When 5 seconds pass
	Then the level of the raw storage is decreased by 5
		And the dough maker is processing
```

```
Scenario: Request raw ingredients #2
	Given the level of the raw storage is between 21 and 25
		And the dough maker is requesting
		And the restocking system is idle
	Whenever 5 seconds pass
	Then the restocking system is restocking
		And the level of the raw storage is decreased by 5
		And the dough maker is processing
```

```
Scenario: Request raw ingredients #3
	Given the level of the raw storage is greater than 25
		And the dough maker is requesting
	When 5 seconds pass
	Then the level of the raw storage is decreased by 5
		And the dough maker is processing
```

```
Scenario: Request raw ingredients #4
	Given the level of the raw storage is 5
		And the dough maker is requesting
	When 5 seconds pass
	Then the raw storage is empty
		And the level of the raw storage is 0
		And the dough maker is processing
```

```
Scenario: Request raw ingredients #5
	Given the level of the raw storage is less than 5
		And the dough maker is requesting
	Whenever 5 seconds pass
	Then the raw storage is empty
```

```
Scenario: Dough processing finishes
	Given the dough maker is processing
	When 5 seconds pass
	Then the dough maker is requesting
		And the dough shaper is shaping
		And the queue of the dough shaper is increased by 1
```

### Dough Shaper

```
Scenario: Dough shaping finishes #1
	Given the dough shaper is shaping
		And the queue of the dough shaper is 1
	When 5 seconds pass
	Then the dough shaper is idle
		And the queue of the dough shaper is 0
		And the baking oven is baking
		And the queue of the baking oven is increased by 1
```

```
Scenario: Dough shaping finishes #2
	Given the dough shaper is shaping
		And the queue of the dough shaper is greater than 1
	Whenever 5 seconds pass
	Then the queue of the dough shaper is decreased by 1
		And the baking oven is baking
		And the queue of the baking oven is increased by 1
```

### Baking Oven

```
Scenario: Baking finishes #1
	Given the baking oven is baking
		And the queue of the baking oven is 1
	When 5 seconds pass
	Then the baking oven is idle
		And the queue of the baking oven is 0
		And the cooling rack is cooling
		And the queue of the cooling rack is increased by 1
```

```
Scenario: Baking finishes #2
	Given the baking oven is baking
		And the queue of the baking oven is greater than 1
	Whenever 5 seconds pass
	Then the queue of the baking oven is decreased by 1
		And the cooling rack is cooling
		And the queue of the cooling rack is increased by 1
```

### Cooling Rack

```
Scenario: Cooling finishes #1
	Given the cooling rack is cooling
		And the queue of the cooling rack is 1
	When 5 seconds pass
	Then the cooling rack is idle
		And the queue of the cooling rack is 0
		And the packaging machine is packaging
		And the queue of the packaging machine is increased by 1
```

```
Scenario: Cooling finishes #2
	Given the cooling rack is cooling
		And the queue of the cooling rack is greater than 1
	Whenever 5 seconds pass
	Then the queue of the cooling rack is decreased by 1
		And the packaging machine is packaging
		And the queue of the packaging machine is increased by 1
```

### Packaging Machine

```
Scenario: Packaging finishes #1
	Given the packaging machine is packaging
		And the queue of the packaging machine is 1
		And the level of the product storage is less than 85
	When 5 seconds pass
	Then the packaging machine is idle
		And the queue of the packaging machine is 0
		And the level of the product storage is increased by 5
```

```
Scenario: Packaging finishes #2
	Given the packaging machine is packaging
		And the queue of the packaging machine is greater than 1
		And the level of the product storage is less than 85
	Whenever 5 seconds pass
	Then the queue of the packaging machine is decreased by 1
		And the level of the product storage is increased by 5
```

```
Scenario: Packaging finishes #3
	Given the packaging machine is packaging
		And the queue of the packaging machine is 1
		And the level of the product storage is greater or equal to 85
		And the dough maker is requesting
	When 5 seconds pass
	Then the packaging machine is idle
		And the queue of the packaging machine is 0
		And the level of the product storage is increased by 5
		And the dough maker is idle
```

```
Scenario: Packaging finishes #4
	Given the packaging machine is packaging
		And the queue of the packaging machine is 1
		And the level of the product storage is greater or equal to 85
		And the dough maker is processing
	When 5 seconds pass
		And dough maker processing finishes
	Then the packaging machine is idle
		And the queue of the packaging machine is 0
		And the level of the product storage is increased by 5
		And the dough maker is idle
		And the dough shaper is shaping
		And the queue of the dough shaper is increased by 1
```

```
Scenario: Packaging finishes #5
	Given the packaging machine is packaging
		And the queue of the packaging machine is greater than 1
		And the level of the product storage is greater or equal to 85
		And the dough maker is requesting
	Whenever 5 seconds pass
	Then the queue of the packaging machine is decreased by 1
		And the level of the product storage is increased by 5
		And the dough maker is idle
```

```
Scenario: Packaging finishes #6
	Given the packaging machine is packaging
		And the queue of the packaging machine is greater than 1
		And the level of the product storage is greater or equal to 85
		And the dough maker is processing
	Whenever 5 seconds pass
		And dough maker processing finishes
	Then the queue of the packaging machine is decreased by 1
		And the level of the product storage is increased by 5
		And the dough maker is idle
		And the dough shaper is shaping
		And the queue of the dough shaper is increased by 1
```

### Product Storage

### Restocking System

```
Scenario: Restock
	Given the restocking system is restocking
	When 10 seconds pass
	Then the raw storage is idle
		And the level of the raw storage is 100
		And the restocking system is idle
```
  