# Petri
```
A language for developing probabalistic soup applications that consist of small code bits that consume and transform data similarly to proteins.
```

## Language Status
Idea Formulation

## Language Idea
Petri is a functional programming language in which applications are developed by writing `funclets`, autonomously invoked functions that are placed into `dishes`, virtual environments that contain data objects required to be processed by the program and programm's funclets that consume existing and create new objects. 

## Object identification
`id` field is reserved for object identification.

## Object Consumption
Whenever a new object is added, all permutations of all paths to existing in the object keys are used to generate a set of sha-256 hashes. Hashes that not already present in the dish are then used to lookup funclets that can partially or fully consume that object. Then a single matching funclet is chosen probabilistically (based on present in the dish amounts of copies of matched funclets) and passed corresponding values that are removed from the object. If, after that operation, the object contains more data, a new attempt to consume it is made. Unconsumed by that process data is then stored for later consumption.

Whenever a new funclet is created, its argument names are used to generate a sha-256 hash that is used to perform a lookup on previously unconsumed data in order to find the data that can be immediately consumed by the funclet. A single found object is then consumed randomly.

### Multi-Object Consumption
Multi-Object consumption is performed in the order object patterns are defined in a multi-object funclet. That is, the second object in the argument list _will be matched only after a match been found for the first argument_. Internally, the funclet is represented as a series of single-object funclets that wrap funclet's code:
```
o1(f0..n), o2(f0..n) -> {
  o1.fx = o2.id;
} -> o1, o2;
```
equals to:
```
// here, `-->` means "funclet without a body"
o1(f0..n) --> 
  o2(f0..n) -> {
    o1.fx = o2.id;
  } -> o1, o2;
```

### Overflow Gates
An overflow gate is activated when a specified number of matching objects exists in the dish whenever a new matching object is added:
```
user(inQueue):50 --> serverRedirect() + user;
```

## Returned Objects
A funclet may return an array of existing or new objects that will be added back into the dish and processed in it, including itself. 
If a funclet returns an object with the same id as the part of a bigger object that was passed to it as a partial match, then that part is inserted back into the object from which the passed data has originated.

### Linking Operator
`+` binary operator can be used to add a field that points to another object:
```
user(inQueue), seat(isFree) -> {
  delete seat.isFree
  set seat.isBisy;            // unfolds into `seat.isBisy = Date.now()`
} -> user+seat, seat;         // user.seat == seat
```
### Cutting Operator
`-` binary operator can be used to cut one object out of another:
```
user(left) -> {
  seat = user.seat;
  set seat.isFree;
} -> user - seat, seat;         // user.seat == undefined
```

### Multiply Operator
`*` can be used to create multiple copies of the same object:
```
dish(started) --> command(accept, users: 10);                   // accept is auto-set to current timestamp

command(accept, users) --> seat(
  isFree: command.accept
) * command.users;
```

## Object Deletion
Objects are deleted automatically in following cases:
- a funclet object that processed some data will be removed from the dish unless it returns itself using the `this` keyword. Therefore, amount of present in the dish funclets becomes an important consideration when developing an application and a useful tool for load and performancd management.
- as soon as all data in an object has been consumed, it is removed from the dish.
- delete is explicitly caled on any of the object matches:
```
    user(loggedOut) -> delete user
```

## Proposed Syntax Examples
### Funclet Definition
```
funclet := type-name '(' argumentList? ')' ('->' code)? (-> user, this;
```
### Dish Initialization
```
dish(started) -> {
  // do stuff
}
```
### Multi-Object Consumption
```
user(name, inQueue), server(hasAvailableSeats) -> {
  user.server = server.id;
  server.hasAvailableSeats = --server.availableSeats > 0;
} -> user, server;
```
## External Objects
External objects can be used to represent external inputs. Whenever a funclet that consumes fields from these objects is created, the corresponding input is opened and listened for events that are then represented as external objects. The input is closed automatically whenever there's no correspondng to it funclets in the dish.
- `console.in`
- `system.fs.root`
- `system.net.socket`
- `system.key.press`
- `system.mouse.press`
- `system.process.signal`
- ...

## External Funclets
Like external objects, external funclets are automatically created whenever corresponding objects are added into the dish:
- `console.message`
- `file.name`
- `tcp.packet`
- `sound.message`
- `process.signal`
- ...

## Dish Exchange Format
( to run a dish across multiple nodes)
```json
[
  [objects],[funclets]
]
```


## ToDo
[x] refine multi-object consumption
[] refine time dilation
[] develop virtual machine
[] develop stdlib
