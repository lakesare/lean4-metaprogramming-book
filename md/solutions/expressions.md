```lean
import Lean
open Lean Meta
```

# Expressions: Solutions

1. Create expression `1 + 2` with `Expr.app`.

```lean
def one : Expr :=
  Expr.app (Expr.app (Expr.const `Nat.add []) (mkNatLit 1)) (mkNatLit 2)

elab "one" : term => return one
#check one  -- Nat.add 1 2 : Nat
#reduce one -- 3
```

2. Create expression `1 + 2` with `Lean.mkAppN`.

```lean
def two : Expr :=
  Lean.mkAppN (Expr.const `Nat.add []) #[mkNatLit 1, mkNatLit 2]

elab "two" : term => return two
#check two  -- Nat.add 1 2 : Nat
#reduce two -- 3
```

3. Create expression `λ x => 1 + x`.

```lean
def three : Expr :=
  Expr.lam `x (Expr.const `Nat [])
  (Lean.mkAppN (Expr.const `Nat.add []) #[Lean.mkNatLit 1, Expr.bvar 0])
  BinderInfo.default

elab "three" : term => return three
#check three    -- fun x => Nat.add 1 x : Nat → Nat
#reduce three 6 -- 7
```

4. [**De Bruijn Indexes**] Create expression `λ a, λ b, λ c, (b * a) + c`.

```lean
def four : Expr :=
  Expr.lam `a (Expr.const `Nat [])
  (
    Expr.lam `b (Expr.const `Nat [])
    (
      Expr.lam `c (Expr.const `Nat [])
      (
        Lean.mkAppN
        (Expr.const `Nat.add [])
        #[
          (Lean.mkAppN (Expr.const `Nat.mul []) #[Expr.bvar 1, Expr.bvar 2]),
          (Expr.bvar 0)
        ]
      )
      BinderInfo.default
    )
    BinderInfo.default
  )
  BinderInfo.default

elab "four" : term => return four
#check four -- fun a b c => Nat.add (Nat.mul b a) c : Nat → Nat → Nat → Nat
#reduce four 666 1 2 -- 668
```

5. Create expression `λ x y => x + y`.

```lean
def five :=
  Expr.lam `x (Expr.const `Nat [])
  (
    Expr.lam `y (Expr.const `Nat [])
    (Lean.mkAppN (Expr.const `Nat.add []) #[Expr.bvar 1, Expr.bvar 0])
    BinderInfo.default
  )
  BinderInfo.default

elab "five" : term => return five
#check five      -- fun x y => Nat.add x y : Nat → Nat → Nat
#reduce five 4 5 -- 9
```

6. Create expression `λ x, String.append "hello, " x`.

```lean
def six :=
  Expr.lam `x (Expr.const `String [])
  (Lean.mkAppN (Expr.const `String.append []) #[Lean.mkStrLit "Hello, ", Expr.bvar 0])
  BinderInfo.default

elab "six" : term => return six
#check six        -- fun x => String.append "Hello, " x : String → String
#eval six "world" -- "Hello, world"
```

7. Create expression `∀ x : Prop, x ∧ x`.

```lean
def seven : Expr :=
  Expr.forallE `x (Expr.sort Lean.Level.zero)
  (Expr.app (Expr.app (Expr.const `And []) (Expr.bvar 0)) (Expr.bvar 0))
  BinderInfo.default

elab "seven" : term => return seven
#check seven  -- ∀ (x : Prop), x ∧ x : Prop
#reduce seven -- ∀ (x : Prop), x ∧ x
```

8. Create expression `Nat → String`.

```lean
def eight : Expr :=
  Expr.forallE `notUsed
  (Expr.const `Nat []) (Expr.const `String [])
  BinderInfo.default

elab "eight" : term => return eight
#check eight  -- Nat → String : Type
#reduce eight -- Nat → String
```

9. Create expression `λ (p : Prop) => (λ hP : p => hP)`.

```lean
def nine : Expr :=
  Expr.lam `p (Expr.sort Lean.Level.zero)
  (
    Expr.lam `hP (Expr.bvar 0)
    (Expr.bvar 1)
    BinderInfo.default
  )
  BinderInfo.default

elab "nine" : term => return nine
#check nine  -- fun p hP => p : (p : Prop) → p → Prop
#reduce nine -- fun p hP => p
```

10. [**Universe levels**] Create expression `Type 6`.

```lean
def ten : Expr :=
  Expr.sort (Nat.toLevel 7)

elab "ten" : term => return ten
#check ten  -- Type 6 : Type 7
#reduce ten -- Type 6
```
