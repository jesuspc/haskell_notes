# Category Theory for Programmers

__Source:__ [Bartosz Milewski's Programming Cafe](http://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-preface/)

## 1. Category: The Essence of Composition

__Category__: Objects and arrows that go between them
  * Arrows compose
    ```tex
    A -> B, B -> C => A -> C
    ```

__Morphism__: The arrows
  * Functions as morphisms
    ```tex
    f(a) -> b, g(b) -> c => g(f(a)) -> c (~> g∘f)
    ```
    ```haskell
      f :: A -> B -- f has type function from type A to type B
      g :: B -> C
      g . f -- composed
    ```
__Composition__
  * Properties:
    * Associative:
      ```tex
      h∘(g∘f) = (h∘g)∘f = h∘g∘f
      ```
    * Identity function (arrow loops from the object to itself):
      ```tex
      Given f :: A -> B then:

      f∘idA = f; idB∘f = f
      ```
      ```haskell
      id :: a -> a -- Polymorphic declaration; a is any type
      id x = x
      ```

## 2. Types and Functions

__Types__: Sets of values
  * Can be finite or infinite
  * To compose there must be a -> from out1 to out2; Types ensure this match
  * Examples in Haskell:
    * __Void__: Not inhabited by any values
      * Functions taking it can be defined but not called since value would be needed
        ```haskell
        -- Curry-Howard isomorphism: from falsity follows anything
        absurd :: Void -> a
        ```
    * __Singleton__: Contains only one value (called unit)
      * That means we can ellide the value from the function call
        ```haskell
        f44 :: () -> Integer
        f44 () = 44 -- The unit value is ()
        ```
      * Funcs from unit to any type are biunivocal with the elements of that set.
      * __Parametrically polymorphic function__
        * Can be implemented with the same formula for any type
          ```haskell
          -- A whole family of those functions can be implemented as:
          unit :: a -> ()
          unit _ = ()
          ```
    * __Bool__: Contains only two values, True and False
      * Functions to __Bool__ are called __predicates__

__Set__ (category): The category where objects are sets and morphism functions
  * Functions map elements from one set to elements of another
  * Functions can map two elements to one but not one to two
  * Identity functions map each elm of a set to itself

__Hask__: Set category for which each set has been extended with __bottom__
  * The category of Haskell types and functions

__Bottom__: Value that corresponds to non-terminating computation
  * In Haskell
    * Notted as **\_|\_**
    * Every runtime error is treated as bottom
    * Is a member of all types
    * undefined evaluates to bottom
    * Functions that may return it are called partial
    * Functions that can not return it are called total (valid for every arg)

__Functions__
  * If no side effects are called pure
    * In Haskell all functions are pure

## 3. Categories Great and Small

Example categories

__No objects__: Zero objects => Zero morphisms

__Free category__
  * Category generated by connecting each node with each other possible node
  * __Free construction__
    * Completing a given structure by extending it with the min number of items
      to satisfy its laws (in the previous case category laws)

__Hom-set__ (C(a,b))
  * Set of morphisms from object a to object b in a category C

__Orders__
  * A morphism is the relation of being less than or equal => __Preorder__
      - => Every hom-set is empty or singleton
        - => C(a,a) contains only the identity
  * If also [a <= b and b <= a => a = b] => __Partial order__
      - => Can not have cycles
      - => Any directed acyclic graph generates it as its free category
  * If also all objects are in a relation with each other => __Total order__
    or __Linear order__
    * Required for most of the sorting algorithms

__Monoids__
  * A set with a binary operation that is:
    * Associative
    * Has a neutral element with respect to the operation
    ```haskell
    class Monoid m whereclass Monoid m where
    mempty  :: m
    mappend :: m -> m -> m
    -- It's programmer's responsability to make sure monoidal properties are satisfied
    ```
  * Can be described as a single category object with a set of morphisms:
    * __Categorical monoid__: One-object category (M)
      - => M(m, m) + binary operator __monoidal product__:
        * g·f should exist because source and target are the same
        * Is associative by rules of category
        * Identity morphism is the neutral element
          - => A set monoid can always be recovered from a category monoid
            - => They are the same

## 4. Kleisli Categories

__Kleisli Category__: A category based on a Monad
  * Its objects are the types of the underlying prgramming language
  * Its morphisms from type A to type B are functions from A to a type derived
    from B using the particular 'embellishment' (endofunctor)
  * It defines:
    * Function composition
    * The identity morphism with respect to the composition operation
  ```haskell
  -- type alias parameterized by a type variable a and equivalent to the pair (a, String)
  type Writer a = (a, String)
  -- Our morphisms are functions from a -> Writer b
  -- Composition is declared with the fish operator
  (>=>) :: (a -> Writer b) -> (b -> Writer c) -> (a -> Writer c)
  -- \x is a lambda of one argument
  -- `let` declare auxiliar variables to be used in the return value defined by the `in` clause
  -- Equality in let pattern matches
  m1 >=> m2 = \x ->
    let (y, s1) = m1 x
        (z, s2) = m2 y
    in (z, s1 ++ s2)
  -- The identity morphism
  return :: a -> Writer a
  return x = (x, "")

  -- Example of the composition in action:

  upCase :: String -> Writer String
  -- `map` applies the toUpper function to the string s
  upCase s = (map toUpper s, "upCase ")

  toWords :: String -> Writer [String]
  toWords s = (words s, "toWords ")

  -- The composition is achieved using the fish operator
  process :: String -> Writer [String]
  process = upCase >=> toWords
  ```

## 5. Products and Coproducts + 6. Simple Algebraic Data Types

__Isomorphism__: A pair of morphisms one inverse to the other (invertible morphism)
  * __Isomorphic objects__: Mapping from object a to object b and
  from b to a, each one inverse to the other
  * __Inverse morphisms__
  ```lex
  g is inverse to f <=>
  f . g = id
  g . f = id
  ```

__Initial object__: The object that has one and only one morphism going to any
object in the category (most arrows from it)
  * It may not exist
  * Its uniqueness is not guarenteed
  * Its uniqueness up to unique isomorphism is guaranteed
  * Examples:
    * Least element in _posets_ (partially ordered sets)
    * The Void type in the Set category

__Terminal object__: The object with one and only one morphism coming to it
from any object in the category (least arrows to it)
  * It may not exist
  * Its uniqueness is not guarenteed
  * Its uniqueness up to unique isomorphism is guaranteed
  * Examples:
    * Biggest element in _posets_ (partially ordered sets)
    * The Unit type in the Set category

__Opposite category__ (Cop of C): C with the arrows reversed
  * If composition redefined accordingly:
    ```lex
    For C with:
    h = g∘f
    => Cop with:
    hop = fop∘gop
    ```
    => Is category

__Products__
  ```lex
  Given two objects a, b we define its product c as:

  The object with two projections for which any other object c' with two
  projections there is a unique morphism m from c' to c that factorizes those
  projections
  ```
  * __Projection__ of a __Cartesian product__:
  ```haskell
  fst :: (a, b) -> a
  fst (x, y) = x
  snd :: (a, b) -> b
  snd (x, y) = y
  ```
  * __Factorize__:
  ```haskell
  -- p, q projections of c; p', q' projections of c'; m factorizer
  p :: c -> a
  q :: c -> b
  p’ = p . m
  q’ = q . m
  ```
  * Leads to __product types__:
    * Haskell's __pairs__ are the cannonical implementation
    ```haskell
    data Pair a b = Pair a b
    --which is equivalent to
    data (,) a b = (,) a b
    --more commas for more dimensions
    ```
    * Non commutative
    * Commutative up to isomorphism
    * The Unit type is the unit of the product
    * => The Set category is a monoidal category with respect to product

__Coproduct__:
  ```lex
  Given two objects a, b we define its coproduct c as:

  The object with two injections such that for any other object c' with two
  injections there is a unique morphism m from c to c' that factorizes those
  injections
  ```
  * __Injection__:
  ```haskell
  -- i, j injections of c; i', j' injections of c'; m factorizer
  i :: a -> c
  j :: b -> c
  i' = m . i
  j' = m . j
  ```
  * Dual of the product (obtained by reversing the arrows in the product pattern)
  * Leads to __sum types__:
    * Haskell's __eithers__ are the cannonical implementation
    ```haskell
    data Either a b = Left a | Right b
    ```
    * Non commutative
    * Commutative up to isomorphism
    * The Void type is the unit of the coproduct
    * => The Set category is a monoidal category with respect to coproduct

* => (a, Void) is equal up to isomorphism to Void
  * => Analogous to a * 0 = a
* => (a, Either b c) is equal up to isomorphism to Either (a, b) (a c)
  * => Analogous to a * (b + c) = ab + bc

* => Those two interwinded monoids form a __Ring__
  * => Therefore they have the properties of an algebraic Ring

# 7. Functors

__Functor__: Mapping between categories preserving the structure of category
  * Given C, D categories, F functor
    * F maps objects in C to objects in D
    * F maps morphisms
      * Preserving connections
      * Preserving composition
      * Mapping correspondant ids
    ```haskell
    -- Given categories C, D
    -- Given a, b in C
    -- Given F functor
    -- Given f from a to b morphism in C:
    f :: a -> b

    -- The image of a in D is:
    F a
    -- Then the image of f in D:
    F f :: F a -> F b
    -- and if h = g . f in C
    F h = F g . F f
    -- and if ida is id in a, idFa id in Fa
    F ida = idFa
    ```
  * Abstracting the functor in Haskell
    * __Typeclasses__: Define a family of types that support a common interface
      * Work both on types and type constructors
      * For a functor:
      ```haskell
      class Functor f where
        fmap :: (a -> b) -> f a -> f b
        -- If f (type constructor, inferred by compiler by identifying it
        -- applying on types) is a functor it implements fmap with the
        -- given signature
      -- With that we can declare Maybe as
      instance Functor Maybe where
        fmap _ Nothing = Nothing
        fmap f (Just x) = Just (f x)
      ```
  * Examples
    * __Maybe__
    ```haskell
    data Maybe a = Nothing | Just a -- is a mapping from type a to type Maybe a
    -- Maybe is a type constructor, a function on type
    -- To be a functor Maybe should be a mapping also on morphisms of a:
    -- For any f :: a -> b we need to produce f' :: Maybe a -> Maybe b
    -- We therefore define the image of f under Maybe as:
    f' :: Maybe a -> Maybe b
    f' Nothing = Nothing
    f' (Just x) = Just (f x)
    -- We implement the morphism-mapping part of a functor as fmap. For Maybe:
    fmap :: (a -> b) -> (Maybe a -> Maybe b) -- 'lifts' a function
    -- Because of currying the type signature can be interpreted as:
    fmap :: (a -> b) -> Maybe a -> Maybe b
    fmap _ Nothing = Nothing
    fmap f (Just x) = Just (f x)
    -- We can prove that here defined Maybe is a Functor and fullfills the properties
    ```
    * __Reader__
    ```haskell
    -- Type constructor is:
    ((-> r)) -- All functions from r, it maps any type a to (r -> a)
    -- With fmap:
    -- fmap :: (a -> b) -> ((r -> a) -> (r -> b))
    -- which eqs to fmap :: (a -> b) -> (r -> a) -> (r -> b)
    instance Functor ((->) r) where
      -- To get h::r->b given f::a->b, g::r->a we can simply compose
      fmap f g = f . g
      -- Which is the same as
      fmap = (.)
    ```
  * Functors between categories compose (like functions between sets)
    * And they have the same properties as morphisms in the __Cat__ category:
      * Category in which objects are categories (without including itself)
        and morphisms are functors
    * Example:
    ```haskell
    square x = x * x

    mis :: Maybe [Int] -- Composition of two functors Maybe and [] acting on a
    mis = Just [1, 2, 3]

    mis2 = fmap (fmap square) mis -- Outer fmap uses the implementation for
    -- Maybe and inner for List
    ```

# 8. Functoriality

* __Bifunctors__
  * Functors of two arguments
    * In objects maps every pair of objects from category C and category D to
      objects in category E
      * Maps from a cartesian product of categories CxD to E
    * In morphisms it maps a pair of morphisms which is the same as a single
      one from the category CxD. It goes from a pair of obj to a pair of obj.
      * Composition follows:
      ```lex
      (f, g) . (f', g') = (f . f', g . g')
      ```
    * => functors in both arguments
  ```haskell
  -- In this class all three categories are the category of Haskell types
  class Bifunctor f where
    bimap :: (a -> c) -> (b -> d) -> f a b -> f c d
    -- Maps a pair of funcs to the lifted func (f a b -> f c d)
    bimap g h = first g . second h
    first :: (a -> c) -> f a b -> f c b
    first g = bimap g id
    second :: (b -> d) -> f a b -> f a d
    second = bimap id
  ```
