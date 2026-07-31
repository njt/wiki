---
url: https://joomy.korkutblech.com/posts/2026-07-28-why-rocq-is-better.html
title: Why Rocq is better than Lean for program verification
author: Joomy Korkut
date_fetched: 2026-08-01
date_published: 2026-07-28
tags: [rocq, lean, formal-verification, coinductive-types, program-extraction]
---

A write-up on why I don't give in to the hype and switch to Lean for formal verification of programs.

Especially now, with AI's well-publicized success in mathematics, I get asked why I still use [Rocq](https://rocq-prover.org/) instead of giving in to the hype and switching to [Lean](https://lean-lang.org/). Most of this post started life as a provocatively titled slide in a recent [LangSec](https://langsec.org/spw26/abstracts.html) keynote: "Why Rocq is better than Lean for program verification."

I am not trying to start a flame war here. If you are a more mature person than I am, feel free to mentally replace "better" with "a better fit for my work today" everywhere below. One more caveat before I start: this post is about *program* verification, not formalizing mathematics, where I will happily admit Lean has real momentum.

## Language-level issues

### Coinductive types and cofixpoints

Wojciech Różowski and Joachim Breitner of Lean FRO developed support for [coinductive predicates](https://www.youtube.com/watch?v=vEt07N_v-Yo) in Lean. Their feature made it into [Lean 4.25](https://lean-lang.org/doc/reference/latest/releases/v4.25.0/) as the `coinductive` command ([implementation](https://github.com/leanprover/lean4/blob/master/src/Lean/Elab/Coinductive.lean)). It is useful for bisimulations and other coinductive proofs, but it does not provide executable cofixpoints or programs that can be extracted.

I also want codata in `Type` that I can actually run. Rocq gives me that directly with `CoInductive` and `CoFixpoint`. Lean has no corresponding declarations in `Type`; its alternatives use ordinary functions and structures or library encodings.

The closest Lean experiment I found to a Rocq-style codata declaration is Alex Keizer's [QPFTypes](https://github.com/alexkeizer/QPFTypes), a proof-of-concept package for generic codata. Its `codata` command turns a specification into a library encoding and generates destructor, corecursor, and bisimulation principles. Unlike Rocq's `CoInductive`, it is not a kernel declaration. The examples use QPFTypes' pinned Lean 4.25.0 toolchain (latest supported at the time of the post).

#### Declaring codata

I do not mean this as a dunk on QPFTypes: it says "proof of concept" right there in the readme. But the rough edges show up quickly in examples that are completely ordinary in Rocq. For instance, Rocq accepts this parameterless coinductive type directly:

```coq
CoInductive co_unit : Type :=
| co_unit_loop : co_unit.
```

```lean
codata CoUnit where
  | loop : CoUnit

/--
error: Due to a bug, codatatype without any parameters don't quite
work yet. Please try adding parameters to your type
-/
```

This one can be written off as an implementation bug but the next two are structural. Mutually coinductive declarations are routine in Rocq but not supported by QPFTypes:

```coq
CoInductive tree (a : Type) : Type :=
| node : a -> forest a -> tree a
with forest (a : Type) : Type :=
| fnil : forest a
| fcons : tree a -> forest a -> forest a.
```

```lean
mutual
  codata Tree α where
    | node : α → Forest α → Tree α
  
  codata Forest α where
    | fnil : Forest α
    | fcons : Tree α → Forest α → Forest α
end

/--
error: invalid mutual block: either all elements of the block must be
inductive/structure declarations, or they must all be definitions/
theorems/abbrevs
-/
```

Indexed coinductive families are also part of Rocq's ordinary coinductive fragment but outside the QPF encoding. In this example, the index is a clock that advances at each step; the same pattern occurs with protocols, phases, sizes, and state machines:

```coq
CoInductive istream (a : Type) : nat -> Type :=
| icons : forall n, a -> istream a (S n) -> istream a n.
```

```lean
codata IStream α : Nat → Type where
  | icons : α → IStream α (n + 1) → IStream α n

/--
error: Unexpected type; type will be automatically inferred. Note that
inductive families are not supported due to inherent limitations of QPFs
-/
```

Under the hood, QPFTypes generates `Cofix` machinery. The `codata` command hides most of it for simple, non-mutual, non-indexed cases, but once I leave that fragment I am either calling the low-level `MvQPF.Cofix.corec` and `bisim` API myself or I am out of luck. In Rocq, the examples above are just declarations.

Rocq's direct support is not painless; anyone who has argued with its guardedness checker knows that. On the proof side, Rocq users also have mature tools such as [Paco](https://github.com/snu-sf/paco) and Damien Pous's [coinduction](https://github.com/damien-pous/coinduction) library. They help with coinductive predicates and relations, but they do not replace `CoFixpoint` for programs.

#### Extracting cofixpoints

A native Rocq cofixpoint extracts to an actual lazy OCaml value. For example, in the [game tree library](https://github.com/bloomberg/game-trees) I worked on, the Rocq `unfold_cotree` function extracts to:

```ocaml
type 'a cotree = 'a __cotree Lazy.t
and 'a __cotree =
| Conode of 'a * 'a cotree colist

(* other definitions ... *)

(** val unfold_cotree : ('a1 -> 'a1 colist) -> 'a1 -> 'a1 cotree **)

let rec unfold_cotree next init =
  lazy (Conode (init, (comap (unfold_cotree next) (next init))))
```

With QPFTypes, construction and observation instead go through the generic `MvQPF.Cofix.corec` and `MvQPF.Cofix.dest` operations. The program keeps the generic `Cofix` representation rather than becoming the direct lazy tree above. That is fine as mathematics, but it is not the program I would write by hand.

If you want to poke at QPFTypes yourself, [BadCoinduction.lean](../assets/why-rocq-is-better/BadCoinduction.lean) contains the full experiment behind this section: working `Colist` and `Cotree` definitions, examples of the generated interface, and checked failures for parameterless, mutual, and indexed codata. Its header records the exact QPFTypes commit and command needed to rerun the test.

#### Current Lean alternatives

At this point, a Lean programmer may be shouting at the screen: nobody reaches for QPFTypes just to write a stream. Fair enough. QPFTypes is here because it comes closest to a Rocq-style codata declaration. Day-to-day Lean uses several other tricks.

For streams, mathlib's [`Stream'`](https://leanprover-community.github.io/mathlib4_docs/Mathlib/Data/Stream/Defs.html) is the obvious answer. It is just `Nat → α`: ask for position `n` and get an element back. It runs, and mathlib supplies corecursors along with extensionality, bisimulation, and coinduction lemmas, so it is pleasant to prove things about. But it is not a lazy constructor whose tail is another stream. It solves streams, not arbitrary mutual or indexed codata.

Another option is an explicit state machine: keep some state, write a step function, and use it as a corecursor. Lean's current [`Iter` interface](https://lean-lang.org/doc/reference/latest/Iterators/) packages this pattern for sequences and computes one step at a time, on demand. An iterator may carry a [`Productive`](https://lean-lang.org/doc/reference/latest/Iterators/Iterator-Definitions/#finite-and-productive-iterators) proof that it will eventually yield a value or finish; `Iter.repeat` already has one. With a custom iterator, I have to supply the step interface, its invariants, and perhaps that productivity proof. This works, but now I am doing the plumbing between a state machine and the sequence myself. A Rocq `CoFixpoint` checks its recursive calls for guardedness and gives me the coinductive value directly.

[`Thunk`](https://lean-lang.org/doc/reference/latest/Basic-Types/Lazy-Computations/) buys laziness, but not coinduction. Compiled Lean delays a thunk until it is forced and then caches the result. The logic sees a `Unit → α`, which means total definitions involving thunks remain available to proofs, although the caching is invisible. A thunk neither permits recursion nor checks that recursion eventually produces a constructor. An extracted Rocq cofixpoint uses similar runtime laziness only after its recursion has passed the guardedness checker.

[`partial def`](https://lean-lang.org/doc/reference/latest/Definitions/Recursive-Definitions/#partial-functions) is the blunt instrument. The compiler runs the recursive body, but the logic gets only an opaque constant. Lean checks neither termination nor productivity here; it accepts both a thunked producer of the natural numbers and a producer that immediately calls itself forever. An `unsafe def` also runs, but theorem-safe declarations cannot mention it at all. Batteries' [`MLList`](https://github.com/leanprover-community/batteries/blob/main/Batteries/Data/MLList/Basic.lean) shows the usual arrangement: a private unsafe lazy implementation, an opaque public interface, and producers such as `fix` and `iterate` written as `partial def`s. Those producers cannot be unfolded in proofs the way an observed Rocq cofixpoint can. Lean also has [`partial_fixpoint`](https://lean-lang.org/doc/reference/latest/Definitions/Recursive-Definitions/#partial-fixpoints), which retains equations, but it does not accept this kind of constructor-and-thunk recursion.

QPFTypes avoids that opacity: it gives me a corecursor and bisimulation principles. But then I am back to its generic `Cofix` representation and the declaration restrictions I tested above.

Why am I worrying about anything beyond streams? Interaction trees. The [interaction trees](https://github.com/DeepSpec/InteractionTrees) library in Rocq represents effectful, possibly nonterminating programs as coinductive trees. I can write programs with them, interpret and extract those programs, and prove equations about the same trees, usually up to weak bisimulation. `Stream'` and `Iter` only give me sequences, not the branching continuations needed for effects. I could run a hand-written effect tree with `Thunk` and `partial def`, but then the recursive producer would be opaque to proofs. Getting all of that in Lean requires a library encoding of codata.

MIT PLV's [lean4-itree](https://github.com/mit-plv/lean4-itree) does this for interaction trees using Mathlib's `PFunctor.M` final coalgebra. The newer [PolyFun](https://github.com/Verified-zkEVM/PolyFun) adds handlers, recursive procedures, traces, strong and weak bisimulation, and proofs of the monad and iteration laws. I can compute with these trees and prove things about them in Lean. They are still library-encoded M-types, though. Lean has no native codata declaration here, and computation keeps the generic representation rather than producing the direct lazy program Rocq extracts.

[HITrees](https://arxiv.org/abs/2510.14558) are not a way around this problem. The authors consider the coinductive Delay-monad approach used by ITrees, but Lean's lack of native coinductive types rules it out. Their trees are inductive instead, and nontermination becomes a higher-order recursion effect. A recursive computation is no longer an infinite tree that I can observe and unfold; its recursion gets a meaning only when a handler interprets the effect. The monadic interpretation can run it, and proofs can go through the state-machine interpretation, but the HITree equational theory does not provide the usual recursion-unfolding equation. It works. It also moves nontermination out of the tree and into the interpreter, which I find more awkward than writing a guarded Rocq cofixpoint.

With Lean, I have to give up one of these things or rebuild it on top of generic machinery. Rocq lets me declare codata, write a guarded producer, reason about it by observation, and extract direct lazy code. That is what I want from coinduction.

### Nested inductive types and predicates

[Meven Lennon-Bertrand pointed this out to me](https://lipn.info/@mevenlennonbertrand/116608679656852130), and I expanded the example below. Lean accepts many nested inductive definitions, but its checker rejects some definitions accepted by Rocq. I ran into this while writing my proof pearl [_A Rose Tree Is Blooming_](https://joomy.korkutblech.com/papers/game-trees-cpp26.pdf), which relies on this part of Rocq and would have been considerably uglier to write in Lean. The rose tree example is too large for this post, so here is the same issue with JSON schemas.

Suppose you have a JSON schema language and want to prove the usual things about validation: field names line up, dropping a field from the schema is safe, and validating a prefix works. Routine stuff if you are building a verified system. Defining JSON and schemas is no trouble in either language:

```coq
Inductive json : Type :=
| jnull : json
| jstr  : string -> json
| jnum  : nat -> json
| jarr  : list json -> json
| jobj  : list (string * json) -> json.

Inductive schema : Type :=
| sany : schema
| sstr : schema
| snum : schema
| sarr : schema -> schema
| sobj : list (string * schema) -> schema.
```

```lean
inductive JSON where
  | null : JSON
  | str  : String → JSON
  | num  : Nat → JSON
  | arr  : List JSON → JSON
  | obj  : List (String × JSON) → JSON

inductive Schema where
  | any  : Schema
  | strS : Schema
  | numS : Schema
  | arrS : Schema → Schema
  | objS : List (String × Schema) → Schema
```

The interesting part is the validation relation. An object schema like `{ "name": string, "age": number }` should validate an object `{ "name": "Alice", "age": 30 }` by checking, pairwise, that the field names match and that each value validates against the corresponding sub-schema. Rocq can store all of that in one [`Forall2`](https://rocq-prover.org/doc/V9.0.0/stdlib/Stdlib.Lists.List.html#Forall2) derivation:

```coq
Inductive valid : schema -> json -> Prop :=
  | valid_any : forall j, valid sany j
  | valid_str : forall s, valid sstr (jstr s)
  | valid_num : forall n, valid snum (jnum n)
  | valid_arr : forall elem_schema elems,
      Forall (fun j => valid elem_schema j) elems ->
      valid (sarr elem_schema) (jarr elems)
  | valid_obj : forall schema_fields json_fields,
      Forall2
        (fun (sf : string * schema) (jf : string * json) =>
          fst sf = fst jf /\ valid (snd sf) (snd jf))
        schema_fields json_fields ->
      valid (sobj schema_fields) (jobj json_fields).
```

The projections are deliberate. Rocq 9.0 rejects an equivalent tuple-pattern lambda around the recursive occurrence as non-strictly positive; its syntactic checker does not see through that pattern match. The projection-based declaration above compiles.

Lean 4.32.1 rejects the corresponding object constructor:

```lean
inductive ValidCombined : Schema → JSON → Prop where
  | obj :
      Forall₂
        (fun (sf : String × Schema) (jf : String × JSON) =>
          sf.1 = jf.1 ∧ ValidCombined sf.2 jf.2)
        schemaFields jsonFields →
      ValidCombined (.objS schemaFields) (.obj jsonFields)

error: (kernel) invalid nested inductive datatype 'And',
nested inductive datatypes parameters cannot contain local variables.
```

Lean accepts several nearby forms: `Forall₂ ParRed`, direct recursion through `And` and `Exists`, and `Forall₂ (fun sf jf => Valid sf.2 jf.2)`. In the rejected declaration, the recursive occurrence passes through both `Forall₂` and `And`, and the kernel reports the inner `And`. A different case, `Forall₂ (Eval env)`, fails at `Forall₂` because its relation parameter captures the constructor-local `env`.

Lean can retain the list structure by splitting the combined relation into two `Forall₂` derivations:

```lean
inductive Valid : Schema → JSON → Prop where
  | obj :
      Forall₂
        (fun (sf : String × Schema) (jf : String × JSON) =>
          sf.1 = jf.1)
        schemaFields jsonFields →
      Forall₂
        (fun (sf : String × Schema) (jf : String × JSON) =>
          Valid sf.2 jf.2)
        schemaFields jsonFields →
      Valid (.objS schemaFields) (.obj jsonFields)
```

No indices or separate length proof are needed. Dropping the head remains structural, although Lean must destruct both derivations:

```coq
Lemma Forall2_tail :
  forall (A B : Type) (R : A -> B -> Prop)
         (a : A) (b : B) xs ys,
    Forall2 R (a :: xs) (b :: ys) ->
    Forall2 R xs ys.
Proof.
  intros. inversion H; assumption.
Qed.
```

```lean
theorem Valid.dropHead
    (hv : Valid (.objS ((k, s) :: schemaFields))
               (.obj ((k', j) :: jsonFields))) :
    Valid (.objS schemaFields) (.obj jsonFields) := by
  cases hv with
  | obj hnames hvalues =>
      cases hnames
      cases hvalues
      exact .obj
        ‹Forall₂ _ schemaFields jsonFields›
        ‹Forall₂ _ schemaFields jsonFields›
```

Splitting the relation loses the single proof object that pairs each name equality with its recursive validation. I can recover that pairing with a mutual `ValidFields` relation, but then Lean's `induction` tactic does not support mutual inductive types, and the generated recursor takes a motive for every relation. A custom induction theorem can hide that setup. Rocq keeps the standard `Forall2` representation, and `Scheme` can generate combined principles when a mutual definition is needed.

Lean can express the same propositions without an index-based encoding, but it makes me rearrange the declaration and build more proof machinery. Neither system automatically supplies an elementwise induction principle for nested data such as `Term` containing `list Term`; both need a stronger recursor for hypotheses about the list elements (Meven also told me that Rocq 9.2 fixes it, see footnote for details).

You can run the whole comparison yourself: [NestedPain.v](../assets/why-rocq-is-better/NestedPain.v) is the Rocq 9.0.0 side, and [NestedPain.lean](../assets/why-rocq-is-better/NestedPain.lean) is the Lean 4.32.1 side. The Lean file wraps expected failures in `#guard_msgs`, so the errors are checked during compilation instead of being left as untested comments.

## Program extraction

Eventually, I need these definitions to become programs I can run.

Lean is opinionated about extraction: its standard toolchain compiles through its own runtime. Sure, this is a genuine plus if you are writing Lean libraries and the runtime's design fits your needs. And Lean's standard toolchain deserves real credit on performance: Kim Morrison's verified [`lean-zip`](https://kim-em.github.io/blog/2026-7-24-why-lean-is-faster-than-rust/) can even compress faster than pure-Rust `miniz_oxide`! That is impressive. But performance is not the whole story for extraction. I also care whether I can read the generated code, choose its target, or put it through a verified pipeline.

Lean does not hand you a menu of alternate extraction backends, and the compilation pipeline it has offers no end-to-end correctness proof (hence the rare runtime bugs like [the one Kiran Gopinathan found](https://kirancodes.me/posts/log-who-watches-the-watchers.html)), nor is it readable in my opinion (which is normal as it is not designed for it! The produced code is very runtime-specific).

In contrast, Rocq gives me several routes from definitions to runnable programs, with different trade-offs on TCBs, readability, etc.:

*   [OCaml, Haskell, Scheme](https://rocq-prover.org/doc/master/refman/addendum/extraction.html)
*   a [verified extraction pipeline to Malfunction](https://github.com/MetaRocq/rocq-verified-extraction),
*   [Rust](https://github.com/AU-COBRA/coq-rust-extraction),
*   [Elm](https://github.com/AU-COBRA/coq-elm-extraction),
*   Clight and WebAssembly via [CertiRocq](https://github.com/CertiRocq/certirocq), a verified compiler (some parts are in progress),
*   and now C++, via [Crane](https://github.com/bloomberg/crane) (which I have had a hand in, and tries to focus on [readability](https://bloomberg.github.io/crane/papers/crane-rocqpl26.pdf) of generated code).

Odds are, one of those routes will work.

### Verified programs in practice

Rocq lets me write programs that actually run and machine-check properties of the same source code. These examples happen to be games because I thought writing games in Rocq would be fun. Crane extracts their Rocq logic and event loops to C++, and my [rocq-crane-sdl2](https://github.com/joom/rocq-crane-sdl2) bindings connect the result to SDL2. The screenshots link to playable WebAssembly builds.

#### Rocqman

Its proofs cover the gameplay transitions used by the frame loop. Score never decreases, lives and remaining collectibles never increase, terminal states are fixed points of `tick`, and the pause and end-screen transitions are checked.

#### Rocqsweeper

The proofs cover the Minesweeper rules and input layer. The first click is safe; flagging preserves mine and adjacency data; flood fill preserves mines and cannot increase hidden safe cells; cursor movement stays in bounds; and mouse events are interpreted at the expected cells.

#### Reversirocq

The game uses the same game tree library for its Reversi rules (added by Charles C. Norton) and coinductive alpha-beta AI. Its theorems cover legal-move enumeration and game results, and relate alpha-beta to minimax on the finite prefix being searched.

The proof boundary stops at the Rocq source. It does not include SDL, Crane, the generated C++, or the native runtime. Within that boundary, the proofs concern logic used by the running programs, not a separate model.

## Ecosystem

The ecosystem is the part I cannot casually replace. Most of the infrastructure below is written in Rocq; a few entries are external tools with an established Rocq backend or component.

*   **Abstractions:**
    *   [interaction trees](https://github.com/DeepSpec/InteractionTrees): a coinductive datatype that represents effectful, possibly nonterminating programs as trees of external events, so you can give denotational semantics to impure code and reason about it equationally.
    *   [choice trees](https://github.com/vellvm/ctrees): interaction trees extended with internal nondeterministic choice, which is what you want when modeling concurrency or other nondeterministic systems.
*   **Program verification frameworks:**
    *   [Iris](https://iris-project.org/): a higher-order concurrent separation logic framework that is more or less the workhorse for reasoning about stateful and concurrent programs ([Iris-Lean](https://github.com/leanprover-community/iris-lean) is worth mentioning here and is rapidly developing; it is already capable of a lot, though it has not been exercised to the extent that Iris has).
    *   [CFML](https://github.com/charguer/cfml): a framework for proving OCaml programs correct against higher-order separation-logic specifications. Its front end imports OCaml source into Rocq; its library generates characteristic formulae and provides proof tactics.
    *   [Perennial](https://github.com/mit-pdos/perennial): an Iris-based framework for verifying concurrent, crash-safe storage and distributed systems. It uses [Goose](https://github.com/goose-lang/goose) to connect the proofs to runnable programs written in a subset of Go.
    *   [VST](https://vst.cs.princeton.edu/): the Verified Software Toolchain, a separation logic for proving functional correctness of C programs, based on CompCert semantics.
    *   [BRiCk](https://github.com/bedrocksystems/BRiCk): a program logic and toolchain for verifying real C++ programs.
*   **Verification tools with Rocq backends or components:**
    *   [Frama-C](https://frama-c.com/): a platform for analyzing and deductively verifying C programs that can discharge its obligations to Rocq.
    *   [Why3](https://www.why3.org/): a platform for deductive verification with its own language, which dispatches goals to many backend provers and can export obligations to Rocq for interactive proof.
    *   [Cerberus](https://www.cl.cam.ac.uk/~pes20/cerberus/): an executable formal semantics for a large, practical fragment of C. Its CHERI C memory model has a Rocq implementation.
*   **Formal semantics and verified compilers for real languages:**
    *   [CompCert](https://compcert.org/): a formally verified optimizing C compiler, the flagship example of the whole field.
    *   [Vellvm](https://vellvm.github.io/vellvm/): a specification of LLVM IR in Rocq with an abstract semantics and an executable interpreter proved to refine it.
    *   [Vélus](https://velus.inria.fr/): a verified compiler from Lustre to CompCert's Clight.
    *   [WasmCert](https://github.com/WasmCert/WasmCert-Coq): a mechanized formal semantics of WebAssembly.
    *   [JSCert](https://github.com/jscert/jscert): a mechanized formal semantics of JavaScript, tracking the ECMAScript 5 specification.
*   **Lighter-weight verification by translation:**
    *   [hs-to-rocq](https://github.com/plclub/hs-to-rocq): translates Haskell source into Rocq so you can verify it after the fact.
    *   [rocq-of-ocaml](https://github.com/formal-land/rocq-of-ocaml): the same idea for OCaml.
    *   [rocq-of-python](https://github.com/formal-land/rocq-of-python): the same idea for Python.
    *   [rocq-of-rust](https://github.com/formal-land/rocq-of-rust): the same idea for Rust, alongside [Aeneas](https://github.com/AeneasVerif/aeneas), which turns borrow-checked Rust into a pure functional model for verification (and does also target Lean).
*   **Program synthesis:**
    *   [Fiat Crypto](https://github.com/mit-plv/fiat-crypto): derives correct-by-construction, high-performance cryptographic arithmetic, the kind that ends up shipping in browsers and TLS libraries.
    *   [Rupicola](https://github.com/mit-plv/rupicola): a relational-compilation toolkit for turning low-level functional Gallina programs into imperative [Bedrock2](https://github.com/mit-plv/bedrock2) programs.
*   **Verified lexing and parsing:**
    *   [Narcissus](https://github.com/mit-plv/fiat): derives correct-by-construction encoders and decoders for binary formats.
    *   [Verbatim](https://github.com/egolf-cs/Verbatim): a verified regular-expression-based lexer.
    *   [CoStar](https://github.com/slasser/CoStar): a verified parser based on the ALL(*) algorithm.

You might skim that list and think, "None of this is a big deal; I can vibe-port the piece I need to Lean over a weekend." Maybe you can. But hold that thought for one more section.

I should admit that some of these projects are not actively maintained. In my experience, though, I can usually throw an agent at one and get it building and running again pretty easily.

## Regulatory acceptance

I have no firsthand experience of taking a tool through certification, and I am not an expert on regulatory acceptance. This point came from Meven, who noted that it may matter more to my European colleagues than it does to me.

ANSSI has published criteria for using Rocq in Common Criteria evaluations. The CompCert project reports that in 2026 the compiler was successfully qualified for the MFC_NG computer on ATR 42/72 aircraft, in work by AbsInt with guidance from Airbus.

I cannot say what a Lean port would be required to do in either setting. Even a clean weekend port would not automatically inherit that history; whether it would be accepted is a question for people familiar with these processes.

## But what about the AI agents?

This is the question I actually get asked the most. I know all the hype is on making AI agents write Lean, but the agents are perfectly good at writing Rocq too.

*   Rocq dates to the late 1980s and has a substantial corpus of code and documentation. There is a lot of Rocq out there.
*   Current models adapt well to unfamiliar languages when given documentation and examples. This is not 2024 anymore.

"But the AI only knows the popular language" is an argument with a short shelf life, and it is not a reason for me to switch proof assistants.

## Epilogue

Lean is doing serious program-verification work (look at mvcgen and Velvet!) but moving my work there would still mean reshaping definitions and replacing extraction pipelines, libraries, and institutional history I already rely on. Given the work I actually do, Rocq is the better fit for me today.
