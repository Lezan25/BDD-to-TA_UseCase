# # Simplified I4.0 Use Case — BDD Scenarios (Abstract)

## Model

```
model(SimplifiedI4Factory, [entities below], [scenarios below])
```

---

## Entities

```
entity(user,
  [],
  [var(ordered_amount, num(0))],
  state_var(user, [idle, waiting, ordering])
)
```

```
entity(raw_storage,
  [],
  [var(level, num(100))],
  state_var(raw_storage, [idle, empty])
)
```

```
entity(dough_maker,
  [],
  [],
  state_var(dough_maker, [idle, requesting, processing])
)
```

```
entity(dough_shaper,
  [],
  [var(queue, num(0))],
  state_var(dough_shaper, [idle, shaping])
)
```

```
entity(baking_oven,
  [],
  [var(queue, num(0))],
  state_var(baking_oven, [idle, baking])
)
```

```
entity(cooling_rack,
  [],
  [var(queue, num(0))],
  state_var(cooling_rack, [idle, cooling])
)
```

```
entity(packaging_machine,
  [],
  [var(queue, num(0))],
  state_var(packaging_machine, [idle, packaging])
)
```

```
entity(product_storage,
  [],
  [var(level, num(0))],
  state_var(product_storage, [idle])
)
```

```
entity(restocking_system,
  [],
  [],
  state_var(restocking_system, [idle, restocking])
)
```

---

## Scenarios

### User

```
sc(
  User_starts,
  state_is(user, idle)
  ∧ state_is(dough_maker, idle),
  trigger(once, on_time(0)),
  set_state(user, waiting) ; set_state(dough_maker, requesting)
)
```

```
sc(
  User_starts_ordering,
  state_is(user, waiting),
  trigger(once, on_time(30)),
  set_state(user, ordering)
  ; set_between(prop(user, ordered_amount), num(1), num(8))
)
```

```
sc(
  User_orders_products_1,
  state_is(user, ordering)
  ∧ prop_cmp(prop(user, ordered_amount), >, num(0))
  ∧ prop_cmp(prop(user, ordered_amount), ≤, prop(product_storage, level))
  ∧ state_not(dough_maker, idle),
  trigger(repeat, on_time(5)),
  dec_prop(prop(product_storage, level), prop(user, ordered_amount))
  ; set_between(prop(user, ordered_amount), num(1), num(8))
)
```

```
sc(
  User_orders_products_2,
  state_is(user, ordering)
  ∧ prop_cmp(prop(user, ordered_amount), >, num(0))
  ∧ prop_cmp(prop(user, ordered_amount), ≤, prop(product_storage, level))
  ∧ state_is(dough_maker, idle)
  ∧ prop_cmp(prop(product_storage, level), >, num(50)),
  trigger(repeat, on_time(5)),
  dec_prop(prop(product_storage, level), prop(user, ordered_amount))
  ; set_between(prop(user, ordered_amount), num(1), num(8))
)
```

```
sc(
  User_orders_products_3,
  state_is(user, ordering)
  ∧ prop_cmp(prop(user, ordered_amount), >, num(0))
  ∧ prop_cmp(prop(user, ordered_amount), ≤, prop(product_storage, level))
  ∧ state_is(dough_maker, idle)
  ∧ prop_cmp(prop(product_storage, level), ≤, num(50)),
  trigger(repeat, on_time(5)),
  dec_prop(prop(product_storage, level), prop(user, ordered_amount))
  ; set_state(dough_maker, requesting)
  ; set_between(prop(user, ordered_amount), num(1), num(8))
)
```

```
sc(
  User_orders_products_4,
  state_is(user, ordering)
  ∧ prop_cmp(prop(user, ordered_amount), >, num(0))
  ∧ prop_cmp(prop(user, ordered_amount), >, prop(product_storage, level)),
  trigger(once, on_time(5)),
  set_state(user, waiting)
)
```

---
### Dough Maker

```
sc(
  Request_raw_ingredients_1,
  prop_cmp(prop(raw_storage, level), ≥, num(6))
  ∧ prop_cmp(prop(raw_storage, level), ≤, num(20))
  ∧ state_is(dough_maker, requesting),
  trigger(once, on_time(5)),
  dec_prop(prop(raw_storage, level), num(5))
  ; set_state(dough_maker, processing)
)
```

```
sc(
  Request_raw_ingredients_2,
  prop_cmp(prop(raw_storage, level), ≥, num(21))
  ∧ prop_cmp(prop(raw_storage, level), ≤, num(25))
  ∧ state_is(dough_maker, requesting),
  trigger(repeat, on_time(5)),
  set_state(restocking_system, restocking)
  ; dec_prop(prop(raw_storage, level), num(5))
  ; set_state(dough_maker, processing)
)
```

```
sc(
  Request_raw_ingredients_3,
  prop_cmp(prop(raw_storage, level), >, num(25))
  ∧ state_is(dough_maker, requesting)
  ∧ state_is(restocking_system, idle),
  trigger(once, on_time(5)),
  dec_prop(prop(raw_storage, level), num(5))
  ; set_state(dough_maker, processing)
)
```

```
sc(
  Request_raw_ingredients_4,
  prop_cmp(prop(raw_storage, level), =, num(5))
  ∧ state_is(dough_maker, requesting),
  trigger(once, on_time(5)),
  set_state(raw_storage, empty)
  ; set_prop(prop(raw_storage, level), num(0))
  ; set_state(dough_maker, processing)
)
```

```
sc(
  Request_raw_ingredients_5,
  prop_cmp(prop(raw_storage, level), <, num(5))
  ∧ state_is(dough_maker, requesting),
  trigger(repeat, on_time(5)),
  set_state(raw_storage, empty)
)
```

```
sc(
  Dough_processing_finishes,
  state_is(dough_maker, processing),
  trigger(once, on_time(5)),
  set_state(dough_maker, requesting)
  ; set_state(dough_shaper, shaping)
  ; inc_prop(prop(dough_shaper, queue), num(1))
)
```

---
### Dough Shaper

```
sc(
  Dough_shaping_finishes_1,
  state_is(dough_shaper, shaping)
  ∧ prop_cmp(prop(dough_shaper, queue), =, num(1)),
  trigger(once, on_time(5)),
  set_state(dough_shaper, idle)
  ; set_prop(prop(dough_shaper, queue), num(0))
  ; set_state(baking_oven, baking)
  ; inc_prop(prop(baking_oven, queue), num(1))
)
```

```
sc(
  Dough_shaping_finishes_2,
  state_is(dough_shaper, shaping)
  ∧ prop_cmp(prop(dough_shaper, queue), >, num(1)),
  trigger(repeat, on_time(5)),
  dec_prop(prop(dough_shaper, queue), num(1))
  ; set_state(baking_oven, baking)
  ; inc_prop(prop(baking_oven, queue), num(1))
)
```

---
### Baking Oven

```
sc(
  Baking_finishes_1,
  state_is(baking_oven, baking)
  ∧ prop_cmp(prop(baking_oven, queue), =, num(1)),
  trigger(once, on_time(5)),
  set_state(baking_oven, idle)
  ; set_prop(prop(baking_oven, queue), num(0))
  ; set_state(cooling_rack, cooling)
  ; inc_prop(prop(cooling_rack, queue), num(1))
)
```

```
sc(
  Baking_finishes_2,
  state_is(baking_oven, baking)
  ∧ prop_cmp(prop(baking_oven, queue), >, num(1)),
  trigger(repeat, on_time(5)),
  dec_prop(prop(baking_oven, queue), num(1))
  ; set_state(cooling_rack, cooling)
  ; inc_prop(prop(cooling_rack, queue), num(1))
)
```

---
### Cooling Rack

```
sc(
  Cooling_finishes_1,
  state_is(cooling_rack, cooling)
  ∧ prop_cmp(prop(cooling_rack, queue), =, num(1)),
  trigger(once, on_time(5)),
  set_state(cooling_rack, idle)
  ; set_prop(prop(cooling_rack, queue), num(0))
  ; set_state(packaging_machine, packaging)
  ; inc_prop(prop(packaging_machine, queue), num(1))
)
```

```
sc(
  Cooling_finishes_2,
  state_is(cooling_rack, cooling)
  ∧ prop_cmp(prop(cooling_rack, queue), >, num(1)),
  trigger(repeat, on_time(5)),
  dec_prop(prop(cooling_rack, queue), num(1))
  ; set_state(packaging_machine, packaging)
  ; inc_prop(prop(packaging_machine, queue), num(1))
)
```

---
### Packaging Machine

```
sc(
  Packaging_finishes_1,
  state_is(packaging_machine, packaging)
  ∧ prop_cmp(prop(packaging_machine, queue), =, num(1))
  ∧ prop_cmp(prop(product_storage, level), <, num(85)),
  trigger(once, on_time(5)),
  set_state(packaging_machine, idle)
  ; set_prop(prop(packaging_machine, queue), num(0))
  ; inc_prop(prop(product_storage, level), num(5))
)
```

```
sc(
  Packaging_finishes_2,
  state_is(packaging_machine, packaging)
  ∧ prop_cmp(prop(packaging_machine, queue), >, num(1))
  ∧ prop_cmp(prop(product_storage, level), <, num(85)),
  trigger(repeat, on_time(5)),
  dec_prop(prop(packaging_machine, queue), num(1))
  ; inc_prop(prop(product_storage, level), num(5))
)
```

```
sc(
  Packaging_finishes_3,
  state_is(packaging_machine, packaging)
  ∧ prop_cmp(prop(packaging_machine, queue), =, num(1))
  ∧ prop_cmp(prop(product_storage, level), ≥, num(85))
  ∧ state_is(dough_maker, requesting),
  trigger(once, on_time(5)),
  set_state(packaging_machine, idle)
  ; set_prop(prop(packaging_machine, queue), num(0))
  ; inc_prop(prop(product_storage, level), num(5))
  ; set_state(dough_maker, idle)
)
```

```
sc(
  Packaging_finishes_4,
  state_is(packaging_machine, packaging)
  ∧ prop_cmp(prop(packaging_machine, queue), =, num(1))
  ∧ prop_cmp(prop(product_storage, level), ≥, num(85))
  ∧ state_is(dough_maker, processing),
  trigger(once, on_time(5) & on_end(dough_maker, processing)),
  set_state(packaging_machine, idle)
  ; set_prop(prop(packaging_machine, queue), num(0))
  ; inc_prop(prop(product_storage, level), num(5))
  ; set_state(dough_maker, idle)
  ; set_state(dough_shaper, shaping)
  ; inc_prop(prop(dough_shaper, queue), num(1))
)
```

```
sc(
  Packaging_finishes_5,
  state_is(packaging_machine, packaging)
  ∧ prop_cmp(prop(packaging_machine, queue), >, num(1))
  ∧ prop_cmp(prop(product_storage, level), ≥, num(85))
  ∧ state_is(dough_maker, requesting),
  trigger(repeat, on_time(5)),
  dec_prop(prop(packaging_machine, queue), num(1))
  ; inc_prop(prop(product_storage, level), num(5))
  ; set_state(dough_maker, idle)
)
```

```
sc(
  Packaging_finishes_6,
  state_is(packaging_machine, packaging)
  ∧ prop_cmp(prop(packaging_machine, queue), >, num(1))
  ∧ prop_cmp(prop(product_storage, level), ≥, num(85))
  ∧ state_is(dough_maker, processing),
  trigger(repeat, on_time(5) & on_end(dough_maker, processing)),
  dec_prop(prop(packaging_machine, queue), num(1))
  ; inc_prop(prop(product_storage, level), num(5))
  ; set_state(dough_maker, idle)
  ; set_state(dough_shaper, shaping)
  ; inc_prop(prop(dough_shaper, queue), num(1))
)
```

---
### Restocking System

```
sc(
  Restock,
  state_is(restocking_system, restocking),
  trigger(once, on_time(10)),
  set_state(raw_storage, idle)
  ; set_prop(prop(raw_storage, level), num(100))
  ; set_state(restocking_system, idle)
)
```
