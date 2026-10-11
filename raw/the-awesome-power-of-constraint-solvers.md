---
url: https://btmc.substack.com/p/the-awesome-power-of-constraint-solvers
date_fetched: 2026-10-11
---

With my kids1 having derailed every single one of my side projects (including this blog), I thought to revisit a topic very dear to me: **constraint solving and constrained optimization**. I already wrote an article about “Machine Reasoning”2, which is a made up umbrella term for all sorts of classical AI techniques (in contrast to modern Machine Learning-based approaches), but it was a very high level overview focused on listing different types of constraint solvers that didn’t really provide any particularly useful piece of information. Not very interesting, so here’s a better one.

My entire professional career and academic research has involved constraint solving or constrained optimization in some capacity. Here are some use cases the tools I worked on have been applied to:

- Verifying that distributed program specifications were free of race conditions and deadlocks, and then generating correct-by-construction program skeletons (containing just the network communication) from those specifications. 
- Generating mechatronic system architectures, including electrical power systems in aircraft and gearboxes in cars. 
- Parameter optimization, including improving bicycle aerodynamics and greatly improving the modulation ratio of a domestic water boiler. 

Each of the three bullet points used a different type of solver. The first used a chain of verification tools that ultimately invoked a Satisfiability Modulo Theories (SMT) solver. The second used a combination of a Boolean Satisfiability (SAT) solver and a Constraint Programming (CP) solver. The last one used a custom solver implementing a large portfolio of Derivative Free and Surrogate-based optimization algorithms.

Most programmers are completely unfamiliar with these sorts of tools and the wide variety of very hard problems they can tackle.

If you are into game dev you might be familiar with Wave Function Collapse, which is basically a Constraint Programming technique applied to the procedural generation of textures and tile-maps.

All those LLM math proofs coming out are using Lean (a proof assistant) and are probably making very liberal use of the ‘Grind’3 tactic for automated proof search, which uses SMT solver techniques under the hood.

But this is all a little too abstract, so let’s solve a cool problem together.

# The Problem: An Adder Circuit

Here’s what we want to accomplish: Using nothing but NAND-gates, a “functionally complete” logic gate that can be used to implement any boolean formula, implement a 1-bit Full-Adder circuit.

A NAND-gate takes 2 input bits A and B and outputs one bit C that is the negation of the conjunction of A and B:

A Full-Adder takes 2 input bits A and B plus an input carry bit Ci, and outputs a sum bit S plus an output carry bit Co4, behaving as follows:

So for example, if A=1, B=0, and Ci=1, then the output is Co=1 and S=0. Your CPU and GPU are full of adder circuits (pun intended), though not made up of only NAND-gates.

We want to use the bare minimum amount of NAND-gates necessary, and we want the configuration with the least depth as an approximation of propagation delay (less delay = better performance). Here’s a solution:

9 NAND gates. Depth 6. Would you be able to come up with this solution by hand? Very likely, it’s not a hard problem. *But can you write a program that finds it*? 

Also, is this the best we can do? If we increase the depth can we get away with less gates? If we use more gates can we pull off a lower depth? Or maybe can we improve both metrics at the same time?

**Give that some thought before proceeding with the rest of the article.**

# Constraint Satisfaction Problems

The problem above, of finding that Full-Adder circuit, can be modeled as a *Constraint Satisfaction Problem* (CSP). Any NP problem can be modeled as a CSP because constraint satisfaction is itself an NP-Complete problem.

If we add the objective of finding the lowest amount of NAND-gates necessary and/or the lowest depth required, then it becomes a Constrained Optimization Problem (COP). Multi-objective optimization is a can of worms5, so we’ll do a little trick to have a single objective: For a given fixed number of NAND-gates, find the lowest depth. If there is no solution, increase the number of NAND-gates.

Unless P=NP, any algorithm that can solve this problem must have worst-case exponential complexity: **O(2 n)**. That might sound really bad, but it’s worst-case complexity. In practice, state-of-the-art solvers can eat a problem like the above for breakfast, as you’ll soon see.

# Modeling the Problem

A constraint satisfaction problem consists of decision variables, the domains of those variables (what values they can take), constraints relating those variables, and in the case of constrained optimization, an objective.

We’ll make use of the excellent Python package CPMpy because it has a super nice API and can invoke a ton of different solvers under the hood.

Here’s our starting point:

```
import cpmpy as cp
def main():
  model = cp.Model()
  # We'll model the problem here
  solver = cp.SolverLookup.get("ortools", model)
  has_solution = solver.solve()
  print(f"Status: {solver.status()}")
  if has_solution:
    # We'll output the solution here
    pass
if __name__ == "__main__":
    main()
```
OR-Tools is Google’s suite of constraint solving and optimization software. The solver lookup above is actually loading a specific solver from that suite, CP-SAT, which has won a constraint solving competition every year for 13 years straight, it’s that good. If you run this program you’ll get the following result:

`Status: ExitStatus.FEASIBLE (0.019812 seconds)`There are no variables and no constraints, so the problem is trivial. Other statuses we can get include:

- OPTIMAL: The solver found a solution and proved it is the best possible one. 
- UNSATISFIABLE: The solver proved there is no solution. 
- UNKNOWN: The solver was not able to solve the problem in the time given. 

Now let us start setting up the problem. First, we’ll just model the inputs and outputs of the system and impose that it does indeed add the bits:

```
a = cp.boolvar(name="a")
b = cp.boolvar(name="b")
c_i = cp.boolvar(name="c_i")
s = cp.boolvar(name="s")
c_o = cp.boolvar(name="c_o")
model.add(2 * c_o + s == a + b + c_i)
```
(CPMpy uses a lot of operator overloading abuse to let you write the above)

If we run the program we’ll get FEASIBLE again and it’ll take about the same time. We can also print the solution it found (within `if has_solution`):

```
if has_solution:
  print(f"{a.name}: {a.value()}") 
  print(f"{a.name}: {b.value()}") 
  print(f"{c_i.name}: {c_i.value()}")
  print(f"{s.name}: {s.value()}")
  print(f"{c_o.name}: {c_o.value()}")
```
I just get false on all of them, which makes sense, 2 * 0 + 0 = 0 + 0 + 0. Try adding a constraint setting `c_o == True` and see what happens (make sure to remove it after).

But we’re just directly relating the outputs to the inputs here, this tells us nothing useful. What we want is to propagate those inputs through some number of NAND gates, such that, *for all possible inputs*, the output is as expected. We’ll do this in multiple steps.

# Modeling NAND gates

Each NAND gate will be modeled as 4 variables: 2 inputs, 1 output, and one variable tracking the depth where this NAND gate sits:

```
NUM_GATES = 9
class NANDGate:
  def __init__(self, id: int):
    self.id = id
    self.a = cp.boolvar(name=f"nand_{id}_a")
    self.b = cp.boolvar(name=f"nand_{id}_b")
    self.c = cp.boolvar(name=f"nand_{id}_c")
    # we need to give lower and upper bounds to integer variables
    # depth can never exceed the number of gates, so max is NUM_GATES
    self.depth = cp.intvar(1, NUM_GATES, name=f"nand_{id}_depth")
    self.connections_to_a = []
    self.connections_to_b = []
    self.connections_from_c = []
  def constrain_value_computation(self, model):
    model.add(self.c == ~(self.a & self.b))
```
The class itself is just a way to bundle the data together, it serves no other purpose.

We’ll tackle those connection lists later (more variables!). The depth will also come into play later. For now, we just want to add some NAND gates to the problem and enforce that they compute the right value, meaning output C has to be the negation of the conjunction of inputs A and B.

```
nand_gates = []
for i in range(0, NUM_GATES):
  gate = NANDGate(i+1)
  nand_gates.append(gate)
  gate.constrain_value_computation(model)
```
We can also print the values of these new variables in the solution, as a sanity check. It’s not really the information we ultimately care about, but it tells us if we screwed something up in the modeling, you can remove this later if you want:

```
if has_solution:
  # ...
  for gate in nand_gates:
      print(f"{gate.a.name}: {gate.a.value()}")
      print(f"{gate.b.name}: {gate.b.value()}")
      print(f"{gate.c.name}: {gate.c.value()}")`
```
# Modeling Connections

Now we want these NAND gates to connect to each other. Meaning, if we decide that NAND gate 1’s output C is connected to NAND gate 2’s input A, then the values of C and A must be equal. We’ll model connections as follows:

```
class GateConnections:
  def __init__(self, from_gate: NANDGate, to_gate: NANDGate):
    self.c_a = cp.boolvar(name=f"nand_{from_gate.id}_c_to_nand_{to_gate.id}_a")
    self.c_b = cp.boolvar(name=f"nand_{from_gate.id}_c_to_nand_{to_gate.id}_b")
    self.from_gate = from_gate
    self.to_gate = to_gate
  def constrain_value_propagation(self, model):
    model.add(self.c_a.implies(self.from_gate.c == self.to_gate.a))
    model.add(self.c_b.implies(self.from_gate.c == self.to_gate.b))
def compute_connections(gates: list[NANDGate]):
  connections = []
  for i in range(len(gates)):
    for j in range(i + 1, len(gates)):
      con = GateConnections(gates[i], gates[j])
      connections.append(con)
      gates[j].connections_to_a.append(con.c_a)
      gates[j].connections_to_b.append(con.c_b)
      gates[i].connections_from_c.append(con.c_a)
      gates[i].connections_from_c.append(con.c_b)
  return connections
```
Each `GateConnections` instance is actually modeling 2 different connections between 2 gates, one for each of the input ports of the target. We could have kept them separate but this shortens the blog post. Each of the connections from output to input is modeled as a boolean variable signaling that decision. If `c_a` is true, then that means `c` is connected to `a`. Note that an output port can connect to more than one input port, so having both `c_a` and `c_b` be true is perfectly fine.

The `constrain_value_propagation` method adds constraints that ensure that the value of the output is propagated to the target input if the connection exists.

There are no backward connections in an adder, so the loop that creates connections (and the respective connection variables) only creates them from lower ID gates to higher ID gates. This cuts a lot of the search space that consists of nothing but swaps of gate IDs, a form of symmetry breaking.

Now, the connection constraints:

```
connections = compute_connections(nand_gates)
for con in connections:
  con.constrain_value_propagation(model)
for gate in nand_gates:
  model.add(cp.sum(gate.connections_to_a) == 1)
  model.add(cp.sum(gate.connections_to_b) == 1)
  model.add(cp.sum(gate.connections_from_c) >= 1)
```
The first loop should be obvious, but for the second loop we’re saying the following: for each NAND gate, each of its input ports must be connected to exactly 1 other port (exactly one of the connection variables involving it must be true), and the output port must connect at least once, but can connect any number of times (at least one of the connection variables involving it must be true).

If you actually try to run the program now, you’ll get `ExitStatus.UNSATISFIABLE`. The reason is that there’s no way to connect all the ports with 9 NAND gates that can only connect forward: the first NAND gate’s inputs can’t connect to anything and the last NAND gate’s output can’t connect to anything!

# Modeling Mappings

We “forgot” (pedagogically) about the input and output ports of the adder itself. We need to implement a mapping from them to the NAND gates’ ports. We model it similarly to connections, but a mapping is input to input or output to output. First a small refactoring:

```
class Adder:
  def __init__(self):
    self.a = cp.boolvar(name="a")
    self.b = cp.boolvar(name="b")
    self.c_i = cp.boolvar(name="c_i")
    self.s = cp.boolvar(name="s")
    self.c_o = cp.boolvar(name="c_o")
    self.a_mappings = []
    self.b_mappings = []
    self.c_i_mappings = []
    self.s_mappings = []
    self.c_o_mappings = []
  def constrain_value_computation(self, model):
    model.add(2 * self.c_o + self.s == self.a + self.b + self.c_i)
```
The loose variables that represented the adder are now also bundled together, and we also track their mappings much like we tracked connections in the `NANDGate` instances. There is only one `Adder` instance but now we can pass it around. Our main function now starts as follows:

```
model = cp.Model()
adder = Adder()
adder.constrain_value_computation(model)
nand_gates = []
# ...
```
With this small refactoring in place, we can implement mappings:

```
class GateMappings:
  def __init__(self, gate: NANDGate, adder: Adder):
    self.gate = gate
    self.adder = adder
    self.a_a = cp.boolvar(name=f"adder_a_nand_{gate.id}_a")
    self.a_b = cp.boolvar(name=f"adder_a_nand_{gate.id}_b")
    self.b_b = cp.boolvar(name=f"adder_b_nand_{gate.id}_b")
    self.b_a = cp.boolvar(name=f"adder_b_nand_{gate.id}_a")
    self.c_i_a = cp.boolvar(name=f"adder_c_i_nand_{gate.id}_a")
    self.c_i_b = cp.boolvar(name=f"adder_c_i_nand_{gate.id}_b")
    self.s_c = cp.boolvar(name=f"adder_s_nand_{gate.id}_c")
    self.c_o_c = cp.boolvar(name=f"adder_c_o_nand_{gate.id}_c")
  def constrain_value_propagation(self, model):
    model.add(self.a_a.implies(self.gate.a == self.adder.a))
    model.add(self.b_b.implies(self.gate.b == self.adder.b))
    model.add(self.a_b.implies(self.gate.b == self.adder.a))
    model.add(self.b_a.implies(self.gate.a == self.adder.b))
    model.add(self.c_i_a.implies(self.gate.a == self.adder.c_i))
    model.add(self.c_i_b.implies(self.gate.b == self.adder.c_i))
    model.add(self.s_c.implies(self.gate.c == self.adder.s))
    model.add(self.c_o_c.implies(self.gate.c == self.adder.c_o))
```
These mappings act much like connections, and we can treat them as such:

```
def compute_mappings(adder: Adder, gates: list[NANDGate]):
  mappings = []
  for gate in gates:
    mapping = GateMappings(gate, adder)
    mappings.append(mapping)
    gate.connections_to_a.append(mapping.a_a)
    gate.connections_to_a.append(mapping.b_a)
    gate.connections_to_b.append(mapping.b_b)
    gate.connections_to_b.append(mapping.a_b)
    gate.connections_to_a.append(mapping.c_i_a)
    gate.connections_to_b.append(mapping.c_i_b)
    gate.connections_from_c.append(mapping.s_c)
    gate.connections_from_c.append(mapping.c_o_c)
    adder.a_mappings.append(mapping.a_a)
    adder.a_mappings.append(mapping.a_b)
    adder.b_mappings.append(mapping.b_b)
    adder.b_mappings.append(mapping.b_a)
    adder.c_i_mappings.append(mapping.c_i_a)
    adder.c_i_mappings.append(mapping.c_i_b)
    adder.s_mappings.append(mapping.s_c)
    adder.c_o_mappings.append(mapping.c_o_c)
  return mappings
```
Note that we’re also adding mappings to the lists of connections in the gates. That’s because if an input port is mapped, it cannot also be connected to another port, the constraint that is it connected exactly once must also consider the option of mapping it instead (similar logic applies to output ports).

Then we can add the constraints:

```
# add this before the connection sum constraints so the mappings are included there
mappings = compute_mappings(nand_gates, adder)
for mapping in mappings:
  mapping.constrain_value_propagation(model)
```
The problem is solvable now but the configuration I got most definitely does not compute an addition:

What’s going on here?

# For All Possible Inputs

The issue is this constraint:

`model.add(2 * self.c_o + self.s == self.a + self.b + self.c_i)`This constraint just says the solver needs to find a value for these variables that makes the constraint true, it doesn’t say that for all possible inputs the correct sum is computed. So it can simply pick 1 input/output pair where the configuration of NAND gates happens to give the right result, even if the result is wrong for all other inputs.

Imagine we were doing something much simpler: implementing AND. When both inputs are True or both inputs are False, AND and OR agree, so an implementation of “OR” would pass as a solution for “AND” for certain inputs. Does this make sense? We need it to give the right solution *for all* inputs, simultaneously.

That “for all” there in the sentence above is quite important, it is the universal quantifier (∀). The problem we’re trying to solve is actually of the form:

Which is a synthesis problem, it states: there exists some program *p* such that, for all inputs *i*, the program constraints *P(p)* imply the specification constraints *S(p, i)* for those inputs.

There are various ways to deal with that universal quantifier:

- Some solvers have native support for quantifiers, but performance tends to be rather dubious (it’s a very hard problem in general). 
- Two solvers can work together in a Counter-Example Guided Inductive Synthesis (CEGIS) loop, each handling one of the quantifiers. 
- We can just unroll the universal quantifier for all possible inputs. 

We only have 8 possible inputs, so the latter option is easily the best choice for this Full-Adder use case.

But how do we actually model that? Each variable can only take one value, and we want to propagate 8 different inputs through them. The answer is to create 8 different sets of the “value” variables, and each input will be routed through one of those sets. The connection variables will then route all 8 sets of value variables simultaneously. We’ll start by making the following changes:

```
class NANDGate:
  def __init__(self, id: int):
    self.id = id
    # Note the 8 as the first input, a is now an array of 8 boolean variables
    self.a = cp.boolvar(8, name=f"nand_{id}_a")
    self.b = cp.boolvar(8, name=f"nand_{id}_b")
    self.c = cp.boolvar(8, name=f"nand_{id}_c")
# ...
class Adder:
  def __init__(self):
    self.a = cp.boolvar(8, name="a")
    self.b = cp.boolvar(8, name="b")
    self.c_i = cp.boolvar(8, name="c_i")
    self.s = cp.boolvar(8, name="s")
    self.c_o = cp.boolvar(8, name="c_o")
```
All of the value variables get an 8 as the first argument, turning them into arrays. CPMpy has an array-oriented API, so `a + b` is the same as `[a[0] + b[0], a[1] + b[1], a[2] + b[2], …]` . We do need to change the connection implication constraints because they don’t work directly on arrays, we need to say that we want the conjunction of those expressions, using `cp.all()`:

```
class GateConnections:
  # ...
  def constrain_value_propagation(self, model):
    model.add(self.c_a.implies(cp.all(self.from_gate.c == self.to_gate.a)))
    model.add(self.c_b.implies(cp.all(self.from_gate.c == self.to_gate.b)))
class GateMappings:
  # ...
  def constrain_value_propagation(self, model):
    model.add(self.a_a.implies(cp.all(self.gate.a == self.adder.a)))
    model.add(self.b_b.implies(cp.all(self.gate.b == self.adder.b)))
    model.add(self.a_b.implies(cp.all(self.gate.b == self.adder.a)))
    model.add(self.b_a.implies(cp.all(self.gate.a == self.adder.b)))
    model.add(self.c_i_a.implies(cp.all(self.gate.a == self.adder.c_i)))
    model.add(self.c_i_b.implies(cp.all(self.gate.b == self.adder.c_i)))
    model.add(self.s_c.implies(cp.all(self.gate.c == self.adder.s)))
    model.add(self.c_o_c.implies(cp.all(self.gate.c == self.adder.c_o)))
```
Now, we’re ready for the input constraints, we just need to define the truth table:

```
def constrain_truth_table(model, adder: Adder):
  table = [
    (0, 0, 0, 0, 0),
    (1, 0, 0, 1, 0),
    (0, 1, 0, 1, 0),
    (0, 0, 1, 1, 0),
    (1, 1, 0, 0, 1),
    (1, 0, 1, 0, 1),
    (0, 1, 1, 0, 1),
    (1, 1, 1, 1, 1),
  ]
  for i in range(8):
    array = cp.cpm_array(
      [adder.a[i], adder.b[i], adder.c_i[i], adder.s[i], adder.c_o[i]])
    model.add(array == table[i])
def main():
  # ...
  # right before we lookup the solver:
  constrain_truth_table(model, adder)
```
We now have a working full-adder generator. We didn’t write a visualizer so it’s hard to tell on your end (I have a vibe coded one that exports the solution to netlistsvg), but we could also write an exporter to some logic circuit emulator and test it there.

Anyway, just believe me for now. Run the program and it’ll very quickly tell you the problem is feasible and provide a solution. Here’s what mine looks like:

Looks rather… familiar? It’s the same as the one we were looking for, just shifted around. Running the program a bunch of times seems to generate very similar images each time, but that’s no confirmation. If we ask for all solutions using `solver.solveAll` instead of `solver.solve`, we get that there are 24576 solutions, but many of those solutions are *symmetric* (meaning equivalent), they just have elements swapped around that don’t make any difference to the result. Dealing with symmetries and graph isomorphisms is out of scope for this article, however.

But here’s the million dollar question: Is there a way to make a Full-Adder with only 8 NAND-gates? We just have to set `NUM_GATES` to 8 and solve again.

In just 99 milliseconds on my dying M1 MacBook Pro I’m told in no uncertain terms that nope, you cannot make a Full-Adder with only 8 NAND-gates:

`Status: ExitStatus.UNSATISFIABLE (0.099592 seconds)`As a reminder, this doesn’t mean the solver failed to find a solution, it explored the entire design space in 99 milliseconds. *There is no solution*.

Some solvers can even produce machine-verifiable (albeit unreadable) mathematical proofs that there really is no solution. If we remove the truth table constraints we get nonsensical solutions again, but now with 8 gates instead of 9.

What about depth though? Can we do better than 6?

# Modeling Depth

Spoiler alert: no, 6 is the best we can do, there is in fact only 1 way to make a Full-Adder out of NAND-gates (those 24576 solutions are all equivalent). But let’s get confirmation anyway.

We already added the depth variables, but we didn’t relate them in any way. The easiest way to fit depth into our existing constraints is to enforce that if two gates are connected, then the depth of the target gate is greater:

```
def constrain_depth(model, connections: list[GateConnections]):
  pairs = set()
  for con in connections:
    if (con.from_gate.id, con.to_gate.id) in pairs:
      continue
    model.add(con.c_a.implies(con.to_gate.depth > con.from_gate.depth))
    model.add(con.c_b.implies(con.to_gate.depth > con.from_gate.depth))
    pairs.add((con.from_gate.id, con.to_gate.id))
```
Then our objective is to minimize the max depth, simple enough:

```
def main():
  # ...
  constrain_depth(model, connections)
  model.minimize(cp.max([gate.depth for gate in nand_gates]))
  # ...
  if has_solution:
    print(f"Depth: {model.objective_value()}")
```
And the result is, sadly:

```
Status: ExitStatus.OPTIMAL (0.476214 seconds)
Depth: 6
```
Depth 6 is as low as we can go with NAND gates.

# Neat! How do I implement a Solver?

This is normally the part of the blog post where I tell you how to implement some crazy thing in C, in this case a constraint solver, but sadly this time I won’t. The article is already huge and a competitive constraint solver requires quite a lot of engineering. It wouldn’t be an article, more like a book. Here are some starting points for different solver types:

- Conflict-Driven Clause Learning (CDCL, used in SAT). 
- Arc-Consistency Algorithm #3 (AC-3, used in CP) 
- Simplex Algorithm (used in Linear Programming). 

Maybe I’ll make a “Crafting Solvers” in honor of Bob Nystrom’s “Crafting Interpreters” someday.

# Conclusion

Constraint Solving is an extremely powerful approach for tackling hard problems that would otherwise require highly-specialized algorithms and heuristics.

Despite having exponential complexity in general, state-of-the-art solvers can chew through (many) real-world problems with ease. Sadly with larger adders it begins to struggle (2-bit is still fair game), though you can always make larger adders out of Full-Adders as opaque blocks to make it tractable (though the optimal solution then involves adding extra gates to handle the carry computations in parallel, to minimize propagation delay).

Here’s a gist with the complete code above.

See you on the next one.

Totally not my atrocious time management skills, it is 100% the kids’ fault.

Not to be confused with “reasoning” in LLMs.

The Isabelle theorem prover calls the same sort of tactic ‘sledgehammer’. Now that’s a name.

A half-adder would lack the input carry.

Many solvers do not natively support multiple objectives, and there are many different ways of setting up the problem when there are multiple competing objectives. Further reading: Multi-objective Optimization
