# Basic Optics
Based on

- Video: [Basic optics: lenses, prisms, and traversals in Haskell](https://www.youtube.com/watch?v=geV8F59q48E)
- Blogpost: [A (very short) intro to optics](https://www.47deg.com/blog/optics/)

## Optics

lens = getter + setter

### Operators

- `^.` [view](#Get)
- `.~` [set](#Set)
- `%~` [over](#Over)
- `%` [composition](#Composition)

#### View
```haskell

data Name   = Name String String
data Person = Person Name Age

view name p

-- Or

p ^. name -- like `p.name`
```

#### Set

```haskell
set name "B" p 

-- or

p & name .~ "B" -- like `p.name = "B"`
```

#### Over

Get + setter fn()

```haskell
over name ("Mr. " <>) p -- <> is a semigroup

-- or

p & name %~ ("Mr. " <>)
```

#### Composition

Compose lenses and get the type: from `Person` to `String` (firstname).


```haskell
name        :: Lens' Person Name
firstname   :: Lens' Name String

name % firstname :: Lens' Person String

p ^. (name % firstname) -- view with composition
p & name % firstname .~ "NewName" -- set with composition
```
Note `Lens'` are _simple_ lenses


## Affine traversals
 
`AffineTraversal` gives back a Maybe like; `Just x`/`Nothing` 

### Preview

```haskell

data User = Person { _name :: Name, _age :: Age }
          | Entity { _legalName :: Name } 

preview age user

user ^? age
```

If there's no age (because the user is of type Entity) it will return `Nothing`

### Composition

Following with above example

```haskell
makeLenses ''User
```

Will produce

```haskell
name :: AffineTraversal' User Name
firstName :: Lens' Name String


name % firstName :: AffineTraversal' User String
```

Composition with a `AffineTraversal'` will give back a `AffineTraversal'`. This is because it might fail, getting `Nothing`

## Prism

AffineTraversal + builder

```haskell
data Event = UserAdded    User
           | OrderShipped Order
makePrisms ''Event

review _UserAdded u -- creates an Event
_UserAdded # u      -- creates an Event

```

### Pattern Guards 

```haskell
getName :: User -> String
getName u = 
    | Just pd <- u ^? _Person = pd ^. name % firstNane
    | Just ed <- u ^? _Entity = ed ^. legalNane
```

It will do 

-  `u ^? _Person` and if it's `Just pd` _return_ `pd ^. name % firstName`
- same for second pattern matching

----

- https://www.youtube.com/watch?v=geV8F59q48E&t=1168s
- https://hackage.haskell.org/package/optics-0.3/docs/Optics.html
- https://hackage.haskell.org/package/lens-tutorial-1.0.4/docs/Control-Lens-Tutorial.html